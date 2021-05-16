---
layout: post
title: SQL Injection insight [Part 1] - Basic SQL Injection,
date: 2021-05-04 00:32:20 +0700
description: Bài viết đầu tiên trong loạt bài tìm hiểu về lỗi SQL Injection
img: 2021/05/05/210505_intro_sqlinjection.png
fig-caption: # Add figcaption (optional)
tags: [Security, WebSecurity]
---
>Injection đặc biệt là SQL Injection phải nói là nổi tiếng thuộc hàng top trong nhóm những lỗ hổng ứng dụng Web. Nổi tiếng theo nghĩa là hầu hết ai làm việc liên quan tới ứng dụng Web, thậm trí rộng hơn là Công nghệ thông tin đều biết hoặc từng nghe về nó. Tuy nhiên nổi tiếng cỡ vậy nhưng theo thống kê của OSWAP thì nhiều năm liền nó vẫn nằm ở top 1 các lỗ hổng nguy hiểm nhất đối với ứng dụng web. Trong thời gian tới tôi sẽ viết một loạt bài chia sẻ về SQL Injection là những ghi chú của tôi khi tìm hiểu về Web security. Rất hy vọng rằng các thông tin mà tôi có được hữu ích cho mọi người.

## Roadmap

Để dễ follow các ghi chú về chủ đề này, tôi sẽ mô tả sơ qua về chiến lược (tactics) của tôi. Tóm gọn lại gồm các bước sau:
* Đầu tiên tôi sẽ giới thiệu sơ qua về SQL Injection (Nó là gì? Để làm gì? Tìm ở đâu? Sương sương mấy cái liên quan ngoài lề trước khi bắt đầu). 
* Sau đó đào sâu vào phân tích các chiến lược phát hiện, build payload khai thác lỗi, cách dùng một số công cụ khai thác lỗi này một cách tự động. 
* Cuối cùng là các áp dụng các nội dung đã tìm hiểu vào thực chiến qua các bài CTF (Cature the Flag) Web.   

## Một số kiến thức cơ bản cần có trước khi bắt đầu

Loạt bài này cần một số kiến thức cơ bản bắt buộc phải có trước khi bắt đầu vào việc (Nếu đã từng tham gia lập trình web app. Ví dụ: như đã từng code web sử dụng php-mysql thì tuyệt).
* Có kiến thức cơ bản về web, mô hình hoạt động của ứng dụng web, các khái niện cơ bản như kiến thức về giao thức http, session, cookies. Tối thiểu nhất hiểu được cái hình này nói gì.

![dynamic web]( {{site.url}}/assets/img/2021/05/05/210505_dynamic_web.png){:width="600px"}


* Kiến thức về cú pháp truy vấn SQL. Không cần sâu lắm nhưng phải biết các ý nghĩa/cú pháp của các chỉ thị bên dưới.

![sql verb]( {{site.url}}/assets/img/2021/05/05/210505_sql_verb.PNG){:width="400px"}

![sql modifier]( {{site.url}}/assets/img/2021/05/05/210505_sql_modifiers.PNG){:width="400px"}

![sql data type]( {{site.url}}/assets/img/2021/05/05/210505_sql_data_type.PNG){:width="400px"}

![sql special charactor]( {{site.url}}/assets/img/2021/05/05/210505_sql_special_charactor.PNG){:width="400px"}

Nếu chưa có các kiến thức này (nhìn các hình bên trên mà không hiểu nó nghĩa là gì) thì tôi khuyên bạn nên tìm hiểu chúng trước vì nó là các kiến thức nền tảng cơ bản để có thể bắt đầu các phần sau. 

>Còn nếu bạn đã sẵn sàng rồi thì **Let's go**

## Giới thiệu về SQL Injection

###  SQL Injection là gì?

Đã có rất nhiều bài viết mô tả SQL Injection là gì. Tôi không nhắc lại ở đây, chỉ xin lấy một ví dụ nho nhỏ để mô tả định nghĩa cũng như sự nguy hiểm của nó. 

Tôi vẫn nhớ những ngày đầu lập trình web bằng php-mysql. Tính năng login của web app tôi hay viết thế này.

