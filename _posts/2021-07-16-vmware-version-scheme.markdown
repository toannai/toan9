---
layout: post
title: VMware versioning,
date: 2021-07-16 00:32:20 +0700
description: VMware versioning scheme
img: 2021/07/16/sematic_versioning.png
fig-caption: # Add figcaption (optional)
tags: [Linux]
---

Cũng tại thời gian này trời nắng nóng, cứ về nhà cơm nước, tắm rửa xong là chui tọt vào phòng ngủ luôn. Không thèm ngồi vào bàn làm gì. Nay quyết tâm lắm nên mới ngồi vào bàn viết cái gì đó (anw kinh nghiệm là muốn làm gì thì cứ phải ngồi vào bàn chứ không vạ vật là ngủ ngay). Vãn hy vọng những gì mà mình notes lại có ích cho mọi người, không nhiều thì ít. Nên ai mà có hay đọc blogs của em thì nhớ cho một like và sub fb của em nhé. Viết một bài dù ngắn hay dài tính ra cũng mất nhiều thời gian phết đó.

Làm việc với VMware cũng nhiều nhưng bình thường rất ít khi để ý về vấn đề versioning của nó. Mà không phải mình mình đâu mà cả mấy bác coi như là VMware Administrator luôn cũng ít khi để ý đến nó (Vì có bao giờ mấy bác patches mấy đâu mà để ý =))). Nên khi mình hỏi đến cũng ậm ừ lắm. Do vậy, mình cũng dành cả một buổi chiều để tìm hiểu chút ý về vấn đề này, cốt là tự trả lời một số câu hỏi bản thân đưa ra.

### Software versioning

Khi phát triển ra một sản phẩm phần mềm (software) để quản lý và maintain nó người ta sẽ đặt cho nó một cái tên hoặc đánh cho nó một số version nhất định (VD: Ubuntu 16.04 LTS <=cũng được gọi là => Ubuntu Xenial Xerus). Để đánh số phiên bản cho software người ta có đưa ra cái khái niệm **scheme**  - Một lược đồ đánh số phiên bản được tạo ra để theo dõi các phiên bản khác nhau của phần mềm. Có 2 loại scheme.

* Internal version number: Chỉ có giá trị trong tổ chức. Cái này tăng nhiều lần trong 1 ngày. 
* Release version: Dùng khi sản phẩm đã được release. Ít thay đổi hơn, và việc đặt tiên này thường áp dụng semantic versioning.

Đương nhiên ở đây ta chỉ cần quan tâm tới Release version còn thì bên trong tổ chức họ đánh số thế nào thì mặc xác họ, chả ảnh hưởng gì đến ta cả.

### Semantic versioning 

"Semantic versioning" - một scheme được sử dụng khá rộng rãi. Cấu trúc của nó gồm các thành phần chính sau:

![sematic_versioning]( {{site.url}}/assets/img/2021/07/16/sematic_versioning.png){:width="200px"}

Ngoài bộ ba thành phần chính **Major.Minor.Patch** nó có thể kèm theo một số các thành phần phụ khác như pre-release tag và optional build meta tag.

Một số nhà phát triển dị hơn còn dụng thêm 1 số kí hiệu: alpha" (a), "beta" (b), or "release candidate" (rc) để thêm vào versioning cho phần mềm của mình. Nên các bạn đừng ngại nhiên khi thấy một số version như sau: 0.5, 0.6, 0.7, 0.8, 0.9 → 1.0b1, 1.0b2 (with some fixes), 1.0b3 (with more fixes) → 1.0rc1 (which, if it is stable enough), 1.0rc2 (if more bugs are found) → 1.0.





