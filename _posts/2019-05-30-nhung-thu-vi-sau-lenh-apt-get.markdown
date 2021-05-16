---
layout: post
title: Những thú vị sau lệnh apt-get mà ít người để ý,
date: 2019-05-30 00:32:20 +0700
description: # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Linux]
---
>Tản mạn về apt, apt-get, apt-cache, mọi người từng xài bao nhiêu % trong nội dung bài này của tôi rồi?

Thống kê một trong những lệnh gõ nhiều nhất trên linux trong cuộc đời tôi có lẽ là **apt-get**. Nhẩm tính từ năm 1 đại học đã xài linux và gõ apt-get điên đảo rồi, cái hồi mà distro hay sử dụng là ubuntu tầm 9.X thì phải. Giờ đã lên 19 rồi (trùng hợp thế đúng 10 năm 10 base version). Nay có thời gian "take a note" về lệnh này, chỉ là sự điểm lại nhe nhẹ chứ không có ý cao siêu dạy người nên mọi người đừng chém tôi nhé. Ai thấy chán quá xin bỏ qua một cách nhẹ nhàng.

![apt-get]( {{site.url}}/assets/img/2019/05/30/front.jpg){:width="600px"}

### 1. Giới thiệu,

Apt-get nằm trong gói ***Advanced Package Tool***. Gói này gồm 3 tool rất quen là apt-get, apt-cache và apt (Cái này chắc mọi người ít nghe). Trong đó apt-get chịu trách nhiệm quản lý gói (Gỡ, xóa, update, download, ...). Apt-cache dùng cho việc search gói, thông tin gói. Còn lại **apt** (Từ phiên bản ubunutu 16.04) thấy các bác khẳng định chắc nịch là phiên bản cải tiến, là sự nâng cấp tổng hợp tính năng của apt-get + apt cache. Đọc tới đây tôi nghĩ có lẽ mọi người nên tập thói quen xài apt thay apt-get dần đi, không có ngày các bác ý hứng lên lại thay luôn apt-get, apt-cache bằng apt thì xong.

Apt-get, apt-cache, apt là bộ công cụ quản lý gói sử dụng dpkg để quản lý các gói deb. Lấy ví dụ  ***chỉ*** dùng dpkg cài gói A, check thông tin trong gói A yêu cầu phải cài gói B trước. Tuy nhiên tự dpkg nó không làm việc này vì scope của nó là chỉ tác động only trên gói đang cài A thui. Nên thông thường không thêm option gì nó sẽ báo lỗi là không cài được vì thếu dependencies packages. (Dĩ nhiên ép nó cài vẫn cài dược nhưng cài xong nó sẽ không chạy). Tuy nhiên việc hành xử lại khác khi xài apt-get hoặc apt cài gói. Lúc này apt-get, apt sẽ tìm kiếm và tải gói (nếu gói không có sẵn) trên repo về. Sau đó dùng dpkg để cài gói A. Nếu gói A yêu cầu gói B. Thì job cài gói A sẽ được cho vào queue để đó. Apt-get, apt sẽ tải gói B về, cài gói B ngon nghẻ. Sau đó móc từ queue ra cài gói A. Lúc này gói A cài được vì B đã được cài rồi.


### 2. Man page rất dài và dai nhưng tôi chỉ xin liệt kê vài lệnh hay dùng:

```
apt-get update             	->  apt update
apt-get upgrade            	->  apt upgrade
apt-get dist-upgrade       	->  apt full-upgrade
apt-get install package    	->  apt install package
apt-get remove package     	->  apt remove package
apt-get purge packate      	->  apt purge package
apt-get clean				->  apt clean
apt-get autoclean			->  apt autoclean
apt-get autoremove         	->  apt autoremove
apt-cache search string    	->  apt search string
apt-cache policy package   	->  apt list -a package
apt-cache show package     	->  apt show package
apt-cache showpkg package  	->  apt show -a package
```

Chắc list kia mọi người khá quen quá rồi. Tôi chỉ note qua 1 chút thôi, không giải thích nhiều. 

#### *Đầu tiên tôi focus vào làm rõ sự khác biệt giữa update, upgrade và dist-update*

Xét trường hợp sử dụng apt-get/apt khi thực hiện cài 1 gói việc đầu tiên sẽ đọc file /etc/apt/source.list (Hoặc các file trong thư mục /etc/apt/source.list.d nếu thư mục này được cấu hình load) tìm các location rồi download các meta data (chỉ duy nhất là thông tin: tên, version, ...) của gói cài đặt lưu tại database local như category về các gói. Sau này toàn bộ việc tìm kiếm các gói để cài đặt sẽ thực hiện tìm kiếm trên database này. Database như bản đồ để biết lấy gói nào ở đâu.
* apt-get/apt update là lệnh kích hoạt việc chạy cập nhật database local. Không thực hiện update các gói.
* apt-get/apt upgrade là lệnh thực hiện việc update gói. Tuy nhiên nó chỉ update gói hiện tại mà không thực hiện update các gói phụ thuộc.
* apt-get/apt dist-update thì khác. Nó sẽ update gói hiện tại và toàn bộ các gói phụ thuộc. 