>Link github: [link app demo in github](https://github.com/toannai/sql_injection_tutorial/tree/master/tut1)

```php
      <?php
         // Login
         include("config.php"); // Load config from config file
         session_start();
   
         if($_SERVER["REQUEST_METHOD"] == "POST") {
            // username and password sent from form 
            
            $myusername = $_POST['username'];
            $mypassword = $_POST['password']; 
            
            $sql = "SELECT username FROM users WHERE username = '$myusername' and password = '$mypassword'";
            $result = mysqli_query($db,$sql);
            $row = mysqli_fetch_array($result);
            #$active = $row['active'];
            
            $count = mysqli_num_rows($result);
            
            // If result matched $myusername and $mypassword, table row must be 1 row
            
            if($count == 1) {
               // ...... Tạo session va redirect ve trang main sau dang nhap
               echo "Hello: ".$row['username'].". Login success <br>"; 
                 
            } else {
               // ..... Quay tro lai trang dang nhap
               echo "Please login! <br>";
            }
         }
      ?>
```

Mô tả qua một chút: Phần kiểm tra đăng nhập của tôi sẽ lấy username lưu vào biến $myusername và password lưu vào biến $mypassword từ biến môi trường ``$_POST`` là dữ liệu form submit được nhập vào bởi người dùng. Sau khi lấy được username và password thì tôi sẽ dùng nó để build lên câu lệnh sql **SELECT * FROM users WHERE username = '$myusername' and password = '$mypassword'**. Lệnh này được gửi xuống thi hành ở csdl mysql bên dưới. Nếu kết quả trả về là tồn tại 1 bản ghi (username và password đều đúng <=> Match với ít nhất 1 row nào trong table users) thì sẽ tạo session cho user đã login và redirect về màn hình làm việc của user đã login. Ngược lại nếu không trả lại bản ghi nào (username hoặc password sai hoặc cả hai sai <=> Không match row nào trong table users) sẽ redirect quay trở về trang đăng nhập bắt người dùng đăng nhập lại. 

Với trường hợp thông thường username là admin và password là 123456A (Password đúng). Kết quả là đăng nhập thành công, câu query thực tế được build lên và gửi xuống csdl là.

![sql true]( {{site.url}}/assets/img/2021/05/05/210505_truepass.PNG){:width="600px"}

Tuy nhiên thay đổi một chút. Trong quá trình đăng nhập tôi không nhập vào username và password thông thường thay vào đó tôi nhập username là ``admin' -- -`` và password nhập đại một cái gì đó chả hạn abc123 (Password linh tinh sai). Ta lại thấy đăng nhập thành công với user là admin. Lúc này câu lệnh truy vấn csdl sẽ có dạng

![sql wrong]( {{site.url}}/assets/img/2021/05/05/210505_sql_wrongpass.png){:width="600px"}

Lý giải cho việc đăng nhập thành công với user admin là phần kiểm tra password bôi vàng (```-- -' and password = 'abc123'```) do đứng sau dấu ```--``` là ký tự comment trong mysql nên sẽ bị bỏ qua khi thi hành trong truy vấn csdl. Lúc này truy vấn chỉ còn lại là.

```
SELECT username FROM users WHERE username = 'admin'
```

Do user admin nghiễm nhiên nằm trong csdl nên kết quả trả lại luôn đúng và luôn tồn tại một bản ghi có username là admin trong bảng users app báo login thành công.

***Như vậy chả cần password của user admin ta cũng có thể đăng nhập thành công. Thật là nguy hiểm phải không***

Qua ví dụ tạm định nghĩa thế này:

>SQL Injection là một kỹ thuật lợi dụng những lỗ hổng về câu truy vấn của các ứng dụng. Được thực hiện bằng cách chèn thêm một đoạn SQL để làm sai lệnh đi câu truy vấn ban đầu, từ đó có thể khai thác dữ liệu từ database theo mục đích của kẻ tấn công.

### Mục tiêu SQL Injection?

Có quá nhiều mục tiêu nhắm đến khi attacker khai thác SQL Injection. Ở đây tôi chỉ liệt kê vài nội dung mà tôi biết và phân loại **sơ sơ** dựa trên quan điểm cá nhân mà thôi.
* Đầu tiên là để khai thác các dữ liệu nhạy cảm. Ví dụ đánh cắp dữ liệu thông tin khách hàng, nội bộ tổ chức.
* Vượt qua các cơ chế xác thực (Authentication), phân quyền (Authorization). Cụ thể là khai thác để vượt qua các cơ chế xác thực để vào hệ thống trái phép, leo quyền từ user này sang các user khác, nâng đặc quyền user thông thường sang các user có đặc quyền cao hơn.
* Tác động vào csdl, sửa đổi các thông tin trong hệ thống. Trong thời gian đi làm tôi đã từng gặp những case pentest vào hệ thống sửa đổi csld để trừ tiền trong tài khoản ngân hàng hoặc ghi nợ vào tài khoản điện thoại của các VIP trong tập đoàn V.
* Upload shell/reverser shell chiếm quyền điều khiển server, làm cửa hậu để xâm nhập hệ thống nội bộ của tổ chức.
... 

Còn rất nhiều các mục tiêu khác nữa mà khó có thể lường trước được vì chúng rất đa dạng.

### Tìm lỗi SQL Injection ở đâu?

Đứng trước một ứng dụng cần thực hiện pentest, lúc mới bắt đầu tôi đặt ra câu hỏi ***"Tìm lỗi SQL Injection ở đâu bây giờ?"***. Dĩ nhiên là đâu cũng được, miễn là có lỗi, tìm ra được lỗi :)

Quay lại ví dụ, dễ nhận thấy việc SQL Inject được attacker thực hiện dưới góc độ người dùng, họ gửi các macilious input (Từ nay cứ nói tới cái gì độc độc là bắt đầu bằng cụm từ ***macilious - Độc hại***) vào logic của ứng dụng theo chủ đích của mình. Điều này đồng nghĩa với việc vị trí bị khai thác phải là các vị trí mà ứng dụng đọc dữ liệu từ người dùng và người dùng có thể thay đổi dữ liệu đó. Tổng kết lại có các vị trí sau ta cần đặc biệt chú ý.

* GET URL query parameters (Biến query trên URL ứng dụng gửi qua phương thức GET)
* POST Payload (Dữ liệu là các nội dung gửi trong payload của phương thức POST)
* HTTP Cookies (Biến cookie lưu dưới trình duyệt/ứng dụng phía người dùng)
* HTTP Header, Special: User Agent (Header mà người dùng/ứng dụng gửi lên. Thường gặp nhất là User Agent)

Bài viết đầu tiên tôi viết khá dài rồi nên là tôi xin dừng lại tại đây. Bạn nào ghé qua đọc mà có ý kiến góp ý cho tôi xin để lại dưới comment giúp tôi. Cảm ơn bạn.

>PS: Loạt bài viết có tham khảo nhiều nguồn. Trong đó nội dung chủ yếu trong loạt bài giảng **SANS 542 - Web App Penetration Testing and Ethical Hacking**.
