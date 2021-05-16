---
layout: post
title: Linux OS đã làm những việc gì trong quá trình boot?,
date: 2019-05-31 00:32:20 +0700
description: # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Linux]
---
Tôi hỏi thật lúc ngủ dậy a/e thường làm những gì? Có hay làm gì bất chính không đấy =))

![Wake up]( {{site.url}}/assets/img/2019/05/31/wakeup.jpg)

>Đây là tôi mỗi sáng ngủ dậy ae ạ

Đùa ae tí, thật ra là tôi có dự định làm 1 số bài về boot process trong linux. Nội dung nôm na là "Chuyện gì đã xảy ra từ khi ta ấn nút power on đến khi ta login thực sự vào trong 1 server linux". Tôi nghĩ đây là 1 chủ đề khá hay vì có thể nó sẽ có ích khi mà một ngày đẹp trời ae không thể login vào server? - What the hell? tai sao vậy ta? (Good reason to start!)

![Why?]( {{site.url}}/assets/img/2019/05/31/why.png)

OK, không nói nhiều start luôn,

## **Linux Boot process**

Cả buổi tối ngồi đọc rất nhiều bài viết cả tiếng anh và tiếng việt. Các bài viết nhìn chung khá dễ hiểu và phân tích chi tiết. Đấy là **con người ta**, còn mình thì chưa biết viết ntn đây? OK nhưng không sao tôi vẫn viết, viết theo cách của tôi.


### Overview - flowchat boot process, 

Tôi thì tôi thích mấy cái hình anh em ạ, nhìn cái hình nó dễ hiểu hơn là đọc chữ - đau đầu dã man. Nhờ google tôi kiếm được cái hình này ae ạ.


![Boot process]( {{site.url}}/assets/img/2019/05/31/linux_boot_process0.png)

Từ cái hình tôi chia ra làm 3 phase chính

* BIOS stage
* Boot loader stage
* Kernel stage

Vì vậy tối sẽ bám theo sườn này và kết hợp với cái hình kia để mô tả ke ke,

### BIOS - phase 1, beep beep tiếng kêu thần thánh,

Máy ae mà **đời đầu** dùng máy tính case hẳn đíu thể nào quên tiếng beep thần cmn thánh. Vào quán Net giá 2,5k/1h háo hức vl khi bật lên máy kêu một tiếng beep, biết ok ngon rồi. Mà cứ kêu vài tiếng beep beep coi như là xong cmnl máy hỏng (So sad) bác chủ đổi máy cho cháu. Sau ms biết tiếng beep này bắt nguồn từ đoan **bios** đóa ae.

Mô tả quá trình này nó là đoạn màu xanh ở đây ae ạ.

![Bios stage]( {{site.url}}/assets/img/2019/05/31/bios1.PNG)

Bios là các đoạn mã ASM (assembly) nhúng vào một dạng bộ nhớ không mất dữ liệu khi khởi động lại (bền vững) mà nó gọi là ROM (nghe quen quá). Vai trò của nó đơn giản là. 
* POST (Power On Self Test) : Check/Test xem phần cứng máy (RAM, CPU, Ổ Cứng) có ok, sẵn sàng dùng không? Nếu ok kêu beep 1 cái rồi vào phase sau. Nếu không ok kêu beep liên tục và stop luôn (Móa hóa ra beep ở đây à, ae sửa máy cho gái chú ý nha). Việc này có vai trò đảm bảo khi máy đã bật lên là phải đủ hết, ngon hết không lỗi làm gì nha.
* Find MBR: OK nếu đã check ok rồi thì nó bắt đầu tìm vào thiết bị được cấu hình boot (Cái mà ae hay làm khi cài win để config boot device, thứ tự ưu tiên các kiểu đó). Sau đó tìm tới cái vị trí đầu tiên (sector đầu tiên) trong biết bị được lựa chọn boot. Đọc mảnh đầu tiên (độ dài khoảng 512b) gọi là MBR (Master boot record). Trong đó có thông tin vị trí của boot loader để mò vào thực hiện quá trình Boot Loader (Chương trình mồi). 

>Nghịch tí

Táy máy tôi thử dump cái MBR này xem? nó ntn

```
[root@localhost mbr]# dd if=/dev/sda of=mbr.bin bs=512 count=1
1+0 records in
1+0 records out
512 bytes (512 B) copied, 0.000387061 s, 1.3 MB/s
[root@localhost mbr]# file mbr.bin
mbr.bin: x86 boot sector; partition 1: ID=0x83, active, starthead 32, startsector 2048, 2097152 sectors; partition 2: ID=0x8e, starthead 170, startsector 2099200, 81786880 sectors, code offset 0x63
```
OK, có vẻ như là dump được, ae thử đọc coi cái mbr này cấu trúc ntn cái coi :) tôi gợi đòn thôi, ae tự đọc nha.

### Boot Loader - phase 2,

