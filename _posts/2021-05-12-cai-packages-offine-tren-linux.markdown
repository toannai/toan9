---
layout: post
title: Cài packages trên linux khi không có mạng internet,
date: 2021-05-12 00:32:20 +0700
description: Cài packages trên linux khi không có mạng internet 
img: 2021/05/12/nointernet.jpg
fig-caption: # Add figcaption (optional)
tags: [Linux]
---


Gần đây có 2 người bạn hỏi tôi chung một câu hỏi là "Làm sao để cài gói trong linux khi server không có mạng?". 

![No internet up]( {{site.url}}/assets/img/2021/05/12/nointernet.jpg)

Một câu hỏi khi có nhiều hơn 1 người hỏi là đáng để viết một cái note nho nhỏ rồi, dù thật ra nó cũng chả có gì to tát. Qua kinh nghiệm bản thân tôi cũng có đề xuất ra một số cách như sau:

#### Cách 1: Quay tay 

**Quay tay level 1**: Tìm gói trên internet theo distro và version của distro để down load về và copy vào máy cần cài. Tùy thuộc vào distro mà dùng rpm hay là dpkg để cài gói .rpm hoặc .deb vừa copy vào. Ok cách này nếu mà các gói cần cài ít dependencies thì dùng cũng ổn. Cơ mà nếu mà gói cần cài mà có nhiều dependencies thì bách nhục. Down gói này nó lại cần gói kia =)) Quay tay quá nhiều mắc mệt. Nhục.

#### Cách 2: Lại quay tay

**Quay tay level 2**: Chuẩn bị một máy ảo cùng distro cùng version với máy cần cài offine. Search google thần trưởng để tìm mấy plugin/command download các gói cần cài kèm dependencies. Google tạm tôi thấy nói thế này.
* RPM: https://ostechnix.com/download-rpm-package-dependencies-centos/
* DEB: https://stackoverflow.com/questions/13756800/how-to-download-all-dependencies-and-packages-to-directory

Dĩ nhiên tiếp theo vẫn là copy các gói download được vào máy cần cài và dùng rpm hoặc dpkg để cài các gói đó.
    
*Cách này nói chung cũng okie nhưng cũng có một số lần bạn tôi tâm sự là xài nó không có được vẫn thiếu gói - Chắc do ông hướng dẫn chưa có đủ cái "Tâm" hoặc chưa đủ "Tầm" hoặc là do bạn tôi đen thôi*

#### Cách 3: Quay tay tiếp

**Quay tay level 3**: Vẫn là chuẩn bị một máy ảo cùng distro và cùng version. Sau đó enable mode cache với apt hoặc yum. Với các loại package manage thông thường trước khi cài gói có thể set option để nó download các gói về client cache trước khi cài. Và sau đó ta vào thư mục cache để copy các file cài về.
* RPM: Bật cache bằng option ```keepcache=1``` và thư mục lưu các file packages cache (file .rpm) là tham số ```cachedir=xxx``` trong file /etc/yum.conf
* DEB: Cache mặc định được bật và đường dẫn lưu các file cache (file .deb) là ```/var/cache/apt/archies/```

Biết thư mục lấy các file cache này ta chỉ cần enable cache packages sau đó cài các gói cần cài lên máy ảo. Bước tiếp theo là chui vào thư mục cache copy lấy các file .deb hoặc .rpm copy lên máy cần cài để cài thôi.

Cách này thì đảm bảo 100% là thành công. Trừ khi hôm đấy ngáo và copy nhầm :)

#### Cách 4: Sử dụng local tunnel qua ssh

Có kênh ssh và máy ssh client vẫn vào được mạng. OK, như cách vẫn làm của ae công ty tôi là dựng 1 cái proxy trên máy client (Có thể dùng burpsuite) rồi local tunnel vào qua SSH để expose proxy này để cài gói sử dụng yum và apt bình thường như không có gì xảy ra.

Đại loại là tôi có vài cách như vậy chia sẻ với mọi người,


