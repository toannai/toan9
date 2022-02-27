---
layout: post
title: SQL Injection insight [Part 5] - Union base SQL Injection,
date: 2021-05-16 00:32:20 +0700
description: Phân loại SQL Injection và kinh nghiệm xử lý từng loại,
img: 2021/05/16/210516_sql_injections.png
fig-caption: # Add figcaption (optional)
tags: [Security]
---
Buổi hôm nay tối sẽ đi sâu vào phân tích SQL Injection UNION base. Theo tôi đây không hẳn là một loại SQL Injection mà thực sự nó là một kỹ thuật trong khai thác SQL Injection thì hợp lý hơn. Nhưng hãy quên cái đó đi, không quan trọng cho lắm. Chúng ta bắt đầu nào.


## 1. Nhắc lại về cú pháp lệnh UINION trong SQL Query
Trong SQL query về cơ bản cú pháp lệnh UNION như sau:

```
SELECT a, b FROM table1 UNION SELECT c, d FROM table2
```

Kết quả của câu lệnh sẽ là một tập kết quả với 2 cột bao gồm cột a và b trong bảng table1 và cột c, d trong bảng 2.

| a   | b    |
|-----|------|
| c   | d    |

Điều kiện để câu lệnh UNION có thể thực hiện được là:
* Câu lệnh query trước và sau UNION phải trả lại cùng số cột. Theo ví dụ trên cả 2 query đều trả lại 2 cột.
* Data types mỗi cột trả lại phải tưởng thích (cùng hoặc có thể convert lẫn nhau) giữa các queries. Theo ví dụ trên a với c, b với d phải tương thích nhau.
Bạn hãy nhớ lấy điều này, ta sẽ dùng tới nó ở phần sau.

## 2. UNION base SQL Injection

Khi một ứng dụng có lỗ hổng SQL injection kết quả của query có thể được trả lại bên trong response của ứng dụng, bằng việc lợi dụng từ khóa UNION có thể giúp ta lấy được dữ liệu từ các bảng khác trong database. Đây là kết quả của SQL Injection UNION attack.

Okie, thử lấy một ví dụ nhỏ:

Giả sử truy vấn nguyên bản ban đầu như thế này (với product_id là được truyền vào từ người dùng)

```
SELECT product_name, product_type from products WHERE product_id=1
```

Bằng cách nào đó ta có thể inject vào được truy vấn này thành

```
SELECT product_name, product_type from products WHERE product_id=1 UNION SELECT username, password FROM users WHERE username='admin'
```

Lúc này kết quả trả lại sẽ gồm 2 cột

| Orange   | fruit     |
|----------|-----------|
| admin    | 123qwea@  |

Trong đó raw đầu là kết quả truy vấn của mệnh đề trước UNION. Raw thứ 2 là kết quả của mệnh đề truy vấn sau UINION.

Tạm gọi phần trước UNION là **Original Query** và phần sau là **Inject query** (Vì phần trước là phần ứng dụng có sẵn, phần sau ta cố tình chèn vào).

Kết hợp phần điều kiện thực hiện lệnh UNION ở **mục 1** ta dễ dàng thấy để có thể thực hiện Injection UNION attack ta cần làm rõ các nội dung sau:

* Có bao nhiêu cột đang được trả về từ Original query?
* Những cột được trả về từ Original query thuộc loại dữ liệu nào và thích hợp để giữ những giá trị nào từ kết quả từ truy vấn ta thực hiện inject vào (Inject query)?

Phần sau sẽ cùng nhau làm rõ các chiến lược thực hiện từng yêu cầu này.

## 3. Xác định số cột yêu cầu trong SQL Injection UNION attack
Khi thực hiện một cuộc tấn công SQL injection UNION, có hai phương pháp hiệu quả để xác định có bao nhiêu cột đang được trả về từ truy Original query.

