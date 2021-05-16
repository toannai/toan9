---
layout: post
title: Software Engineer si tình, còn Tôi đã đục lỗ file .deb như thế nào,
date: 2020-06-07 00:32:20 +0700
description: # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Linux]
---
Tiêu đề vừa sến và nghe **"tít"** như của mấy anh hacker nhỉ. Nhưng không, tôi là sysadmin thôi, nay nhân tiện đọc để viết bài tôi kiếm được chuyện khá hay ho muốn kể cho mọi người nghe luôn nên đặt title **kêu** như vậy.

>Description: Hàng ngày làm việc với file .deb nhiều nhưng hẳn là không nhiều người để ý trong file deb nó có cái gì vậy? OK, không sao nay ta thử **đục lỗ** một file deb xem nó có gì? Hy vọng nó có thông tin gì đó hay ho giúp ích cho cv hàng ngày.


## 1. .deb file - Câu chuyện của một Software Engineer "Si tình",

![Intro debian deb]( {{site.url}}/assets/img/2019/06/02/debianlove.png)

[Ian Ashley Murdock](https://en.wikipedia.org/wiki/Ian_Murdock) (1973-2015) một founder của project Debian (Một distro cúa Linux). Ngay từ hồi sinh viên (khoảng năm 1993-1994) đã có si mê một em gái vô cùng. Bằng chứng là do quá "Si mê"  em gái đóa và sau này cũng là vợ ông, ông đặt cho tên của dự án distro linux của mình **Debian** là từ ghép của tên 2 người  **Deb**ra Lynn + **Ian** Ashley Murdock. Không dừng mức độ "Si tình" ở đóa **.deb** - Một chuẩn đóng gói nhị phân trên  debian cũng được đặt dựa trên chữ cái đầu của chính cô bạn gái đóa **Deb**ra ... (không biết trong debian còn cái gì mang tên gái nữa không?) ... Chẹp đúng là lúc yêu nhau nhìn đâu cũng thấy nhau.

Nhưng chuyên tình hồi đầu lãng mạn nhưng càng về sau thì càng lãng xẹt. Sau vài năm yêu nhau 2 người đã cưới nhau, có với nhau 3 đứa nhóc. Nhưng thật chớ chêu tưởng là lên đỉnh của hạnh phúc rùi :) họ lại ly thân vào khoảng năm 2008 (Lý do được giải thích: Chả hiểu tại sao).

>Chuyện đùa mà có thật mà như đùa nhỉ?, vui chút thôi mọi người đừng ném gạch tội cho tôi.

Các pakage .deb được sử dụng chủ yếu trên các distro của dòng Debian (Debian, Ubuntu, ..). Làm việc với các gói debian người ta hay sử dụng công cụ dpkg (command line) hoặc synaptic (GUI) - Hai cái này chắc quá quen thuộc rồi tôi không cần giới thiệu nữa. À còn cái này nữa thú vị chắc nhiều người ko để ý. Chuẩn deb này hay ở chỗ có thể sử dụng công cụ alien để conver sang các format khác (bao gồm cả rpm) - Nhưng cũng giới hạn thui không ngon 100% được.

He, phần giới thiệu kết thúc nha.

## 2. Cùng nhau "đục lỗ" file .deb

Theo Google tôi được biết File deb bản chất là một file nén theo chuẩn nén ar của Unix. Ok là file nén thì tôi thử giải nén xem nó có cái gì? 

Để cho quen, lấy ví dụ là toy telnet đem ra mổ đi: [telnet_0.17-41.2_amd64.deb](https://debian.pkgs.org/10/debian-main-amd64/telnet_0.17-41.2_amd64.deb.html)

Không quá khó để google ra cách giải nén

![deb1]( {{site.url}}/assets/img/2019/06/02/deb1.PNG)

Quá trình giải nén tạo ra ba file: **control.tar.xz, data.tar.xz, debian-binary**. Giờ tôi sẽ khám phá từng file một.

### - debian-binary

Đây là một file text. Theo mạng thì nội dung file này là thông tin version của định dạng file. Ở đây tôi thấy ghi là 2.0. (Thấy ghi tới 2019 thì format deb này mới có version 2.0 thui.)


![deb2]( {{site.url}}/assets/img/2019/06/02/deb2.PNG)

### - data.tar.xz

Do là 1 file nén, thật dễ để giải nén nó ra coi có chi. 

![telnet]( {{site.url}}/assets/img/2019/06/02/telnet.png)

Ở đây tôi thấy 1 thư mục y cấu trúc y hệt như cấu trúc của 1 số folder trong file system. ```/usr/bin/...```
Phải chăng bản chất của việc cài gói phần mềm deb là việc dpkg giải nén rồi copy các file trong cấu trúc này vào các vị trí tương tự trên file system của ta khi cài đặt chăng? để kiểm chứng lại nhận định tôi cài thử telnet vào máy ubuntu đang dùng và kiểm tra lại thì thấy đúng thế thật :) Á đù mình giỏi quá ahehe,

