---
layout: post
title: Sử dụng hex trong query MySQL,
date: 2021-05-08 00:32:20 +0700
description: Sử dụng hex trong query MySQL
img: 2021/05/08/210508_mysql.png
fig-caption: # Add figcaption (optional)
tags: [Security, WebSecurity]
---

Hôm nay  tôi vô tình đọc trên mạng được một query rất lạ. Thấy hay ho nên cũng đọc xem tại sao. Đại loại là

```
mysql> SELECT X'4D7953514C';
        -> 'MySQL'
mysql> SELECT 0x0a+0;
        -> 10
mysql> SELECT 0x5061756c;
        -> 'Paul'
```
Để hiểu thì cũng hơi dài dòng một chút.

### Hexadecimal Literals trong mysql

* Ký tự viết ở dạng hexa thường được sử dụng cú pháp 0x'val' hoặc X'val'. Trong đó val à tập các ký tự hexa {0-9,A-F}. 

>Chú ý 0x thì x case-sensitive (Tức x nhỏ thì khác X to).

Lấy ví dụ: toan9 chuyển sang hex sẽ là 0x746f616e39 hoặc X746f616e39

* Cú pháp hex viết theo ký hiệu 0x'val' hoặc X'val' thì val phải có độ dài là một số chẵn. Nghĩa là

![odd]( {{site.url}}/assets/img/2021/05/08/210508_odd.JPG){:width="400px"}

* MySQL coi hexa string là binary string

![binary]( {{site.url}}/assets/img/2021/05/08/210508_binary.JPG){:width="400px"}

* Đương nhiên MySQL cũng hỗ trợ convert sang hex

![convert]( {{site.url}}/assets/img/2021/05/08/210508_convert.JPG){:width="400px"}

### Hệ quả

Như vậy để hiểu tại sao

```
mysql> SELECT X'4D7953514C';
        -> 'MySQL'
```

Thì quá rõ rồi. 

Nhưng cái này thì hơi khó hiểu

```
mysql> SELECT 0x0a+0;
        -> 10
```
Bật mysql lên thử thì thấy thế này

![hex add]( {{site.url}}/assets/img/2021/05/08/210508_hex_add.JPG){:width="300px"}

Quay lại trên thì rõ ràng thấy 

```
0x0a+0 -> (0x)(0a) + 0 = 10 + 0 = 10
```

Chưa biết dùng khi nào nhưng hy vọng có ngày dùng tới,
