---
layout: post
title: Kỹ thuật tấn công Poison Null Byte(aka Embedding Null Code),
date: 2021-05-10 00:32:20 +0700
description: Kỹ thuật tấn công Poison Null Byte/Embedding Null Code
img: 2021/05/10/210510-poison-null-byte.jpg
fig-caption: # Add figcaption (optional)
tags: [Security]
---
Nhân tiện làm một challenge trên tryhackme liên quan tới Poison NULL byte tôi cũng tìm hiểu và ghi chép chút ít về kỹ thuật này.

NULL byte trong nhiều  ngôn ngữ được sử dụng để detect việc kết thúc của một chuỗi string. Kỹ thuật tấn công Poison NULL Byte đã lợi dụng việc này để inject các ký tự NULL Byte vào ứng dụng, làm cho ứng dụng không xử lý postfix NULL terminators như thông thường, việc xử lý bị thay đổi theo ý đồ của kẻ tấn công.

Kỹ thuật này rất hay được sử dụng để bypass filter file extension hoặc để tấn công directory traversal.

Một số biến thể hay gặp:

```
PATH%00
PATH[0x00]
PATH[alternate representation of NULL character]
<script></script>%00
```

Để dễ hiểu ta lấy ví dụ sau:

Giả sử ta có đoạn code php như sau:
```
$whatever = addslashes($_REQUEST['whatever']);
include("/path/to/program/" . $whatever . "/header.htm");
```

Bằng việc thao tác URL sử dụng postfix NULL bytes, ta dễ dàng truy cập UNIX password file và bỏ qua đoạn nối chuỗi **. "/header.htm"** ở phía sau bằng việc sử dụng URL: 
```
http://vuln.example.com/phpscript.php?whatever=../../../etc/passwd%00
```
Lý do là vì khi trình thông dịch PHP đọc tới %00 thì nó thấy đây là ký tự kết thúc nên dừng tại đây và bỏ qua phần sau. Hệ quả là dưới ứng dụng sẽ thi hành đoạn mã sau:
```
$whatever = addslashes($_REQUEST['whatever']);
include("/path/to/program/" . "../../../etc/passwd")
```
Rõ ràng ứng dụng đã không hoạt động như thiết kế ban đầu mà bị lái theo ý đồ của kẻ khai thác.

***Tham khảo***

https://owasp.org/www-community/attacks/Embedding_Null_Code

https://defendtheweb.net/article/common-php-attacks-poison-null-byte
