---
layout: post
title: Hướng dẫn sử dụng lệnh top trong linux,
date: 2019-05-25 00:32:20 +0700
description: # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Linux]
---

Tôi hay có thói quen khi cài server thì dùng automation tool (saltstack/ansible) hardening một số cấu hình trên server (set password requirement, ssh key, disable ssh root, limit ip ssh, siết iptables) + cài luôn tmux, htop, systats luôn anh em ạ. Do có sẵn nên khi kiểm tra stats svr tôi hay dùng htop màu mè, trực quan, tiện lợi- Lâu dần thành quen cmnl. Sướng rồi khổ không chịu được nên là vào svr nào mà không có htop phải dùng top bực bội vô cùng – nhìn vào như mớ bòng bon :'( nhưng đời như c, ghét của nào trời trao của ấy, dạo này chuyển cty mới rất nhiều svr thường chỉ có sẵn top, và đặc biệt ko có internet nên cài htop lại phải tunnel các kiểu mệt vl, thế là thôi hôm nay rảnh rang tôi đọc man top và note lại đây vài ý vậy.

###  Hiểu các thông tin hiển thị của top,

Đầu tiên cùng nhau ngó qua xem top cung cấp cho ae những thông tin gì nào?
![top ]( {{site.url}}/assets/img/2019/05/25/top-1.png)

* Bắt đầu từ dòng 1, từ trái -> phải ta sẽ thấy current system time – uptime – user logon system – load average.

![top - info 1]( {{site.url}}/assets/img/2019/05/25/1.png)

* Dòng 2 liệt kê số lượng các task. Nói chung khá quen đọc cái hiểu liền, chỉ có khái niệm zombie là hơi mới 🙂 Zomebie process là process mà đã kết thúc (exit) nhưng entry vẫn chiếm trong bảng process table vì tiến trình cha sinh ra nó vẫn còn sống. Lý giải tại sao không “được” giải phóng mà vẫn phải giữ lại ae có thể đọc tại đây https://www.howtogeek.com/119815/htg-explains-what-is-a-zombie-process-on-linux/

![top - info 2]( {{site.url}}/assets/img/2019/05/25/2.png)

* Dòng 3 hiển thị tỉ lệ sử dụng cpu time. us – user space, sys – system space, ni – waste cpu for nice/renice.

![top - info 3]( {{site.url}}/assets/img/2019/05/25/3-1.png)

* Dòng 4 hiển thị tỉ lệ sử dụng bộ nhớ. Ae có thể tham khảo bài viết này của tôi [Nội dung người lớn - Click để hiển thị](https://toannn.com/notes/xac-dinh-luong-ram-free-tren-linux.html)

![top - info 4]( {{site.url}}/assets/img/2019/05/25/4-1.png)

* Dòng 5: Thông tin về tỉ lệ swap sử dụng.

![top - info 5]( {{site.url}}/assets/img/2019/05/25/4-2.png)

* Dòng 6: Thông tin chi tiết về tỉ lệ sử dụng tài nguyên của từng process. Ở đây có 1 số cột la lạ, còn lại khá clear rồi, dĩ nhiên tôi chỉ điểm qua 1 số cột là lạ thôi.

![top - info 6]( {{site.url}}/assets/img/2019/05/25/5.png)

PR và NI: NI – nice là độ ưu tiên của process. Nghe đâu giá trị này từ -20 tới 19. Càng thấp ưu tiên càng cao. Độ ưu tiên này được xài đến khi thiếu/hết MEM các process có độ ưu tiên thấp (nice lớn) sẽ bị queue xử lý sau (nếu chưa chạy) hoặc kill đi (nếu đang chạy mà MEM hết). Tương tự PR là scheduling priority là độ ưu tiên xử lý của cpu. Khi cpu hết các process có độ ưu tiên thấp sẽ bị queue xử lý sau process có độ ưu tiên cao.

VIR, RES, SHR, %MEM: Các trường này mô tả tỉ lệ sử dụng MEM của process. VIR (Virtal Memory): kích thước bộ nhớ ảo process sử dụng (Bao gồm programe’s code, data của process trong MEM và cả lượng bộ nhớ bị swap ra disk). RES (Resident): kích thước bộ nhớ vật lý (Physical memory) mà process sử dụng. %MEM là tỉ lệ sử dụng MEM của process. SHR là kích thước vùng nhớ chia sẻ mà process sử dụng.



### Các option top hay dùng,

*  Đầu tiên bật top lên. Nhấn h để xem các help của top.

![top - h]( {{site.url}}/assets/img/2019/05/25/top_h.png)

Xem nào cũng kha khá tùy chọn đó chứ. Nhưng thôi, chắc tôi sẽ chỉ tập trung vào 1 số tùy chọn chính hay dùng thôi ae ạ. 

* Sau khi thoát ra khỏi help bằng q. Thử nhấn f dùng mũi tên lên/xuống để di chuyển, nhấn tab để chọn thêm/bớt trường hiển thị. Ví dụ chọn thêm trường swap. Di chuyển xống swap và nhấn tab.

![top - swap]( {{site.url}}/assets/img/2019/05/25/sw.png)

Cũng trong cửa số đó nếu con trỏ màu trắng đang ở trường đó ae có thể nhấn s để sort theo trường.

![top - s]( {{site.url}}/assets/img/2019/05/25/soft_base.png)

Lúc này kq xuất hiện thêm trường swap và được sort rất tử tế.

![top - sort]( {{site.url}}/assets/img/2019/05/25/topsort.png)

*  Trong lệnh top nhấn c để show ra command của từng process

![top - sort]( {{site.url}}/assets/img/2019/05/25/command.png)

Lặp lại nhấn c sẽ ẩn command 😉

>Quy ước các option sau này cũng thế nhấn lần 1 thì hiển thị thông tin option, nhấn lại lần nữa thì ẩn nha ae.

*   Shift + M để sort theo Memory & Shift + P để sort theo CPU
*   Nhấn d để thay đổi interval time
*   Nhấn z để đổi màu processs
*   Đặc biệt nếu chạy top với quyền root có thể nhấn k rồi nhập pid để kill process. Nhấn r để renice
*   Nhấn 1 để hiện thông tin về từng core


Top có rất nhiều tùy chọn. Ở đây tôi chỉ note một số tùy chọn mà tôi hay dùng. Rất mong có gì đó hay ho cho ae nhất là các ae mới vào nghề vậy.

Cảm ơn ae,