Thoạt nhìn có vẻ apt-get/apt dist-update có vẻ ngon hơn apt-get/apt update nhưng thật ra rất nguy hiểm vì có thể làm miss match các gói update phụ thuộc vì có thể có 1 gói khác ngoài gói ta update đang xài các gói phụ thuộc này mà khi ta update lên không đúng version gói đó yêu cầu. Tốt nhất nên hạn chế sử dụng.

>Vừa nói về meta data của các package. Vậy khi lưu ở local nó nằm ở đâu? Tôi tìm 1 tí thì thấy nó nằm ở đây ```/var/lib/apt/lists```. Hấp nó thôi :)

![List db dir]( {{site.url}}/assets/img/2019/05/30/note1.PNG)

Tail thử file xem có gì hay ho không?
![List db dir]( {{site.url}}/assets/img/2019/05/30/note2.PNG)

OK fine, đó là text file đọc ngon mọi người ạ.

#### *Thứ 2 tôi phân biệt giữa: apt-get/apt remove và purge*

Với apt-get/apt remove khi gỡ gói vẫn giữ lại cấu hình cũ (thường là file cấu hình). Còn với apt-get/apt purge thì khi gỡ gói xóa bỏ luôn toàn bộ các cấu hình trước đó.

#### *Thứ 3 tôi phân biệt giữa apt-get/apt clean, apt-get/apt autoclean và apt-get/apt autoremove*

Tôi cài thêm gói tcptraceroute huyền thoại vì cái này hay dùng:

![List install tcptraceroute]( {{site.url}}/assets/img/2019/05/30/note3.PNG)

Giả sử ngẫu nhiên tôi táy máy tìm thấy thư mục sau ```/var/cache/apt/archives```  liên quan tới apt tool.

![list cache dir]( {{site.url}}/assets/img/2019/05/30/note4.png)


Nhìn kỹ có cả gói tcptraceroute.deb tôi vừa cài. Hóa ra là các gói deb trước khi cài nó được tải về đây. **Cái này hay** cài offine 1 gói mà nhiều gói dependencies ta có thể kiếm 1 máy dùng lệnh cài để tải về rồi copy qua máy khác cùng loại cài là ngon nghẻ nhỉ (Dĩ nhiên có thể lựa chọn xài tunnel =)) nhưng đây là 1 phương án khác cũng hay ho)

Em đi xa quá, quay lại với bài viết,

Gõ lệnh apt-get clean vào thư mục kia thì thấy xóa cmn hết file.

![list cache clear]( {{site.url}}/assets/img/2019/05/30/note5.PNG)

HÓa ra lệnh ```apt-get clean``` là xóa cache file deb ae ạ. 

Đọc 1 chút thấy lệnh ```apt-get autoclean``` làm việc tương tự là dọn dẹp thư mục ```/var/cache/apt/archives``` chỉ khác ở cái thay bằng xóa hết thì nó chỉ xóa các gói "no longer" tức là các gói mà repo không còn nữa? (Ví dụ như gói được cache là version cũ mà repo nó có version mới hơn) chứ không xóa toàn bộ như clean. Cái này demo khó quá =)) chưa nghĩ ra demo ntn ae ạ, chả nhẽ tự dựng cái repo demo thì ghê quá nha ;)

Còn lệnh apt-get/apt autoremove là lệnh tự động tìm loại bỏ các gói đã được cài để thỏa mãn dependencies của một gói X nào đó trước đây nhưng hiện không được sử dụng bởi bất cứ gói nào ( Có thể Do X đã được gỡ đi nhưng nó chưa được gỡ bỏ) 

Hy vọng mọi người vẫn hiểu tôi đang nói gì :)

![Understand]( {{site.url}}/assets/img/2019/05/30/understand.jpg){:width="200px"}


Chắc mọi người choáng hết đầu vì tôi viết dài quá rồi. Nhưng không sao, chịu khó chút tôi viết nốt tí thôi =))


### 3. Giờ tôi nói qua tí về mấy file cấu hình của apt-get/apt-cache/apt-get

#### Đầu tiên là file huyền thoại: /etc/apt/source.list.
Ví dụ 1 phần của file (chắc quá quen thuộc rồi)
```
root@svr:~# cat /etc/apt/sources.list
# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://archive.ubuntu.com/ubuntu bionic main restricted
# deb-src http://archive.ubuntu.com/ubuntu bionic main restricted
```

Nói qua format của file này. Theo man page nó như sau:

```Type [ options ] uri suite [component1] [component2] [...]```

