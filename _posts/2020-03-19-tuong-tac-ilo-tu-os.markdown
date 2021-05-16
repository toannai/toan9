---
layout: post
title: Tương tác với iLo từ OS,
date: 2020-03-19 00:32:20 +0700
description: Tương tác với iLo từ os # Add post description (optional)
img: 2020/03/19/200319_ilo.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Linux]
---

Lâu lâu không viết bài gì về job cả, toàn nói đạo lý, tâm sự thầm kín hoặc là cà khịa mà thôi. Bữa nay dở tay quay lại viết lách chút về công việc. 


Lúc chiều có con server khách hàng bị ngáo. OS vào được nhưng ILO không thể naò vào được. Anh cùng team bảo thử từ ÓS chọc lên tương tác ILO kiểm tra xem có vấn đề gì không
. Sau hồi search google quả đúng là có thể làm như thế được được. Hay thiệt mà mấy lâu nay đâu có biết vậy nay note về cái này.


### Nguyên lý,

Việc từ OS có thể chọc ngược điều khiển ILO về cơ bản là do firmware và Kernel có expose ra một interface giúp application có thể tương tác trực tiếp với iL0. Google thì ra cái chuẩn này [IPMI - Wiki](https://en.wikipedia.org/wiki/Intelligent_Platform_Management_Interface). 

OK nghĩa là từ OS hoàn toàn có thể tương tác vớ ILO dễ dàng. iLO là đối với server HP tuy nhiên đã là chuẩn thì khả năng 99,9% các hãng Dell, CISCO cũng support việc này trên các console server của họ.


### Từ OS Ubuntu chọc vào tương tác với ILO

Trong bài viết này tôi sẽ demo việc từ OS là Ubuntu tương tác với ILO qua thư viện OpenIPMI.

Đầu tiên ta cài các gói cần thiết

```
[root@server ~]# apt-get install -y openipmi
[root@server ~]# apt-get install -y ipmitool
```

Sau đó cần enable module của nhân hỗ trợ IPMI

```
[root@server ~]# modprobe ipmi_devintf
[root@server ~]# modprobe ipmi_si
```

Và từ giờ ta có thể làm kha khá nhiều việc hay ho từ đây. Ví dụ như

#### 1. Xem lại cấu hinhf network ILO

```
[root@server ~]# ipmitool lan print
```

#### 2. Restart network ILO

```
[root@server ~]# ipmitool mc reset warm
```

#### 3. Thay đổi cấu hình netowrk ILO

```
[root@server ~]# ipmitool lan set 1 ipaddr 192.168.1.211
[root@server ~]# ipmitool lan set 1 netmask 255.255.255.0
[root@server ~]# ipmitool lan set 1 defgw ipaddr 192.168.1.1
```

#### 4. Tạo user ILO

```
[root@server ~]# ipmitool user set name 2 admin
[root@server ~]# ipmitool user set password 2
Password for user 2:
Password for user 2:
[root@server ~]# ipmitool channel setaccess 1 2 link=on ipmi=on callin=on privilege=4
[root@server ~]# ipmitool user enable 2
```

#### 5. Anw, Và tà hoàn toàn có thể làm một số trò hay ho như dùng bash shell hoặc python để làm gì đó với ILO nếu ta muốn.


>Tham khảo [https://dev-random.net](https://dev-random.net/configuring-hp-ilo-through-linux-automatically/)