---
layout: post
title: Từ SSL/TLS handshake - back to basic, đến vấn đề chọn IDS/IPS,
date: 2022-01-02 00:32:20 +0700
description: Hồi còn làm tại VCS có xảy ra một case là có PCAP và SSL/TLS private key server (Có thể kèm master secret) liệu có thể đọc được dữ liệu trong PCAP không? Hôm ấy cũng tranh cãi khá nhiều, sau đó tôi không follow case này cũng không rõ kết quả mãi tới gần đây công việc lại va phải nên nay tôi đọc lại để trả lời cho câu hỏi này :) ... Cuộc sống mà không trả lời câu hỏi này hôm nay thì chắc mai lại gặp lại thôi =))
img: 2022/01/02/sslvatls.jpg
fig-caption: # Add figcaption (optional)
tags: [Security]
---

Hồi còn làm tại cty cũ một lần XLSC có xảy ra một case là ae có được PCAP và khách hàng sẵn sàng cung cấp SSL/TLS private key (Có thể kèm master secret), ae tập trung tìm xem liệu có thể đọc được dữ liệu trong PCAP không? Hôm ấy cũng tranh cãi khá nhiều, tuy nhiên sau đó tôi không follow case này cũng không rõ kết quả. Mãi tới gần đây công việc lại va phải nên nay tôi đọc lại để trả lời cho câu hỏi này :) ... Cuộc sống mà không trả lời câu hỏi này hôm nay thì chắc mai lại gặp lại thôi =)) Kinh nghiệm là có gặp gì đó chưa rõ ràng hãy cố mà trả lời bằng được.

# Tả phí lù liên quan,

Qua một thời gian làm dài làm việc tôi nhận ra rằng có lẽ ý tưởng xiên suốt của networking là ý tưởng phân tầng (Layer).  Việc bảo vệ dữ liệu sử dụng mã hóa có lẽ cũng phụ thuộc vào ý tưởng này. Ta có thể lựa chọn mã hóa dữ liệu ở một tầng nhất định trong OSI. Tầng nào cũng được tùy thuộc vào security protocol sử dụng.

![security protocol layer]( {{site.url}}/assets/img/2022/01/02/protocol-stack.png){:width="700px"}


Phổ biến nhất mà ai chắc cũng từng nghe là HTTPS và SSL/TLS. Có thể nhiều a/e đều đã biết nhưng tôi vẫn cứ reviews nhanh một chút: TLS cơ bản là người kế nhiệm - phiên bản thay thế của SSL. Còn HTTPS là một sự "implementation" (cài đặt cho một t/h cụ thể) của SSL/TLS cho giao thức HTTP, chính vì quan hệ "Trèo đầu cưỡi cổ" vậy sẽ không có sự tách rời của HTTPS và SSL/TLS.

**Nhắc lại**: Có 2 loại mã hóa chính là đối xứng và bất đối xứng - cái này là thứ mà ai cũng từng nghe thao thao bất tuyệt cả thời sinh viên do vậy tôi nghĩ cũng không cần nhắc lại nó là gì ở đây.

![algorithm]( {{site.url}}/assets/img/2022/01/02/algorithms.png){:width="400px"}
 

Ứng dụng tiêu biểu của mã hóa (đặc biệt là mã bất đối xứng) là để Mã hóa & Chống chối bỏ (Xác thực tôi là tôi chứ không phải là ai khác).

# SSL/TLS handshake, 

Quay trở lại vấn đề với SSL/TLS (Từ đây nếu không nhắc gì xin coi SSL/TLS là TLS cho ngắn gọn). Quá trình mã hóa này sử dụng kết hợp cả mẫ hóa đối xứng và bất đối xứng. Mô tả cơ bản của quá trình này gồm 2 step:

+ Step 1: Hai bên (Client & Server) sẽ sử dụng mã hóa bất đối xứng để xác nhận nhau (Chống chối bỏ) và trao đổi/thống nhất một khóa phiên (session key).
+ Step 2:  Session key ra ở step trước sẽ được sử dụng để mã hóa dữ liệu trao đổi sử dụng khóa đối xứng. 

