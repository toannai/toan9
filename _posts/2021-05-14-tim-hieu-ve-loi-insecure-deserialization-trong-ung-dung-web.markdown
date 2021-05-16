---
layout: post
title: Tìm hiểu về lỗi Insecure deserialization (aka PHP Object Injection) trong ứng dụng web,
date: 2021-05-14 00:32:20 +0700
description: Bài viết cung cấp các thông tin lỗi Insecure deserialization trong ứng dụng web,
img: 2021/05/14/210514_deserialization_infographic.jpg
fig-caption: # Add figcaption (optional)
tags: [Security, WebSecurity]
---

Ngày trước khi mới học lập trình tôi thường hay truyền dữ liệu qua mạng giữa app và server bằng việc trao đổi qua giữa các đoạn text .Client gửi một đoạn text, server đọc đoạn text và xử lý rồi trả lời theo yêu cầu của client (Khá là dễ hình dung). Sau này học lập trình hướng đối tượng. Trong một trương trình tôi có thể sử dụng object để lưu giữ thông tin, mỗi object có các thuộc tính chứa các loại dữ liệu cơ bản (int/float/string/...), các object tương tác với nhau qua method. Tôi vẫn băn khoan liệu có thể thể nào tương tác giữa 2 trương trình chạy trên 2 máy khác nhau bằng việc gửi/nhận các object (thay bằng text) mà 2 bên vẫn có thể hiểu và xử lý được dữ liệu trong object đó không (Kiểu 2 bên thống nhất 1 cú pháp chung để gửi/đọc object qua mạng)????

Câu trả lời cho câu hỏi này là hoàn toàn có thể. Việc này được thực hiện thông qua quá trình Serialization và Deserialization trong các ngôn ngữ lập trình. Buổi hôm nay sẽ cùng nhau phân tích về quá trình này. Cũng như phương pháp khai thác lỗ hổng liên quan tới quá trình này khi pentest ứng dụng.

## Serialization vs Deserialization,

**Serialization** là quá trình chuyển đổi một cấu trúc dữ liệu phức tạp như object và các trường dữ liệu của nó thành một "flatter" format (anw không biết dịch sang tiếng việt thế nào). Format này có thể dễ dàng gửi nhận dưới dạng các bytes stream tuần tự. Việc serilize này giúp cho dễ dàng hơn trong việc:

* Ghi dữ liệu phức tạp vào bộ nhớ để giao tiếp liên tiến trình, lưu dữ liệu vào file, database
* Gửi/nhận dữ liệu phức tạp thông qua mạng, giữa các phành phần của ứng dụng hay trong các lời gọi API

Khi serialization trạng thái của đối tượng được duy trì tức là các thuộc tính và giá trị tương ứng của thuộc tính của đối tượng được giữ nguyên. 

**Deserialization** là quá trình ngược của serialization. Là quá trì restore by stream trở lại đối tượng nguyên bản, đảm bảo được trạng thái khi nó được serialize.

Byte stream sau khi được serialization có thể ở dạng binary hoặc text (XML, JSON, ...) tùy ngôn ngữ. Khả năng đọc được cũng có thể khác nhau (Đương nhiên dạng string thì đọc ngon rồi còn như binary thì khóc).

Hầu hết các ngôn ngữ lập trình đều support serialization và deserialization. 

Quá trình serialization và deserialization được tóm tắt bằng hình sau:

![Serialization]( {{site.url}}/assets/img/2021/05/14/210514_deserialization_diagram.jpg)


Lấy ví dụ về 1 đoạn code php thực hiện serialization và deserialization:

```php
<?php
class User{
   public $username;
   public $status;
}
$user = new User;
$user->username = 'vickie';
$user->status = 'not admin';
$serialized_string = serialize($user); //serialize
$unserialized_data = unserialize($serialized_string); //deserialize
echo "<h2>serialize</h2>";
var_dump($unserialized_data); 
echo "<h2>Deserialize</h2>";
var_dump($unserialized_data->status);
?>
```

Kết quả in ra sẽ như sau:

![Demo]( {{site.url}}/assets/img/2021/05/14/210514_demo1.JPG)

## Insecure Deserialization là gì,

Insecure Deserialization là lỗi xảy ra trên ứng dụng khi người khai thác (pentester/hacker) lợi dụng quá trình deserialization phá vỡ logic hoạt động theo thiết kế ban đầu của ứng dụng, thực hiện các hành động theo ý của người khai thác. Dĩ nhiên kết quả của hành động này có thể là độc hại hay không tùy vào chủ ý của người khai thác. 

Trong thực tế, attacker đã sử dụng nó thực hiện một loạt các kiểu tấn công như path traversal, Denial of Service  (DDos) attack, tạo ra reverse shell (để RCE) hoặc thi hành các đoạn mã độc hại trên target.


## Ví dụ về lỗi Insecure Deserialization,

Tôi sẽ lấy một ví dụ nhỏ demo khai thác Insecure deserialization trên PHP. Với PHP điều kiện để lỗi Insecure Deserialization có thể khai thác là:

