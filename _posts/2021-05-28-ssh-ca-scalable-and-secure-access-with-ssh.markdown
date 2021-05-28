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

Từ phiên bản 5.4 (released 2010-03-08) OpenSSH đã hỗ trợ OpenSSH Certificates (SSH-CA). Cơ chế của SSH-CA là dựa trên nền tảng chữ ký số. Trong mô hình này ta sẽ có một sign server - Server này sẽ ký lên cho ta các certificate (Chứng chỉ). 