![telne2]( {{site.url}}/assets/img/2019/06/02/telnet2.PNG)


Tới đây tạm kết luận rằng nội dung file data.tar.xz chính là nội dung binary, docs, ... thứ mà thực sự là nội dung gói phần mềm ta sẽ cài vào hệ thống bố trí theo cấu trúc nó sẽ được copy vào - Anw, Diễn đạt hơi khó hiểu nhưng vẫn hy vọng mọi người hiểu ý tôi :)


### - control.tar.xz

Trời dài quá file cuối cùng rồi, cũng phải thử mổ nó ra xem coi nó là gì nào? Mọi người đừng nản nha :)

![deb3]( {{site.url}}/assets/img/2019/06/02/deb3.PNG)

Nội dung trong đây lại là vài file nhỏ. Tò mò giờ tôi tiếp tục xem cá file nhỏ là gì?

* control: Có vẻ như File này chứa thông tin về gói. Quá dễ để hiểu nội dung của thông tin trong nó :)

```
Package: telnet
Source: netkit-telnet
Version: 0.17-41.2
Architecture: amd64
Maintainer: Mats Erik Andersson <mats.andersson@gisladisker.se>
Installed-Size: 163
Depends: netbase, libc6 (>= 2.15), libgcc1 (>= 1:3.0), libstdc++6 (>= 5)
Replaces: netstd
Provides: telnet-client
Section: net
Priority: standard
Description: basic telnet client
 The telnet command is used for interactive communication with another host
 using the TELNET protocol.
 .
 For the purpose of remote login, the present client executable should be
 depreciated in favour of an ssh-client, or in some cases with variants like
 telnet-ssl or Kerberized TELNET clients.  The most important reason is that
 this implementation exchanges user name and password in clear text.
 .
 On the other hand, the present program does satisfy common use cases of
 network diagnostics, like protocol testing of SMTP services, so it can
 become handy enough.
```
>Đùa móa, ra đây là cái file chết tiệt control là nguyên nhân bắt ta phải cài thêm các gói denpends các kiểu đây, ghét!

* md5sums: HÓa cái file nè chứa hash của toàn bộ file trong **data.tar.xz**

```
root@svr:/home/toanvnu/deb/control# cat md5sums 
f647f104f97573583daf301e29df827d  usr/bin/telnet.netkit
8d1846a637fc6c5d73c9b9b91a3e63ab  usr/share/doc/telnet/README
a8aa951b88ca4f4c4835d88865fb4ee8  usr/share/doc/telnet/README.old.gz
58c8e990fc09e14511db5aab2f6f4164  usr/share/doc/telnet/changelog.Debian.gz
40bf56de99b3770b9186dd30c03361e8  usr/share/doc/telnet/changelog.gz
fb622cfdfebf11742ba07c5f8a4c0e48  usr/share/doc/telnet/copyright
911004a24f40df624643ac0679c9f669  usr/share/lintian/overrides/telnet
a40db5ee63525645b3187f10447dac0d  usr/share/man/man1/telnet.netkit.1.gz
c4c7b74fe26fab1b2dfce7fe61758d97  usr/share/menu/telnet
```
Tui đoán, mà chắc cũng là đúng luôn thui :) file này chắc để đảm bảo tính toàn vẹn của các file data.tar.xz.

* Ba file còn lại **postinst, postrm, prerm** là các file script bash thông thường thôi. Ngoan quá, càng dễ vì đúng là sở trường của sysadmin. Ba file này đọc qua qua trên mạng thì thấy đại ý nội dung như sau:

	+ postinst -> File được chạy su khi copy dữ liệu nén trong phần data.tar.xz thực sự vào hệ thống.
	
	![deb4]( {{site.url}}/assets/img/2019/06/02/deb4.PNG)

	+ prerm -> Script được chạy trước khi remove dữ liệu thực sự đã cài trong gói khỏi hệ thống khi ta thực hiện hành động gỡ bỏ gói (apt-get remove ...).
	 
	![deb5]( {{site.url}}/assets/img/2019/06/02/deb5.PNG)

	+ postrm -> Script được chạy sau khi remove data thực sự khỏi hệ thống khi ta thực hiện hành động gỡ bỏ gói (apt-get remove).

	![deb6]( {{site.url}}/assets/img/2019/06/02/deb6.PNG)

>Vậy có preinst không? có chứ sao lại không? chỉ là package này không có thui. Ý nghĩa của nó tương tự.

Đọc qua *thêm* thì thấy trong 1 file deb theo chuẩn thì nên có 4 file này, tuy nhiên là option ạ (Nghĩa là có một, một vài hoặc toàn bộ có/không) không bắt buộc đó, tùy vào ta muốn thao tác ở pharse nào thui.

