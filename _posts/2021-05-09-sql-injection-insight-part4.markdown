---
layout: post
title: SQL Injection insight [Part 4] - Error base SQL Injection,
date: 2021-05-08 00:32:20 +0700
description: Phân loại SQL Injection và kinh nghiệm xử lý từng loại
img: 2021/05/09/210509_intro_sqlinjection.png
fig-caption: # Add figcaption (optional)
tags: [Security]
---

>Bài trước do nội dung quá dài nên tôi xin cắt nhỏ ra để mọi người tiện theo dõi.

## Ứng dụng sử dụng demo

Trước khi bắt đầu nội dung chính ngày hôm nay tôi xin mô tả chút về ứng dụng rất đơn giản sẽ được sử dụng để demo ở các phần sau. 

**Tên ứng dụng**: Tìm kiếm thông tin người dùng từ username.

**Mô tả:** ứng dụng của tôi sử dụng biến username nhận dữ liệu từ người dùng qua phương thức GET trong url. Sau đó tìm trong cơ sở dữ liệu mysql của mình để trả lại thông tin cho người dùng match với username mà người dùng vừa truyền vào.

[Link code github](https://github.com/toannai/sql_injection_tutorial/tree/master/tut2)

***URL:***
```
http://site1.local/tut2/index.php?username=admin
``` 
***SQL Query***
```
SELECT * FROM users WHERE username = 'admin'
```
***Result:*** 
```
Username: admin
Password: 123456A
Address: Hanoi
```
Okie, tới đây thì sẵn sàng vào nội dung chính của buổi hôm nay rồi.

## Error base SQL Injection

Error-based SQLi là một dạng của in-band SQL Injection. Kỹ thuật này dựa trên message trả về của database server để lấy thông tin về cấu trúc của cơ sở dữ liệu. Trong nhiều trường hợp thì chỉ cần error-based SQL Injection thôi cũng đủ để lấy toàn bộ thông tin database.

Nguyên nhân của lỗi này là do nhiều lập trình viên hay có thói quen hiển thị lỗi ngay trên ứng dụng để dễ troubleshoot. Điều này là tiện lợi trong quá trình phát triển nhưng vô hình chung lại tạo ra kẽ hở để bị lợi dụng tấn công.

Quay trở lại với ví dụ bên trên khi ta nhập vào ```admin'``` 

![visible]( {{site.url}}/assets/img/2021/05/09/210509_error_base1.JPG){:width="600px"}

Dễ thấy Input ```admin'``` gây ra lỗi - một ví dụ của error base SQL Injection, cụ thể là lỗi sai cú pháp *right syntax to use near ''admin**'**'' at line 1*. Thông báo lỗi này vừa là dấu hiệu giúp chúng ta nhận diện ra input đang mắc lỗi sql injection. Ngoài ra chúng cũng vô cùng hữu ích vì cung cấp cho chúng ta thêm thông tin để ta biết rằng query đang sai, cụ thể là ở phần ```''admin'''``` (có vẻ như đang thừa 1 quote) từ đó ta có thể điều chỉnh input sao cho phù hợp.

Tuy nhiên nếu thử với input là ``admin''`` thì không thấy báo lỗi mà cũng không có kết quả nào match với đầu vào.

![visible]( {{site.url}}/assets/img/2021/05/09/210509_errror_base2.JPG){:width="600px"}

Output **User admin" isn't exist in database** lý giải cho kết quả này, do username nhận vào sẽ là ``admin"`` nên tìm  trong db không thấy giá trị nào match cả . Kỳ lạ thật tại sao ``admin'`` thì không được mà ``admin"`` thì được. Tuy nhiên cũng không cần quan tâm đâu, chỉ xin nhớ rằng.

>Hãy chú ý đọc thông báo lỗi vì có thể chúng cung cấp cho ta nhiều thông tin hữu ích để ta điều chỉnh payload

>Trong payload nếu sử dụng sigle quote (') hoặc double quotes ("") mà gây ra lỗi thì hãy thử thêm vào thêm 1 quote nữa xem sao.

## Phân tích một số playload thú vị sử dụng

#### Yêu cầu
Tính năng của phần mềm là nhập username và lấy thông tin, nhưng tôi lại muốn lấy toàn bộ danh sách toàn bộ users một cách trái phép (Coi như hack vô hệ thống lấy users table luôn). Thực tế việc này thực hiện dễ dàng bằng sửa câu query thành.

```
SELECT * FROM users WHERE username = '' or 1=1
```

Do biểu thức logic của toán tử WHERE là or nên chỉ cần 1 vế 1=1 có giá trị TRUE thì toàn bộ điều kiện sau WHERE là true, coi như SELECT toàn bộ.

#### Thử 1 vài payload để giải quyết yêu cầu (Bypass ứng dụng sinh ra query trên)

* Pay load đầu tiên: Thử ``' or 1=1 --``. Kết quả trả lại lỗi.

![list all 0]( {{site.url}}/assets/img/2021/05/09/210509_list_all0.JPG){:width="600px"}

 Nhưng khi thử với ``' or 1=1 -- -`` thì không lỗi và lấy được all users.

![list all 1]( {{site.url}}/assets/img/2021/05/09/210509_list_all.JPG){:width="600px"}

Nguyên nhân được cho là dấu comment đứng cạnh quote ``--'`` là không hợp lệ. Để điều chỉnh cần thêm một dấu các phía sau ``--``. Tuy nhiên để an toàn tốt nhất nên sử dụng luôn ``-- -``.

* Một payload khác: Thử ``' or 1=1 #`` không lỗi và lấy được all users.

![List all 2]( {{site.url}}/assets/img/2021/05/09/210509_list_all1.JPG){:width="600px"}

>Qua đây có thể thấy rằng có thể dùng các ký tự comment môt cách khéo léo để bỏ qua phần sau của query - Phần qua rất khó đoán được nội dung developer làm gì.

## Payload tương đương 

Tôi lại nhập vào một số payload sau:

* Payload ``ad*\\*min`` và ``ad min`` thì thấy kết quả trả lại là thông tin của user admin.

![List all 2]( {{site.url}}/assets/img/2021/05/09/210509_comment1.JPG){:width="600px"}


![List all 2]( {{site.url}}/assets/img/2021/05/09/210509_comment1.JPG){:width="600px"}

Để lý giải tại sao tôi lại lấy ví dụ này, câu trả lời là nhiều trường hợp payload bị WAF (Web application firewall) chặn. Có chế của việc chặn lọc này dựa trên việc tìm kiếm các cụm từ nhạy cảm nhất định (blacklist). Để bypass ta cần chê dấu/làm mờ các từ hoặc cụm từ đó (obfuscate) trong payload để WAF không nhận ra. Soi vào ví dụ trên giả sử ``admin`` nằm trong blacklist thì tôi sẽ biến đổi thành ``ad*\ \*min`` hoặc ``ad  min`` rất có thể sẽ qua mặt được.

Và ngoài kỹ thuật sử dụng concat, comment tring để bypass WAF như trên ta còn có thể sử dụng các phương pháp subtring, hexString, ASII hoặc kết hợp nhiều phương pháp trong một payload. Anw bạn cũng có thể sáng tạo ra một cách của riêng bạn mục đích là để payload nó **rối mù lên** (dĩ nhiên payload vẫn phải thực hiện được ý đồ của người viết khi sinh ra SQL dưới ứng dụng) và WAF không thể nào nhận ra. Nhiều samples payload hay có thể xem ở tài liệu [2] phần tài liệu tham khảo cuối bài.

Hôm nay tôi xin dừng tại đây. Có nội dung nào cần trao đổi các bạn có thể để lại dưới comment hoặc inbox qua page fb cho tôi. Cảm ơn các bạn.

**Tham khảo**

[1] https://github.com/payloadbox/sql-injection-payload-list/blob/master/README.md

[2] https://portswigger.net/web-security/sql-injection/cheat-sheet

