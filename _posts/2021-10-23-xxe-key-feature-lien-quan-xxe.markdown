---
layout: post
title: Key features của XML entities liên quan tới XXE,
date: 2021-10-23 00:32:20 +0700
description: Key features của XML entities liên quan tới XXE
img: 2021/10/23/xxe_keyfeatures.png
fig-caption: # Add figcaption (optional)
tags: [Security, WebSecurity]
---

Trong notes này, chúng ta sẽ đề cập tới một số key features của XML có liên quan đến lỗ hổng XXE.

### XML là gì?

XML là viết tắt của "extensible markup language". XML là một ngôn ngữ được thiết kế để lưu trữ và truyền dữ liệu. Giống như HTML, XML sử dụng cấu trúc dạng cây gồm các tags và data. Không giống như HTML, XML không sử dụng các tags được định nghĩa trước, chính vì vậy các thẻ có thể được đặt tên để mô tả dữ liệu (Để chỉ nhìn tag là biết nó dùng để define cho cái gì). Trước đây XML khá thịnh hành như một định dạng truyền tải dữ liệu ("X" trong "AJAX" là viết tắt của "XML"). Nhưng sự phổ biến của nó hiện đã giảm hẳn và đang dần bị thay thế bởi JSON.

### XML entities là gì?

XML entities là một cách thể hiện một item của data trong tài liệu XML, thay vì sử dụng chính dữ liệu đó. Các entities khác nhau được xây dựng trong đặc tả của ngôn ngữ XML. Ví dụ: các entities &lt; và &gt; đại diện cho các ký tự **<** và **>**. Đây là các metacharacters được sử dụng để biểu thị các thẻ XML và do đó thường phải được biểu diễn bằng cách sử dụng các entities của chúng khi chúng xuất hiện trong dữ liệu.

### Document type definition là gì?

XML document type definition (DTD) chứa các khai báo có thể định nghĩa cấu trúc của tài liệu XML, các kiểu dữ liệu mà nó có thể chứa và các items khác. DTD được khai báo trong phần tử DOCTYPE và có thể tùy chọn định nghĩa ở đầu tài liệu XML. DTD có thể hoàn toàn độc lập trong chính tài liệu (được gọi là "internal DTD") hoặc có thể được load từ nơi khác (được gọi là "external DTD") hoặc có thể kết hợp cả hai.

### Các XML custom entities là gì?

XML cho phép các custom entities được định nghĩa trong DTD. Ví dụ:

```<! DOCTYPE foo [<! ENTITY myentity "my entity value">]>```

Định nghĩa này có nghĩa là mọi cách sử dụng tham chiếu thực thể & myentity; trong tài liệu XML sẽ được thay thế bằng giá trị đã xác định: "my entity value".

### Các XML external entities là gì?

Các XML external entities là một loại thực thể tùy chỉnh có định nghĩa nằm bên ngoài DTD nơi chúng được khai báo.

Khai báo của một XML external entities sử dụng từ khóa SYSTEM và phải chỉ định một URL mà từ đó giá trị của thực thể sẽ được load. Ví dụ:

```<! DOCTYPE foo [<! ENTITY ext SYSTEM "http://normal-website.com">]>```

URL có thể sử dụng file:// protocol, do đó XML external entities có thể được load file. Ví dụ:

```<! DOCTYPE foo [<! ENTITY ext SYSTEM "file:///path/to/file">]>```

Các XML external entities cung cấp các thức để thực hiện các cuộc tấn công XXE(XML external entity attacks). 