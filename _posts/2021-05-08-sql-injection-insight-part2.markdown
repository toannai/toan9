---
layout: post
title: SQL Injection insight [Part 2] - Template tư duy trong quá trình khai thác,
date: 2021-05-08 00:32:20 +0700
description: Khai thác lỗi sql injection và những template tư duy
img: 2021/05/08/210508_intro_sqlinjection.png
fig-caption: # Add figcaption (optional)
tags: [Security, WebSecurity]
---

Quay trở lại với các notes về SQL Injection. Quá trình tiến hành khai thác SQL Injection theo tôi thấy không phải là luôn luôn phải nghĩ ra những payload(1) mới toanh mà đôi khi chỉ cần áp dụng các template hoặc kinh nghiệm đã gặp trước đây là cũng có thể thành công rồi. Buổi hôm nay là chia sẻ về một số tư duy **có sẵn** đó.

Quay lại ví dụ ở bài trước, khi payload attack truyền vào ứng dụng sinh ra câu lệnh SQL sẽ có dạng:

![components]( {{site.url}}/assets/img/2021/05/08/210508_components.png){:width="600px"}

Việc phát hiện, exploit(2) bản chất là việc tìm kiếm các prefixes, payload, suffixes phù hợp để tác động lên ứng dụng để nó phản hồi theo mục đích của kẻ tấn công. 


### Balancing quotes - Cân đối việc sử dụng dấu nháy 

STRING là một trong những kiểu dữ liệu phổ biến và được dùng nhiều nhất trong SQL. Các chuỗi STRING thường được bao bởi Quotes (Cặp dấu nháy). Điều đặc biệt là SQL có thể cho lựa chọn nháy đơn ('..') hoặc nháy kép ("...") để bao STRING. Bao STRING nếu bắt đầu bằng nháy đơn thì phải đóng bằng nháy đơn, ngược lại nếu bắt đầu bằng nháy kép thì phải đóng bằng nháy kép. Chính vì vậy xác định nháy đơn hay kép là nội dung rất quan trong khi build payload có string.

Hãy so sánh 2 kết quả bên dưới đây:

* Chọn đúng quotes. Exploit thành công

![true quote]( {{site.url}}/assets/img/2021/05/08/210508_string_quote1.JPG){:width="600px"}

* Chọn sai quotes. Exploit thất bại

![false quote]( {{site.url}}/assets/img/2021/05/08/210508_string_quote2.JPG){:width="600px"}

Quote trong payload cần phải tương thích với quote mà tác giả ứng dụng (Người viết nên ứng dụng) đã sử dụng ở prefixes và suffixes.

Như vậy hãy chú ý xác định quotes khi build payload khai thác. Đơn giản là khi không dùng được sigle quotes - nháy đơn ('...') hãy suy nghĩ tới việc thử dùng double quotes - nháy kép ("...") xem sao, có khi lại có kết quả bất ngờ.

### Balancing columns numbers - Cân đối số cột sử dụng 

INSERT và UNION là 2 câu lệnh rất hay được sử dụng trong viết payload SQL Injection. Việc thành thạo 2 câu lệnh này như là một điều kiện bắt buộc khi muốn thành thạo khai thác SQL Injection.

```
INSERT INTO planets_tbl(name,planet,heads) VALUES('Zaphod','Betelgeuse',2);

SELECT id,username,password FROM user1_tbl WHERE username='Zaphod' UNION SELECT id2,username2,password2 from user2_tbl;
``` 

Một trong những yêu cầu để dùng được câu lệnh INSERT và UNION là xác định số cột của table mà ta INSERT vào hoặc số cột câu lệnh SELECT phía trước UNION. Để làm việc này ta cũng có một số template **tư duy**. Tuy nhiên nội dung khá dài và liên quan tới phần sau nên tôi sẽ phân tích chi tiết ở bài viết sau.

Điểm thứ 2 cần nhớ là phải chú ý tư duy về xác định số cột của truy vấn.

### Balacing Data type - Cân đối kiểu dữ liệu sử dụng

Quay lại với 2 câu lệnh INSERT và UNION, ngoài số lượng các cột cần nhận dạng kiểu dữ liệu kết hợp với cột để payload có thể thực thi được. Các kiểu dữ liệu có thể không match hoàn toàn 100% nhưng yêu cầu là phải tương thích hoặc có thể convert với kiểu dữ liệu mà tác giả ứng dụng sử dụng.

Anw, Hơi thốn tí, cách duy nhất để xác định kiểu dữ liệu trong khi viết payload là **đoán** và **thử** mà thôi.


Trên đây là 3 nội dung mà cần tập trung/chú ý vào ở các phần sau của loạt bài hướng dẫn vì phần sau trong các tình huống cụ thể chúng ta sẽ đi vào phân tích 3 nội dung này rắt nhiều, mục tiêu là để chúng ta hình thành tư duy template trong quá trình build payload. Buổi hôm nay chắc tôi cũng xin dừng tại đây. Bạn nào đọc qua có góp ý gì xin để lại dưới comment giúp mình. Cảm ơn các bạn.


>**Ghi chú**:
>* (1)Payload: Đoạn mã thực hiện để injection vào khai thác sql injection.
>* (1)Exploit: Khai thác.