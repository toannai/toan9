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

![Event Info]( {{site.url}}/assets/img/2021/10/10/event_info.png)

Windows Event logs được chia thành các kiểu sau:

* Security: Ghi lại access control và các security setting infomation. Các thông tin được ghi trong log này được thiết lập bằng Group Policy. Ví dụ: Sự kiện Success/Failed logon, access file/folder, ... .Nhìn trung thì đây là loại log quan trọng nhất khi forensic sự cố security. Các loại log còn lại bên dưới chủ yếu sử dụng để troubleshoot lỗi trong quá trình vận hành hệ thống/service.

* System: Log liên quan tới các service, system component, Driver, resource của windows. ví dụ: Service started/stoped, reboot/shutdown, ... 

* Application: Các software event. Ví dụ: SQL server failed, ...

* Custom: Các loại custom application log. Ví dụ: Directory Service, DNS Server, FRS, ... 

Với mỗi kiểu lại chia thành các level khác nhau mô tả mức độ quan trọng của thông tin. Các level này bao gồm:

####  Level ở mức độ Administration (Quản trị):

* Error: Các vấn đề nghiêm trọng. Có thể gây mất mát dữ liệu hoặc lỗi tính năng. VD: Service không start được
* Warning: Ít nghiêm trọng hơn. Lỗi không xảy ra ngay mà tiềm ẩn xảy ra trong tương lai. VD: Disk sắp full
* Infomation: Các thông tin không thêm. VD: Service đã được start

####  Level ở mức độ Security:

* Success Audit: Audited Security Event thành công. Ví dụ: Logon thành công
* Failed Audit: Audited security Event không thành công. Ví dụ: Không thể kết nối tới ổ shared

### Event logs được lưu trữ ở thư mục nào trong hệ điều hành?

Với Win NT/XP/2k3 lưu tại `%systemroot%\System32\config` với đuôi mở rộng .evt

Với Win 7/8/10/2k8/2k12/2k16 lưu tại `%systemroot%\System32\winevt\logs` với đuôi mở rộng .evtx

Nếu như linux lưu log dưới dạng text file thì Windows lưu các file này dưới dạng binary và không thể đọc được bằng text editor thông thường. 

Dĩ nhiên là ta cũng có thể thay đổi vị trí lưu log bằng việc thay đổi key trong registry tại đường dẫn sau:

`HKLM\SYSTEM\CurentControlSet\Services\EventLog\Application`
`HKLM\SYSTEM\CurentControlSet\Services\EventLog\System`
`HKLM\SYSTEM\CurentControlSet\Services\EventLog\Security`


### Các điểm cần tập trung khi forensic sử dụng Event Logs?

Trong những lần đi xử lý sự cố không phải toàn bộ loại log cần phải lục tung vì như đã nói ban đầu log trên Windows rất nhiều nếu coi hết thì chắc Bumzz mất. Do vậy để ngắn gọn thường chỉ chỉ cần tập trung vào các nhóm sau:

* User authentication và logon 
* User behavior and action
* File/Folder/Share access 
* Các sửa đổi liên quan tới cấu hình Security 

Bản chất cốt lõi là trả lời cho các câu hỏi: Kẻ xấu xâm nhập qua đâu, vào lúc nào, như thế nào, đã làm gì trên hệ thống (Lấy được gì/sửa đổi gì). Để giải quyết các câu hỏi này ta cần tập trung vào các Event security Category sau:

![cate log]( {{site.url}}/assets/img/2021/10/10/log_cate.png)

Thỉnh thoảng ta vẫn hay băn khăn khi đưa ra tiêu chuẩn thiết lập log cho máy chủ thì cần nhưng thông tin nào? Có lẽ bảng trên là một câu trả lời rõ ràng cho câu hỏi này :)


Trên đây là các thông tin cơ bản liên quan tới Event logs của Windows. Ở phần sau ta sẽ đi vào một số kịch bản cụ thể để ứng dụng các thông tin này. Tôi nghĩ đó sẽ là phần hay ho hơn mớ thông tin này rất nhiều. Còn bây giờ muộn rồi, thôi đi ngủ thôi.

Good night!

**Tham khảo:** 

SAN FOR500





