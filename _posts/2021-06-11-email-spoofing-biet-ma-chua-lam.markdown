---
layout: post
title: Email spoofing vấn đề nhức nhối,
date: 2021-06-11 00:32:20 +0700
description: Email spoofing vấn đề nhức nhối, có thể biết mà không làm,
img: 2021/06/11/210611_email_spoofig.jpg
fig-caption: # Add figcaption (optional)
tags: [Security]
---

Gần đây công ty tôi nhận được một số thông tin khách hàng phản hồi rằng họ nhận được các email mạo danh công ty (email spoofing) gửi vào hòm mail của họ. Vấn đề này không mới, cty/tổ chức nào cũng có thể gặp. Cú cái nhiều cái biết rõ mà không thể làm gì (vì sao tôi nói sau).

Mục đích của email spoofing cũng không gì là lạ lẫm (Đặc biệt với ae system đã làm mail kiều gì cũng gặp). Có thể là spam quảng cáo, lừa đảo hoặc dải mã độc, ransomware. Một người dùng thông thường khi nhận được mail từ một địa chỉ email **họ nghĩ là** tin cậy không lý gì họ không mở, không click, không làm theo. Và chỉ sơ ý thôi là cũng có thành nạn nhân rồi. Hậu quả để lại nhiều khi không chỉ ảnh hưởng tới cá nhân nạn nhân mà đôi khi ảnh hưởng cả tổ chức (VD: Tình huống bị ransomware).

Trước đây khi audit email của gov/com tôi rất hay gặp vấn đề này, bữa nay tôi muốn chia sẻ một số kinh nghiệm của tôi với các ae system. Hy vọng đọc xong bài này ae sẽ bớt chút thời gian coi lại hệ thống của mình, sẽ có những action cụ thể. Mỗi người góp tay một chút sẽ làm thế giới tốt đẹp hơn một tí (nghe to tát không)  ( ͡° ͜ʖ ͡°)

### Những điều cần biết trước khi bắt đầu,

Ở đây có cả những ae đã biết rồi và những ae mới. Cũng có thể ae biết rồi mà chưa biết gọi tên nó là gì (T_T).

Theo wiki "Email spoofing is the creation of email messages with a forged sender address. ". Tiếng việt có thể hiểu là việc tạo ra các email message giả mạo người gửi (From user).

Nói về email spoofing chắc phải mất một buổi sáng (Đi training tôi hay mất tầm đó). Nhưng buổi nay chỉ xin nói một case khá phổ biến hay gặp nhất là spoofing bên ngoài domain tổ chức mà thôi. Hình thức này mô tả bằng hình sau:

![fucker]( {{site.url}}/assets/img/2021/06/11/210611_fake_mail.png){:width="500px"}

Mô tả: Lẽ ra chỉ mail server của khách hàng @customer mới được phép gửi từ sender@customer gửi cho receiver@mydomain nằm trên mail server của cty tôi @mydomain. Nhưng bằng cách nào đó mail server Fucker server lại lén lút mạo danh sender@customer làm việc này mà khi nhận mail tôi receiver@mydomain không nhận ra mình nhận từ nguồn không hợp lệ và cứ tin là mình đang nhận mail từ mail server @customer gửi.

Nguyên nhân chính là do yếu tố lịch sử, giao thức SMTP ban đầu quá đơn giản, kém bảo mật. Không có các cơ chế xác thực. Mãi sau này phải người ta phải nghĩ ra rất nhiều các cơ chế mở rộng (extends) để khắc phục dần nhược điểm này. 

