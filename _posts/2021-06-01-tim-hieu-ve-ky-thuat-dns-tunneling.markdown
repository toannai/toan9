---
layout: post
title: Tester 101 - DNS Tunneling,
date: 2021-06-01 00:32:20 +0700
description: Tìm hiểu về kỹ thuật DNS Tunneling,
img: 2021/06/01/210601_intro_dns_tunneling.png
fig-caption: # Add figcaption (optional)
tags: [Linux]
---
Trước đây hồi làm ở v có đợt hóng thấy redteam có sử dụng DNS Tunneling để bypass firewall. Mấy lần định test mà quanh đi quẩn lại quên, nay nhân dịp có làm một task liên quan nên đọc và viết bài này luôn. 

## Tunneling - khái niệm quá quen thuộc,

Tunneling là một kỹ thuật không mới. Khi nghe về nó ta dễ dàng hình dung tới mô hình nhồi một protocol trong một protocol khác để tạo một kênh kết nối giữa hai điểm.

![tunnel]( {{site.url}}/assets/img/2021/06/01/210601_tunnel_ssh.png){:width="500px"}



Mục đích của tunneling có thể là để tạo ra VPN, GRE tunnel, ... tuy nhiên đôi khi nó còn bị lợi dụng để qua mặt (bypass) các Firewall vì trong rất nhiều tình huống hệ thống gần như không có bất kì liên lạc nào với internet ngoại trừ một số protocol nhất định, muốn trao đổi thông tin với internet qua các protocol khác đành phải đi qua con đường lách luật là nhồi nó vào các protocol được cho phép. Thế giới thì đa dạng Tunneling từ SSH, ICMP, HTTP, ... đủ cả.

Như cái tên của nó, DNS Tunneling là kỹ thuật nhồi dữ liệu (thường thấy là TCP/UDP) trong các bản tin DNS.  

Để phân tích về kỹ thuật này tôi sẽ đi ngược một chút là demo trước rồi phân tích sau cho nó ngược với bình thường.

## Demo DNS Tunneling,

### Chuẩn bị:
1. Server có IP public
2. Một domain mà bản có thể control các bản ghi

### Bước 1: Tạo một name server để quản lý các bản ghi DNS cho domain của bạn

Bình thường khi mua domain nhà cung cấp hay free luôn ta luôn cái dịch vụ quản lý DNS - bản chất là cho phép ta tạo các bản ghi cho domain của ta trên DNS server của họ. 

Như vậy khi user hỏi (query) một bản ghi DNS bất kỳ nào đó liên quan đến domain của ta (Có thể cần qua một chuỗi hỏi đáp với các server DNS trung gian) thì thằng cuối cùng trả lời bạn sẽ là DNS server của nhà cung cấp domain theo thông tin ta cấu hình trên WEB. 

Tuy nhiên các nhà cung cấp domain cũng cho phép ta custome một chút. Lúc này thay bằng việc DNS của nhà cung cấp trả lại thông tin họ sẽ cho phép ta forward việc trả lại này tới server DNS của ta. Việc này dễ dàng thực hiện bằng việc cấu hình custome Name Server. 

Tôi hay mua domain ở namesilo thì thực hiện như sau:

* Đầu tiên tạo 2 bản ghi nameserver cho domain trỏ vào ip public của vps làm dns server

![record 1]( {{site.url}}/assets/img/2021/06/01/210601_record1.png)

* Sau đó add 2 bản ghi nameserver vừa tạo vào domain của ta

![record 2]( {{site.url}}/assets/img/2021/06/01/210601_record2.png)

* Test lại bằng cách từ máy bất kỳ (có internet) nslookup domain của ta, cùng lúc tcpdump port 53 trên vps làm dns server mà thấy dns request ta thực hiện là thành công.

![record 2]( {{site.url}}/assets/img/2021/06/01/200601_result.png)

>Chú ý mở port 53 tcp&udp của vps nha. Vì dịch vụ DNS dùng 2 port này.


 


**Tham khảo**

https://medium.com/@galolbardes/learn-how-easy-is-to-bypass-firewalls-using-dns-tunneling-and-also-how-to-block-it-3ed652f4a000