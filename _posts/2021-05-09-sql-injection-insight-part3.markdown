---
layout: post
title: SQL Injection insight [Part 3] - Một số tiêu trí loại phân SQL Injecion,
date: 2021-05-08 00:32:20 +0700
description: Phân loại SQL Injection và kinh nghiệm xử lý từng loại
img: 2021/05/09/210509_intro_sqlinjection.png
fig-caption: # Add figcaption (optional)
tags: [Security, WebSecurity]
---

Bài tiếp theo trong loạt SQL Injection Insight tôi sẽ đi vào phân loại lỗi SQL Injection. Việc phân loại đương nhiên chỉ là tương đối nghĩa là có thể không rõ ràng/chồng lấn nhau. Tuy nhiên điều này sẽ giúp cho ta dễ dàng hơn trong việc áp dụng payload phù hợp với từng loại đó - các lỗi cùng loại có thể có các cách ứng xử giống nhau. Let's start.

## Visible SQL Injection vs Blind SQL Injection

Cách phân biệt đầu tiên là dựa trên dấu hiệu phản hồi lỗi trở lại của ứng d. Theo dấu hiệu này tạm chia thành Visible và Blind. 

* Với Visible SQL Injection chúng ta có thể nhìn thấy kết quả của việc Injection. Kết quả này có thể là success hoặc false nhưng người dũng vẫn nhìn thấy.

Ví dụ: 

![visible]( {{site.url}}/assets/img/2021/05/09/210509_visible_sql.JPG){:width="600px"}

* Ngược lại, vói Blind thì kết quả Injection sẽ không hiển thị ra và ta không thể nhìn thấy nó dù ứng dụng có lỗi, vẫn có thể khai thác được. Với Blind sql injeciton ta có 2 kiểu thường gặp là Blind-boolean-based SQLi và Blind-time-based SQLi (Hai kiểu này sẽ làm rõ dần ở các phần sau).

* Rõ ràng Visible SQL Injection thì dễ phát hiện, khai thác hơn Blind SQL Injection. Tuy nhiên việc tấn công bằng Blind SQL Injection không phải là không thể, vẫn có một số cách để phát hiện và khai thác. Muốn biết các kỹ thuật này có thể theo dõi những phần tiếp theo của loạt bài viết.

## In-band(Inline) vs Out-Of-Band

Phân loại in-band(inline) với Out-Of-Band là dựa trên tiêu chí kênh kết nối tấn công và kênh phản hồi. 

* In-band(inline): Kênh kết nối tấn công gửi payload và kênh phản hồi là cùng một kênh. 2 Loại in-band(Inline) phổ biến nhất là Error-base SQLi và Union-Base SQLi. Ta dễ thấy 2 loại này đều phản hồi lại cho attacker/pentester luôn ngay khi có kết quả ở cùng một kênh (thậm trí là cùng kết nối TCP/UDP luôn).

* Of-of-band: Kênh kết nối tấn công gửi payload và kênh phản hồi ở hai kênh khác nhau. Loại này nghe có vẻ hơi khó hiểu. Nhưng hãy hình dung thế này. Một số CSDL SQL có hỗ trợ gửi đi các request DNS và HTTP (VD trường hợp của SQL server là command xp_dirtree) thông qua việc thi hành các câu lệnh SQL. Trong quá trình detect và tấn công attacker/pentester có thể lợi dụng gọi các lệnh SQL này, cố gắng truyền vào giá trị request (máy chủ dns/http) là thông tin domain/máy chủ của họ qua input từ phía user (Input đương nhiên là các payload sql command). Nếu có phản hồi từ phía ứng dụng (có log request của dns hoặc http access trên máy chủ của họ) thì coi như là ứng dụng có lỗ hổng và SQL payload được thực thi thành công. Trong nhiều trường hợp kết quả của truy vấn còn có thể gửi qua kênh này dựa vào việc attacker truyền vào request http gửi đi thông tin kết quả của truy vấn SQL Injection. Out-of-band thường được sử dụng với blind SQL Injection.

Kỹ thuật Out-of-band mô tả bằng hình sau:

![outband]( {{site.url}}/assets/img/2021/05/09/210509_out_band.jpeg){:width="600px"}



***Tham khảo***

https://infosecwriteups.com/out-of-band-oob-sql-injection-87b7c666548b>