Và như đặc điểm của man page là quá đầy đủ và phức tạp nên đọc không hiểu gì luôn. [Link full không che](https://manpages.debian.org/jessie/apt/sources.list.5.en.html)

Thôi thôi mệt lắm, ở đây tôi chỉ nêu qua đại ý vài cái chính thôi cho dễ hiểu.


* Type: Ở đây là kiểu nén file cài đặt (Archive type). Có 2 kiểu chủ yếu là deb (File trương trình dạng đã biên dịch rồi - binary) hoặc deb-src (File  source code. Dĩ nhiên nếu sử dụng source code thì tool apt-get/apt sẽ thực hiện build trên máy ta rồi sau đó cài luôn. Cái này ít gặp, thông thường gói cài là binary thôi).

* options: Có thể có hoặc không. Nó kiểu như điều kiện gì đó. Format là setting=value. Ví dụ: [arch=amd64] tức là chỉ dùng cho arch là amd64.

* uri: Đường dẫn tới repo lưu file mà apt-get download gói. URI có thể là  có thể là ftp, http, https hậc file:///, ổ đĩa.

* suite: Code name của bản phân phối os đang dùng. Ở ví dụ trên: Với ubuntu 18 tên của nó là bionic. 

* component: Có thể có nhiều component với các y/n khác nhau. Thông thường loại ta hay gặp là để phân loại riêng ra các nhóm .deb file dựa trên giấy phép mà nó tuân thủ. Họ debian (ubuntu, debian) nó hay tuân theo cái giấy phép gọi là DFSG (Debian Free Software Guideline - Quy định việc sử dụng, phân phối, thương mại, ... phần mềm). [DFSG link không che](https://en.wikipedia.org/wiki/Debian_Free_Software_Guidelines). Với ***main*** tuân thủ nghiêm ngặt DFSG và các gói này được release bởi Debian distribution. Contrib - Tuân thủ DFSG nhưng không do Debian distribution release. None-free - Không tuân thủ DFSG. ... cái này nói dài lắm, chỉ nêu ra cái cơ bản thôi.

Dễ hiểu chỉ cần nhớ rằng nó sẽ giúp ta xác định vị trí của packages file trên repo thì nằm ở những thư mục nào mà thôi. Thích thì ta chi vào view luôn trên trình duyệt luôn. Thấy nó bố trí thành các thư mục y hệt đó.

![list cache clear]( {{site.url}}/assets/img/2019/05/30/repo2.png)


#### Sau đó nói chút về file /etc/apt.conf hoặc thư mục /etc/apt.conf.d/

Ai đã xài APT tools tôi cá là đây là những file/thư mục mà mọi người ít sờ tới nhất, hầu hết toàn để mặc định. Như vậy là chuẩn rồi vì file này ghi quá rắc rối, không trong sáng và lại dùng vs mục đích bình thường thì cũng không cần sửa. Nhưng ngày hôm nay đi lạc hướng chút, cùng khai - phá nó coi xem có gì?

List dir ra thấy như sau:

![list dir]( {{site.url}}/assets/img/2019/05/30/apt_dir.PNG)

Dễ nhận thấy mỗi file đều có gán 1 số thứ tự ở đầu. Số này dùng để chỉ thứ tự load của nó chứ không phải người ta để thế cho vui đâu.

Theo tài liệu từ man page thì nói nó được tổ chức theo dạng cây. Thử lấy 1 file coi xem ntn?

```
root@svr:/etc/apt/apt.conf.d# cat 10periodic 
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "0";
APT::Periodic::AutocleanInterval "0";
```
Tổ chức theo cây thì tôi mô tả như sau: 

![list dir]( {{site.url}}/assets/img/2019/05/30/function.PNG)

Function group có nhiều nhưng thường có 2 loại hay dùng thôi: 

* APT: Điều chỉnh hành vi của apt tool
* ACQUIRE: Điều chỉnh cách thức download gói.

Mỗi loại có rất nhiều Function và Option khác nhau. Chi tiết mọi người đọc ở man page nha, quá dài tôi không thể viết hết được, chỉ mô tả cấu trúc qua qua cho dễ hình dung, hiểu rồi cần gì thì tra cứu thôi. [Link APT full không che](http://manpages.ubuntu.com/manpages/cosmic/man5/apt.conf.5.html)

Tôi điều chỉnh rất ít cấu hình thỉnh thoảng chỉ cấu proxy nên sử dụng Acquire function group.

```
cat /etc/apt/apt.conf.d/02proxy.conf
Acquire::http::Proxy "http://user:password@proxy.server:port/";
Acquire::https::Proxy "http://user:password@proxy.server:port/";
```

Tới chỗ này mọi người thử hình dung coi cái cây nó trông ntn nhỉ :-)

Bài viết dài quá, xin dừng tại đây không viết dài viết dai lại thành viết dại. Cảm ơn mọi người đã đọc tới đây.

>Have a good night,