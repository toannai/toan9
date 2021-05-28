---
layout: post
title: SSH CA - Scalable and secure access with SSH,
date: 2021-05-28 00:32:20 +0700
description: SSH CA - Scalable and secure access with SSH
img: 2021/05/28/20210528_ssh_intro.jpg
fig-caption: # Add figcaption (optional)
tags: [Linux]
---

Tôi sẽ không cần nói nhiều về SSH nữa vì nó là giao thức quá quen thuộc với mọi người rồi. Bất kể bạn là ai - sysadmin/dev thậm trí là tester, nếu đã từng làm việc với Linux thì chắc chắn đều đã từng sử dụng SSH. Trước hết phải nói là SSH quá ngon, dùng trên Open-SSH quá ổn định. Tuy nhiên trong thời gian làm việc với SSH/OpenSSH team của tôi cũng gặp một số vấn đề phát sinh khá "nan giải" cần giải quyết. Đó cũng là lý do tôi viết bài này. 

Chúng tôi có hàng vài trăm server Linux nằm ở nhiều sites khách hàng khác nhau. Thỉnh thoảng các server này lại cần cấp quyền cho đội dự án ssh vào trong một thời gian ngắn (1 vài tiếng đến 1 vài ngày) để fix bug hoặc update. Người triển khai và người vận hành thuộc 2 team khác nhau. Team triển khai deploy xong bàn giao lại cho team vận hành. Ngoài ra, công ty công nghệ nên người cũ hôm nay mai có khi nghỉ việc, lại có người mới vào - tính ra cũng thường xảy ra. Bài toán quản lý SSH credential thật sự là đau đầu. Lịch sử để lại là username/password auth rõ ràng là không ổn - vừa kém an toàn lại chưa giải quyết được vấn đề nêu ra. OpenSSH có hỗ trợ private-public key cái này an toàn hơn chút nhưng cũng vẫn chưa giải quyết được các vấn đề vừa nêu. Chúng tôi cũng tìm hiểu một số cách quản lý khác như sử dụng LDAP, Kerberos thậm trí còn phá triển hẳn một project KeyServer (xác thực password + otp) để quản lý credential tập trung, các phương án này nhìn có vẻ ổn nhưng lại gặp vấn đề là các hệ thống nằm phân tán, server nhiều như lợn con việc maintain thường xuyên để đảm bảo hệ thống key management/agent ổn định rất khó, nhiều khi lỗi xảy ra (vd server sai giờ một chút) thì không thể vào được server. Tìm kiếm một hồi chúng tôi **tạm** đã tìm được lời giải bằng việc sử dụng SSH Certificates (SSH - CA). Dĩ nhiên đây không phải là một câu trả lời hoàn hảo nhưng chắc là tốt nhất trong lúc này.

Khi lựa chọn một phương án để giải quyết bài toán của mình, chúng tôi quan tâm đến 3 vấn đề: Authentication + Authorization + Accounting. Quá trình Authentication xác nhận user có quyền truy cập hệ thống, Authoriation - Cấp quyền user được phép thực hiện và Accounting - Loging toàn bộ quá trình từ khi người dùng đăng nhập đến khi đăng xuất khỏi hệ thống để cần để điều tra sau này. Tôi nêu điều này ra ở đây là để người đọc bài viết chú ý hơn vào các yếu tố này, có phương án chuẩn bị cho mình bất kể cả dùng SSH-CA hay không.

Từ phiên bản 5.4 (released 2010-03-08) OpenSSH đã hỗ trợ OpenSSH Certificates (SSH-CA). Cơ chế của SSH-CA là dựa trên nền tảng chữ ký số. Trong mô hình này ta sẽ có một Sign server tập trung. Người dùng (user) sinh ra cặp public-private key của mình. Mỗi lần cần đăng nhập server, user sẽ gửi public key lên Sign Server này để ký (Sign server ký bằng private key của mình). Ký xong, Sign server sẽ trả lại cho user một Certificate (Chứng chỉ). Trong chứng chỉ cũng chứa các thông tin thời gian hợp lệ sử dụng Certificate, danh sách tên hệ thống được vào - Các thông tin này sẽ quyết định việc user sẽ được phép ssh vào những server nào và trong khoảng thời gian nào. Đến đây khi người dùng ssh lên Server đích sẽ cung cấp private key của  mình và Ceritificate của mình đã được ký. Server đích do có CA.pub (public key của CA) của Sign Server (được cài từ trước) sẽ verify tính hợp của Ceritificate, bao gồm cả kiểm tra thời gian được phép login hệ thống cũng như tên hệ thống được phép đăng nhập (tên hệ thống sẽ được cấu hình trong server). Nếu toàn bộ thông tin thỏa mãn đang nhập thành công, trái lại đăng nhâp thất bại. Quá trình này mô tả bằng hình vẽ dưới.

![ssh cert flows]( {{site.url}}/assets/img/2021/05/28/20210528_ssh_cert_flow.jpg){:width="600px"}

 
