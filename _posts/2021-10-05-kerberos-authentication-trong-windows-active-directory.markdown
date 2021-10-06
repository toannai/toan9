---
layout: post
title: Kerberos authentication trong Windows Active Directory,
date: 2021-10-05 00:32:20 +0700
description: Kerberos authentication trong Windows Active Directory,
img: 2021/10/05/event-4771-kerberos-authentication-illustration.jpg
fig-caption: # Add figcaption (optional)
tags: [Windows]
---


>Thời sinh viên học môn ANM toàn học về mã hóa, secure protocol các loại. Bài tập toàn là tìm điểm yếu để khai thác secure protocol. Nói thật toàn lý thuyết đíu hiểu gì. Cơ mà hôm nay viết về một cái ứng dụng của thứ đíu hiểu ngày đó. 

Kerberos là một secure protocol quan trọng được sử dụng để authen trong Windows. Hiểu được follow của nó sẽ có kha khá tác dụng, lý dải cho nhiều thứ hay ho trong thực tế. Có ích cho cả Sysadmin (để troubleshoot) hay Security Engineer (Để hiểu về cách thức khai thác). OK nào bắt đầu thôi.  

Khi người dùng đăng nhập vào một máy tính Windows trong một Domain (Microsoft nó gọi là Active Directory - AD), password họ nhập vào sẽ được hash (băm) và hash này được sử dụng làm key encrypt timestamp để gửi đến Key-Distribution Center (KDC), KDC này đơn giản là một service nằm trên Domain Controller của AD. Timestamp đã encrypt này sau đó được gửi như một AS-REQ (Authentication Server Request). Khi nhận được AS-REQ này KDC xác minh thông tin đăng nhập của người dùng bằng cách decrypt yêu cầu với hash của password của người dùng lưu trong NTDS.DIT file của AD (File data của AD) và xác minh đảm bảo timestamp yêu cầu này nằm trong giới hạn có thể chấp nhận được. Sau đó, KDC phản hồi bằng AS-REP (Authentication Server Reply).

![as-req-and-reply]({{site.url}}/assets/img/2020/06/23/as-request-and-as-reply.png){:width="600px"}

AS-REP chứa TGT được mã hóa bằng key của KRBTGT (key này chính là hash của password krbtgt user trên AD) cũng như một số dữ liệu khác được mã hóa bằng key của người dùng. Tài khoản KRBTGT là tài khoản được tạo khi promoting DC lần đầu tiên và được sử dụng để xác thực Kerberos. Việc thay đổi mật khẩu tài khoản KRBTGT có những tác động rất nghiêm trọng đến nó, sẽ được đề cập ở phần sau.

Tới đây, người dùng đã được xác thực trong domain, họ vẫn cần quyền truy cập vào các dịch vụ trên máy tính mà họ đang cần đăng nhập. Điều này được thực hiện bằng cách yêu service ticket (TGS) cho một dịch vụ chính thông qua TGS-REQ. Dịch vụ được đại diện thông qua service principal name (SPN) của nó. Có rất nhiều SPN, phần lớn trong số đó có thể được tìm thấy ở đây. Để truy cập vào máy thực, SPN của ‘HOST’ được yêu cầu. HOST là principle chứa tất cả các built-in services nằm trên Windows. 


![as-req-and-reply]({{site.url}}/assets/img/2020/06/23/tgs-req.png){:width="600px"}

Trong TGS có chứa một Privileged Attribute Certificate (PAC). Thông tin chính trong PAC là user và memberships của user đó.


![tac]({{site.url}}/assets/img/2020/06/23/tac.png){:width="600px"}


GroupID là danh sách các dịch vụ mà người dùng đó có quyền truy cập. Để ngăn chặn việc giả mạo, TGS được mã hóa bằng cách sử dụng hash mật khẩu của dịch vụ đích. Trong trường hợp HOST / ComputerName, đây sẽ là hash mật khẩu tài khoản Computer. Lý do hash mật khẩu tài khoản được sử dụng để mã hóa / giải mã ticket là vì đó là thông tin bí mật duy nhất được chia sẻ duy nhất giữa account và  KDC / Domain Controller.


![tgs-req]({{site.url}}/assets/img/2020/06/23/tgs-reply.png){:width="600px"}


Sau khi nhận được TGS, thông qua TGS-REP, dịch vụ đích sẽ giải mã ticket bằng hash mật khẩu của nó (trong trường hợp này, đó là hash mật khẩu của Computer password) và tìm trong PAC của TGS để xem liệu các SID nhóm thích hợp có hiện diện hay không, xác định quyền truy cập. Điểm khác biệt chính với phiếu dịch vụ là KDC thực hiện xác thực (TGT) trong đó dịch vụ thực hiện ủy quyền (PAC trong TGS). Sau khi được xác nhận, người dùng được phép truy cập gốc của dịch vụ HOST và người dùng sau đó được đăng nhập vào máy tính của họ.

Toàn bộ quá trình đăng nhập này có thể được xem trong Wireshark khi ghi lại quá trình đăng nhập cho người dùng khác.

![wireshark]({{site.url}}/assets/img/2020/06/23/wireshark.png){:width="600px"}


Tham khảo

https://posts.specterops.io/kerberosity-killed-the-domain-an-offensive-kerberos-overview-eb04b1402c61

