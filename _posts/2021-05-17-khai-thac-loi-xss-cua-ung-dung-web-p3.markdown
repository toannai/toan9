---
layout: post
title: Cross Site Scripting (XSS) [Part 3] - Mổ xẻ lỗi store XSS (aka persistent XSS),
date: 2021-05-18 00:32:20 +0700
description: Tìm hiểu về lỗi Cross Site Scripting trên ứng dụng web - P3,
img: 2021/05/18/210517_intro_xss.png
fig-caption: # Add figcaption (optional)
tags: [Security]
---
Buổi hôm nay chúng ta sẽ đi sâu vào phân tích lỗi store XSS, mô tả những tác động của kiểu tấn công store XSS vào ứng dụng web, và quan trọng hơn là tìm hiểu làm thế nào để tìm kiếm và khai thác lỗi store XSS.

## Store XSS là gì?

Theo định nghĩa của **portswigger** thì store XSS (aka persistent XSS) xuất hiện khi một ứng dụng nhận dữ liệu từ một nguồn không đáng tin cậy (untrusted source) và dữ liệu này được phản hồi *sau đó* theo một cách không an toàn. Cần phân biệt **sau đó** với **ngay lật tức** vì đây chính là một trong những điểm khác biệt của store XSS với Reflected XSS.

## Lấy ví dụ về store XSS

Giả sử một website cho phép người dùng submit comments lên các blog posts, các post này được hiển thị cho người dùng. Khi người dùng nhấn nút submmit comment thì một request HTTP sẽ như sau:

```
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Length: 100

postId=3&comment=This+post+was+extremely+helpful.&name=Carlos+Montoya&email=carlos%40normal-user.net
```

Ngay sau khi comment được submit tất cả người dùng ghé thăm blog post sẽ nhận được đoạn text sau trong response của ứng dụng.

```
<p>This post was extremely helpful.</p>
```

Tuy nhiên do ứng dụng không thực hiện xử lý dữ liệu, người khai thác có thể submit một macilious coment như sau:

```
<script>/* Bad stuff here... */</script>
```
Trong request comment được encode URL như sau:

```
comment=%3Cscript%3E%2F*%2BBad%2Bstuff%2Bhere...%2B*%2F%3C%2Fscript%3E
```
Lúc này toàn bộ người dùng khi ghé thăm blog post trình duyệt sẽ thực thi đoạn mã sau trong context của session:

```
<p><script>/* Bad stuff here... */</script></p>
```
Bad stuff có thể là đoạn mã lấy cookie gửi thông tin này lên server hoặc thực hiện các hành động ngoài mong muốn của người dùng.

## Vài điểm phân biệt giữa Reflected XSS và Store XSS

Qua ví dụ trên ta nhận thấy vài điểm khác biệt cơ bản của Reflected XSS và Store XSS như sau:

| Reflected XSS                                                        | Store XSS                      |
| ---------------------------------------------------------------------| ------------------------------ |
| Ứng dụng phản hồi ngay lật tức                                       | Ứng dụng phản hồi sau đó       |
| Payload không được lưu vào cơ sở dũ liệu                          | Payload được lưu vào cơ sở dữ liệu|
| Yêu cầu người dùng click vào link (tạo request)                 | Không cần người dùng click vào link |
| Tác động của việc tấn công chỉ ảnh hưởng tới một cá nhân đơn lẻ  | Ảnh hưởng tới số lượng lớn người dùng |

## Phương pháp tìm kiếm và khai thác lỗi store XSS

### Vài điểm lưu ý:

Để tìm kiếm/khai thác hổng store XSS bạn cần phải xác định entry points - những điểm người khai thác có thể control data vào quá trình xử lý của ứng dụng và exit points - những điểm dữ liệu xuất hiện ở response của ứng dụng.

Entry points có thể bao gồm ở các vị trí sau:
* Parameters hoặc other data trong URL query string hoặc message body (visible or invisible value).
* The URL file path.
* HTTP request headers.
* Mọi điểm mà website nhận dữ liệu, ...

Exit points thì có thể là mọi vị trí trong website, phần được render cho người dùng cuối sử dụng.

### Khai thác

Cần chú ý rằng điểm không khai thác dược reflexed XSS thì cũng có thể bị store XSS và ngược lại - 2 loại lỗi này nhìn chung không có sự liên hệ với nhau nhiều.

Bước đầu tiên trong việc kiểm tra các lỗ hổng store XSS là xác định các liên kết giữa các entry points và exit points, theo đó dữ liệu được gửi đến một entryp point được phát ra từ một exit point.

Việc tìm kiếm store XSS trong nhiều trường hợp khá là "Thách thức" bởi các lý do sau:
* Về nguyên tắc, dữ liệu được gửi đến entry point có thể xuất hiện ra từ bất kỳ exit point nào. Ví dụ: Tên hiển thị do người dùng cung cấp (entry point) có thể xuất hiện trong audit log chỉ hiển thị cho một số người dùng ứng dụng nhất định có quyền xem audit log này.
* Dữ liệu lưu trữ hiện tại có thể thường xuyên bị ghi đè bởi các hành động khác trong ứng dụng. Ví dụ tính năng search có thể hiển thị các từ khóa tìm kiếm gần đây đôi khi nhanh chóng bị bị thay thế bởi các từ khóa tìm kiếm của các người dùng khác.

Để xác định toàn bộ các liên kết giữa entry và exit points ta có thể kiểm tra bằng việc submit vào mỗi entry point một giá trị (data) và theo dõi phản hồi của ứng dụng để phát hiện các trường hợp xuất hiện giá trị đã gửi - exit point. Đặc biệt có thể chú ý đến các chức năng ứng dụng có liên quan, chẳng hạn như nhận xét trên các bài đăng trên blog. Khi giá trị đã gửi được quan sát trong một phản hồi, bạn cần xác định xem dữ liệu có thực sự được lưu trữ trên các yêu cầu khác nhau hay không, thay vì chỉ được phản ánh trong phản hồi ngay lập tức.

Khi ta đã xác định được các liên kết giữa các entry point và exit point trong quá trình xử lý của ứng dụng, mỗi liên kết cần được kiểm tra cụ thể để phát hiện xem có lỗ hổng store XSS hay không. 

Về phương pháp kiểm tra nhìn chung cũng giống như phương pháp tìm kiếm các lỗ hổng XSS được phản ánh ở phần trước.

**Tham khảo**

[1] https://portswigger.net/web-security/cross-site-scripting/stored

