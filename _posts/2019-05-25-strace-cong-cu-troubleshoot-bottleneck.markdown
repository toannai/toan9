---
layout: post
title: Strace - tool hay để khoanh vùng troubleshoot bottleneck,
date: 2019-05-25 00:32:20 +0700
description: Strace - tool hay để khoanh vùng troubleshoot bottleneck # Add post description (optional)
img: 2019/05/25/190509_strace_intro.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Linux]
---

Chiều nay có 1 case khá thú vị, một lệnh apt-get dùng qua proxy thời gian đầu lúc khởi chạy wait rất lâu (2 – 3 phút), dấu hiệu thực tế cho thấy nguyên nhân không phải do mạng vì ngay sau 2 – 3 phút ban đầu đó việc down gói, un-deb, cài gói diễn ra rất nhanh. Thật khó lý giải tại sao. Lát sau nhờ tool strace giúp khoanh vùng được vấn đề . Tối về rảnh dang ngó qua thấy kha khá điều hay ho về cái tool này và note tại đây.

Nguyên văn của man page “strace – trace system calls and signals” – Đại ý là nó sẽ giúp ta hiển thị ra theo thứ tự chạy của trương trình toàn bộ các system call mà nó gọi (system call – tạm gọi là các lời gọi hệ thống mà chương trình sử dụng để giao tiếp với core os VD: tạo file, mở/đóng file, ..). 

###  Thử với dòng lệnh đầu tiên

![starce first]( {{site.url}}/assets/img/2019/05/25/strace1.png)

Nhìn cũng thấy kích thích đấy nhưng nếu ae nào chưa từng lâp trình hệ thống cũng khó hiểu phết 🙂 run dế quá.

### Thêm chút mắm với option –r

![starce first]( {{site.url}}/assets/img/2019/05/25/strace1.png)

Thấy có thêm cột thời gian dành cho từng system call. Ngon quá từ đây sẽ biết được bị “dắt” – bottleneck ở system call nào – giúp khoanh vùng khá hiệu quả trong dự đoán nguyên nhân.

### Thêm chút muối với option -c 

![starce first]( {{site.url}}/assets/img/2019/05/25/strace3.png)

Tới đây thì đúng là dành cho các ae ăn xổi rồi. Như VD phần đa thời gian dành cho system call nmap – Google chút [Nội dung người lớn - Click để hiển thị](http://man7.org/linux/man-pages//man2/munmap.2.html) => Phần lớn thời gian lệnh trên là dành cho map/unmap file vào bộ nhớ, buồn cười nhỉ có vẻ hóa ra phần lón t/g tạo file là tốn cho tính toán nên đặt nó ở đâu trên disk chứ chứ không phải là cứ một phát là biết đặt luôn vào đâu.

>Ngoài việc truyền vảo arg là execute job có thể truyền vào pid (Cái này hay ho đây) nhưng case study dài quá ae tự đọc thôi =)). [
Nội dung người lớn - Click để hiển thị](https://blog.tanelpoder.com/2013/02/21/peeking-into-linux-kernel-land-using-proc-filesystem-for-quickndirty-troubleshooting/)

### Thêm chút mắm mắm, muối muối nữa chắc ae phải đọc man page nha,


Quay lại case lúc chiều, dùng strace cứ đến 1 đoạn có 1 system call lúc gọi rất tốn t/g nhưng 2 ae không hiểu nó làm gì vì vốn không phải là dân lập trình hệ thống, lát sau ông anh cùng team nhận ra có 1 đoạn gọi ra net phân giải dns cứ bị loop lặp đi lặp lại (retry) anh ý đánh giá có thể do retry dns. Thử comment dns phát thì ăn ngay (Problem solved). Mà Csh dùng proxy rồi mà mấy tool đó còn phân giải dns làm gì nữa (dị).

Nói chung nghe strace cũng lâu mà nay mới để ý, đúng là amz tool cho khoanh vùng troubleshoot bottleneck.

 