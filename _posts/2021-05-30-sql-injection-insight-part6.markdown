---
layout: post
title: SQL Injection insight [Part 6] - Blind SQL Injection,
date: 2021-05-30 00:32:20 +0700
description: Phân loại SQL Injection và kinh nghiệm xử lý từng loại,
img: 2021/05/30/210530_sql_injections.png
fig-caption: # Add figcaption (optional)
tags: [Security, WebSecurity]
---
Sau mấy ngày chuyển chủ đề cho đỡ chán. Buổi nay tôi lại tiếp tục bài viết tiếp theo trong Series SQL Injection insight. Nói chung sẽ cố gắng viết theo kiểu cuốn gói viết tới đâu xong tới đó tránh để bị **mốc** vì lười. Nội dung hôm nay sẽ về Blind SQL Injection.

## Blind SQL Injection là gì
Blind SQL injection xuất hiện khi một ứng dụng chứa lỗ hổng SQL Injection nhưng khi khai thác các HTTP responses không chứa kết quả của truy vấn SQL hoặc bất kỳ lỗi nào cơ sở dữ liệu liên quan. Blind - Tiếng việt nghĩa là **mù**, đúng nghĩa không nhìn thấy bất cứ biểu hiện gì ở phía ứng dụng khi cõ lỗi bị khai thác.

## Khai thác blind SQL Injection bằng việc trigger (kích hoạt) điều kiện các responses


## Khai thác blind SQL Injection bằng việc trigger SQL error (lỗi SQL)


## Khai thác blind SQL Injection bằng việc trigger time dealays


## Khai thác bind SQL Injection sử dụng kỹ thuật out-of-band (OAST) 


** Tham khảo**

[1] [portswigger.net](https://portswigger.net/web-security/sql-injection/blind)