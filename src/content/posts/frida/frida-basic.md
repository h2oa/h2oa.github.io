---
title: frida - những bước đầu tiên
published: 2025-01-23
updated: 2025-01-24
description: 'frida - những bước đầu tiên'
image: ''
tags: [frida]
category: 'Mobile security'
draft: false 
lang: ''
---

# I. Mở đầu

Mới chuyển sang môi trường mới được một thời gian ngắn, nhìn lại thấy bản thân còn quá lười và cần cần học thêm nhiều thứ vì nơi làm việc mới cũng thoải mái thời gian, nếu không học thêm thì ... chắc còn cái nịt. Vô tình trong lúc tìm cách bypass frida detection tìm được blog https://eternalsakura13.com/2020/07/04/frida/ đầy đủ và hay quá! Cũng là cảm hứng để xây một blog cho riêng mình, củng cố kiến thức, ghi lại cái mới học được, tự giám sát bản thân để không lười biếng nữa. Học lại frida thôi ...

# II. Cài đặt môi trường

Thực ra trước giờ chưa từng thực sự hiểu frida là gì, tóm gọn lại thì nó giúp "hook" vào các hàm trong mobile và có thể sửa lại logic trong hàm đó trước khi nó được gọi, vi diệu! Về phần này có thể đọc thêm blog của anh Nhật, vẫn hay lên đọc đi đọc lại mỗi khi cần: https://viblo.asia/p/hook-android-java-voi-frida-EbNVQZrb4vR. Rất tuyệt cho người mới như mình.

Frida cần cài đặt ở 2 bên: frida server và frida client. Và version 2 bên phải giống nhau.

Frida server cài trong điện thoại, repo https://github.com/frida/frida/releases. Muốn biết điện thoại dùng arm64 hay x86 check `uname -m`:

```
"aarch64" is 64-bit ARMv8-A, which is Android's "arm64-v8a" ABI.
"armv7l" is 32-bit ARMv7-A, which is Android's "armeabi-v7a" ABI.
"x86_64" is 64-bit x86, which is Android's "x86_64" ABI.
"i386" or "i686" are the basic 32-bit x86 or the Pentium Pro variant respectively, which is Android's "x86" ABI.
```

![alt text](images/{A4546326-676C-42E7-ACF3-D59E81D83FB3}.png)

Thường hay `adb push` vào `data/local/tmp`, rồi chạy frida server lên thôi `./frida-server-name &`, dấu `&` cho chạy ngầm:

![alt text](images/{00C43D22-3527-4EB3-8BE3-7C542E947831}.png)

Listen port `-l 0.0.0.0:1337`, chưa rõ tác dụng, chắc sau dùng để debug.

Frida client cài trong môi trường máy tính:

```
pip3 install frida-tools
frida --version
frida-ps --version
```

Cài version theo ý muốn:

```
pip3 install frida==16.6.4
```

# III. Frida cơ bản

Check các package apk đang chạy `frida-ps -Ua`:

![alt text](images/{C1928C7C-13A5-435E-84F4-77975CC21F0D}.png)

Chương trình đơn giản spawn ứng dụng:

```
Java.perform(function x() {
    console.log("hello")
})
```

```
frida -U -f <package.name> -l <frida.js>
```

Chạy và in ra console "hello":

![alt text](images/{D73BAF2B-16C9-41D8-85DD-FE942E0E09DE}.png)

# IV. Một vài bài CTF luyện tập

## PicoCTF - droids1

apk file: https://github.com/h2oa/h2oa.github.io/tree/master/assets/ctf/one.apk

![alt text](images/{A5F0DCAB-9181-484E-A432-9744E9639800}.png)

Hàm `getFlag()` so sánh input nhập vào với `String password = ctx.getString(R.string.password);`, nếu giống sẽ trả về flag (lấy từ lib `hellojni`). Dùng frida hook vào hàm `getFlag()` để in ra giá trị `password`. Chuyển đoạn Java `String password = ctx.getString(R.string.password);` sang js:

![alt text](images/{B6B7D3B9-2B71-40EB-8DF0-9EE05E792DEF}.png)

```
var passwordResId = ctx.getResources().getIdentifier('password', 'string', ctx.getPackageName());
var password = ctx.getString(passwordResId);
```

Frida script:

```
Java.perform(function () {
    console.log("Start hooking ...");
    var FlagstaffHill = Java.use('com.hellocmu.picoctf.FlagstaffHill');
    FlagstaffHill.getFlag.implementation = function(input, ctx) {
        var passwordResId = ctx.getResources().getIdentifier('password', 'string', ctx.getPackageName());
        var password = ctx.getString(passwordResId);
        console.log("password: ", password);
        var result = this.getFlag(input, ctx);
        return result;
    };
});
```

Có thể hook không thành công vì khi `Java.perform()` được gọi thì ứng dụng chưa sẵn sàng, có thể giải quyết bằng cách đợi 2 giây rồi mới thực hiện, lúc này ứng dụng sẵn sàng sẽ hook thành công được:

```
function main() {
    console.log("Start hooking ...");

    setTimeout(function() {
        Java.perform(function() {
            var FlagstaffHill = Java.use("com.hellocmu.picoctf.FlagstaffHill");
            FlagstaffHill.getFlag.implementation = function(input, ctx) {
                var passwordResId = ctx.getResources().getIdentifier('password', 'string', ctx.getPackageName());
                var password = ctx.getString(passwordResId);
                console.log("password: ", password);
                var result = this.getFlag(input, ctx);
                return result;
            }
        });
    }, 2000);
}

setImmediate(main);
```

