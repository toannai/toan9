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

Lấy ví dụ chúng ta muốn lấy password của user Administrator trong bảng Users có cột cột Username và Password. Ta có thể thực việc này bằng việc gửi một tá các inputs để test nhầm thu thập từng ký tự của Password bằng thuật toán Binary Search nổi tiếng - Khoanh vùng dần dần để xác định từng ký tự (Các bạn có thể google đọc thêm về phương pháp này). 

Để làm việc này chúng ta bắt đầu với Input như sau:

```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
```

Nếu message "Welcome back" được trả lại thì chứng tỏ điều kiện Inject có giá trị là true bởi vậy ký tự đầu tiên của Password sẽ lớn hơn m.

Tiếp tục ta sẽ gửi input dưới đây:

```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 't
```

Nếu ứng dụng không trả lại message "Welcome back" điều đó chỉ ra rằng điều kiện Injection là FALSE bởi vậy ký tự đầu tiên của Password nó sẽ không lớn hơn t.

....

Cho đến cuối cùng khi đã xác định được ký tự đầu tiên của password ta sẽ verified lại bằng payload sau và quan sát message trả lại.

```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's
```

Cứ thế ta lần lượt xác định các ký tự tiếp theo trong password của user Administrator cho tới ký tự cuối cùng.

## Khai thác blind SQL Injection bằng việc trigger SQL error


Ở ví dụ trước mặc dù kết quả trả lại của truy vấn không hiển thị với người dùng tuy nhiên phía ứng dụng lại phản hồi trở lại cho ta những dấu hiệu tương ứng để giúp ta nhận biết kết quả của truy vấn là TRUE hay FALSE. Tuy nhiên trong một số tình huống kỹ thuật này cũng không thể thực hiện được do phía ứng dụng không thể hiện sự khác biệt nào. 

Trong tình huống này chúng ta có thể thử phương án quan sát phản hồi của ứng dụng trả lại dựa trên điều kiện trigger SQL error phụ thuộc vào điều kiện Injection. Việc này có thể được thực hiện dựa trên việc sửa đổi câu query làm cho nó gây ra database eror nếu điều kiện inject là TRUE và không gây ra database error nếu điều kiện injection là FALSE.

Để mô tả ý tưởng này giả sử ta gửi 2 request bao gồm TrackingId như sau:

```
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```
Các inputs này có sử dụng từ khóa CASE để kiểm tra điều kiện và trả lại trả lại một biểu thức tùy thuộc vào điều kiện TRUE hoặc FALSE. Với input thứ nhất biểu thức CASE có kết quả là 'a'. Điều này làm cho kết quả của lệnh SQL ban đầu cũng không có lỗi. Với input thứ 2, phép tính 1/0 sẽ lỗi chi cho 0, lỗi này rất có thể sẽ làm thay đổi HTTP response của ứng dụng và chúng ta có thể sử dụng chúng để phân biệt với trường hợp trả lại điều kiện là TRUE.

Sử dụng kỹ thuật này linh hoạt áp dụng theo cách đã mô tả ở ví dụ phần trên (Sử dụng binary search) ta lại có thể kiểm tra từng ký tự sau mỗi lần thử.

```
xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
```

## Khai thác blind SQL Injection bằng việc trigger time dealays

Trong nhiều trường hợp không thể sử dụng cả 2 kỹ thuật trên vì không có sự khác biệt giữa phản hồi của ứng dụng khi câu lệnh SQL có kết quả True/False, hoặc Database error có hay không. Trong những trường hợp này ta có thể thử một phương pháp khác gọi là TIME DELAYS.

Các câu query SQL thường xử lý một cách bất động bộ bởi ứng dụng, bởi vậy delay việc thi hành các câu lệnh SQL sẽ gây ra delay HTTP response. Điều này cho phép chúng ta suy ra điều kiện đưa vào dựa trên thời gian thực hiện trước khi nhận được HTTP response.

Kỹ thuật trigger TIME DELAY phụ thuộc rất nhiều vào kiểu database được sử dụng. Trên MS SQL Server ta có thể kiểm tra và kích hoạt TIME DELAY dựa trên cách diễn đạt là TRUE hay không.

```
'; IF (1=2) WAITFOR DELAY '0:0:10'--
'; IF (1=1) WAITFOR DELAY '0:0:10'--
```

Câu lệnh đầu tiên sẽ không trigger TIME DELAY bởi 1=2 có kết quả là flase. Câu lệnh thứ 2 sẽ delay 10s bởi điều kiện 1=1 có kết quả là True.

Tương tự như các trường hợp mô tả trên ta lại có thể kiểm tra từng ký tự tại mỗi lần test bằng câu lệnh sau.

```
'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--
```




## Khai thác bind SQL Injection sử dụng kỹ thuật out-of-band (OAST) 


** Tham khảo**

[1] [portswigger.net](https://portswigger.net/web-security/sql-injection/blind)