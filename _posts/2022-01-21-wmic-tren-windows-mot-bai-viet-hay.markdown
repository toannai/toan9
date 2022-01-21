---
layout: post
title: WMI trên Windows - Một công cụ hữu ích,
date: 2022-01-21 00:32:20 +0700
description: WMI trên Windows - Một công cụ hữu ích
img: 2022/01/21/intro.png
fig-caption: # Add figcaption (optional)
tags: [Security]
---

Trong thời gian làm việc với Windows tôi cũng có nghe tới Windows Management Instrumentation (WMI). Thỉnh thoảng cũng có đọc qua qua nhưng không hiểu/nhớ nhiều lắm. Gần đây đọc được một tài liệu của FireEye khá thú vị về WMIC. Hôm nay có thời gian tôi tóm tắt lại nội dung của bài report đó hy vọng nó có chứa thông tin gì đó hữu ích cho các bạn đọc blog của tôi, vậy thôi. Tôi rất thích các bài report của FireEye vì nó có chứa nhiều thông tin tôi chưa biết hoặc thắc mắc lại được giải thích rất cặn kẽ và dễ hiểu.

## Giới thiệu qua về WMI,

Nói hơi dài dòng tí thì ban đầu người ta đề xuất ra một cái chuẩn để quản lý và kết hợp các thông tin từ các hệ thống quản lý hardware và software khác nhau gọi là Web-Based Enterprise Management (WBEM). WBEM này được base trên Common Information Model (CIM) schema (CIM - Schema có thể hiểu là một cấu trúc chuẩn định nghĩa sẵn để các bên tuân theo giúp dễ intergration với nhau). WMI này của MS là implement cho cái WEBEM này.

Nhưng thôi dài dòng quá, hãy tạm hiểu trên Windows WMI nó như là công cụ giúp truy xuất thông tin hệ thống và qua nó ta còn có thể thay đổi một số cấu hình của hệ thống Windows. WMI này có thể sử dụng local hoặc remote. (Port tương tác với WMI có thể là 135 - DCOM hoặc 5985 - WINRM http/https).

Đây là công cụ mà có từ Win NT 4.0 và Win 95 và nó được MS bảo tồn cho tới tận những bản windows sau này. Công cụ này bắt đầu được cộng đồng security để ý tới từ khi nó được sử dụng bởi stuxnet, được sử dụng với kha khá các mục đích: System reconnaissance, antivirus and virtualmachine (VM) detection, code execution, lateral movement, persistence và data threft. Nghe có mùi "lợi dụng" nhưng hiểu về nó cũng rất có ích cho cả Sysadmin nữa.





