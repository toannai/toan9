---
layout: post
title: Cross Site Scripting (XSS) [Part 2] - Phân tích chi tiết lỗi XSS và các vấn đề liên quan,
date: 2021-05-17 00:32:20 +0700
description: Tìm hiểu về lỗi Cross Site Scripting trên ứng dụng web - P2,
img: 2021/05/17/210517_intro_xss.png
fig-caption: # Add figcaption (optional)
tags: [Security, WebSecurity]
---
Các bài viết của tôi thường được cắt ngắn với thời lượng nhỏ để tránh gây sự mệt mỏi cho người theo dõi. Và buổi hôm  nay tối sẽ tiếp tục đi phân tích những nội dung liên quan tới lỗi XSS.

## Phân loại lỗi XSS
Theo hầu hết các tài liệu thì lỗi XSS được chia làm 3 loại.
* Reflected XSS
* Store XSS
* DOM XSS

### Reflected XSS: 
Là kịch bản tấn công đơn giản nhất của XSS. Nó phát sinh khi một ứng dụng nhận được dữ liệu tấn công (payload) trong request HTTP và phản hồi lập tức dữ liệu đó ngay trong respone theo cách không an toàn. Chính lý do này nên nó được gọi là Reflected - Phản chiếu.

Ví dụ: Có một ứng dụng nhận dữ liệu từ biến GET có tên là message và hiện thị cho người dùng.
Trong app có đoạn code sau:
```
<?php
...
echo "<p>Status: ".$message</p>
...
?>
```
Lúc này khi truyền vào biến message đoạn text **All is well**
```
https://insecure-website.com/status?message=All+is+well.
```
Kết quả phản hồi
```
<p>Status: All is well.</p>
```
Do ứng dụng không thực hiện làm sạch dữ liệu bởi vậy nên dễ dàng bị khai thác theo cấu trúc sau:
```
https://insecure-website.com/status?message=<script>/*+Bad+stuff+here...+*/</script>
```
Trình duyệt không nhận ra được đây là malicious script, kết quả là đoạn javascript độc hại sẽ được thực thi.
```
<p>Status: <script>/* Bad stuff here... */</script></p>
```
Nếu người dùng truy cập URL được cấu trúc bởi người khai thác như bên trên, thì mailcious script sẽ được thực thi ngay trên chính browser của người dùng, trong phạm vi (context) của session người dùng ứng dụng. Chính bởi điểm này mà script có thể thực hiện mọi action duyệt, thu thập mọi ngoại dữ liệu dưới quyền của người dùng với sự trợ giúp đắc lực của DOM.

## Vấn đề offucate