* Phương pháp đầu tiên liên quan đến việc đưa vào một loạt các mệnh đề ORDER BY và tăng chỉ số cột được chỉ định cho đến khi xảy ra lỗi. Ví dụ: giả sử điểm chèn là một chuỗi được trích dẫn trong mệnh đề WHERE của truy vấn ban đầu, bạn sẽ gửi:

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
etc.
```

Chuỗi payload này sửa đổi truy vấn ban đầu để sắp xếp các kết quả theo các cột khác nhau trong tập kết quả. Cột trong mệnh đề ORDER BY có thể được chỉ định bởi chỉ mục của nó, vì vậy ta không cần biết tên của bất kỳ cột nào. Khi chỉ mục cột được chỉ định vượt quá số cột thực tế trong tập kết quả, cơ sở dữ liệu trả về lỗi, ví dụ với MySQL lỗi như sau:
```
The ORDER BY position number 3 is out of range of the number of items in the select list.
```
Ứng dụng thực sự có thể trả về lỗi cơ sở dữ liệu trong phản hồi HTTP của nó hoặc nó có thể trả về một lỗi chung hoặc đơn giản là không trả về kết quả nào. Với điều kiện này ta có thể phát hiện một số khác biệt trong phản hồi của ứng dụng, từ đó có thể suy ra có bao nhiêu cột đang được trả về từ truy vấn.

* Phương pháp thứ hai liên quan đến việc gửi một loạt các payload UNION SELECT với một số giá trị NULL khác nhau:

```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
....
```
Nếu số lượng NULL không khớp với số cột, cơ sở dữ liệu trả về lỗi, ví dụ với MySQL sẽ trả lại lỗi:
```
All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.
```
Ứng dụng thực sự có thể trả lại thông báo lỗi này hoặc có thể chỉ trả lại một lỗi chung hoặc không có kết quả. Khi số lượng NULL khớp với số cột, cơ sở dữ liệu trả về một hàng bổ sung trong tập kết quả, chứa các giá trị NULL trong mỗi cột. Việc này nếu thành công sẽ làm thay đổi HTTP response, tùy mức độ mà sự thay đổi khác nhau. Nếu may mắn, ta sẽ thấy một số nội dung bổ sung trong phản hồi, chẳng hạn như một hàng bổ sung trên bảng HTML. Nếu không, các giá trị NULL có thể gây ra một lỗi khác, chẳng hạn như NullPointerException. Trường hợp xấu nhất, phản hồi inject thành công (đủ số lượng trường NULL) có thể không phân biệt được với phản hồi inject thất bại (số lượng trường NULL sai) làm cho phương pháp xác định số lượng cột này không hiệu quả.

## 4. Tìm kiếm các cột có kiểu dữ liệu hữu ích trong SQL Injection UNION attack

Lý do để thực hiện một cuộc tấn công SQL injection UNION là lợi dụng **Original query** để có thể truy xuất kết quả **Inject query**. Nói chung, dữ liệu mà ta muốn truy xuất thông thường ở dạng chuỗi (string/char), vì vậy ta cần tìm một hoặc nhiều cột trong kết quả truy vấn ban đầu có kiểu dữ liệu hoặc tương thích với dữ liệu chuỗi.

Cách làm là sau khi đã xác định số lượng cột cần thiết, ta có thể thăm dò từng cột để kiểm tra xem nó có thể chứa dữ liệu chuỗi hay không bằng cách gửi một loạt các payload UNION SELECT lần lượt đặt một giá trị chuỗi vào mỗi cột. Ví dụ: nếu truy vấn trả về bốn cột, bạn sẽ gửi:
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

## 5. Một số vấn đề đau đầu khác
Quên những gì liên quan tới UNION đi. Vẫn cần một thứ nữa mà ta cần phải tập trung đó là xác định truy vấn phía sau UINON ngoài số cột và kiểu dữ liệu, đó chính cần biết tên cột và tên bảng cho query này? Đây là một câu hỏi đau đầu cần thực hiện. Để giải quyết câu hỏi này tôi cũng chia sẻ một số kinh nghiệm như sau.



**Tham khảo**

[1] https://portswigger.net/web-security/sql-injection/union-attacks