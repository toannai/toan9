---
layout: post
title: Kerberos authentication trong Windows Active Directory,
date: 2021-10-05 00:32:20 +0700
description: Kerberos authentication trong Windows Active Directory,
img: 2021/10/05/event-4771-kerberos-authentication-illustration.jpg
fig-caption: # Add figcaption (optional)
tags: [Windows]
---


>Thời sinh viên học môn ANM toàn học về mã hóa, secure protocol các loại. Bài tập toàn là tìm điểm yếu để khai thác secure protocol. Nói thật toàn lý thuyết nghe chả hiểu gì. Cơ mà hôm nay viết về một cái ứng dụng của thứ chả hiểu ngày đó. 

Kerberos là một secure protocol quan trọng được sử dụng để authen trong Windows. Hiểu được flow hoạt động của nó sẽ có kha khá tác dụng, lý dải cho nhiều thứ hay ho trong thực tế. Có ích cho cả Sysadmin (để troubleshoot lỗi) hay Security Engineer (Để hiểu về nguyên lý một số kiểu khai thác). OK nào bắt đầu thôi.  

Để dễ hiểu tôi sẽ mô tả quá trình đăng nhập của một người dùng vào một service là một Computer chạy Windows ở trong domain sử dụng keberos authen (Các services khác tương tự thôi nha).

Khi người dùng đăng nhập vào một Computer chạy Windows trong một Domain (Microsoft nó gọi là Active Directory - AD), password do họ nhập vào sẽ được hash (băm) và hash này được sử dụng làm key encrypt timestamp để gửi đến Key-Distribution Center (KDC), KDC này đơn giản là một service nằm trên Domain Controller của AD. Timestamp đã encrypt này sau đó được gửi như một AS-REQ message (Authentication Server Request). Khi nhận được AS-REQ này KDC xác minh thông tin đăng nhập của người dùng bằng cách decrypt yêu cầu key giải mã là hash của password của người dùng lưu trong NTDS.DIT file của DC (File data của AD) và xác minh đảm bảo timestamp yêu cầu này nằm trong giới hạn có thể chấp nhận được. Sau đó, KDC phản hồi bằng AS-REP message (Authentication Server Reply).


![as-req-and-reply]({{site.url}}/assets/img/2021/10/05/as-request-and-as-reply.png){:width="600px"}


AS-REP chứa TGT được mã hóa bằng key của KRBTGT (key này chính là hash của password krbtgt user trên AD) cũng như một số dữ liệu khác được mã hóa bằng key là hash password của người dùng. Tài khoản KRBTGT là tài khoản được tạo khi promoting DC lần đầu tiên và password của nó được sinh ngẫu nhiên. Thật ra cũng không cần quan tâm password của nó lắm chỉ cần quan tâm tới hash password của nó thôi vì nó là thứ được sử dụng để xác thực Kerberos. 

Tới đây, người dùng đã được xác thực trong domain, tuy nhiên họ vẫn cần quyền truy cập vào các dịch vụ nhất định (chứ không phải chỉ là xác thực mình với domain mà thôi), cụ thể ở đây là Computer mà họ đang cần đăng nhập. Điều này được thực hiện bằng cách người dùng tiếp tục yêu cầu thêm một service ticket (TGS) thông qua TGS-REQ. Dịch vụ (kể cả là một Computer) được đại diện thông qua service principal name (SPN). Để truy cập vào Computer mong muốn, SPN yêu cầu ở đây sẽ là của Computer được yêu cầu. HOST là principle chứa tất cả các built-in services nằm trên Windows. 


![as-req-and-reply]({{site.url}}/assets/img/2021/10/05/tgs-req.png){:width="600px"}


Trong TGS có chứa một Privileged Attribute Certificate (PAC). Thông tin chính trong PAC là user và memberships của user đó.


![tac]({{site.url}}/assets/img/2021/10/05/tac.png){:width="600px"}


GroupID là danh sách các dịch vụ mà người dùng đó có quyền truy cập (Để truy cập được Computer mong muốn thì Computer SPN ID phải nằm trong ds này). Để ngăn chặn việc giả mạo, TGS được mã hóa bằng cách sử dụng hash password của dịch vụ đích cần truy cập. Trong trường hợp ComputerName, đây sẽ là hash password của Computer Account (Trên AD Computer cũng là một account và có password và AD quản lý password này và nó được định kỳ thay đổi. Lỡ không may bị miss match Computer password gây ra lỗi **Relation ship** huyền thoại đóa). Lý do hash password service được sử dụng để encrypt / decrypt GTS ticket là vì đó là thông tin bí mật duy nhất được chia sẻ duy nhất giữa service và KDC / Domain Controller.


![tgs-req]({{site.url}}/assets/img/2021/10/05/tgs-reply.png){:width="600px"}


Sau khi nhận được TGS, thông qua TGS-REP, service đích sẽ giải mã TGS ticket bằng hash password của nó (trong trường hợp này, đó là hash password của Computer) và tìm trong PAC của TGS để xem liệu các SID thích hợp có hiện diện hay không nhằm xác định quyền truy cập. Điểm chính cần chú ý ở đây là KDC thực hiện authen (TGT) và service sẽ thực hiện autho(PAC trong TGS). Sau khi được xác nhận, người dùng được phép truy cập gốc của HOST principal và người dùng sau đó được đăng nhập vào máy tính của họ.

Toàn bộ quá trình đăng nhập này có thể được xem trong Wireshark khi ghi lại quá trình đăng nhập cho người dùng khác. (Nhưng tôi lười lắm không caputure đâu)

![wireshark]({{site.url}}/assets/img/2021/10/05/wireshark.png){:width="600px"}


Tham khảo

https://posts.specterops.io/kerberosity-killed-the-domain-an-offensive-kerberos-overview-eb04b1402c61

