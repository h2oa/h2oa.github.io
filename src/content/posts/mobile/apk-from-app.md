---
title: Trích xuất apk từ app
published: 2025-03-05
description: 'Trích xuất apk từ app'
image: ''
tags: [apk, mobile]
category: 'Mobile security'
draft: false 
lang: ''
---

# I. Mở đầu

Có được file apk chắc chắn giúp phân tích các hành vi của ứng dụng dễ dàng hơn. Ghi lại một số cách lấy file apk từ ứng dụng. Tập trung chủ yếu android trước, ios tìm hiểu sau ...

# II. Command

Với các app tải từ CHPlay về, có thể dùng lệnh trích xuất apk. Mang luôn ehust ra test.

Tìm được path của ứng dụng xong, pull hết file apk về:

![alt text](images/{63AED607-8A4A-4C39-A5BA-CBB54DDC3961}.png)

Trường hợp này ứng dụng sử dụng Android App Bundles nên có các các split apks (bao gồm `base.apk` và các `split_config.*.apk`). Muốn install lại cần dùng `adb install-multiple` thay vì install bình thường:

![alt text](images/{AA9743B1-62CE-4B70-BB30-E24638D0C11D}.png)

Khi phân tích app bằng jadx, base.apk chứa code java, kotlin, ... các lib .so nằm trong file `split_config.<structure>.apk`.