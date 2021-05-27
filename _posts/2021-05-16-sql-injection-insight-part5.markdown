---
layout: post
title: SQL Injection insight [Part 5] - Union base SQL Injection,
date: 2021-05-16 00:32:20 +0700
description: Phân loại SQL Injection và kinh nghiệm xử lý từng loại,
img: 2021/05/16/210516_sql_injections.png
fig-caption: # Add figcaption (optional)
tags: [Security, WebSecurity]
---

Bài viết tiếp theo trong loạt bài SQL Injection insight tôi sẽ viết về Union Base SQL Injection. Nói đây là 1 loại SQL Injection thì có vẻ không đúng cho lắm, sẽ phù hợp hơn nếu nghĩ nó là một kỹ thuật trong khai thác SQL Injection thì hợp lý hơn. Vì nó là phương pháp được sử dụng trong nhiều trong quá trình khai thác ở các loại lỗi SQL injection khác nhau, cả visible, blind, error base, ....

Khi một ứng dụng có lỗ hổng SQL injection kết quả của query có thể được trả lại bên trong response của ứng dụng, từ khóa UNION có thể được sử dụng để giúp ta lấy được dữ liệu từ các bảng khác trong database. Đây là kết quả của SQL Injection UNION attack.

Nhìn lại cấu trúc của câu lệnh UNION trong SQL query:

```
SELECT a, b FROM table1 UNION SELECT c, d FROM table2
```

Kết quả của câu lệnh sẽ là một tập kết quả với 2 cột bao gồm cột a và b trong bảng table1 và cột c, d trong bảng 2.

| a   | b    |
|-----|------|
| c   | d    |

Điều kiện để câu lệnh UNION có thể thực hiện được là:
* Các queries phải trả lại cùng số cột
* Data types mỗi cột phải tưởng thích giữa các queries
Để thực hiện SQL Injection UNION attack phải đảm bảo 2 điều kiện này.Việc này liên quan tới làm rõ các nội dung sau:
* Có bao nhiêu cột đang được trả về từ truy vấn ban đầu?
* Những cột nào được trả về từ truy vấn ban đầu thuộc loại dữ liệu phù hợp để giữ kết quả từ truy vấn được inject vào?

## Xác định số cột yêu cầu trong SQL Injection UNION attack
Khi thực hiện một cuộc tấn công SQL injection UNION, có hai phương pháp hiệu quả để xác định có bao nhiêu cột đang được trả về từ truy vấn ban đầu.

Phương pháp đầu tiên liên quan đến việc đưa vào một loạt các mệnh đề ORDER BY và tăng chỉ số cột được chỉ định cho đến khi xảy ra lỗi. Ví dụ: giả sử điểm chèn là một chuỗi được trích dẫn trong mệnh đề WHERE của truy vấn ban đầu, bạn sẽ gửi:

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
etc.
```

Chuỗi trọng tải này sửa đổi truy vấn ban đầu để sắp xếp các kết quả theo các cột khác nhau trong tập kết quả. Cột trong mệnh đề ORDER BY có thể được chỉ định bởi chỉ mục của nó, vì vậy bạn không cần biết tên của bất kỳ cột nào. Khi chỉ mục cột được chỉ định vượt quá số cột thực tế trong tập kết quả, cơ sở dữ liệu trả về lỗi, chẳng hạn như:
```
The ORDER BY position number 3 is out of range of the number of items in the select list.
```
Ứng dụng thực sự có thể trả về lỗi cơ sở dữ liệu trong phản hồi HTTP của nó hoặc nó có thể trả về một lỗi chung hoặc đơn giản là không trả về kết quả nào. Với điều kiện bạn có thể phát hiện một số khác biệt trong phản hồi của ứng dụng, bạn có thể suy ra có bao nhiêu cột đang được trả về từ truy vấn.

Phương pháp thứ hai liên quan đến việc gửi một loạt các trọng tải UNION SELECT chỉ định một số giá trị rỗng khác nhau:

```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
etc.
```
Nếu số lượng null không khớp với số cột, cơ sở dữ liệu trả về lỗi, chẳng hạn như:
```
All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.
```
Một lần nữa, ứng dụng thực sự có thể trả lại thông báo lỗi này hoặc có thể chỉ trả lại một lỗi chung hoặc không có kết quả. Khi số lượng rỗng khớp với số cột, cơ sở dữ liệu trả về một hàng bổ sung trong tập kết quả, chứa các giá trị null trong mỗi cột. Ảnh hưởng đến phản hồi HTTP kết quả phụ thuộc vào mã của ứng dụng. Nếu may mắn, bạn sẽ thấy một số nội dung bổ sung trong phản hồi, chẳng hạn như một hàng bổ sung trên bảng HTML. Nếu không, các giá trị null có thể gây ra một lỗi khác, chẳng hạn như NullPointerException. Trường hợp xấu nhất, phản hồi có thể không phân biệt được với phản hồi do số lượng giá trị rỗng không chính xác, làm cho phương pháp xác định số lượng cột này không hiệu quả.

## Tìm kiếm các cột có kiểu dữ liệu hữu ích trong SQL Injection UNION attack
Lý do để thực hiện một cuộc tấn công SQL injection UNION là để có thể truy xuất kết quả từ một truy vấn được đưa vào. Nói chung, dữ liệu thú vị mà bạn muốn truy xuất sẽ ở dạng chuỗi, vì vậy bạn cần tìm một hoặc nhiều cột trong kết quả truy vấn ban đầu có kiểu dữ liệu hoặc tương thích với dữ liệu chuỗi.

Sau khi đã xác định số lượng cột cần thiết, bạn có thể thăm dò từng cột để kiểm tra xem nó có thể chứa dữ liệu chuỗi hay không bằng cách gửi một loạt các trọng tải UNION SELECT lần lượt đặt một giá trị chuỗi vào mỗi cột. Ví dụ: nếu truy vấn trả về bốn cột, bạn sẽ gửi:
```
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```

Nếu kiểu dữ liệu của một cột không tương thích với dữ liệu chuỗi, truy vấn được đưa vào sẽ gây ra lỗi cơ sở dữ liệu, chẳng hạn như:
```
Conversion failed when converting the varchar value 'a' to data type int.
```
Nếu lỗi không xảy ra và phản hồi của ứng dụng chứa một số nội dung bổ sung bao gồm giá trị chuỗi được chèn vào, thì cột có liên quan sẽ phù hợp để truy xuất dữ liệu chuỗi.


**Tham khảo**

[1] https://portswigger.net/web-security/sql-injection/union-attacks