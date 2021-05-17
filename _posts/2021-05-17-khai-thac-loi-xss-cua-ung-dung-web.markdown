---
layout: post
title: Cross Site Scripting (XSS) [Part P1] - Giới thiệu về lỗi XSS,
date: 2021-05-17 00:32:20 +0700
description: Tìm hiểu về lỗi Cross Site Scripting trên ứng dụng web - P1,
img: 2021/05/17/210517_intro_xss.png
fig-caption: # Add figcaption (optional)
tags: [Security, WebSecurity]
---
Cũng như SQL Injection, XSS cũng được coi là một trong những lỗi phổ biến, có thâm niên lịch sử lâu đời "trong giới securiy" nên chắc là ai cũng nghe qua một lần rồi. Tuy vậy trong loạt bài về Websecurity dù ít hay nhiều cũng phải nhắc tới nó vì trên thực tế mức độ gặp phải nó ***thường xuyên*** chả kém gì SQL Injection là mấy. Ở công ty tôi có chuyên gia lỗi XSS, trong tháng rồi bạn ý cũng đã kiếm được vài ngàn $ dù chỉ focus vào lỗi này (XSS Guy). Nhưng thôi không dài dòng nữa chúng ta bắt đầu nội dung của buổi hôm nay thôi.

## Khởi động,

Vì lỗi này được khai thác ở client site trên nền Javascript nên trước khi bắt đầu tôi nghĩ chúng ta nên điểm lại chút kiến thức về Javascript.

Đầu tiên cần phải hiểu rằng Javascript không phải là Java (Cái này ai cũng biết rồi). Và nó là một  ngôn ngữ thuần hướng đối tượng (Nghĩa là cái gì trong Javascript cũng là đối tượng).

#### Khai báo biến và kiểu dữ liệu

#### Cấu trúc điều khiển

#### Cấu trúc lặp

#### DOM
* Document Object Model (DOM) - Mô hình tài liệu đối tượng. Là chuẩn cho phép Javascript tự động truy cập update cấu trúc, nội dung hoặc style của page. 

Theo DOM cấu trúc của tài liệu (Document) được tổ chức dạng cây như sau:

![DOM]( {{site.url}}/assets/img/2021/05/17/210517_dom.png)

Dễ thấy Document được coi như một cây. HTML tag là root. Root có 2 con là HEAD và BODY. Mỗi Item là một child.

* Vừa nói Javascript là ngôn ngữ hướng đối tượng. Mỗi đối tượng có thuộc tính và phương thức. Truy cập thuộc tính theo cú pháp **object.property** và gọi thương thức theo cú pháp **object.method()** 

Ví dụ: đối tượng document có thuộc tính referrer tham chiếu bằng document.referrer. Có method là alert() khi gọi sẽ là document.alert("Demo alert").

Vài method hay gặp

![Method]( {{site.url}}/assets/img/2021/05/17/210517_method.JPG)

* Vài snippet code DOM huyền thoại hay dùng như sau:


