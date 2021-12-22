---
layout: post
title: DNS nào thì recursive, interactive, forwarder cũ mà mới,
date: 2021-12-21 00:32:20 +0700
description: DNS nào thì recursive, interactive, forwarder cũ mà mới, nhiều khi dùng mà cứ loạn lên dùng mà nhiều ông cứ bị lẫn. Hôm nay tôi tổng hợp tại đây để sau cấu hình DNS thì không bị loạn lên nữa.
img: 2021/12/21/dnsintro.jpg
fig-caption: # Add figcaption (optional)
tags: [Linux]
---

DNS là cái quá cũ rồi, ai cũng hiểu nó làm gì. Tuy nhiên gần đây có tới 2 người hỏi tôi về cấu hình DNS. Nguyên nhân cơ bản là đang lẫn lộn giữa Recursive & Interactive vs Forwarder (Lý thuyết thì ai cũng biết nhưng thực tế nó hay lẫn lộn). Hôm nay tôi viết cái bài này để làm rõ 3 khái niệm này (Theo tôi hiểu ná). Hy vọng các CCEI bạn của tôi xin nhẹ tay tôi chỉ là Sysadmin không phải network admin.

## Nhắc lại về Recersive và Interactive Query

Trước hết do là người Việt nên tôi tạm dịch tạm ra tiếng Việt vậy.

- Recersive -  Đệ quy 
- Interactive - Tương tác 

Sau khi biết nghĩa tiếng việt rồi thì chỉ cần nhìn cái hình này là hiểu và nhớ. Một cái mang tính "đệ quy" và một cái mang tính "Tương tác qua lại".

![DNS query]( {{site.url}}/assets/img/2021/12/21/recersive_interactive.PNG)

Đọc đến đây bạn có tự hỏi là thế hàng ngày máy tính của tôi nó dùng interactive hay recersive query vậy? Câu trả lời đọc tiếp phần sau.

## DNS recursive resolver

DNS server sẽ quyết định việc mình phục vụ client theo kiểu Recersive hay Interactive. Với bind9 thì enable bằng option sau:

```
recursion yes | no;
```

Trong mạng thông thường thì các dns nội bộ mà ta hay sử dụng thường là loại **DNS recursive resolver**. Với loại này quá trình query DNS của máy tính chúng ta là sự kết hợp của cả 2 kiểu query Recersive và Interactive.

![DNS resolver]( {{site.url}}/assets/img/2021/12/21/resolver.PNG)

Tuy nhiên các DNS recursive resolver này cũng có nhược điểm là có nguy bị lợi dụng để tạo ra kiểu tấn công **DNS Amplification Attack**

Vậy google dns có phải là kiểu **DNS recursive resolver** hay không? Thử dùng wireshark để bắt gói tin thì DNS hỏi dâm trí thì tôi thấy.

![DNS dantri]( {{site.url}}/assets/img/2021/12/21/dantri.PNG)

OK hỏi một phát google 8.8.8.8 trả lời luôn không bắt tôi đi hỏi lòng vòng => Vậy google là một **DNS recursive resolver** rồi.

## DNS Forwarder

Tôi là người quản trị mạng. Nếu dùng một **DNS recursive resolver** nghĩa là tôi phải mở port DNS tới ANY DEST. Điều này tôi không hề muốn vì như vậy hay bị mấy ông Security Soi mói. Vậy có cách nào không? Câu trả lời là có sử dụng DNS Forwarder.

DNS cấu hình forward thì không phải là **DNS recursive resolver** vì nó không query interactive tới một loạt con DNS khác để tổng hợp các câu hỏi cho client.

Việc cấu hình forward được thực hiện trên DNS server (tạm gọi là DNS local). Lúc này DNS local sẽ không thực hiện phân giải mà forward các query mà DNS local không có/không biết sang một DNS Forwarder bên ngoài. DNS bên ngoài sẽ là thằng trả lời câu hỏi cho client. DNS nội bộ chỉ làm nhiệm vụ forward không hơn không kém. 

![DNS forwarder]( {{site.url}}/assets/img/2021/12/21/forward.PNG)

Kết quả là ta chỉ cần mở từ DNS nộ bộ tới DNS Forwarder bên ngoài mà thôi không cần phải mở ANY DEST. Vừa lòng mấy ông chưa.

Ext: DNS server nó còn có rule forward theo ACL nữa (VD domain này thì forward đến DNS này xử lý, Domain kia thì forward tới DNS kia xử lý) các ông có thể tìm hiểu nha khi cần.

Thôi tôi đi ngủ,