## 3. Thử tạo 1 file deb xem sao?

Đoạn trên là đoạn tôi hướng dẫn mọi người xem hàng của người ta, giờ tôi hướng dẫn mọi người tự làm hàng - Tự tay pack một file .deb của mình.

*Đề bài*: Tôi muốn đóng 1 file .deb mà khi cài nó sẽ bung ra 1 file là toannn vào vị trí ```/usr/local/sbin```? Okie, Let's go.

**Bước 1.** Tạo nội dung data cài đặt. Chính là nội dung trong **data.tar.xz** mà ta vừa nói ban đầu

Đầu tiên tôi tạo 1 thư mục là ```toannn``` đi.

```
root@svr:/home/toanvnu/deb# mkdir toannn
```
Do theo đề bài "Tôi muốn đóng 1 file .deb mà khi cài nó sẽ bung ra 1 file là ```toannn``` vào vị trí ```/usr/local/sbin``` do vậy trong thư mục hiện thời tôi sẽ tạo cấu trúc thư mục tương tự, match với location file system muốn cài vào ```usr/local/sbin/toannn```

```
root@svr:/home/toanvnu/deb# mkdir -p toannn/usr/local/sbin/
root@svr:/home/toanvnu/deb# touch  toannn/usr/local/sbin/toannn
root@svr:/home/toanvnu/deb# echo "Hello world" > toannn/usr/local/sbin/toannn 
root@svr:/home/toanvnu/deb# chmod 755 toannn/usr/local/sbin/toannn 
```

**Bước 2.** Tạo file control, postinst, postinst là các nội dung cần cho **control.tar.gz**. Trong đó file control là bắt buộc. Hai file postinst, postinst  là optional (Vừa nói ở trên đó).

Tool đóng gói tôi dùng nó bắt đặt các file này trong thư mục DEBIAN là thư con của thư mục gốc ```toannn``` (chs) nhưng họ bắt thì mình phải làm thôi.

```
root@svr:/home/toanvnu/deb# mkdir toannn/DEBIAN
root@svr:/home/toanvnu/deb# touch toannn/DEBIAN/control
root@svr:/home/toanvnu/deb# touch toannn/DEBIAN/postinst
root@svr:/home/toanvnu/deb# chmod 755 toannn/DEBIAN/postinst 
root@svr:/home/toanvnu/deb# touch toannn/DEBIAN/prerm
root@svr:/home/toanvnu/deb# chmod 755 toannn/DEBIAN/prerm
```
>Vì hai file **postinst** và **prerm** là 2 file thi hành nên họ yêu cầu phải change mode thành 755. Mọi người chú ý nha.

Nhìn lại cấu trúc thư mục vừa tạo ra.

![tree]( {{site.url}}/assets/img/2019/06/02/tree.PNG)

Nội dung 3 file như sau

File control
```
Package: toannn.com 
Version: 1.1-1
Section: base
Priority: optional
Architecture: all
Depends:  awk
Maintainer: toannn <toanvnu@gmail.com>
Description: Linux system information
 This is a simple deb file 
```
File postinst: Mục tiêu là trước khi bung ra nội dung package vào thư mục t/ư trên hệ thống sẽ tạo ra file ```/tmp/postinst```
```
touch /tmp/postinst
```
File prerm: Mục tiêu là sau khi remove package khỏi hệ thống sẽ tạo ra file ```/tmp/prerm```
```
touch /tmp/prerm
```

**Bước 3.** Tiến hành tạo file deb

Gõ lệnh bên dưới tại thư mục cha của thư mục toannn

```
root@svr:/home/toanvnu/deb# dpkg-deb --build toannn/
dpkg-deb: building package 'toannn.com' in 'toannn.deb'.
root@svr:/home/toanvnu/deb# 
```
Hay hay, có vẻ đã tạo ra file deb thành công.

**Bước 4.**  Ktra lại
Sau khi tạo ra file deb, tôi thêm bước kiểm chứng xem file của ta hoạt động có như mong muốn không?

* Cài 


![install]( {{site.url}}/assets/img/2019/06/02/install.png)

* Remove

![remove]( {{site.url}}/assets/img/2019/06/02/remove.png)

Đóa đúng như mong đợi. Ok có vẻ không quá phức tạp nhỉ,

Vài viết hơi dài. Ban đầu cũng không muốn viết dài ntn đâu, đọc đau đầu lắm nhưng khổ cái tính tôi cứ thích làm ra ngõ - ngách mới yên ạ. Mong mọi người thông cảm.

Chúc tất cả mọi người có một chuyện tình đẹp và có cái happy ending nha, đừng như bác **Ian** nha, thốn lắm,

__Hà Nội 02/06/2019, ngày nắng mưa lẫn lộn,__