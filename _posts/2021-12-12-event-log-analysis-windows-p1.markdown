---
layout: post
title: Event logs analysis 2 - Một số kịch bản thực tế hay sử dụng,
date: 2021-12-13 00:32:20 +0700
description: Event logs analysis 2 - Một số kịch bản thực tế hay sử dụng,
img: 2021/12/13/ev_top.png
fig-caption: # Add figcaption (optional)
tags: [Windows, Forensic]
---

Ở bài viết trước ta đã có một số hiểu biết cơ bản nhất về Windows Event Logs. Bài viết này ta sẽ tiếp tục đi vào một số kịch bản cụ thể ứng dụng trong quá trình forensic Windows Event Logs thực tế. Đây không phải là tất tần tật các việc cần làm nhưng sẽ là gợi ý không tồi cho các bạn mới khi mà ta không biết bắt đầu từ đâu :). OK, chúng ta bắt đầu thôi.

### Phân tích sự kiện đăng nhập trong Windows,

> Để chuẩn bị cho phần này tôi nghĩ trước tiên các bạn hãy đọc bài viết "Tất tần tật các hình thức logon trong Windows" của tôi để có cái nhìn tổng quát về các kênh logon mà Windows Support cũng như cách thức để block các hình thức logon này.

Tracking account là một trong những task hay được thực hiện nhất khi review Events Logs. Việc tracking này sẽ giúp ta trả lời các câu hỏi Account nào bị compromise. Vào thời điểm nào, kéo dài bao lâu. Qua hình thức logon nào và nguồn gốc các truy cập từ đâu. Các thông tin này là vô cùng quan trọng trong quá trình forensic ở giai đoạn đầu.

### Phân tích việc truy cập File và Folder, network shared,

### Phân tích việc create/exit process,

### Phân tích việc cố tình thay đổi cấu hình thời gian hệ thống,

### Phân tích việc sử dụng các external driver,