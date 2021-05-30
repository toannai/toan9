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

Việc phát hiện và khai thác blind SQL Injection về cơ bản là sẽ khó hơn các loại khác. Tuy nhiên không phải là không thể. Cũng có những cách thức đặc hiệu để xử lý nó. Ta sẽ cùng nhau phân tích chi tiết ở các phần sau.

## Khai thác blind SQL Injection bằng việc trigger (kích hoạt) điều kiện các responses

Phân tích về trường hợp này ta sẽ lấy ví dụ về một ứng dụng tracking cookies để tổng hợp phân tích về việc sử dụng website. Một request ứng dụng khi gửi liên server sẽ gửi kèm một Cookie header như sau:

```
Cookie: TrackingId=u5YD3PapBcR4lN3e7Tj4
```
Khi request gửi TrackingId đến server, phía ứng dụng sẽ sinh ra câu truy vấn SQL như sau:

```
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'
```
Query này chứa lỗi SQL Injection nhưng kết quả của nó sẽ không được trả lại phía user. Tuy nhiên ứng dụng sẽ có những biểu hiện khác nhau phụ thuộc vào kết quả trả lại của lệnh truy vấn trên. Nếu trả lại data (Ghi nhận TrackingId) thì ứng dụng sẽ trả lại message "Welcome back" bên trong page.

Hành vi này là đủ để có thể khai thác blind SQL Injection và thu thập thông tin bằng việc trigger các điều kiện khác nhau của response tương ứng điều kiện Inject vào ứng dụng. Ví dụ giả sử ta gửi các request bao gồm các giá trị TrackingId như sau:

```
…xyz' AND '1'='1
…xyz' AND '1'='2
```
Giá trị đầu tiên sẽ trả lại kết quả bởi vì AND '1'='1 có giá trị là TRUE và message "Welcome back" sẽ được hiển thị. Trong khi đó giá trị thứ 2 ứng dụng sẽ không hiển thị gì vì điều kiện Inject có giá trị là FALSE. Với mỗi lần Inject đơn lẻ ta có thể xác định được câu trả lời tương ứng (True hoặc False) nên việc khai thác nhìn chung mất thời gian một chút.

Lấy ví dụ chúng ta muốn lấy password của user Administrator trong bảng Users có cột cột Username và Password. Ta có thể thực việc này bằng việc gửi một tá các inputs để test nhầm thu thập từng ký tự của Password. Để làm việc này chúng ta bắt đầu với Input như sau:

```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
```

Nếu message "Welcome back" được trả lại thì chứng tỏ điều kiện Inject có giá trị là true bởi vậy ký tự đầu tiên của Password sẽ lớn hơn m.

Tiếp tục ta sẽ gửi input dưới đây:

```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 't
```



## Khai thác blind SQL Injection bằng việc trigger SQL error (lỗi SQL)


## Khai thác blind SQL Injection bằng việc trigger time dealays


## Khai thác bind SQL Injection sử dụng kỹ thuật out-of-band (OAST) 


** Tham khảo**

[1] [portswigger.net](https://portswigger.net/web-security/sql-injection/blind)