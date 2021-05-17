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

#### Reflected XSS là gì?
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
Nếu người dùng truy cập URL được cấu trúc bởi người khai thác như bên trên, thì malicious script sẽ được thực thi ngay trên chính browser của người dùng, trong phạm vi (context) của session người dùng ứng dụng. Chính bởi điểm này mà script có thể thực hiện mọi action duyệt, thu thập mọi ngoại dữ liệu dưới quyền của người dùng với sự trợ giúp đắc lực của DOM.

#### Các bước để khai thác Reflected XSS

**Bước 1:** Xác định Input vector
Để xác định input vector. Trên mỗi web page, ta phải xác định được toàn bộ các biến mà user được phép định nghĩa và làm thế nào để truyền giá trị vào nó. Nó có thể là hidden or visible inputs. Có thể là các param, header trong HTTP request, POST data, các field hidden thậm trí là các giá trị định nghĩa trước của radio hoặc selection trong HTML. Mỗi biến là một Input vector. Đương nhiên các input đó ta phải có thể Inject được giá trị vào.

**Bước 2:** Phân tích input vector

Sau bước 1, ta tiến hành phân tích từng input vector để tìm lỗ hổng tiềm năng. Để làm việc này ta sẽ Inject thử từng payload vào để thu lại phản hồi của website. Để tránh nhàm chán ta cũng có thể sử dụng các tool tự động (Fuzzer application) tìm lỗi với một chuỗi các attack string (payloads) đã có sẵn. Một vài payload rất phổ biến:

```
<script>alert(123)</script>
“><script>alert(document.cookie)</script>
```
Có rất nhiều payload khác có thể tham khảo [ở đây](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)

**Bước 3** Kiểm tra tác động
Với mỗi lần thử test vào từng Input vector ở phase trước, ta sẽ phân tích kết quả để xác định xem điểm ta vừa thử có thực sự chứa lỗ hổng hay không. Việc này được thực hiện bằng việc phân tích response HTML và tìm kiếm test input trong đó. Một khi tìm thấy, người kiểm thử cũng dễ dàng nhận diện các ký tự đặc biệt không được encoded đúng cách, bị thay thế hoặc filter. Tập hợp các ký tự đặc biệt có thể dùng để khai thác không bị filter sẽ phụ thuộc vào ngữ cảnh của section HTML đó.

Các ký tự bị thay thế trong HTML 
```
> (greater than)
< (less than)
& (ampersand)
' (apostrophe or single quote)
" (double quote)
```
Các ký tự bị thay thế trong html action hoặc javascript code
```
\n (new line)
\r (carriage return)
' (apostrophe or single quote)
" (double quote)
\ (backslash)
\uXXXX (unicode values)
```


#### Vấn đề bypass filter


***Tham khảo***

[1] https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/07-Input_Validation_Testing/01-Testing_for_Reflected_Cross_Site_Scripting.html

[2] https://portswigger.net/web-security/cross-site-scripting