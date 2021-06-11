---
layout: post
title: Cài đặt modem Huawei/GPON của Viettel thành bộ phát wifi,
date: 2019-08-06 00:32:20 +0700
description: Cài đặt modem Huawei/GPON của Viettel thành bộ phát wifi,
img: 2019/08/06/gpon.jpg
fig-caption: # Add figcaption (optional)
tags: [Linux]
---
Tôi biết bài viết này có thể là nhảm nhí vì trên fb của tôi toàn ae làm IT có thể biết cái này rồi. Nhưng không sao, vẫn có thể có người cần vì tôi search trên mạng cũng có người hd mà tôi làm theo không chạy được. Okey giờ tôi làm lại,

![gpon]( {{site.url}}/assets/img/2019/08/06/gpon.jpg){:height="500px"}

## I. Phần chính,

### Giới thiệu,

Ae hay cấu modem để phát wifi bằng mode NAT nhưng tôi làm theo không được. Nên giờ tôi hướng dẫn cấu con modem làm con switch layer2 thôi.


### Bước 1: Reset
Chọc cái que tăm vào lỗ reset trên modem giữ tới khi nào mấy cái đèn nó nhấp nháy liên tọi để reset modem về chế độ của nhà sảm xuất (Thật ra là chế độ do Viettel đặt hàng Huawei) - Cái này ae thừa biết rồi.

Lúc này thông tin đặng nhập nó in ở cái tờ giấy dính ở đít modem. 

PS: Ái chà Viettel giờ ngon quá sau sự cố default password năm ngoái Modem dạo này đã sinh password ngẫu nhiên. 

### Bước 2: Login vào Huawei modem và chiến thôi

* Link login: Paste vô trình duyệt để vào cấu hình modem 

***https://192.168.99.1:80/***

```HTTPS port 80, chơi trội quá nhỉ ae.```

* Vào phần Advance Setup trong màn hình đăng nhập

![Advance]( {{site.url}}/assets/img/2019/08/06/02.PNG){:height="400px"}

* Loại bỏ hết cấu hình trong phần WAN => Configuration: Nói chung có gì thì Delete hết cho trống trơn như hình sau

![w1]( {{site.url}}/assets/img/2019/08/06/03.PNG){:height="400px"}


* Tắt DHCP ở phần LAN

Phần LAN -> LAN Port work mode Chọn toàn bộ các port là port LAN

![L2]( {{site.url}}/assets/img/2019/08/06/05.PNG){:height="400px"}

Phần LAN -> Host configuration: Đặt cấu hình để truy cập Modem. Đặt subnet nào cũng được. Cứ chọn 99 như tôi cũng ok ```(*)```

![L3]( {{site.url}}/assets/img/2019/08/06/06.PNG){:height="400px"}

Phần LAN -> LAN DHCP configuration tắt DHCP đi và cấu hình DHCP Relay để cho máy client khi kết nối nhận ip do modem tổng cấp.

![L4]( {{site.url}}/assets/img/2019/08/06/07.PNG){:height="400px"}

* Đặt lại SSID và password WLAN/Wifi cho dễ nhớ

![L5]( {{site.url}}/assets/img/2019/08/06/10.PNG){:height="400px"}

* Giờ chỉ việc cắm dây LAN từ modem chính vào cái modem vừa cấu hình là xong.

### Bước 3: Kết nối ở client

* Với mobile: Kết nối ok

* Với LAPTOP: Chs nó lại nhận DHCP là ipv6. Giờ đối phó chỉ cần vào phần cấu hình network của card wifi tắt ipv6 của laptop đi (Thật ra giờ có dùng ipv6 đâu nên tắt đi cũng chả ls).

![L5]( {{site.url}}/assets/img/2019/08/06/12.PNG){:height="400px"}

***Nếu cần reset lại máy tính để nó ăn cấu hình nha***

### Muốn vào cấu hình wifi thì làm thế nào?

OK giờ chỉ cần vào cấu hình 1 ip tĩnh cho card wifi đang kết nối vào con wifi huawei này cùng dải mạng với dải đã đặt để manage ở bước 2 ``(*)`` . Sau đó vào trình duyệt địa chỉ ***https://192.168.99.1:80/*** cấu như bình thường.

Ví dụ: nếu bước 2 đặt như tôi thì đặt ip máy tính cấu hình là 192.168.99.2 netmask 255.255.255.0 rồi ping sang 192.168.99.1 mà ok là triển bình thường thôi.


## II. Phần lảm nhảm,

Tôi học CNTT nhưng ngày xưa ở xóm trọ tôi rất lười, ba bốn cái chuyện mạng mẽo này kia chả bao giờ tôi sờ vào, mặc mn muốn làm gì thì làm. Rồi, ờ quê tôi có ông chú họ rất kiểu "khéo tay hay làm" dù ở TP nhưng cứ mỗi lần về nhà tôi chơi lại giúp hí hoáy sửa cái lọ cái chai. OK fine :) tôi ngưỡng mộ quá ý. Có lúc rảnh ngồi ngẫm, ss ra thấy mình thật vô trách nhiệm & nhỏ nhen ý. Vậy là từ đó về sau tôi chủ động hơn trong ba bốn cái chuyện ít ra mình có khả năng. CS mà :) thỉnh thoảng cần thay đổi gì đó cho nó mới mẻ chính mình.

Hi, làm thợ điện cũng không khó nhỉ? không rõ thợ sửa ống nước thế nào? 