Boot Loader - Là Chương trình mồi khởi động. Mục tiêu của nó là đến cuối cùng là phải load đc kernel vào.  
Tôi nghe nói có 2 loại hay dùng là Grub hoặc LILO. Nhưng giờ thấy toàn dùng grub2 (version 2) vì thấy bảo nó ngon hơn LILO. Nên từ giờ tôi chỉ default nói grub thui cho đơn giản, quốc dân. 

>Khi cài os ( nhất là debian) đoạn cuôi cuối ae hay được hỏi có cài boot loader không? và cài nó vào đâu? (ae hay chọn Yes cài luôn trên disk cài os). Bt xong, Cũng không để ý hóa ra nay tìm mới biết nó được lưu vào đây nè ae.

![Boot loader]( {{site.url}}/assets/img/2019/05/31/grub1.PNG)

Tôi đi tiếp cái map flow ban đầu thì quá trình này là đoạn màu cam.

![Boot loader]( {{site.url}}/assets/img/2019/05/31/bootloader1.png)


Ngay bắt đầu quá trình Boot Loader màn hình sẽ hiển thị cái menu boot huyền thoại chính là menu của trương trình grub2. 

![Boot loader]( {{site.url}}/assets/img/2019/05/31/grubmenu.PNG)

Sau khi chọn OS boot từ menu grub này thì quá trình boot loader mới thực sự bắt đầu.

Bắt đầu Bootloader sẽ đồng thời load vào MEM 2 cái (Như trên hình flow kìa ae).

* Kernel: Phần core của OS. (Cái này nghe ai cũng biết rùi)
* initramfs (hoặc initrd): Là một loại mini file system được load toàn bộ vào RAM phục vụ cho kernerl dùng tạm trong thời gian đầu của quá trình?

>Để xem nó boot gì???? tôi mò vào file cấu hình grub ```cat /boot/grub2/grub.cfg```

![Boot loader]( {{site.url}}/assets/img/2019/05/31/grubmenu2.PNG)

>Ngó ở hình tối thấy tôi đang dùng ```vmlinuz-3.10.0-957.el7.x86_64``` => Đây chính là file nén kernel và ```initramfs-3.10.0-957.el7.x86_64.img``` => Initrd. Chính là 2 cái tui vừa nói ở trên.

>Nghịch thử tui view thử file **initramfs** coi (Nhớ cài thêm gói ```binwalk``` nha ae)

![Boot loader]( {{site.url}}/assets/img/2019/05/31/bin.PNG)


>Content initramfs y hệt như thư mục gốc của linux. Ờ thì nó là mini file system mà :). 
>Tự hỏi sao nó load luôn cmn file system chính vào lại còn load cái này làm gì? OK, tự trả lời "Khả năng cho nhanh và đỡ lỗi vì rõ ràng việc load vào 1 file system cồng kềnh có đủ thứ trên đời vào rất mất t/g và đang load dắt ở đâu thì xong cmnr boot stop, pc died :'( khỏi cứu vãn, khỏi check disk or fsck khi bad sector được. Ngoài ra linux có trò encrypt Partition. Rõ ràng việc dùng một Mini Filesystem là bước đệm để decrypt real Filesystem trước khi thực tế mount vào. Linux nó tính cả rồi ae ạ".

Tới cuối phase này kernel đã được load vào rồi. Và lúc này nó đang sử dụng mini Filesystem.

### Kernel - phase3, bước cuối để ae chuẩn bị login vào server 

![Boot loader]( {{site.url}}/assets/img/2019/05/31/kernel_last.png)

Như vậy sau khi load vào được kernel và initramfs vào thì kernel sẽ tạm thời mount sử dụng Mini Filesystem của initramfs trong MEM. 

Ngay sau đó nó sẽ chạy 1 script tên là **linuxrc** trong Mini Filesystem. Script này sẽ đọc file fstab để mount vào Filesystem thực tế. Đến khi mount xong Filesystem thực tế như lẽ thường của cs nó sẽ unmount, đá đít, loại bỏ Mini Filesystem ra (chả thế để mà thờ à?? bỏ cho nhẹ đầu). 

Tiếp sau quá trình hoàn thành việc mount Filesystem. Linux sẽ tiếp tục chạy 1 process Init. Init process có thể là SystemV (Init) hoặc mới hơn là SystemD. 

Init process luôn là process thủy tổ (cha) của toàn bộ các process khác luôn có pid là 1.

![Init]( {{site.url}}/assets/img/2019/05/31/init.PNG)

Từ proccess này sẽ thực hiện toàn bộ các công việc khởi tạo, chạy các script, service, ... các kiểu con đà điểu, túm lại là toàn bộ nhũng cái đã được thiết lập sẵn. 

Trong đoạn cuối quá trình, Init process sẽ load và chạy bash shell lên - Màn hình đen xì dành cho ae mê gõ lệnh =)) sau đó nữa là load cái gói giao diện GUI (Có thể là KDE hoặc GNOME) cho các ae đam mê với đồ họa và thân thiện.


OKey tới đây ae gõ mật khẩu vào và xài thui, kết thúc toàn quá trình.

Làm nhảm nhiều quá, không biết ae có hiểu được không? có đau đầu lắm không =)) 