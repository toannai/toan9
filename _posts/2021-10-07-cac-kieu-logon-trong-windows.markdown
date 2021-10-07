---
layout: post
title: Các hình thức logon trong Windows,
date: 2021-10-07 00:32:20 +0700
description: Các hình thức logon trong Windows,
img: 2021/10/07/intro.jpg
fig-caption: # Add figcaption (optional)
tags: [Windows]
---

Gần đây chúng tôi có một yêu cầu là giới hạn chỉ cho phép user logon theo một số kiểu nhất định. Yêu cầu này đã dẫn tôi dến việc cần phải tìm hiểu các kiểu xác thực mà Windows hỗ trợ từ đó đề ra các biện pháp kiểm soát logon (Allow/Deny) phù hợp. Càng đọc tôi lại càng thấy thú vị vì bài toán ban đầu lại mở ra rất nhiều case có thể tái sử dụng kiến thức này (forensic, monitor). Mỗi ngày tôi sẽ viết ra một chút sự "ngộ" đấy nên các bạn chăm đọc blog của tôi, chỉ mất vài phút lướt fb thôi nhưng có khi lại có những nội dung hay ho giúp ích cho công việc cũng nên.

Theo tài liệu của microsoft mô tả [tại đây](https://docs.microsoft.com/en-us/windows-server/identity/securing-privileged-access/reference-tools-logon-types) thì có 7 hình thức logon mà windows hỗ trợ: Interactive (also known as, Logon locally), Network, Batch, Service, NetworkCleartext, NewCredentials, RemoteInteractive.



