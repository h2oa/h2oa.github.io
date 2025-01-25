---
title: Hackthebox CTF - Mobile
published: 2025-01-25
description: 'Hackthebox CTF - Mobile'
image: ''
tags: [mobile, hackthebox, ctf]
category: 'CTF'
draft: false 
lang: ''
---

# I. Very Easy

## Don't Overreact

APK file: https://github.com/h2oa/h2oa.github.io/blob/main/assets/apks/dontoverreact.apk

Từ tên challenge biết APK code bằng React, mở jadx xem file `index.android.bundle`:

![alt text](images/{B7255B28-6157-47A9-AB10-00964A056A18}.png)

Thấy chuỗi base64 khả nghi, decode ra flag:

![alt text](images/{2FC71F0A-25C3-4BDE-B50A-22A53039144C}.png)

## Cat

APK file: https://github.com/h2oa/h2oa.github.io/blob/main/assets/apks/cat.ab

File được cho có extension `.ab`, đây là android backup file. Dùng tool https://github.com/nelenkov/android-backup-extractor unpack thành tar file:

![alt text](images/{B6C15A9F-30F7-4A74-A757-7B43A46A38BB}.png)

Tìm kiếm một hồi, nhận ra trong đống ảnh chỉ có 1 ảnh không phải mèo, flag ở trong ảnh này luôn:

![alt text](images/{CBCFEB1B-8F18-41CC-8E21-DEBD06F0DD35}.png)

Bài này chắc muốn nói đến các file backup android `.ab` bị leak không an toàn.

## APKey

APK file: https://github.com/h2oa/h2oa.github.io/blob/main/assets/apks/APKey.apk

![alt text](images/{3DC81E59-BDAC-487C-86C3-CE75E08E1F51}.png)

File apk ban đầu chưa align và sign, nếu install trực tiếp sẽ bị lỗi. Cần align và sign lại bằng APKTools, sau đó có thể install thành công vào android.

![alt text](images/{8F34D0D8-E1D2-4AFA-AF2C-587091E153C1}.png)

Chức năng login, quan sát code trong file apk:

![alt text](images/{99DC2505-7C45-46C1-9248-C1A4FDC48559}.png)

`this.f928c` là name, `this.d` là password, sau khi login hàm xử lý như sau:

![alt text](images/{BD11A9BF-F2F4-420C-8E22-A38815CA4AE1}.png)

Password nhập vào qua một loạt biến đổi cần có giá trị `a2a3d412e92d896134d9c9126d756f`, thử password là `123` sẽ mã hóa thành `c4ca4238a0b92382dcc509a6f75849b`:

```
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

class Main {
    public static void main(String[] args) {
        String obj = "1";
        try {
            MessageDigest messageDigest = MessageDigest.getInstance("MD5");
            messageDigest.update(obj.getBytes());
            byte[] digest = messageDigest.digest();
            StringBuffer stringBuffer = new StringBuffer();
            for (byte b2 : digest) {
                stringBuffer.append(Integer.toHexString(b2 & 255));
            }
            String str = stringBuffer.toString();
            System.out.println(str);
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }
    }
}
```

![alt text](images/{6915DF77-BB86-4FE2-902F-E0C746904EE1}.png)

Có thể sửa smali để chuyển đoạn password mã hóa từ `a2a3d412e92d896134d9c9126d756f` sang `c4ca4238a0b92382dcc509a6f75849b`, khi đó chỉ cần nhập password `1` sẽ đúng. Decompile bằng APKTool, tìm chuỗi mã hóa ban đầu và sửa lại, compile lại là OK:

![alt text](images/{A073467B-92AF-4C8F-B658-1D57B342D24C}.png)

![alt text](images/{A149003B-4EAB-404B-84BD-93AF134F837C}.png)

## Manager

APK file: https://github.com/h2oa/h2oa.github.io/blob/main/assets/apks/Manager.apk

Challenge này cần connect đến server, ban đầu phải nhập IP và port của machine.

Challenge gồm 2 chức năng login và register, khi register thành công có thêm chức năng update, vừa vào id đã là 5 và role member, có thể cần chiếm account id 1 hoặc role admin:

![alt text](images/{157E4380-CAE1-4293-AE88-DA565F01D4FE}.png)

Login:

![alt text](images/{41D5D571-1A83-4332-9211-479FF68AAD21}.png)

Register:

![alt text](images/{1E6B9B1D-C773-48B8-A57C-97542B43F3AB}.png)

Update:

![alt text](images/{18CCB5BF-06BD-4446-85B3-8C92C0D71379}.png)

Đọc qua code không thấy có gì đặc biệt, đăng ký thử user `admin` thì đã tồn tại:

![alt text](images/{5B5307C0-2835-4699-8C1E-09F85B7B0F47}.png)

Vì `/manage.php` unauth nên có thể update password của `admin`:

![alt text](images/{A5CF5556-3FBA-48D6-B744-E71C6420307C}.png)

Login vào `admin` có flag:

![alt text](images/{0CDA44C8-128A-4D70-9ACC-058E252BDC54}.png)

## Pinned

APK file: https://github.com/h2oa/h2oa.github.io/blob/main/assets/apks/pinned.apk

File này cũng cần align và sign lại.

Bypass SSL pinning, dùng script trên frida codeshare: https://codeshare.frida.re/@akabe1/frida-multiple-unpinning/ là xong (có dịp phải đọc kỹ cách bypass SSL pinning, mà lười quá), bắt request là có flag:

![alt text](images/{5796346C-DC39-4862-B73E-6B29494A28CD}.png)