=> Rõ ràng bằng cách nào đó lấy được Session key thì có thể đọc được dữ liệu trao đổi. Như vậy chỉ cần tập trung vào đây thôi.

TLS handshake chính là Step 1 trong quá trình trên. Nếu vẽ bằng hình thì nó diễn ra thế này:


![tls handshake]( {{site.url}}/assets/img/2022/01/02/tls_handshark.PNG){:width="700px"}

Nhấn mạnh lại (Nói đi nói lại quá nhiều lần) ý kết thúc SSL/TLS handshake phải đảm bảo 2 công việc.

+ Xác nhận client/server - Chống chối bỏ (Đảm bảo rằng Client/Server xác nhận là tôi chính là tôi mà không phải là ai khác)

+ Thống nhất được 1 session key để mã hóa dữ liệu trao đổi giữa 2 bên.

Tùy thuộc vào thuật toán/lược đồ trao đổi khóa mà SSL/tLS handshake diễn ra các bước chi tiết - message khác nhau. Ta sẽ thường thấy loại phổ biến chính là Rivest–Shamir–Adleman(RSA) và Diffie-Hellman (DH).

### SSL/TLS sử dụng RSA

Nếu sử dụng RSA thì quá trình diễn ra như sau:

![tls rsa]( {{site.url}}/assets/img/2022/01/02/ssl_handshake_rsa.jpg)


+ **Client hello** message: client khởi tạo quá trình handshake bằng việc gửi đi "Client hello" message tới server. Message này sẽ bao gồm TLS version, cipher suites (Cipher suites là gì sẽ mô tả ở sau) mà client support, kèm theo một chuỗi các bytes là một random string do client sinh ra gọi là "client random."

+ **Server hello** message: Trả lời của server cho 'client hello' message, message này bao gồm server's SSL certificate, cipher suite mà server đã lựa chọn từ danh sách client gửi lên, kèm một "server random," - một chuỗi bytes là random string được sinh ra bởi the server

+ **Authentication**: Sau khi nhận được 'Server hello', client sẽ verifie server's SSL certificate với certificate authority đã issued SSL Certificate (Bằng các CA cert trong cấu hình sẵn của OS). Việc này để đảm bảo rằng server mà client đang kết nối tới chính là nó chứ không phải là ai khác giả mạo. Nếu Verify là đúng thì quá trình tiếp tục, nếu verify sai thì browser hiện lên cái ô confirm quen thuộc yêu cầu người dùng có tiếp tục không hay là break tại đây :)

![tls warn]( {{site.url}}/assets/img/2022/01/02/browser_warnings_1.png){:width="400px"}

(Dĩ nhiên nếu ok hoặc người dùng chấp nhận rủi do thì quá trình TLS handshake với tiếp tục các step sau)

+ **The premaster secret**: Client sẽ gửi lại một chuỗi bytes có giá trị là random string gọi là "premaster secret". Premaster secret được mã hóa (encrypted) bằng public key của server và duy nhất có thể decrypted bằng private key của server server. (Client lấy public key từ SSL certificate của server).

+ **Private key được sử dụng**: Server sẽ decrypt message từ client để lấy premaster secret.

+ **Tạo Session key**: Cả Client và Server cùng sinh ra session keys từ client random, server random và premaster secret. Đương nhiên là sẽ ra cùng một kết quả.

+ **Client is ready**: Client gửi lại "finished" message được mã hóa bằng session key.

+ **Server is ready**: Server gửi lại "finished" message được mã hóa a session key.

+ **Secure symmetric encryption achieved**: Quá trình handshake hoàn thành, và kết nối ở pharse sau được tiếp tục với session key đã thỏa thuận ở 2 bên Client và Server.

### SSL/TLS sử dụng Diffie Hellman (DH)

