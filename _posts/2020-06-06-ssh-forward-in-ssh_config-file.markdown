---
layout: post
title: Tất tần tật về SSH tunneling trong linux,
date: 2020-06-06 00:32:20 +0700
description: # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Linux]
---

Làm ở công ty B firewall tầng tầng lớp lớp nên kỹ năng sử dụng ssh tunnel gần như một kỹ năng ngoài luồng bắt buộc nếu như không muốn mất thời gian làm CR (Connection Request) mở kết nối. Trên Windows thì mấy tool như Xshell, putty hỗ trợ khá mạnh - Có GUI tiện lợi nhưng trên linux thì tool toy GUI ít hơn nên mỗi lần muốn tunnel phải gõ lệnh chay khá là mệt mỏi vì cứ phải nhớ option và vị trí args sao cho phù hợp. Mấy nay thấy một cách khá hay là cấu hình trong file `/etc/ssh_config` hoặc `~/.ssh/config`. Nay note lại đây khi cần sử dụng luôn đỡ phải search.

PS:  Bài viết không đề cập tới các cấu hình ở phía open ssh-server. Nói chung nếu không sửa gì sshd_config thì mặc định là các tính năng bên dưới đã được enable ở phía server. Dĩ nhiên bạn nào cố tình tắt đi thì phải biết đường mà bật lên thì mấy cái bên dưới mới chạy được rồi.

#### Cấu hình local forward port

Để cấu hình localforward port chỉ cần thêm vào đoạn cấu hình sau

```
Host tunnel
    HostName database.example.com
    LocalForward 9906 127.0.0.1:3306 #Thêm vào đoạn này vào snippet Host
    ...
    ...
```

Lúc này khi ta ssh vào host tunel (gõ ```ssh tunnel```), trên máy chạy ssh client sẽ binding port 9906 là port forward cho port 3306 trên server database.example.com. Hay nói cách khác khi ta connect vào port 9906 máy ssh client (máy mà ta gõ lệnh ```ssh tunnel```) thì coi như ta đang connect vào port 3306 trên host database.example.com.

Extends: Muốn connect tới địa chỉ khác không chỉ trên máy database.example.com chỉ cần thay địa chỉ và port của host đó vào chỗ 127.0.0.1:3306 mà thôi. Dĩ nhiên điều kiện là database.example.com phải thông tới địa chỉ và port đó.

#### Cấu hình remote forward port

Để cấu hình remote forward port ta thêm vào đoạn cấu hình sau

```
Host tunnel
    HostName database.example.com
    RemoteForward  8080 127.0.0.1:8081 #Thêm vào đoạn này vào snippet Host
    ...
    ...
```

Lúc này khi ssh vào host tunnel (gõ ```ssh tunnel```), trên máy database.example.com sẽ binding port 8080 là port remote forward cho port 8081 trên máy ssh client. Hay nói cách khác khi trên máy database.example.com connect vào port 8080 thì như là đang connect vào port 8081 trên máy ssh client ( máy mà ta thực hiện gõ lệnh ```ssh tunnel```).

Extends: Muốn connect tới đị chỉ khác không chỉ là host ssh client ta có thể thay địa chỉ 127.0.0.1:8081 bằng host và port mà ta cần kết nối. Dĩ nhiên là host chạy ssh client phải connect được tới địa chỉ và port đó.

#### Cấu hình host ssh-server như một SOCK proxy

Open ssh-server vốn nó đã có tính năng của một SOCK proxy server. Để cấu hình ta chỉ việc thêm vào đoạn sau trong file cấu hình của ssh client.

```
Host tunnel
    Hostname example.com
    DynamicForward localhost:5555 #Thêm vào đoạn này vào snippet Host
```
Lúc này khi ssh vào host tunnel (gõ ```ssh tunnel```), trên máy ssh client (máy chạy lệnh ```ssh tunnel```) sẽ binding port 5555 như là port SOCK proxy giúp ta kết nối vào sock proxy server chạy trên ssh-server. Lúc này chỉ cần sử dụng các phần mềm hỗ trợ SOCK5 là có thể kết nối vào port 5555 để truy cập vào SOCK proxy server này.

Extends: 

**Với điều kiện là vẫn giữ nguyên phiên ssh phía trên (Phiên mà gõ ```ssh tunnel```)**.

Đặc biệt ssh-client cũng hỗ trợ forward dữ liệu qua SOCK, để cấu hình ta chỉ cần thêm vào ssh client đoạn sau.

```
Host sftpserver
   HostKeyAlias sftp.example.com
   ProxyCommand /usr/bin/nc -x 127.0.0.1:5555 %h %p #Thêm vào đoạn này vào snippet Host
```
Như vậy là khi ta ssh vào sftpserver thì ssh-client sẽ tự động forward kết nối của chúng ta qua SOCK proxy server tại 127.0.0.1:5555.


#### Góc linh hoạt,

Ta cũng có thể sử dụng kết hợp các cách thức trên một cách linh hoạt theo ý đồ của chúng ta. He, rõ ràng là open-ssh có thể giúp chúng ta làm cơ số các việc hay ho các bạn nhỉ.

