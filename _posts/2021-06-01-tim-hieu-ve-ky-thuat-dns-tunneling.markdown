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

## Từ Tunneling quá quen thuộc đến DNS Tunneling,

Tunneling là một kỹ thuật không mới. Khi nghe về nó ta dễ dàng hình dung tới mô hình nhồi một protocol trong một protocol khác để tạo một kênh kết nối giữa hai điểm.

![tunnel]( {{site.url}}/assets/img/2021/06/01/210601_tunnel_ssh.png){:width="500px"}

Mục đích của tunneling có thể là để tạo ra VPN, GRE tunnel, ... tuy nhiên đôi khi nó còn bị lợi dụng để qua mặt (bypass) các Firewall vì trong rất nhiều tình huống hệ thống gần như không có bất kì liên lạc nào với internet ngoại trừ một số protocol nhất định, muốn trao đổi thông tin với internet qua các protocol khác đành phải đi qua con đường lách luật là nhồi nó vào các protocol được cho phép. Thế giới thì đa dạng Tunneling từ SSH, ICMP, HTTP, ... đủ cả.

Như cái tên của nó, DNS Tunneling là kỹ thuật nhồi dữ liệu (thường thấy là TCP/UDP) trong các bản tin DNS. 

Kỹ thuật này mô tả sơ qua như sau: Dữ liệu gửi từ client sẽ được mã hóa rồi chuyển đổi thành text và được cắt thành từng đoạn text nhỏ (<255). Các đoạn text nhỏ này được dùng làm subdomain cho domain gốc gửi lên dns server dưới dạng các DNS query bản ghi txt. Khi lên đến dns server các bản tin query này sẽ được lưu lại đến một số lượng nhất định, khi đủ sẽ được ráp lại với nhau thành một khối. Đồng thời mỗi lần nhận được query từ các sub domain này dns server sẽ gửi lại phản hồi cũng là từng mảnh dữ liệu đã được chuyển đổi thành text và cắt nhỏ qua nội dung record txt lại cho client. Client tương tự cũng lưu lại từng bản tin riêng lẻ khi đủ sẽ ráp lại thành một bản tin đầy đủ. OKey như vậy là quá trình truyền tin giữa client và server đã được thực hiện (có thể truyền và nhận dữ liệu).


## Demo DNS Tunneling,

### Chuẩn bị:

1. Server có IP public
2. Một domain mà bản có thể control các bản ghi

### Bước 1: Tạo một DNS server để quản lý các bản ghi DNS cho domain của bạn

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

### Bước 2: Cài đặt DNS tunnel Server và DNS tunnel Client

Có nhiều project hỗ trợ tunnel, sử dụng đại 1 cái là [iodine](https://github.com/yarrick/iodine). Với Iodine sẽ cung cấp sẵn 2 tool Server và Client - đương nhiên rồi. Project này sẽ cung cấp cho ta phương tiện tạo kênh kết nối thông qua DNS Tunneling trong suốt giữa Client và Server. Dữ liệu truyền cũng được mã hóa luôn, chả khác gì VPN. 

Tải project về Server và không quá khó để build ra binary

```
apt-get install build-essential -y
apt-get install libz-dev -y
apt-get install -y pkg-config -y

git clone https://github.com/yarrick/iodine.git
cd iodine
make
```

OKie lúc này trong thư mục ```./bin``` ta đã có binary rồi, iodined - server và iodine - client.

#### Chạy DNS tunneling server trên DNS server bằng lệnh sau:

P/S: Chạy bằng quyền root nhé vì port 53 muốn binding cần quyền root.

```
./iodined -c -f 10.0.0.1 toan9.com
```

Không quên mật khẩu để mã hóa dữ liệu tunneling.

![Run server]( {{site.url}}/assets/img/2021/06/01/210601_run_server.JPG){:width="700px"}

Lúc này:
* 10.0.0.1: Là ip tun interface của DNS tunneling server. Chọn 1 ip bất kỳ tốt nhất là khác dải mạng với toàn bộ các interface trên máy chủ.
* toan9.com: Domain của ta mà dns server quản lý
* -c: để disable check client IP/port on trên mỗi request
* -f: Để server chạy ở foreground
* Các option khác tham khảo ở [github.com](https://github.com/yarrick/iodine/blob/master/src/iodined.c)

Kết quả của việc này là DNS server sẽ có 1 tun interface

![ip config]( {{site.url}}/assets/img/2021/06/01/210601_ip_server.JPG){:width="700px"}

#### Trên client chạy lệnh để tạo DNS tunneling bằng lệnh sau:

Ta copy file iodine về client và chạy lệnh sau để run client

```
./iodine -f toan9.com
```

Trong đó:
* -f: Chạy client ở foreground
* toan9.com: Domain ta quản lý bằng DNS server

Không quyên nhập mật khẩu trùng mật khẩu lúc ta nhập trên server

![client pass]( {{site.url}}/assets/img/2021/06/01/210601_client_pass.JPG){:width="600px"}

Lúc này trên client cũng xuất hiện một tun ip (giống như VPN vậy đó)

![client ip]( {{site.url}}/assets/img/2021/06/01/210601_client_ip.JPG){:width="600px"}

OKie xong hết rồi lúc này ta dễ dàng giao tiếp với server qua kênh tunnel mã hóa hẳn hoi chả khác gì VPN thông qua DNS Tunneling.

![client ping]( {{site.url}}/assets/img/2021/06/01/210601_client_test_ping.JPG){:width="600px"}

Tới đây rồi ta có thể làm nhiều trò: SSH, SOCK proxy, .... Nhưng mà cái kênh kết nối nay truyền dữ liệu hơi chậm, chậm như những năm 80 vậy nên chỉ dành cho những ai rất kiên nhẫn mà thôi.

## Mổ xẻ một chút,

Dùng Wireshark capture lại các gói tin trên interface máy client ta sẽ chỉ toàn thấy gói tin của bản tin DNS (Đương nhiên rồi).

![client wrk]( {{site.url}}/assets/img/2021/06/01/210601_wireshark_overview.JPG){:width="600px"}

Chọn đại một cặp query - response ta sẽ thấy như sau

![client query]( {{site.url}}/assets/img/2021/06/01/210601_wireshark_txt_query.JPG){:width="600px"}

![resq query]( {{site.url}}/assets/img/2021/06/01/210601_wireshark_txt_response.JPG){:width="600px"}

Kết quả chứng tỏ về cơ bản như ta đã mô tả ở trên.

### Phát hiện thế nào

Bằng tay: Captrue gói tin và đọc bằng wireshark ta dễ dàng nhận thấy vài điểm sau:

* Số lượng gói tin DNS đột biến tăng cao, các request chủ yếu là request TXT record
* Rất nhiều subdomain dị được query bản ghi TXT. Kết quả trả về trong nội dung bản ghi TXT cũng dị không kém (Thông thường bản ghi TXT ở dạng text khá trong sáng chứ không dị thế này)

Bằng máy:

* Firewall layer 4: Chịu!
* Nhiều loại NextGen FW (Ví dụ Paloalto) cũng hỗ trợ việc detect DNS tunneling rồi. Việc không phát hiện chỉ là xem lại bạn đã cấu hình đúng và đủ tính năng hay chưa thôi.

>PS: Cảm ơn a #langtubongdem đã pre hỗ trợ cho bài viết,

**Tham khảo**

[medium.com](https://medium.com/@galolbardes/learn-how-easy-is-to-bypass-firewalls-using-dns-tunneling-and-also-how-to-block-it-3ed652f4a000)