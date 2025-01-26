---
title: Hackthebox CTF - Mobile
published: 2025-01-25
updated: 2025-01-26
description: 'Hackthebox CTF - Mobile'
image: ''
tags: [mobile, hackthebox, ctf, reverse]
category: 'CTF'
draft: false 
lang: ''
---

# Mở đầu

File APK đều lưu tại https://github.com/h2oa/h2oa.github.io/blob/main/assets/apks/ dạng zip, pass giải nén đều là `hackthebox`.

# I. Very Easy

## Don't Overreact

Từ tên challenge biết APK code bằng React, mở jadx xem file `index.android.bundle`:

![alt text](images/{B7255B28-6157-47A9-AB10-00964A056A18}.png)

Thấy chuỗi base64 khả nghi, decode ra flag:

![alt text](images/{2FC71F0A-25C3-4BDE-B50A-22A53039144C}.png)

# II. Easy

## Cat

File được cho có extension `.ab`, đây là android backup file. Dùng tool https://github.com/nelenkov/android-backup-extractor unpack thành tar file:

![alt text](images/{B6C15A9F-30F7-4A74-A757-7B43A46A38BB}.png)

Tìm kiếm một hồi, nhận ra trong đống ảnh chỉ có 1 ảnh không phải mèo, flag ở trong ảnh này luôn:

![alt text](images/{CBCFEB1B-8F18-41CC-8E21-DEBD06F0DD35}.png)

Bài này chắc muốn nói đến các file backup android `.ab` bị leak không an toàn.

## APKey

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

File này cũng cần align và sign lại.

Bypass SSL pinning, dùng script trên frida codeshare: https://codeshare.frida.re/@akabe1/frida-multiple-unpinning/ là xong (có dịp phải đọc kỹ cách bypass SSL pinning, mà lười quá), bắt request là có flag:

![alt text](images/{5796346C-DC39-4862-B73E-6B29494A28CD}.png)

## APKrypt

Bước đầu cần align và sign lại. Ứng dụng có một chức năng duy nhất là nhập VIP code:

![alt text](images/{575F0A66-4EAB-40B0-BFE2-06948BB1448D}.png)

![alt text](images/{040275F0-E38D-45F3-A823-FAED23936743}.png)

Có thể hook vào hàm `md5()` để trả về `735c3628699822c4c1c09219f317a8e9` hoặc sửa smali (như bài `APKey`), bài này thử dùng cách hook:

```
function main() {
    console.log("Start hooking ...");
    setTimeout(function() {
        Java.perform(function() {
            let hookMainActivity = Java.use("com.example.apkrypt.MainActivity");
			hookMainActivity.md5.implementation = function(str) {
				console.log("hooked in md5");
				var returnStr = "735c3628699822c4c1c09219f317a8e9";
				return returnStr;
			}
        });
    }, 2000);
}

setImmediate(main);
```

![alt text](images/{480F8313-0B32-4D31-A23E-6D26DAEBC5CE}.png)

## FastJson and Furious

Nhập chuỗi submit xong bị out luôn khỏi ứng dụng:

![alt text](images/{E9F7381C-52D1-4956-B4B3-CBD0ABB8A82F}.png)

![alt text](images/{6D969C2D-C18C-4667-96F9-8B0E27E7411E}.png)

Trước hết cần luồng code đi vào `if (calcHash.length() > 0)`.

![alt text](images/{91BD340F-17FE-4E03-9E21-1408717D3781}.png)

Hàm `calcHash()` có kiểm tra `if (succeed)` nhưng `succeed` được set bằng `false`:

![alt text](images/{35F2452A-52E5-4B57-B5D1-C63D37E95B68}.png)

Chỉ cần hook và sửa giá trị `succeed` thành `true`, nhập vào chuỗi JSON hợp lệ pass điều kiện `parseObject.keySet().size() != 2` thì đều ra được các flag hash khác nhau (tất nhiên chưa đúng):

```
function main() {
    console.log("Start hooking ...");
    setTimeout(function() {
        Java.perform(function() {
            let hook = Java.use("hhhkb.ctf.fastjson_and_furious.MainActivity");
			hook.succeed.value = true;
			hook.calcHash.implementation = function(str) {
				console.log("hooked in calcHash");
				var flag = this.calcHash('{"a":"b", "c":"d"}');
				console.log("flag: " + flag);
				return "123";
			}
        });
    }, 2000);
}
 
setImmediate(main);
```

Chưa hiểu challenge này lắm, đọc write up thấy là CVE https://jfrog.com/blog/cve-2022-25845-analyzing-the-fastjson-auto-type-bypass-rce-vulnerability/, payload `{"@type":"hhhkb.ctf.fastjson_and_furious.Flag","success":true}`

CVE này sẽ gọi được class bất kỳ và control được thuộc tính trong class đó, nên ý tưởng sẽ gọi tới class `Flag` và set lại thuộc tính `succeed` thành true. Ủa payload là `success`, có mỗi cách giải thích ở tên hàm là `setSuccess`:

![alt text](images/{08A363E2-FA62-46AF-AEA3-5EEC48F2AC67}.png)

Đánh giá ý tưởng ra đề hay mà khâu flag không hay.

## Anchored

