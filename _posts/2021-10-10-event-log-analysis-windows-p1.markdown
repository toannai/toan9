---
layout: post
title: Event logs analysis 1 - Những thông cơ bản về Event logs,
date: 2021-10-10 00:32:20 +0700
description: Event logs analysis 1 - Những thông cơ bản về Event logs,
img: 2021/10/10/ev_top.png
fig-caption: # Add figcaption (optional)
tags: [Windows, Forensic]
---

Trước đây phần nhiều thời gian tôi dành cho việc làm việc trên các hệ thống linux. Thỉnh thoảng có đá qua làm một chút với Windows. Đang làm với Linux quen sang làm Windows rất là bực. Khó chịu nhất là việc troubleshoot lỗi trên Windows. Info trong log rất nhiều và rối dắm. Đến cái công cụ view log dùng cũng muốn đập cho phát vì tra cứu rất hay treo. Mà thôi bỏ qua nói vậy hết cả ngày, nhiều việc không thích vẫn phải làm cho tốt. Hôm nay rảnh tôi vui vì đọc được tài liệu khá hay nên sẽ dành thời gian để note lại một số thứ liên tới Event logs Windows. Nói thì buồn cười nhưng mà thôi cứ làm vậy. Notes của tôi - tôi note, ai rảnh đọc chơi chơi.

### Log và Event log

Log là cái gì? Nếu vác ra để định nghĩa ra cũng mơ hồ. Chỉ cần mang máng là những thông tin (Message/Event) mà trong quá trình developer viết mã (code) phần mềm ghi ra mà thôi. Log có nhiều mục đích nhưng phổ biến nhất là để hỗ trợ troubleshoot hoặc truyền đạt thông tin gì đó tới người dùng. 

Đặc điểm của các message này là dữ liệu dạng text - không có cấu trúc. Cũng không mấy khi được tuân theo một chuẩn ngon nghẻ cho lắm, mỗi ông ghi một kiểu khá loạn xạ dù cũng nhiều chuẩn được đề xuất. 

Để tránh việc mỗi phần mềm ghi log một kiểu, không có thống nhất - vô tổ chức không đâu vào đâu thông thường trên mỗi hệ điều hành người ta thường build sẵn một service log managment. Service này làm nhiệm vụ quản lý log cho toàn bộ hệ điều hành bao gồm cả log ứng dụng. Việc này ngoài chỗ làm cho việc ghi log ngăn nắp - tránh mỗi ông ghi một kiểu tầm bậy tầm bạ cũng có ý nghĩa làm giảm độ phức tạp khi viết ứng dụng - Ứng dụng không cần code thêm phần log management mà chỉ cần gọi api của hệ điều hành quoẳng log vào đó cho hệ điều hành tự xử lý. Phổ biến trong Linux có syslog/rsyslog, windows thì có Event logging. Lợi ý trực quan nhất là nhờ có log management service mà ta không cần phải quá đau đầu khi phải mò đi tìm log của phần mềm a,b,c nằm ở đâu nữa mà chỉ cần vô một chỗ thôi là có log của toàn bộ rồi (vd: Linux ta quá quen thuộc với /var/log hoặc Windows ta có Event Viewer). Nhưng cũng có nhiều ông khùm khoằm không chịu gọi api ghi log của hệ điều hành mà cố tình ghi log ra chỗ x,y,z theo ý mình - Thật mệt với mấy người này phải không?

Event Viewer: Ứng dụng giúp đọc Event logs trên môi trường Windows. 


### Các thông tin nào có trong Windows Events?

Các thông tin được ghi lại bao gồm:

* Software
* Hardware
* Thông tin system functions
* Security

Mỗi một message log trong Windows họ gọi là một Event. Nhiều Event thì gọi là Event logs/Windows Events (Định nghĩa thật loằng ngoằng phải không?). Khi đọc một event để forensic thông thường ta sẽ cần phải chú ý các thông tin sau:
s

![Event Info]( {{site.url}}/assets/img/2021/10/10/event_info.png)

### Event logs được lưu trữ ở thư mục nào trong hệ điều hành?