>Để biết SMTP ban đầu nó thế nào ae hãy thử cài postfix core mà coi. Thật sự chả có cái gì luôn. Không xác thực, không mã hóa gì, ..., không gì hết chỉ có gửi và nhận. Sau này muốn có thêm các tính năng ta sẽ phải cài thêm các module/add-on đảm nhiệm các extension mở rộng này. Xem list modules/add-on tại đây](http://www.postfix.org/addon.html#auth) 


### Các cách chống lại Email spoofing,

Để giải quyết vấn đề Email spoofing người ta sử dụng cơ chế Email authentication, or validation. Đại ý của cơ chế này là bằng cách nào đó để bên gửi và bên nhận có thể xác nhận "mình là mình" với bên kia mà không phải một Fucker nào đó. Vài phương pháp có thể là:

* Sử dụng chữ ký số: DKIM
* Sử dụng bản ghi DNS: SPF/DMARC

Tới đây nội dung thế nào thì chắc tôi không cần nói nữa vì nó quá quen thuộc rồi. Nói lại sợ nhàm. 

Tuy nhiên một điểm luôn cần phải nhớ là "Email authentication" có tính hai chiều <==>.

* Khi nhận mail luôn cần kiểm tra để xác thực nguồn gửi có tin cậy hay không (Nguồn mình nhận đúng là đối tác) (1). Để thực hiện chỉ cần enable tính năng verify SPF/DKIM/DMARC trên thiết bị nhận mail từ internet.
* Khi gửi mail phải chứng minh là mình là mình (2). Để thực hiện cần cấu hình SPF/DMARC hoặc ký DKIM.

Rất nhiều ae chỉ làm một chiều, thường là (2) mà bỏ quên chiều (1) còn lại. 

Hoặc một số case ae cũng biết có những người biết chiều (1) nhưng lại không làm vì lý do "sợ"

### Biết nhưng không làm, cách vượt qua sợ hãi,

Quay lại lý do "Cũng có những người biết chiều (1) nhưng lại không làm". Mà cái tội biết mà không làm thì thực sự đáng trách. (Y_Y)

Lý do mà nhiều ae hay đưa ra là họ sợ DROP nhầm mail của customer. DROP nhầm theo nghĩa là mail gửi đến đúng mà mail gateway bảo Fake và DROP luôn gây ra bị miss mail, sếp quở trách. Các ae hay lý giải nguyên nhân cho việc DROP này như sau: "Nhiều mail server quản trị viên không cấu hình Mail Authenticaion (SPF/DKIM/DMARC) thế là enable tính năng check lên là mail gateway nó thịt (DROP) luôn những mail này". Do sợ hãi thôi cứ tắt đi cho lành. Và hậu quả bạn biết đó - email spoofing lại tiếp diễn như sự thật hiển nhiên. Hậu quả vừa trình bày trên.

Vậy cách xử lý ở đây là gì? (Để giải quyết có thể cần phân tích sâu các email bị DROP)

Ở đây tôi chỉ dám đề xuất phương án để ae vượt qua "Sợ hãi" ở trên. Ae vừa nói lo sợ xảy ra chuyện với các mail không cấu Email Authen vậy đơn giản là ta cứ by pass cho nó. Vì các domain không cấu SPF/DKIM/DMARC hiện giờ rất ít hoặc ít quan trọng (Không cấu google nó chém luôn hoặc không sớm muộn cũng bị cho vào Blacklist - ai cũng biết). Cùng lúc đề xuất **Chém mạnh tay - DROP** luôn bọn FAIL (Cả softfail và HarfFail) vì bọn này chắc chắn 99% là Fake - Tội biết mà không làm là tội to. 

Hiện nay các loại mail gateway quá mạnh, bóc tách quá sâu header để ae thực hiện việc này. Lấy ví dụ với IRON PORT của CISO -  SP mail gateway quá ngon mà quá nhiều ae dùng.

![IRON PORT]( {{site.url}}/assets/img/2021/06/11/210611_ironport.JPG){:width="500px"}

>Hãy mạnh dạn tick SoftFail và Fail và chọn action là DROP.

### Kết,

Cảm ơn ae đã đọc tới đây vì quá kiên nhẫn. Và đừng quên kiểm tra lại mail gateway của mình enable verify Email Authentication nhé (◉ω◉) (◉ω◉) (◉ω◉)