![tls dh]( {{site.url}}/assets/img/2022/01/02/ssl_handshake_diffie_hellman.jpg)

+ **Client hello**: Client sẽ gửi một 'client hello' message có chứa TLS version, client random, và một list các cipher suites client support.

+ **Server hello**: server sẽ phản hồi lại client SSL certificate của nó, một cipher suite lựa chọn từ danh sách client gửi lên, và the server random. (Tới đây vẫn giống khi sử dụng RSA)

+ **Server's digital signature**: Server sử dụng private key của nó để encrypt client random, server random, và các paramater dùng trong thuật toán Diffie Hellman (DH). This Dữ liệu được mã hóa này coi như là server's digital signature, được mã hóa bằng server private key cái mà matches với public key trong SSL certificate của server.

+ **Digital signature confirmed**: The client giải mã server's digital signature với public key của server, verify rằng server sở hữu đúng private key tức là Server chính là nó không phải là một ai khác giả mạo. Sau đó Client gửi lại server các DH parameter của nó.

+ **Client and server calculate the premaster secret**: Thay bằng việc client sinh ra premaster secret và gửi nó it cho server như ở RSA handshake, the client và server sẽ sử dụng các DH parameters đã trao đổi với nhau để tính toán premaster secret một các độc lập. Dĩ nhiên tính toán độc lập nhưng kết quả cuối vẫn sẽ khác nhau. Hãy nhớ rằng thuật toán DH đảm bằng có lấy được DH Paramsters cũng không thể xác định được premaster key.

+ **Session keys created**: Now, the client and server calculate session keys from the premaster secret, client random, and server random, just like in an RSA handshake.

+ **Client is ready**: Từ đây các bước sau thực hiện tương tự như sử dụng RSA để thực hiện handshake.
+ **Server is ready**
+ **Secure symmetric encryption achieved**

### Chú ý: 

Tôi thấy đoạn trên hơi dài nên chỉ tóm tắt mấy một điểm chú ý bạn cần nhớ thôi:

>Điểm khác nhau cơ bản giữa việc sử dụng RSA và DH trong TLS handshake chính là việc RSA thì truyền Pre-master key trong message trao đổi giữa Client và Server. Còn nếu sử dụng DH thì không trao đổi Pre-master key mà trao đổi DH parameter.  

## Có được PCAP và SSL/TLS Private key server (kèm master secret nếu có) liệu có thể decrypt được data hay không?"

Câu trả lời là vừa có vừa không :d

+ Như vừa nói ở trên "Điểm khác nhau cơ bản giữa việc sử dụng RSA và DH trong TLS handshake chính là việc RSA thì truyền Pre-master key trong message trao đổi giữa Client và Server. Còn nếu sử dụng DH thì không trao đổi Pre-master key mà trao đổi DH parameter"

