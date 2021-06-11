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

![fucker]( {{site.url}}/assets/img/2021/06/11/210611_fake_mail.png){:width="600px"}

Nguyên nhân chính là do yếu tố lịch sử, giao thức SMTP ban đầu quá đơn giản, kém bảo mật. Không có các cơ chế xác thực, mã hóa. Mãi sau này phải người ta phải nghĩ ra rất nhiều các cơ chế mở rộng (extends) để khắc phục dần các nhược điểm này. Tuy nhiên do tính không đồng nhất mọi sự bổ sung sau này cho ta cảm giác nó chỉ là sự chắp vá.

>Để biết SMTP ban đầu nó thế nào ae hãy thử cài postfix core mà coi. Thật sự chả có cái gì luôn. Không xác thực, không mã hóa gì, ..., không gì hết chỉ có gửi và nhận. Sau này muốn có thêm các tính năng ta sẽ phải cài thêm các module/add-on đảm nhiệm các extension mở rộng này OpenDKIM, dkim-milter, ... [Xem list plugin tại đây](http://www.postfix.org/addon.html#auth) 




