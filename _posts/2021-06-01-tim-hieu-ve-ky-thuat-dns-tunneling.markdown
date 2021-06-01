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




**Tham khảo**

https://medium.com/@galolbardes/learn-how-easy-is-to-bypass-firewalls-using-dns-tunneling-and-also-how-to-block-it-3ed652f4a000