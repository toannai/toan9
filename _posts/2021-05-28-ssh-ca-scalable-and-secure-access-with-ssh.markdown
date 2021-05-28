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

## Problems

Chúng tôi có hàng vài trăm server Linux nằm ở nhiều sites khách hàng khác nhau. Thỉnh thoảng các server này lại cần cấp quyền cho đội dự án ssh vào một nhóm các server nhất định và trong một thời gian ngắn (1 vài tiếng đến 1 vài ngày) để fix bug hoặc update. Người triển khai và người vận hành thuộc 2 team khác nhau. Team triển khai deploy xong bàn giao lại cho team vận hành. Ngoài ra, cũng như các công ty công nghệ khác người cũ hôm nay làm mai có khi nghỉ việc, lại có người mới vào - tính ra cũng thường xảy ra. Bài toán quản lý SSH credential thật sự là đau đầu. 

Lịch sử để lại là authen sử dụng username + password rõ ràng là không ổn - vừa kém an toàn và chưa giải quyết được vấn đề nêu ra. OpenSSH có hỗ trợ private-public key authen, phương pháp này an toàn hơn chút nhưng cũng vẫn chưa giải quyết được các vấn đề phía trên. Chúng tôi cũng có tìm hiểu một số cách quản lý khác như sử dụng LDAP, Kerberos thậm trí còn phá triển hẳn một project KeyServer (xác thực password + otp) để quản lý credential tập trung, các phương án này nhìn có vẻ ổn nhưng lại gặp vấn đề là các hệ thống nằm phân tán, server nhiều như lợn con việc maintain thường xuyên để đảm bảo hệ thống key management/agent ổn định rất khó, nhiều khi lỗi xảy ra (vd server sai giờ một chút) thì không thể vào được server. Tìm kiếm một hồi chúng tôi **tạm** đã tìm được lời giải bằng việc sử dụng SSH Certificates (SSH - CA). Dĩ nhiên đây không phải là một câu trả lời hoàn hảo nhưng chắc là tốt nhất trong lúc này.

## Analysting - Solving problems

Khi lựa chọn một phương án để giải quyết bài toán đặt ra, chúng tôi quan tâm đến 3 yếu tố (AAA): Authentication + Authorization + Accounting. Mỗi yếu tố giải quyết một nhiệm vụ riêng biệt. Quá trình Authentication xác nhận user có quyền truy cập hệ thống không, Authorization - Cấp quyền user được phép thực hiện và Accounting - Loging toàn bộ sự kiện trên hệ thống suốt quá trình từ khi người dùng đăng nhập đến khi đăng xuất khỏi hệ thống để cần để điều tra sau này. Tôi nêu điều này ra ở đây là để người đọc bài viết chú ý hơn vào các yếu tố này, có phương án chuẩn bị cho mình bất kể cả dùng SSH-CA hay không.

Từ phiên bản 5.4 (released 2010-03-08) OpenSSH đã hỗ trợ OpenSSH Certificates (SSH-CA). Cơ chế của SSH-CA là dựa trên nền tảng chữ ký số. Quá trình này mô tả bằng hình vẽ sau:

![ssh cert flows]( {{site.url}}/assets/img/2021/05/28/20210528_ssh_cert_flow.jpg){:width="700px"}

Trong mô hình này ta sẽ có một Sign server tập trung (CA). Sign server này có cặp khóa private-public của riêng mình và do quản trị viên (admin) quản lý. Người dùng thường (user) cũng sở hữu cặp private - public key của riêng mình. Mỗi lần cần đăng nhập SERVER, user sẽ gửi public key của mình lên cho admin kèm yêu cầu thời gian và một nhóm các server cùng chức năng (role server) cần truy cập. Admin sẽ sử dụng  Sign Server để ký lên public key của user tạo ra một chứng chỉ (Certificate) cho request đó. Trong Certificate cũng chứa các thông tin thời gian hợp lệ sử dụng Certificate, role server được vào - Các thông tin này sẽ quyết định việc user sẽ được phép ssh vào những server có role nào và trong khoảng thời gian nào. Đến đây khi người dùng muốn ssh lên SERVER đích thì sẽ cung cấp private key và Ceritificate của mình đã được ký. SERVER đích do có public key (CA.pub) của Sign Server được cài từ trước (Quản trị viên cài lên SERVER) sẽ dùng nó để verify tính hợp của Ceritificate, bao gồm cả việc kiểm tra thời gian được phép login, role server được phép đăng nhập (Việc kiểm tra role dựa vào so sánh role cấu hình sẵn trên SERVER và role server trong ceritificate). Nếu toàn bộ thông tin thỏa mãn thì việc nhập thành công, trái lại đăng nhâp thất bại. 