+ Nếu SSL/TLS handshake dùng RSA thì việc giải mã có thể thực hiện được (Với 1 số điều kiện nhất định - [https://wiki.wireshark.org/TLS](https://wiki.wireshark.org/TLS)). Lý do PCAP sẽ chứa Pre-master key có thể sử dụng để tính toán ra session key để decrypt data.

+ Nếu SSL/TLS handshake sử dụng Diffie Hellman thì việc giải mã không thể thực hiện được. Lý do là PCAP không hề chứa Pre-master key mà chỉ chứa các DH parameters. Có các DH parameters gần như không thể tính toán được Pre-master key từ đó không lấy được Session key. (Để hiểu chi tiết chắc cần phải đọc về thuật toán này [tại đây](https://tinhte.vn/thread/giai-ngo-https-hoi-2-canh-2-suc-manh-cua-diffie-hellman-dh-key-exchange-protocol-den-tu-dau.3200798/))

+ Ta có thể import SSL/TLS private key server (trong trường hợp sử dụng RSA handshake) hoặc Pre-master key trong bất kỳ trường hợp nào vào wireshark để thực hiện decrypt SSL traffic theo hướng dẫn của wrireshark [https://wiki.wireshark.org/TLS](https://wiki.wireshark.org/TLS)

(Anw chắc sẽ có một bài sử dụng wireshark đọc HTTPS traffic nhưng chắc không phải ở đây vì dài quá rồi, đọc phát ngán)

## SSL/TLS cipher suite,

**Trong bài viết ta có nhắc tới khái niệm Cipher suite. Vậy cipher suite là gì?**

Cipher Suites đơn giản chỉ là một tập các thuật toán mã hóa (cryptographic algorithms) mà client và server sẽ thống nhất sử dụng trong một phiên SSL/TLS. Mỗi thuật toán sử dụng cho 1 số các công việc cụ thể:

* Key exchange
* Bulk encryption
* Message authentication 

 Cấu trúc cơ bản như sau:

![cipher suite]( {{site.url}}/assets/img/2022/01/02/cipher_suite.PNG)

**Ai là người lựa chọn Cipher Suite?**

Thật ra là cả server và client. Client gửi lên danh sách cipher suite trong message client hello và từ danh sách này server sẽ chấm chọn cái nào cả 2 support => Server được ưu tiên hơn 1 chút vì là người chọn sau. Điều này nghĩa là cơ bản ta cũng tác động ở phía server được bằng việc đẩy Priority khi cấu hình list cipher suite trên server để một Cipher suite nào đó có độ "Ưu tiên" cao hơn 1 chút. 


## Các thiết bị IDS/IP và những vấn đề liên quan,

Ngày nay các dữ liệu web truyền trên mạng chủ yếu là được mã hóa HTTPS - SSL/TLS. Điều này làm cho việc truy cập web trở nên an toàn hơn nhưng cũng tạo ra điểm "mù" cho các thiết bị IDS/IPS khi đứng ở giữa, rõ ràng chúng không thể nhìn thấy các traffic này. Muốn IPS/IDS hoạt động ngon lành với SSL/TLS phải giải quyết bài toán này.

Thực tế cho thấy IPS hiện tại đang sử dụng 2 cơ chế chính để đọc dữ liệu SSL/TLS - Dĩ nhiên là đã phải import TLS/SSL Private key server. Nhưng như vừa phân tích, không phải cứ có SSL/TLS Private key server là đọc được Encrypted traffic. 

+ Forward proxy: Loại này nó đứng giữa coi mình như proxy server (fake server dịch vụ thật thật với client, fake client với server dịch vụ thật thật). Đại diện tiêu biểu là Paloalto:

![Forward proxy]( {{site.url}}/assets/img/2022/01/02/pl.PNG)

+ None - Forward Proxy (Chưa biết gọi là loại gì nên đặt tên tạm là vậy): Với loại này cố gắng kiếm pre-master key để decrypt traffic chứ không **thèm** làm fake - Server. Đại diện tiêu biểu là McAfee. Loại này nếu dùng RSA thì không cần cài agent. Còn nếu dùng DH thì phải cài agent để backend nơi mà thực hiện encrypt đẩy Pre-master key về IPS. (Còn không có agent thì gần như IPS chào thua với DH. Lý do tại sao đã giải thích rõ ở phần trên rồi).


![Mc ssl1]( {{site.url}}/assets/img/2022/01/02/mc_1.png){:width="500px"}


![Mc ssl2]( {{site.url}}/assets/img/2022/01/02/mc_2.png){:width="500px"}


*Note*: Bài học là khi lựa chọn IPS/IDS cần phải tỉnh táo với tính năng này lựa chọn loại phù hợp với môi trường hiện tại của mình và đừng quên test lại xem có vấn đề gì không để khẳng định lại xem IPS/IDS có thực sự hoạt động hiệu quả không.


## Tham khảo:
[https://blog.cloudflare.com/keyless-ssl-the-nitty-gritty-technical-details/](https://blog.cloudflare.com/keyless-ssl-the-nitty-gritty-technical-details/)

[https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/)






