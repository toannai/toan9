
---
layout: post
title: Thử thách build lại kernel linux,,
date: 2019-06-02 00:32:20 +0700
description: # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Linux]
---

Đi làm tính ra 4, 5 năm rồi. Làm chỗ nào cũng có xài linux (Dĩ nhiên mức độ ít - nhiều khác nhau). Đã build kha khá, đủ loại phần mềm opensource rồi nhưng chưa bao giờ build kernel cả. Mà build làm éo gì nếu nhu cầu chưa thực sự cần =)). Nhớ hồi xin thực tập có ông pv hỏi mình đã build kernel linux bao giờ chưa =)) mọe hồi đó óe máo. Câu hỏi làm thắng bé vừa sợ vừa ngưỡng mộ vl, nhưng sv biết cái éo gì nên chắc chắn trả lời là "Chưa rồi" - Thật ra tới giờ vẫn chưa mà. OK không sao hôm nay build thử xem?

![Kernel buid1]( {{site.url}}/assets/img/2019/06/02/kernel1.png)

>Trước khi đọc bài này tôi nghĩ ae nên đọc bài linux boot process trước để có kiến thức linux nó load nhân ntn để còn biết cách cài nhân sau build xong. [Nội dung người lớn - Click để hiển thị ...](https://toannn.com/job/2019/05/31/Linux-boot-process.html)

### 0. Quên xừ mất, giới thiệu tí nhỉ?
Hệ điều hành linux - Quá quen thuộc rồi ae nhở? Nghe đồn sau khi Unix đang miễn phí quay ra đòi tiền license mấy bác dev của Unix cay quá rủ nhau ra làm OS free (Clone Unix) với dự án GNU - Phong trào Opensource - Giấy phép Copy Left. Nhưng cái dở phần core OS (phần quan trọng nhất) viết mãi không ổn định, không ngon :)

Cùng lúc bên Hà Lan có ông thạc sĩ tên ***Linus Benedict Torvalds*** viết được cái core ngon - cũng là luận văn ThS của ông luôn. Mấy ông GNU biết được, đem ghép vào OS của mình chạy mượt (quá hên) nên gộp vào dự án của mình luôn. Và để vinh danh ông thạc sĩ liền lấy tên OS là Linux - Tên ông thạc sĩ. 

Đó là câu chuyện của những năm 1991(Vừa xinh Linux vừa bằng tuổi mình). Tới giờ thì Kernel linux nói riêng và GNU nói chung được phát triển bởi rất nhiều các lập trình viên, cty trên t/g. Ai cũng có thẻ đóng góp cho nó bằng việc push code của mình lên repo của dự án theo hướng dẫn [Nội dung người lớn. Click để hiển thị ...](https://www.kernel.org/doc/html/v4.16/process/howto.html). bla, bla tới đây ae biết rồi.

Hết đoạn giới thiệu, Buổi nay mình thử build lại Linux Kernel nha :)




>PS: Tut dưới đây tôi làm trên môi trường là CentOS7 nha ae, các distro khác tương tự thôi.

### 1. Chuẩn bị tool, toy,

* **Cài đặt các gói cần thiết cho việc build**
```
[root@localhost ~]# yum groupinstall "Development Tools" -y
[root@localhost ~]# yum install ncurses-devel bison flex elfutils-libelf-devel openssl-devel bc -y
```

* **Tải về kernel - Đương nhiên rồi**

