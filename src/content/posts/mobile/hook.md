---
title: Một số cách hook trong mobile
published: 2025-02-25
description: 'Một số cách hook trong mobile'
image: ''
tags: [hook]
category: 'Mobile security'
draft: false 
lang: ''
---

# I. Mở đầu

Bên cạnh sử dụng công cụ frida thực hiện hook, học thêm một số kiểu hook khác nâng cao hơn, dùng trong trường hợp bypass app detect frida chẳng hạn. Cảm ơn người anh `hiepnv` đã giúp đỡ trong quá trình học thêm, quá xịn.

# II. hluda

`hluda` có thể hiểu là frida power hơn, của pháp sư trung hoa, search google cái là thấy một đống blog đến từ các pháp sư.

![alt text](images/{4CED6AF7-D784-4C2F-82AE-8DFC3174B8E4}.png)

Cài đặt giống hệt frida thôi, phiên bản `hluda-server` cần tương ứng với frida client, tải về tại: https://github.com/hzzheyang/strongR-frida-android/releases (trong thời điểm hiện tại mới nhất là bản `16.5.6`, mấy bản ở trên repo trống ... cay, nay loay hoay lúc lâu cứ tưởng phải tự build):

![alt text](images/{C52B66EF-44EF-48BD-AE8E-EA71D01377A9}.png)

Về việc xịn hơn frida ở đâu đợi dịp khác rảnh hơn (đỡ lười hơn) viết tiếp.

# III. jshook + fridamod

Cách này dùng module fridamod trên app Magisk. Cài đặt môi trường trước (tham khảo https://doc.jshook.org/#/md6)

Demo cách dùng `magisk` và `zygisk` cho máy thật. Có `magisk` rồi vào phần settings enable `Zygisk` lên (xong phải restart máy):

![alt text](images/{A78F2276-2DB1-4984-BBF7-5F88851095AB}.png)

Xong cài app `jshook` từ https://jshook.org/, cài xong mở lên sẽ hiện trạng thái chưa actived. Chọn mục `Download Magisk/KernelSU module` cài module zygisk:

![alt text](images/{3D984314-1CEE-4327-BDDD-2F070AA2BC72}.png)

Xong quay về app magisk > Modules > Install from storage, cài cái module vừa tải từ jshook về xong sẽ như này:

![alt text](images/{77579CF4-FA21-4EAB-BDEE-C1A7203348AA}.png)

Vào lại app JsHook sẽ thành activated:

![alt text](images/{670D7CEB-32E7-4F93-A325-F8D088CAD205}.png)

Vào tiếp Framework install `Fridamod`:

![alt text](images/{9114933B-8BD9-4971-8754-02A33039009B}.png)

![alt text](images/{A9512F47-398E-4E72-BABC-6D67DFD270B2}.png)

Vào tiếp phần Apps > Chọn App muốn hook, chọn:

- Enable Hook service
- Select injection frame: Default (chính là dùng fridamod)
- Enable scripts: Thêm script hook trong này

Cuối cùng mở app trực tiếp lên, là script hook cũng được chạy trong JsHook, xem ở phần Logs:

![alt text](images/{440A1C84-A920-4FB1-9981-2AD892EDC57E}.png)

Tham khảo thêm blog: https://gitcode.csdn.net/65ed83231a836825ed79c2d1.html

# IV. 