Xét về 3 khía cạnh AAA (mô tả phía trên) lựa chọn phương án đăng nhập này cơ bản thỏa mãn: Authentication - Chỉ user được sign public key mới có thể đăng nhập vào SERVER đích. Authorization - User chỉ được phép ssh vào nhóm các servers cho phép trong certificate. Auditing - Mỗi user đăng nhập với 1 cerificate nhất định OpenSSH server sẽ log lại với một id khác nhau giúp ta phân biệt giữa các user, các log này được chúng tôi đẩy lên hệ thống log tập trung (Hệ thống SIEM của chúng tôi). 

Các máy PC/Laptop của user rất khó để kiểm soát do vậy để tránh việc mất private và certificate chúng tôi cũng triển khai các JUMP server trung gian. User sẽ không thể ssh trực tiếp từ máy cá nhân của mình mà sẽ ssh vào JUMP server, từ JUMP server sẽ SSH vào các server khác. Mỗi user sẽ được cấp một user riêng trên server này. Đảm bảo có 2 Factor Authen đồng cũng bổ sung việc logging hành vi user thực hiện trên đây. JUMP server luôn được bảo vệ với chế độ đặc biệt. 

## Simple demo
Phần này tôi sẽ mô tả chi tiết cách làm ở mức đơn giản nhất. Mô hình thực tế có thể phức tạp hơn ở phần Sign Server.

### Bước 1: Tạo CA Server và cấu hình public key của CA lên SERER đích
* Sau khi SSH vào Sign Server việc đầu tiên cần thực hiện là tạo một CA. Việc này đơn giản chỉ là sinh ra một cặp private - public key trên Sign Server. 

```
# mkdir /root/ca
# cd /root/ca
# ssh-keygen -C CA -f ca
```
Lệnh cuối sử dụng để sinh cặp private - public key sẽ hỏi passpharse để bảo vệ keys, bạn có thể đặt hoặc để rỗng. Tôi thì thích nói không nên để rỗng. Kết quả ta sẽ sinh ra hai cặp keys như sau:

```
# ls -l
total 24
drwx------  2 root root 4096 May 28 11:52 .
drwx------ 13 root root 4096 May 28 11:50 ..
-rw-------  1 root root 2590 May 28 11:44 ca
-rw-------  1 root root  556 May 28 11:44 ca.pub
```
* SSH vào SERVER đích (Đoạn sau của bước 1 này thực hiện trên SERVER đích), ta tạo file ``/etc/ssh/ca.pub`` với nội dung copy từ nội dung của file ca.pub trên Sign Server. Sau đó change lại mode cho file này thành 0644

```
# chmod 0644 /etc/ssh/ca.pub
```
Thêm vào file ``/etc/ssh/sshd_config`` đoạn cấu hình sau

```
TrustedUserCAKeys /etc/ssh/ca.pub
```

Không quên restart lại ssh-server service để apply cấu hình mới
```
# systemctl restart ssh
```

Thế là hoàn thành bước 1.

### Bước 2: Tạo private - public key trên Client
Khi đã SSH vào server client, không quá khó để tạo cho mình một cặp public - private key bằng công cụ ssh-keygen.

```
$ ssh-keygen -t ecdsa
```
Tương tự tôi lại để pass pharse là rỗng để khỏi bị hỏi nhiều gõ mỏi tay, còn bạn để là gì tùy bạn tôi cũng không quan tâm lắm. Kết quả vẫn sinh ra 2 file private - public trong thư mục ``~/.ssh``

```
$ ls -l ~/.ssh 
total 8
-rw------- 1 toannn toannn 505 May 28 12:23 id_ecdsa
-rw-r--r-- 1 toannn toannn 173 May 28 12:23 id_ecdsa.pub
```

### Bước 3: Ký public key của Client 
Copy public key id_ecdsa.pub của client vừa sinh ở bước 3 vào thư mục /root/ca. Chạy lệnh sau để thực hiện ký public key

```
ssh-keygen -s ca -I mfdutra -n root -V +1w -z 1 id_ecdsa.pub
``` 
Giải thích qua một chút các args của lệnh trên

