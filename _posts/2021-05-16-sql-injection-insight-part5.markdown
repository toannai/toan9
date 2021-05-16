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