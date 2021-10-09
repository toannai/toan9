---
layout: post
title: Tất tần tật các hình thức logon trong Windows,
date: 2021-10-07 00:32:20 +0700
description: Tất tần tật các hình thức logon trong Windows,
img: 2021/10/07/intro.jpg
fig-caption: # Add figcaption (optional)
tags: [Windows]
---

Gần đây chúng tôi có một yêu cầu là giới hạn chỉ cho phép user logon theo một số kiểu nhất định. Yêu cầu này đã dẫn tôi dến việc cần phải tìm hiểu các kiểu xác thực mà Windows hỗ trợ từ đó đề ra các biện pháp kiểm soát logon (Allow/Deny) phù hợp. Càng đọc tôi lại càng thấy thú vị vì bài toán ban đầu lại mở ra rất nhiều case có thể tái sử dụng kiến thức này (forensic, monitor). Mỗi ngày tôi sẽ viết ra một chút sự "ngộ" đấy nên các bạn chăm đọc blog của tôi, chỉ mất vài phút lướt fb thôi nhưng có khi lại có những nội dung hay ho giúp ích cho công việc cũng nên.

Theo tài liệu của microsoft mô tả [tại đây](https://docs.microsoft.com/en-us/windows-server/identity/securing-privileged-access/reference-tools-logon-types) thì có 7 hình thức logon mà windows hỗ trợ: Interactive (also known as, Logon locally), Network, Batch, Service, NetworkCleartext, NewCredentials, RemoteInteractive.


## Interactive (Logon Locally)

Microsoft nói "Authentication is interactive when a user is prompted to supply logon information". Điều đó là các hình thức mà cứ yêu hiện lên thông báo cung cấp thông tin đăng nhập là interactive logon. Microsoft cũng chia ra trường hợp interactive logon vơi Local và Domain. Bạn nào muốn hiểu chi tiết nó tương tác ở mức subsystem thì có thể coi [tại đây](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc780332(v=ws.10)?redirectedfrom=MSDN).

Kiểu interactive logon trong thực tế đơn giản nhất là ngồi tại máy login vào màn hình đăng nhập; RUNAS thực hiện tại máy; đăng nhập qua console (kể cả remote console kvm/vmware/...). Đáng đặc biệt hơn là RDP, thậm trí là cả psexec (trường hợp enter username và password) vẫn là interactive logon chứ không phải Net logon nha.

Để control interactive logon ta sử dụng option "Allow/Deny Logon locally" trong GPO (local và AD).

Kiểu logon này được cho phép bằng đặc quyền (privilege) SeInteractiveLogonRight trên windows.

## Net logon

Hình thức logon này tôi không thấy microsoft định nghĩa chắc nịch. Chỉ thấy mô tả một cách chung chung là kiểu **Remote logon** mà **không yêu cầu người dùng cung cấp logon infomation** mà thôi. 

Với hình thức logon này, thông tin xác thực đã được thiết lập trước đó hoặc sử dụng một phương pháp khác để thu thập thông tin xác thực được nhận diện người dùng. Quá trình này xác nhận danh tính của người dùng đối với bất kỳ dịch vụ mạng nào mà người dùng đang cố gắng truy cập. Quá trình này thường ẩn đối với người dùng trừ khi phải cung cấp thông tin đăng nhập thay thế.

Với windows hay dùng một số protocol thực hiện net logon

* Kerberos version 5 protocol
* Public Key certificates
* Secure Socket Layer/Transport Layer Security (SSL/TLS)
* Digest SSP
* NTLM, for compatibility with Windows Server NT 4.0-based systems
* Netlogon Remote Protocol

Một vài kiểu net logon trong thực tế: Câu lệnh Net use * \\SERVER ( bao gồm cả Net use * \\SERVER /u:user) để dùng ổ network; MMC snap-ins to remote computer (VD: RSAT, Eventviewer remote, ...); PowerShell WinRM; PSExec (without explicit creds); Remote Registry; Remote Desktop Gateway (Bước Authenticating to Remote Desktop Gateway).

Để control Net logon ta sử dụng option "Allow/Deny Net logon" trong GPO (local và AD).


## Batch logon

Theo MS "When you use the Add Scheduled Task Wizard to schedule a task to run under a particular user name and password, that user is automatically assigned the Log on as a batch job user right". Điều này có nghĩa là batch logon sẽ xảy ra khi tạo tash scheduler run dưới 1 user khác (có thể cần cung cấp username+password) mà không có sự can thiệp trực tiếp của họ.

Quá trình logon này đương nhiên chỉ diễn ra khi thực thi task schedule chứ không phải khi tạo task.

Kiểu logon này được cho phép bằng đặc quyền (privilege) SeBatchLogonRight trên windows.

Để control batch logon ta sử dụng option "Allow/Deny Batch logon" trong GPO (local và AD).


## Service logon

Service logon được sử dụng khi một dịch vụ được khởi chạy trong context (ngữ cảnh) của người dùng. Logon infomation được cache trong vùng nhớ của lsass process khi dịch vụ được thực thi.

Quá trình logon này đương nhiên chỉ diễn ra khi thực thi service running.

Kiểu logon này được cho phép bằng đặc quyền (privilege) SeServiceLogonRight trên windows.

Để control Service logon ta sử dụng option "Allow/Deny Service logon" trong GPO (local và AD).

Có rất nhiều các thông tin hay ho khác nữa muốn biết chi tiết hãy coi mấy link tôi đã dẫn trong bài nha.

## Tham khảo (khác)

[link1](https://zer1t0.gitlab.io/posts/attacking_ad/?fbclid=IwAR1mS2fiTXJSnALObPQU4NrHjN1uhzuf3WviOefvvec9D6TK6MJX1GZhMzI#interactive-logon)

[link2](https://ldapwiki.com/wiki/Windows%20Logon%20Types)