Trang chính cống của kernel linux [Kernel.org](https://www.kernel.org/). Tôi thích xài wget để download. 

```
[root@localhost kernel]# wget "https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.1.6.tar.xz"
```
* **Giải nén** 

Vì file tải về được nến nên đương nhiên phải giải nén/un-compress ra.

```
[root@localhost kernel]# unxz -v linux-5.1.6.tar.xz 
[root@localhost kernel]# tar xvf linux-5.1.6.tar.xz
```

>Ghê thẹc, source code gì mà hơn 900MB
![Kernel buid1]( {{site.url}}/assets/img/2019/06/02/source.PNG)


### 2. Cấu hình thông số/thuộc tính cho kernel 

Trước khi bắt đầu build kernel đầu tiên ae phải cấu hình thuộc tính cho kernerl. Tức là xác định các modules (driver) nào mà cần cho hệ thống của ae. Task này hơi căng nhỉ vì không rõ mình cần gì nào. Cấu hình này được đặt trong file .config ở thư mục đặt source luôn. 

Tip cho ae lần đầu làm chuyên đó là copy xử nó cái file cấu hình sẵn trên hệ thống vào để khỏi phải suy nghĩ (Cho newbie như mình).

Ae không quên cd vào thư mục chứa source vừa giải nén ra nha trước khi thực hiện nha

```
[root@localhost linux-5.1.6]# cd /root/kernel/linux-5.1.6
```
Và copy file có sẵn trên hệ thống đang chạy vào

```
[root@localhost linux-5.1.6]# cp -v /boot/config-3.10.0-957.el7.x86_64 .config
```

Lựa chọn  khác: ^^ Còn nếu ae vẫn thích thì bên linux nó cũng cung cấp cho ae công cụ để ae tự xử. Ae thử dùng cái menu config này nha. Gõ ```make menuconfig```. Bản chất là nó là công cụ để edit cái file .config để ta đỡ nhầm thôi. 

![Menu module]( {{site.url}}/assets/img/2019/06/02/module_menu.PNG)


### 3. Tiến hành compile

OK mọi thứ đã sắn sàng, ae compile thôi,

```
make 
```

>Chỗ này ae có thể thay đổi số core/thread compile bằng lệnh dưới để việc compile nhanh hơn.

```
make -j $(nproc) #proc là số thread sử dụng. Máy tôi có 4 core nên tôi hay chọn là 4.
```

Và quá trình build bắt đầu. Nhìn ghê vê lù.

![Build]( {{site.url}}/assets/img/2019/06/02/build.PNG)

>Detected: Đoạn build này lâu vc ý ae ạ. Tôi đợi mất cả 1 buổi chiều để build kernel với 1 máy ảo 4G RAM, 2x2CPU. Đâu đó mất 3h liền, càng đợi hun hút. Lời khuyên cho các ae muốn thử là phải chạy ở cái máy khủng khủng một tí.

### 4. Cài thêm module

Sau khi compile xong nhân, ae cần cài thêm các module cho nhân mới. Lệnh để cài thêm module (Đương nhiên là gõ trong thư mục mà ae vừa compile code) 

```
make modules_install
```

Lệnh này đơn giản làm công việc copy modules trong thư mục tương ứng với kiến trúc máy trong thư mục ```arch``` vừa build của nhân mới vào ```/lib/modules```

Không tin ae ls ra coi xem

![Module lib]( {{site.url}}/assets/img/2019/06/02/modules_lib.png)

> Từ hình ae thấy là nó link ra thư mục build lúc đầu nên lần sau ae build thì chú ý là nên copy code vào vào ```/usr/src``` rồi build. Tuyệt đối không được xóa thư mục build kể cả khi đã xong vì nó có link vào đây. 


### 5. Cài đặt nhân mới

Okey, đương nhiên cài xong thì ae phải thử cài xong xem nó có chạy không chứ :v. OK cài cái coi.

Ở bài [Linux boot process](https://toannn.com/job/2019/05/31/Linux-boot-process.html) tôi có nói là nhân được đặt trong thư mục /boot/ dưới dạng một file nén. Như vậy theo phỏng đoán khả năng là nhân mới build chắc cũng phải được copy vào đây. Và thông tin nhân mới cũng phải được cập nhật trong boot loader - Grub2 để khi boot máy còn biết mà load nhân mới vào. Vậy để kiểm chứng tôi sẽ kiểm tra thư mục /boot trước khi cài nhân mới.

![/boot]( {{site.url}}/assets/img/2019/06/02/kernel.PNG)

Kiểm tra cả file cấu hình của grub2 là ```/boot/grub2/grub.cfg```

![grub before]( {{site.url}}/assets/img/2019/06/02/grubmenu2.PNG)

#### Xong xuôi giờ tôi bắt đầu cài nhân mới nha 

* ***Cài nhân mới:***

```
make install
```

Veryfy lại nhận định ban đầu xem sau khi cài nhân mới có thay đổi gì. Đầu tiên tôi kiểm tra thư mục ``/boot/``

![/boot]( {{site.url}}/assets/img/2019/06/02/before.PNG)

Đúng như dự đoán ban đầu, so sánh với hình vừa chụp lại trên nhân mới được copy vào /boot, thêm các file ```/boot/vmlinuz-5.1.6``` ngoài ra có thêm ```/boot/initramfs-5.1.6.img```, ```/boot/System.map-5.1.6```

Kiểm tra lại file cầu hình của grub là ```/boot/grub2/grub.cfg``` thì tôi cũng thấy file này thay đổi, cụ thể có thêm đoạn cấu hình grub cho nhân mớis.

![/boot]( {{site.url}}/assets/img/2019/06/02/config.PNG)

* ***Cập nhật lại grub:***

Tôi thấy việc này có thể ko cần thiết vì ban đầu file cấu hình của grub2 mặc định đã là ```/boot/grub2/grub.cfg``` rùi. Nhưng ko sao làm theo guide nên tôi cứ gõ vào cho chắc tránh rủi do :)

```
grub2-mkconfig -o /boot/grub2/grub.cfg     #Set file cấu hình của grub là /boot/grub2/grub.cfg
grubby --set-default /boot/vmlinuz-5.1.2   #Set default của grub là nhân mới
```

* ***Veryfy lại cấu hình của grub:***

```
grubby --info=ALL | more  #List toàn bộ các nhân được cấu hình trong bootloader - grub2
grubby --default-index    #Kiểm tra index của default kernel (Nhân sẽ được load vào nếu người dùng ko chọn gì)
grubby --default-kernel   #Kiểm tra lại tên của default kernel.
```

### 6. Xem lại kết quả

Trước khi reboot lại tôi kiểm tra lại kernel hiện đang dùng là gì

![uname 1]( {{site.url}}/assets/img/2019/06/02/unamebefore.PNG)

Reboot lại máy ```reboot```

Lúc boot lên màn hình mới thấy có nhân mới rồi ae, haha,

![uname 1]( {{site.url}}/assets/img/2019/06/02/console.PNG)

Vào kiểm tra thì thấy nhân mới được load rồi, ngon quá ae.

![uname 1]( {{site.url}}/assets/img/2019/06/02/uname_after.PNG)


Vậy là kết thúc việc build lại nhân, nói chung không khó lắm chỉ là mất thời gian thui. Thế mà mấy ông pv ngày xưa làm mình đau tim vãi lúa.



### X.PS: Tổng hợp ngắn các bước để build một package opensource sử dụng công cụ cmake

![FBi]( {{site.url}}/assets/img/2019/06/02/fbi.jpg)

Bài khá dài, chắc ae nào đọc tới đây cũng choáng váng cmnr. Thật ra cả bài này tôi chỉ muốn ae nhớ đoạn này thôi: **Để buile một opensource (Áp dụng cho toàn bộ các opensource sử dụng cmake nha)** ta làm các bước sau:

1. Cài công cụ
2. Tải gói về
3. Build
* configure: Mục đích để để sinh ra Makefile - File này lấy thông tin thư viện trong máy build fill vào các macro (biến), chỉ định các thông số build cho phần mềm và quy định các kịch bản thực hiên các task phía sau (make, make install, make uninstall)
* make: Thực hiện kịch bản build trong Makefile
3. Cài đặt
* make instal: Thực hiện kịch bản cài đặt phần mềm trong Makefile

Have a good night ae,

***Tham khảo***

[https://www.cyberciti.biz](https://www.cyberciti.biz/tips/compiling-linux-kernel-26.html)