* Đối tượng tấn công phải sử dụng Magic method (thường xảy ra trong trường hợp là developer override các method này)
* Người khai thác phải thao túng được một hoặc nhiều đoạn mã thông qua quá trình deserialize (Inject được macilious object vào)

### Magic method là gì?

Trong php magic method là các hàm đặc biệt được gọi một cách tự động. Các hàm này thường bắt đầu bằng 2 dấu gạch dưới ``(__<func>)``. Ví dụ: Hàm ai làm php cũng gặp là  ``__construct()``. Tuy nhiên nay ta sẽ chỉ tập trung vào 2 hàm là ``__destructs()`` và ``__wakeup()`` mà thôi vì 2 hàm này sẽ được tự động gọi trong quá trình deserialize.

Đầu tiên ta có đoạn code sau:

```php
<?php
   class Example{
     function __construct(){
         echo "Call contruct method<br>";
     }   
     function __wakeup(){
         echo "Call wakeup method<br>";
     }
     function __destruct(){
         echo "Call destruct method<br>";
     }
   }

   $a = new Example;
   $serialize_data = serialize($a);
   unserialize($serialize_data);
?>
```
Kết quả như sau:

![Demo2]( {{site.url}}/assets/img/2021/05/14/210514_demo_sr.JPG)

Rõ ràng khi đối tượng được tạo method ``__construct()`` sẽ được gọi ngầm định. Khi đối tượng được giải phóng method ``__destruct()`` cũng ngầm định được gọi. Còn khi unserialize thì cả method ``__destruct()`` và ``__wakeup()`` sẽ ngầm định được gọi.

Giả sử ta có một ứng dụng sử dụng method ``__distruct()`` như sau:

```php
class Example2
{
  private $hook;   
  function __construct(){
      // some PHP code...
  }   
  function __wakeup(){
      if (isset($this->hook)) eval($this->hook);
  }
}
// some PHP code...
$user_data = unserialize($_COOKIE['data']);
// some PHP code...
```

Ta có thể RCE sử dụng insecure deserialization để exploit thông qua việc truyền vào một đối tượng để unserialize. Vì lớp Example2 có một magic function chạy hàm eval() mà input được cung cấp bởi người dùng nhưng không được làm sạch.

Để exploit RCE này, đơn giản ta chỉ cần truyền vào biến data cookie một object đã được serialize một thuộc tính **$hook** mà ta mong muốn, ví dụ như đoạn code dưới đây.

```php
class Example2
{
   private $hook = "phpinfo();";
}
print urlencode(serialize(new Example2));
// We need to use URL encoding since we are injecting the object via a URL.
```

Khi ta truyền vào object Example2 vào biến data cookie đoạn mã "phpinfo();" sẽ được chạy. Chi tiết quá trình xảy ra như sau:

1. Bạn truyền một đối tượng Exmple2 đã được serilize vào trương trình giống như biến data cookie
2. Trương trình gọi method unserialize() trên biến data cookie
3. Vì biến data cookie được serialize trong đối tượng Example2, sau khi kết thúc quá trình unserialize() nó cũng tạo ra một instant tương ứng là một đối tượng của Exmple2 object.
4. Do quá trình unserialize() của Example2 tự động gọi method ``__wakeup()`` và đã được override nên ``__wakeup()`` được gọi.
5. Method ``__wakeup`` lấy giá trị $hook của object sau serialize để chạy eval($hook). 
6. $hook có giá trị là "phpinfo();" bởi vậy nó sẽ thành eval("phpinfo()") và được chạy.
7. Quá trình RCE thành công.


Đây là một ví dụ đơn giản nhất mô tả lỗi Insecure Serialize, trong thực tế lỗi này có thể xảy ra không đơn thuần chỉ ở một class/method đơn giản. Có thể là nhiều class/method kết hợp (gọi/truy vấn) với nhau thành chuỗi (Chain) mới gây ra lỗi này, do vậy mà việc kiểm soát rất khó nếu developer không có cái nhìn toàn cảnh khi tương tác giữa các class/method. Trường hợp này gọi là POP Chain. Chi tiết các bạn có thể tham khảo [tại đây!](https://sec.vnpt.vn/2019/08/ky-thuat-khai-thac-lo-hong-phar-deserialization/)

Những năm gần đây theo báo cáo của OSWAP số lượng các framework/ứng dụng mắc phải lỗi này khá cao và các lỗi đều ở mức rất nghiêm trọng (đặc biệt với các ứng dụng sử dụng php hoặc java). Rất hy vọng nội dung hôm nay sẽ có ích cho mọi người trong quá trình develop/audit/pentest để giảm thiểu lỗi này.


**Tham khảo**

[1] https://portswigger.net/web-security/deserialization

[2] https://medium.com/gdg-vit/deserialization-attacks-d312fbe58e7d

[3] https://owasp.org/www-community/vulnerabilities/PHP_Object_Injection

[4] https://sec.vnpt.vn/2019/08/ky-thuat-khai-thac-lo-hong-phar-deserialization/