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

## 1. Phân tích sự kiện đăng nhập trong Windows,

>Để chuẩn bị cho phần này tôi nghĩ trước tiên các bạn hãy đọc bài viết "Tất tần tật các hình thức logon trong Windows" của tôi để có cái nhìn tổng quát về các kênh logon mà Windows Support cũng như cách thức để block các hình thức logon này.

Tracking account là một trong những task hay được thực hiện nhất khi review Events Logs. Việc tracking này sẽ giúp ta trả lời các câu hỏi Account nào bị compromise. Vào thời điểm nào, kéo dài bao lâu. Qua hình thức logon nào và nguồn gốc các truy cập từ đâu. Các thông tin này là vô cùng quan trọng trong quá trình forensic.

#### Một số Event IDs cần quan tâm trong kịch bản này:

* 4624 - Success Logon
* 4625 - Failed Logon
* 4634 / 4647 - Successful Logoff
* 4672 - Account logon với superuser right (Administrator)

#### Hãy nhớ một số điều sau đây:

* Các Event desription cung cấp cho chúng ta các mô tả rất chi tiết. Do vậy nếu quên ý nghía của các trường có thể tham khảo tại phần description này. Tuy nhiên nhược điểm là MS viết quá chi tiết đến nỗi dài không ai muốn đọc luôn :)

![Logon Des]( {{site.url}}/assets/img/2021/12/14/logon_des.PNG){:width="600px"}

* Các logon event không xuất hiện khi máy bị bị một số dạng exploit (RCE, privilege escalation, service exploitation, malware).  Không nên đi tìm hoặc tìm không ra là không có.

#### Ngó qua xem Event Logon 4624 có gì nào?

![Logon event]( {{site.url}}/assets/img/2021/12/14/event-4624.png){:width="600px"}

Không quá khó để luận ra ý nghĩa của các trường. Ở đây chỉ mô tả một số trường quan trọng mà thôi.

Đầu tiên bạn hãy để ý tới **Logged** => Thời gian thực hiện hành vi logon.

* Phần Subject

Xác định "account" - đối tượng mà user request log on tới - Không phải là người dùng đã logged on. Subject thường NULL hoặc là một Service Pricipal. Các thông tin trường này thường không có ý nghĩa nhiều cho việc forensic - Có thể bỏ qua không cần quan tâm. Xem phần New Logon để biết thông tin về người dùng đã logon vào hệ thống. 

* Phần Logon Infomation

**Logon Type** Chỉ ra hình thức logon đã được sử dụng. Từ logon type này ta có thể nhận ra user thực hiện hành vi logon qua kênh nào vd: RDP, Net logon, ... từ đó quá trình hardening/response sẽ hiệu quả hơn (Chả hạn biết chặn các lần logon tương tự của attacker như thế nào). Trong blog của tôi cũng có bài viết mô tả rất chi tiết **Tất tần tật các hình thức logon trong windows** các bạn có thể tham khảo nếu muốn biết sâu hơn.

**Restricted Admin Mode** Logon có sử dụng mode Restricted hay không? Giá trị Yes/No/None(-)

* Phần New logon

Đây là phần chứa nhiều thông tin quan trọng nhất:

**Security ID**: SID của account

**Account Name**: Logon name của account

**Account Domain**: Domain name của account hoặc là DNS name (chữ hoa hoặc thường) hoặc NETBIOS domain name.  Trong trường hợp các đối tượng đặc biệt (như well know security principals) giống như SYSTEM, LOCAL SERVICE, NETWORK SERVICE, ANONYMOUS LOGON trường này sẽ là "NT AUTHORITY".  Nó cũng có thể là "NT Service" trong trường hợp virtual accounts của services.  Nếu là local account nó sẽ là tên của computer.

**Logon ID**: Là một số semi-unique (unique giữa các lần reboots). Số này được sử dụng đẻ nhận dạng logon session, nó được sinh ra từ khi đầu trình khởi tạo sesion.  Mọi events logged sau này trong suốt logon session này sẽ có cùng một Logon ID cho tới khi logoff với event ID là 4647 or 4634 - Trường này sẽ giúp link các event trong phiên logon lại với nhau.

**Linked Login ID**: (Win2016/10) Được sử dụng để User Account Control và interactive logons.  Khi admin interactive logon vào hệ thống với UAC enabled, Windows thực chất sẽ tạo 2 logon sessions - một cái có privileage và  một cái không có privilege.  Điều này được gọi là split token và trường này giúp links 2 sessions với nhau.  (Đù móa đọc thật khó hiểu).

**Network Account Name**: (Win2016/10)  Thấy luôn là "-".  

**Logon GUID**:  Trường ngày giúp correlate (liên kết) logon events trên máy tính local computer với authentication events trên the domain controller.  Giúp link 4624 trên member computer trong miền với 4769 trên DC (Domain controller).  Tuy nhiên  GUIDs không match giữa logon events trên member computers và authentication events trên domain controller.

* Phần Process Information

ProcessID: là ID của process được thi hành khi việc logon bắt đầu

Process Name: Tên process thực hiện quá trinhg logon

* Phần Network Infomation

PHần này giúp nhận diện user thực hiện loggon từ máy client nào. Các thông tin sẽ là None (-) khi logon từ máy local.

**Workstation name**: Tên máy Client

**Source Network Address**: IP của máy client

**Source port**: TCP port thực hiện loggon

* Phần Detailed Authentication Information

Là các thông tin chi tiết về quá trinhg loggon. Phần này cũng không quan trọng lắm trong forensic nên thôi tôi bỏ qua. Bạn nào hứng thú có thể đọc tại đây nha: https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4624

#### Kết hợp Logon và Log off để xác định độ dài phiên

Ta có thể sử dụng trường Logon ID để link 2 sự kiện logon và log off để xác định độ dài của session người dùng.

![Logon session]( {{site.url}}/assets/img/2021/12/14/session_logon.PNG){:width="600px"}

OK ta có thời gian bắt đầu logon (Trường Logged trong event 4624). Ta có độ dài session logon. Có loggon ID để liên kết các action. Từ đó có thể dễ dàng xác định phạm vi các hành động mà user đã thực hiện trong suốt phiên (session) hoạt động của mình. Từ đây có thể xác định attacker đã làm gì trên hệ thống của ta.

#### Quan sát Sự kiện logon còn có giá trị để giảm thiểu rủi do từ việc bị brute force

Trong trường hợp có quá nhiều event 4624 mà kết quả là Failed. Rất có thể ta đang bị bruteforce. Lúc này đừng quên quan sát lại tổng thể event logon nhất là xác định các thông tin các trường snhư Account, logon type, source logon để xác định là kẻ tấn công có nguồn góc từ đâu (IP nào), brute force vào account nào với hình thức brute force qua kênh nào vd: RDP, Netlogon, ... Từ đó có cách xử lý cho hợp lý.

![Logon bruteforce]( {{site.url}}/assets/img/2021/12/14/brute_force.PNG){:width="600px"}

## 2. Phân tích việc truy cập File và Folder, network shared,

Phan 2

## 3. Phân tích việc create/exit process,

Phan 3

## 4. Phân tích việc cố tình thay đổi cấu hình thời gian hệ thống,

Phan 4

## 5. Phân tích việc sử dụng các external driver,

Phan 5