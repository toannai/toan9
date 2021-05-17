---
layout: post
title: Tìm hiểu về lỗi Cross Site Scripting (XSS) trên ứng dụng web,
date: 2021-05-17 00:32:20 +0700
description: Tìm hiểu về lỗi Cross Site Scripting trên ứng dụng web,
img: 2021/05/17/210517_intro_xss.png
fig-caption: # Add figcaption (optional)
tags: [Security, WebSecurity]
---
Cũng như SQL Injection, XSS cũng được coi là một trong những lỗi phổ biến, có thâm niên lịch sử lâu đời "trong giới securiy" nên chắc là ai cũng nghe qua một lần rồi. Tuy vậy trong loạt bài về Websecurity dù ít hay nhiều cũng phải nhắc tới nó vì trên thực tế mức độ gặp phải nó ***thường xuyên*** chả kém gì SQL Injection là mấy. Ở công ty tôi có chuyên gia lỗi XSS, trong tháng rồi bạn ý cũng đã kiếm được vài ngàn $ dù chỉ focus vào lỗi này (XSS Guy). Nhưng thôi không dài dòng nữa chúng ta bắt đầu nội dung của buổi hôm nay thôi.

