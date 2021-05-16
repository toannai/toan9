---
layout: post
title: Hành trình vào một số trang web bị chặn tại Việt Nam,
date: 2020-06-23 00:32:20 +0700
description: Hành trình vào một số trang web bị chặn tại Việt Nam
img: 2020/06/23/200623_vn_flag.jpg
fig-caption: # Add figcaption (optional)
tags: [Security]
---
Hôm nay trên bàn nhậu vô tình anh em có bàn luận vấn đề một số trang web abc.xyz thật là khó truy cập từ việt nam dù cho dùng nhà mạng nào đi chăng nữa, khiến một số anh em bứt rứt :d mà chưa rõ nguyên nhân. Có chút cồn trong người làm khó ngủ cộng tò mò vào thử thì thấy đúng là không vào được. Thử ngồi check một số vấn đề mà anh em đưa ra xem sao.

### Giả thiết 1 chặn bằng việc block trên DNS server 

Hầu hết các ISP của Vietnam khi cung cấp dịch vụ internet đều default DNS server của người dùng là trỏ vào DNS của mình. Việc này sẽ làm giảm băng thông quốc tế (tiết kiệm tiền cho ISP) ngoài ra còn giảm độ trễ và tăng trải nghiệm người dùng (Người dùng sẽ không phải sang Sing hay US để hỏi DNS google, cloudflare, OpenDNS ... làm gì cho xa). Tôi thử dùng nslookup check thử domain bị chặn coi xem ra gì?

![dns]( {{site.url}}/assets/img/2020/06/23/dns.png){:width="400px"}

OK đúng là DNS có vấn đề rồi, query domain của web bị filter không thấy trả lại gì cả. Nhưng nếu chỉ là DNS không trả lại gì (vô tình hay cố ý) thì chỉ cần change DNS là ok thôi. Như vậy tôi thử change DNS coi sao?



### Giả thiết 2 lọc dựa trên inspect content DNS packets

Sau khi đổi DNS máy tính thành DNS google (8.8.8.8) và query lại bằng nslookup thì thấy trả lại ip website có vẻ ok.

![resolv]( {{site.url}}/assets/img/2020/06/23/resolve.png){:width="300px"}

Chứng tỏ không phải ISP chặn bằng việc filter dựa trên inspect content trong DNS packets. Vì nếu inspect nội dung DNS thì việc query này sẽ không thể thực hiện được.

Căn bản vụ DSN lúc này ok rồi, nhưng khi vào bằng browser thì vẫn không được. 

![site]( {{site.url}}/assets/img/2020/06/23/sites.png){:width="600px"}

Nghĩa là việc chỉ change DNS không ăn thua trong tình huống này. Liệu có phải website bị chặn theo IP hay không? Để test thử telnet tới 443 theo ip mà nslookup được.

![ip]( {{site.url}}/assets/img/2020/06/23/telnet.png){:width="400px"}

Kết quả telnet cho thấy vẫn thành công. Chứng tỏ cũng không phải chặn theo ip. Cùi mía sửa thử file host trỏ thẳng vào ip của website. Lúc này khi vào bằng browser thì thấy lỗi kết nối bị reset.

![reset]( {{site.url}}/assets/img/2020/06/23/reset.png){:width="600px"}

Nhìn thông báo kiểu này, theo kinh nghiệm cá nhân thấy đây là một trong những cách thức phổ biến xử lý chặn kết nối của 1 số loại firewall hoặc DLP (Nghĩa là đúng là các ISP đang sử dụng công cụ để chặn kết nối rồi không còn nghi ngờ gì nữa). Nguyên lý cách chặn này là kết nối đang trong quá trình thiết lập bắt tay dở thì một thiết bị ở giữa chen vào bị bắn trở lại gói tin với flag reset làm cho kết nối cứ bị khởi tạo lại liên tục và không thể thiết lập nổi. Để làm việc này chắc chắn các thiết bị này phải phân biệt được các gói tin mà nó cần chặn với các gói tin khác không cần chặn dựa vào thông tin nào đó cụ thể ở đây là domain. Nhưng kết nối của các website ở đây là HTTPS mà? làm sao đọc được trường Host trong header của gói tin HTTPS được nhỉ? Không lẽ nhà mạng đã giải mã được gói tin mã hoá bởi session key trong https? - Vô lý, https giờ không còn an toàn ư? Cơ mà, theo hóng được thì kha khá lần các ISP Việt Nam đã cố broken https nhưng không làm được phải bỏ cuộc và trick cơ mà.


### Giả thiết 3 lọc dựa trên inspect nội dung ở một trong các step của bản tin https

Theo gợi ý của một số người anh là trong SSL/TLS có một số step gửi thông tin server ở dạng không mã hoá. Thử dùng wireshark ở giữa capture lại 1 ssl session (ví dụ session truy cập https://kenh14.vn). Lúc coi dữ liệu gửi đi thì đúng là có thấy một nội dung như sau.

![wỉreshark]( {{site.url}}/assets/img/2020/06/23/wireshark.png){:width="600px"}

OKie. Rõ ràng là hoàn toàn có thể nhìn thấy thông tin domain trong Client Hello của SSL/TLS mà không cần thiết phải decrypt packets gửi bởi SSL/TLS. Rất có thể là các thiết bị firewall, DLP đã sử dụng thông tin này.

=> Chưa dám khẳng định là các ISP dùng cách nào nhưng dựa trên vài info "mò mẫm" ở trên có thể thấy họ đã sử dụng kết hợp 1 vài thứ để filter traffic người dùng (chăn/lọc bỏ). Ít nhất có thể nhìn thấy ở đây là block ở DNS server (dĩ nhiên chỉ DNS server mà ISP quản lý) và ngoài ra rất có thể là còn cả kết hợp việc sử dụng trường server name trong Client Hello của HTTPS để thực hiện ý đồ của mình.

### Nhưng dù bị lọc tôi vẫn muốn vào thì làm sao giờ anh em nhỉ? 

Bạn bè của tôi toàn làm IT nên dăm ba cái filter của nhà mạng cũng không làm khó được mọi người rồi, tôi không dám múa rìu qua mắt thợ. Thôi thì tôi chỉ liệt kê ra đây một vài cách thôi. 

* Đơn giản nhất là xài vpn. Cách này thì tốn tiền vì chưa thấy chỗ nào nó free vpn cả. Nói chung dùng xài thì ổn định nhưng nguy cơ là vẫn có thể bị snip traffic vì dữ liệu của mình toàn bộ chảy qua vpn server. Nếu mà không owner vpnserver cũng không ổn lắm.

* Cách thứ 2 là dùng một free ssl proxy. Ứng viên đầu tiên nghĩ đến khá tốt là dùng Ultrasurf (OK, you are free và khá nổi tiếng trước đây hồi fb bị block công khai). Với ứng viên này hiện có 2 lựa trọn là dùng bộ cài trên máy hoặc plugin trên browser. Ưu điểm là free nhưng nhược điểm là ăn qua proxy nên khả năng cao (80-90%) bị sniff các traffic riêng tư hay bị chèn link quảng cáo vì đời méo có gì là free đâu.

* Cách thứ 3 là dùng Tor: Cách này xem ra nhìn về mặt technical thì ổn nhất vì vừa free mà sử dụng cho web surf hàng ngày cũng hay vì giúp nó giúp ẩn danh trên mạng máy tính.

Dù gì đi nữa thì tôi thấy việc chặn lọc Internet cũng vô nghĩa thật sự, nhiều người thừa thời gian làm những việc vô nghĩa thế nhỉ.