Align và sign lại trước.

README yêu cầu cài đặt trên thiết bị chưa root. Thôi bypass luôn bằng https://codeshare.frida.re/@dzonerzy/fridantiroot/

Giao diện có một chức năng duy nhất là enter email:

![alt text](images/{75116640-D408-4012-8168-41C0E801AD05}.png)

Trong code có nhiều đoạn trả về `Thank you for requesting early access.` , chú ý có load `libanchored.so`:

![alt text](images/{4BE5E406-6AAB-41D2-AF0C-495173F0DFA7}.png)

List xem trong lib này có những hàm nào:

```
Module.enumerateExports("libanchored.so", { 
    onMatch: function(e) {
        console.log("[+] Function: " + e.name + " - Address: " + e.address);
    },
    onComplete: function() {
    }
});
```

Thấy được một số hàm như `Java_com_example_anchored_MainActivity_frf`

![alt text](images/{A649A312-81A5-48F5-B3E9-A59C5F252749}.png)

Đây chính là mấy hàm trả về string trong code client:

![alt text](images/{2E1D306B-F9BC-448D-AD1C-D618A042CA8D}.png)

List theo tiền tố `Java_com_example_anchored_MainActivity`:

```
if (e.name.startsWith("Java_com_example_anchored_MainActivity")) {
    console.log("[+] Function: " + e.name + " - Address: " + e.address);
}
```

Có được các hàm:

```
[+] Function: Java_com_example_anchored_MainActivity_c8 - Address: 0x74de5cf31c
[+] Function: Java_com_example_anchored_MainActivity_frf - Address: 0x74de5cf04c
[+] Function: Java_com_example_anchored_MainActivity_mrm - Address: 0x74de5cfee8
[+] Function: Java_com_example_anchored_MainActivity_prp - Address: 0x74de5cff94
```

Chú ý 3 hàm `frf`, `mrm`, `prp` trước trả về string, đoán là flag. Lấy giá trị trả về các hàm này:

```
var frf = Module.getExportByName('libanchored.so', 'Java_com_example_anchored_MainActivity_frf');
Interceptor.attach(frf, {
    onEnter: function(args) {
        console.log('Inside frf ...');
    },
    onLeave: function(retval) {
        console.log('ftf returned: ' + retval);
    }
});
// tương tự với mrm, prp
```

Không có gì đặc sắc ...

![alt text](images/{169C7BA4-31AC-4524-80C0-DCA4582458A6}.png)

Đọc lại mô tả có thể phải intercept http request? bypass SSL pinning? Merge script codeshared trên với https://codeshare.frida.re/@akabe1/frida-multiple-unpinning/ bắt được request, chứa flag luôn:

![alt text](images/{C8D11DDA-2F35-4C08-BC35-78D8F0E0AA1B}.png)

# III. Medium

## Cryptohorrific

Giải nén xong ra một đống file không hiểu gì:

![alt text](images/{DDD3AFF5-C314-448D-83DA-2A7777244196}.png)

Cấu trúc thư mục:

```
└───hackthebox.app
    ├───Base.lproj
    │   ├───LaunchScreen.storyboardc
    │   └───Main.storyboardc
    └───_CodeSignature
```

Hỏi chatgpt mới rõ đây là cấu trúc file IOS, có lẽ cần chuyển sang `.ipa` để cài app. Tìm mãi không tải nối cái ios emulator nào ... thôi dẹp.

Cũng chú ý tới file `challenge.plist` rất giống flag đã encrypt, chắc phải rev:

![alt text](images/{66C14274-286A-4FA1-A56A-BDFDBCB43C35}.png)

Dùng tool `plistutil` có thể chuyển các file `.plist` thành `.xml` cho dễ đọc:

![alt text](images/{68589322-B652-4207-A966-14678EF7A718}.png)

Flag đã được encrypt `Tq+CWzQS0wYzs2rJ+GNrPLP6qekDbwze6fIeRRwBK2WXHOhba7WR2OGNUFKoAvyW7njTCMlQzlwIRdJvaP2iYQ==`. Tìm thấy `key` và `IV` trong file `challenge` sau khi reverse:

![alt text](images/{12EF7903-BBE6-4C77-9D97-9231C51E73B1}.png)

![alt text](images/{FADCE4AD-C02F-4CF4-950C-C8652E14919E}.png)

## SeeTheSharpFlag

Align và sign lại, vẫn không install được, mở jadx ra thấy có mỗi native lib cấu trúc x86, chắc đây là lý do? (có lẽ cũng vì vậy mà tên file là `com.companyname.seethesharpflag-x86`), liệu có thể build thêm lib arm64 không nhỉ?

Chú ý tới các file `.dll` vì đề bài gợi ý C#, quan sát thấy file `SeeTheSharpFlag.dll` khả nghi. Reverse bằng dnSpy không hiểu sao không được, dùng tool https://github.com/NickstaDB/xamarin-decompress decompile rồi mở bằng dotPeek:

![alt text](images/{AF96B414-BA17-47DD-8B03-2130992CCD48}.png)

![alt text](images/{D07142FD-B1D2-4E71-8509-ED8EA83D979C}.png)

Bài này dạy reverse file `.dll`, chắc vậy.