![alt text](images/{5D5F1A18-43C0-4740-AA9C-F1CAFCE504A8}.png)

Một cách khác là hook vào hàm `Context.getString()`:

```
Java.perform(function() {
    var Context = Java.use("android.content.Context");
    Context.getString.overload('int').implementation = function(resId) {
        var result = this.getString(resId);
        console.log("resId: ", resId, " result: ", result);
        return result;
    }
})
```

## PicoCTF - droids4

![alt text](images/{F69AF8AD-BCE9-4DB8-8391-AA2ABA79C219}.png)

Reverse đoạn code lấy password có thể nhờ chatgpt, thu được password `alphabetsoup`:

![alt text](images/{5E726E70-09E2-4588-B3E0-BAB39447922A}.png)

![alt text](images/{46CE399B-75E1-47B2-BF56-0CF04595503D}.png)

Nhập xong chỉ trả về "call it" vì `return input.equals(password) ? "call it" : "NOPE";`. Lẽ ra khi password đúng cần trả về hàm `cardamom(password)`. Frida hook hàm `getFlag()` rồi gọi hàm `cardamom()` trong đó với input là `password` tìm được ở trên rồi in ra thôi:

```
function main() {
    console.log("Start hooking ...");

    setTimeout(function() {
        Java.perform(function() {
            var FlagstaffHill = Java.use("com.hellocmu.picoctf.FlagstaffHill");
            FlagstaffHill.getFlag.implementation = function(input, ctx) {
                var password = "alphabetsoup";
                var flag = FlagstaffHill.cardamom(password);
                console.log(flag);
                var result = this.getFlag(input, ctx);
                return result;
            }
        });
    }, 2000);
}

setImmediate(main);
```

![alt text](images/{4D8A1C16-18BD-49B9-8696-6E9204AD72E1}.png)

## UnCrackable Level 1

Bài viết tham khảo: https://viblo.asia/p/write-up-owasp-uncrackable-level-1-luyen-tap-co-ban-ve-hooking-functions-bang-frida-va-chen-smali-code-LzD5dgb4ljY

apk file: https://github.com/h2oa/h2oa.github.io/tree/main/src/assets/apks/UnCrackable-Level1.apk

Ứng dụng detect root khi mở:

![alt text](images/{0EA19DE3-3076-46DC-9949-242891E8B7CB}.png)

Nguyên nhân do kiểm tra các điều kiện `c.a() || c.b() || c.c()`

![alt text](images/{B5A46DCD-49E5-40DE-AA00-42F3FE905D62}.png)

Hook frida sửa hết thành `return false`:

```
let c = Java.use("sg.vantagepoint.a.c");
c.a.implementation = function () {
    return false;
};
c.b.implementation = function () {
    return false;
};
c.c.implementation = function () {
    return false;
};
```

Hook xong vẫn không được, xem lại bài viết nhận ra nguyên nhân do root check được gọi ngay khi app khởi động (`onCreate()`), frida chưa kịp hook vào. Chuyển sang ý tưởng hook thay đổi chức năng của buttun OK, để bấm xong không thoát ra khỏi app, viết lại hàm luôn không còn `rertun` nữa:

```
let hookexit = Java.use("java.lang.System");
hookexit.exit.implementation = function() {
    console.log("Exit cancelled");
};
```

Bấm OK xong app không còn thoát ra, chức năng cho nhập input nhưng cần verify:

![alt text](images/{1003FF31-3A5B-40B5-8584-C8B79E581ABC}.png)

Xem hàm `verify()`:

![alt text](images/{2DB8B4D5-6D36-42E3-B5FE-0B1544DD9CEC}.png)

Hook tiếp `a.a(obj)`:

```
let a = Java.use("sg.vantagepoint.uncrackable1.a");
a["a"].implementation = function (str) {
    return true;
};
```

![alt text](images/{C3F9AFB9-DF60-4D3F-AE4D-5AA2868C5CDC}.png)

Cứ tưởng là kết thúc nhưng bài này có flag, input phải là flag, nãy trực tiếp hook cho thành auto true luôn :))))

Xem hàm này:

![alt text](images/{BA8558A1-DE52-4EED-840E-90F81A122DF7}.png)

Flag chính là `bArr`, hook tiếp vào hàm `sg.vantagepoint.a.a.a` rồi in ra kết quả hàm là được:

```
function main() {
    console.log("Start hooking ...");

    setTimeout(function() {
        Java.perform(function() {
            let hookexit = Java.use("java.lang.System");
            hookexit.exit.implementation = function() {
                console.log("Exit cancelled");
            };
            let a_bArr_use = Java.use("sg.vantagepoint.a.a");
            a_bArr_use["a"].implementation = function (bArr, bArr2) {
                var result = this.a(bArr, bArr2);
                console.log("Result: " + result); 
                return result;
            };
        });
    });
}

setImmediate(main);
```

![alt text](images/{91FEC1F6-BD1F-451F-8798-C287236F19B7}.png)

Decode bằng CyberChef > Magic:

![alt text](images/{BAFDDF54-963C-4DC6-BDEC-8F6640EE4416}.png)

# Lưu lại vài link

Paste image và tự lưu vào folder custom: https://www.youtube.com/watch?v=L6KKzbVD-Y8