```
-I mfdutra: Đặt ID của Certificate là mfdutra. (ID này dùng để phân biệt các phiên ssh là của ai)

-n root: Nhận dạng principals (user hoặc host names) trong Certificate. Có thể là một hoặc một vài giá trị cách nhau bằng dấu phẩy. Trường này sẽ gặp lại ở phần sau.

-V +1w: Thiết lập thời gian hợp lệ sử dụng của Certificate là +1w - 1 tuần

-z 1: Xác định serial number được nhúng vào certificate để phân biệt certificate này với các certificate khác được sinh ra từ cùng một CA. Nếu là một số dương (+)a mỗi lần sinh certificate sẽ tăng thêm a. Giá trị mặc định là 0.
```

Kết thúc bước này thư mục /root/ca sẽ có thêm file id_ecdsa-cert.pub 

```
# ls -l 
total 16
-rw------- 1 root root 2590 May 28 11:44 ca
-rw------- 1 root root  556 May 28 11:44 ca.pub
-rw------- 1 root root 1616 May 28 11:52 id_ecdsa-cert.pub
-rw------- 1 root root  171 May 28 11:52 id_ecdsa.pub
```

Dùng tool ssh-keygen để đọc file id_ecdsa-cert.pub ta sẽ gặp lại các options sử dụng khi sign file id_ecdsa.pub

```
# ssh-keygen -Lf id_ecdsa-cert.pub 
id_ecdsa-cert.pub:
        Type: ecdsa-sha2-nistp256-cert-v01@openssh.com user certificate
        Public key: ECDSA-CERT SHA256:hHjkj4NCc9mE23dP/Y3bP/EXCTezNFjj+DXzvLNOR/8
        Signing CA: RSA SHA256:nmRh52BP64InNKRSgEkrpTLV9jJAHmTjblhwPR+hIBc (using rsa-sha2-512)
        Key ID: "mfdutra"
        Serial: 1
        Valid: from 2021-05-28T11:51:00 to 2021-06-04T11:52:27
        Principals: 
                root
        Critical Options: (none)
        Extensions: 
                permit-X11-forwarding
                permit-agent-forwarding
                permit-port-forwarding
                permit-pty
                permit-user-rc                                                                                                                                
```

### Bước 4: Sử dụng file vừa ký ở bước 3 để ssh từ Client lên 

Copy file id_ecdsa-cert.pub vừa ký ở bước 3 vào thư mục ``~/.ssh`` trở lại máy client. Chú ý đúng user đã sinh cặp private - public key bước 2.

```
# cd ~/.ssh                                                                                                                                                       
# ls -l 
total 16
-rw------- 1 root root  505 May 28 11:50 id_ecdsa
-rw------- 1 root root 1616 May 28 11:55 id_ecdsa-cert.pub
-rw------- 1 root root  171 May 28 11:50 id_ecdsa.pub
-rw-r--r-- 1 root root  222 May 28 11:55 known_hosts
```

Sau đó ta thực hiện ssh như bình thường. Kết quả đương nhiên là thành công rồi.

```
$ ssh root@192.168.183.138
The authenticity of host '192.168.183.138 (192.168.183.138)' can't be established.
ECDSA key fingerprint is SHA256:6RE8bsdI/geR0sfVV1AcNvvdhiTf49hMSlAEqjpHK6I.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.183.138' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-143-generic x86_64)
Last login: Fri May 28 15:55:47 2021 from 192.168.183.136
```

Kiểm tra log đăng nhập trên server ở file /var/log/auth.log (Tùy distro mà vị trí file có thể khác) ta thấy nội dung như sau:

```
May 28 17:07:23 svr sshd[28040]: Accepted publickey for root from 192.168.183.136 port 57560 ssh2: ECDSA-CERT ID mfdutra (serial 1) CA RSA SHA256:nmRh52BP64InNKRSgEkrpTLV9jJAHmTjblhwPR+hIBc
May 28 17:07:23 svr sshd[28040]: pam_unix(sshd:session): session opened for user root by (uid=0)
May 28 17:07:23 svr systemd: pam_unix(systemd-user:session): session opened for user root by (uid=0)
May 28 17:07:24 svr systemd-logind[992]: New session 11 of user root.
```

Hãy chú ý đoạn **ID mfdutra (serial 1) CA**. Rõ ràng là ta đã đăng nhập thành công với ID mfduatra vừa tạo ở trên. Thực tế ta có thể sử dụng ID này để phân biệt việc đăng nhập với các certificate khác nhau.

Ok vậy là bước đầu ta đã thành công với việc đăng nhập ssh sử dụng certificate (SSH-CA).

### Bước 3': Ký public key của Client với Principals 

Mai viết tiếp he ..


**Tham khảo**

[1] [engineering.fb.com](https://engineering.fb.com/2016/09/12/security/scalable-and-secure-access-with-ssh/)
