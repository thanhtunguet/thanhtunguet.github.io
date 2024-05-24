---
title: Hướng dẫn cài đặt môi trường cho MacOS phục vụ lập trình Mobile (Phần 2)
date: 2023-03-28 13:10:00 +0700
categories: ["Mobile Development", MacOS]
tags: ["Mobile Development", MacOS]
pin: false
---

## Cài đặt XCode

### Giới thiệu XCode

XCode là một IDE do Apple phát triển và cung cấp cho các nhà phát triển ứng dụng trên hệ điều hành macOS. XCode cung cấp cho người dùng các công cụ để phát triển ứng dụng iOS, macOS, watchOS, và tvOS.
XCode bao gồm một trình soạn thảo mã nguồn, các công cụ để thực hiện gỡ lỗi, và các công cụ để quản lý phiên bản của mã nguồn. Nó cũng cung cấp các công cụ để thiết kế giao diện người dùng và kiểm tra ứng dụng trên các thiết bị khác nhau.
Ngoài ra, XCode cũng hỗ trợ các ngôn ngữ lập trình khác như Swift, Objective-C, C++, và các ngôn ngữ khác. Với XCode, người dùng có thể phát triển ứng dụng cho App Store và các nền tảng khác trên hệ điều hành Apple.

(đoạn này nhờ Notion AI viết)

Cài đặt Xcode thì khá đơn giản, vào App Store và tìm XCode rồi cài đặt là xong.

### Cài đặt Cocoapods

CocoaPods là một công cụ quản lý thư viện (library) cho các ứng dụng iOS và macOS. Nó cho phép bạn dễ dàng thêm, xóa và cập nhật các thư viện từ kho lưu trữ của CocoaPods.

Để cài đặt CocoaPods trên MacOS, bạn có thể làm theo các cách sau:

```bash
brew install cocoapods
```

hoặc

```bash
gem install cocoapods
```

Lưu ý: cách thứ sử dụng `gem` thuộc về ngôn ngữ lập trình Ruby. Nếu bạn thực hiện theo cách này, khuyến khích bạn cài `rbenv` hoặc `rvm` để quản lý và tạo ra môi trường Ruby local cho máy của bạn, tránh làm "pollute" môi trường ruby mặc định của MacOS.

### DevCleaner for XCode

Khuyến khích cài thêm DevCleaner for XCode (cũng ở trong AppStore) để có thể xóa bớt cache và derived data do XCode sinh ra, đặc biệt đối với React Native thì lượng data này là khổng lồ.

## Rosetta và vấn đề kiến trúc chip

Tại thời điểm bài viết này, tháng 3/2023, hầu hết các môi trường lập trình đều đã hỗ trợ song song đầy đủ cả 2 kiến trúc chip Mac Intel và Mac Apple Silicon.

Tuy nhiên, nếu bạn có các dự án cũ hoặc các dependency cũ. Ví dụ `node-sass` v4, v5, ... Bạn sẽ cần quan tâm đến việc hỗ trợ chúng do chúng chưa được cập nhật hỗ trợ với chip ARM của Apple. Và lúc này bạn cần biết về Rosetta

### Rosetta là gì?

Rosetta trên máy Mac là một công cụ giúp các ứng dụng được thiết kế cho kiến trúc Intel chạy trên các máy tính Mac mới sử dụng chip M1 của Apple. Các ứng dụng này được gọi là "ứng dụng không tương thích natively" vì chúng không được thiết kế để chạy trên chip ARM-based của M1.

Rosetta được cài đặt tự động trên các máy tính Mac mới khi cài đặt ứng dụng không tương thích natively. Khi bạn cài đặt một ứng dụng này, hệ thống sẽ tự động tải và cài đặt Rosetta để giúp chạy ứng dụng trên máy tính của bạn. Các ứng dụng chạy thông qua Rosetta có thể chậm hơn và không hoạt động tốt như các ứng dụng được thiết kế để chạy trên chip ARM-based, nhưng nó vẫn giúp cho các ứng dụng không tương thích chạy trên các máy tính Mac mới.

### Bạn cần làm gì với Rosetta?

Mặc định, XCode, terminal sẽ hỗ trợ chip mới nhất nên không được bật Rosetta.

Trong trường hợp bạn cần phát triển ứng dụng cũ với kiến trúc Intel, hãy thực hiện bật Rosetta cho Terminal và Xcode và khởi động lại chúng và tất cả các editor, IDE khác mà bạn dùng để mở project đó.

Ví dụ, để bật Rosetta cho Xcode, bạn làm theo các bước sau:

Bước 1: Nhấp vào biểu tượng Xcode trong thư mục Applications của bạn để mở nó.

Bước 2: Nhấp chuột phải vào biểu tượng Xcode và chọn "Get Info".

Bước 3: Trong cửa sổ "Get Info", chọn tùy chọn "Open using Rosetta".

Bước 4: Đóng cửa sổ "Get Info".

Bước 5: Khởi động lại Xcode.

![picture 1](/assets/images/319239b8a29283a09317fa4eef1773f0689bc243de4a9747904b34d480e075f5.png)

## Java

Hiện tại, phiên bản Java ổn định nhất phục vụ lập trình Android là Java 11 nên việc cài đặt nhiều phiên bản Java trong một máy là không cần thiết.
Ngoài ra, công cụ jEnv cũng không thực sự tốt và linh hoạt như `nvm`, `rbenv` hay `pyenv`.
Do đó, bạn chỉ cần cài cố định một phiên bản Java duy nhất là Java 11.

Ngoài ra, do Java liên quan đến một số vấn đề bản quyền, nên mình gợi ý phiên bản tốt và dễ cài đặt nhất là `Azul JDK`.

Bạn có thể tìm thấy tại đây: [https://www.azul.com/downloads/#zulu](https://www.azul.com/downloads/#zulu)

Azul SDK cũng là SDK built-in được tích hợp sẵn trong Android Studio.

Sau khi cài xong, để chỉ định mặc định Java 11, bạn thêm dòng sau vào `~/.zshrc`:

```bash
export JAVA_HOME="/Library/Java/JavaVirtualMachines/zulu-11.jdk/Contents/Home"
```

## Cài đặt Android Studio

### Giới thiệu về Android Studio

Android Studio là một môi trường phát triển tích hợp (IDE) được phát triển bởi Google dành cho việc phát triển ứng dụng Android. Nó bao gồm một bộ công cụ phát triển hoàn chỉnh, bao gồm trình biên tập mã, trình giải mã mã, trình quản lý dự án, trình kiểm tra lỗi và trình giải mã APK.

Android Studio được xây dựng trên nền tảng IntelliJ IDEA của JetBrains và đã được phát triển từ năm 2013. Nó là một công cụ phát triển Android chính thức được khuyến khích sử dụng bởi Google và được cập nhật thường xuyên để cung cấp cho các nhà phát triển các tính năng mới và cải tiến hiệu suất.

Android Studio cũng cung cấp nhiều tính năng hữu ích cho các nhà phát triển, bao gồm cách xây dựng giao diện người dùng, quản lý tài nguyên, lập trình ứng dụng Android với Java hoặc Kotlin, tạo nhanh các mẫu ứng dụng và nhiều hơn nữa.

Trên tất cả, Android Studio là một công cụ phát triển tuyệt vời cho việc tạo ứng dụng Android chất lượng cao và nó được sử dụng rộng rãi bởi các nhà phát triển trên toàn thế giới.

(đoạn này cũng nhờ AI viết)

### Cài đặt Android Studio

Truy cập [https://developer.android.com/studio](https://developer.android.com/studio) và tải về phiên bản mới nhất của Android Studio. Lưu ý, ở bước này cần chọn chính xác kiến trúc chip đang sử dụng.

![picture 1](/assets/images/584f7b17a60b0691e93cc9be9ba8d111210fa1caf4e53d1fc95112ea1780a1d9.png)

### Cấu hình môi trường Android

Sau khi cài đặt xong, bạn hãy mở Android Studio trong thư mục `Applications`, thực hiện một số bước khởi tạo và chờ cho ứng dụng tải SDK của Android về và lên được giao diện sau:

![picture 1](/assets/images/3117ec54ae636d2f2e3b8076db004039bae13cb0db4be9470f6c55b06cf98497.png)

Tại thời điểm bài này được viết, phiên bản mới nhất là Android Studio Electic Eel.

Chọn biểu tượng ba dấu chấm cạnh nút `New Project`

![picture 2](/assets/images/6e5d4f7e04c4590d1e9b218048a681323cebd7e4eb63e0be45624ff9b648a173.png)

Chọn `SDK Manager` để cấu hình SDK tương ứng.

#### Android Platform Tools

Tại Android Platform Tools, chọn các nền tảng Android bạn cần (thường là phiên bản mới nhất, hoặc cũ hơn nếu bạn làm các project cũ hơn)

![picture 3](/assets/images/0f5913af314d691205a905b667fc6b0e57a44ed06de1f13a67695064dba457ce.png)

#### Android SDK Tools

Tại Android SDK Tools, chọn `Android SDK Command Line Tools`

Bước này sẽ bắt buộc nếu bạn dùng Flutter.

Nhấn OK để cài đặt.

#### Cài đặt biến môi trường cho Terminal

Mở file `~/.zshrc` và thêm vào cuối file:

```bash
# Android
export ANDROID_HOME="${HOME}/Library/Android/sdk"
export PATH="${PATH}:${ANDROID_HOME}/platform-tools"
export PATH="${PATH}:${ANDROID_HOME}/cmdline-tools"
export PATH="${PATH}:${ANDROID_HOME}/tools"
export PATH="${PATH}:${ANDROID_HOME}/tools/bin"
```

Bạn có thể kiểm tra việc cài đặt môi trường bằng cách mở Terminal mới và nhập 2 lệnh để kiểm tra `adb` và `sdkmanager` đã được thêm vào `PATH` hay chưa:

```bash
which adb

which sdkmanager
```

Nếu bạn cài đúng, Terminal sẽ chỉ ra đường dẫn của các file lệnh tương ứng:

![picture 4](/assets/images/53fb3bb1a79188be4b8dcbccb40db6b60b89f31aa32dca3c7f334df89e695baf.png)

## Cài đặt Flutter

Nếu bạn sử dụng Flutter, bạn sẽ cần cài đặt Flutter SDK

### Tải về Flutter SDK

Truy cập [https://docs.flutter.dev/get-started/install/macos](https://docs.flutter.dev/get-started/install/macos) để tải về SDK Flutter mới nhất, nhớ chọn đúng kiến trúc tương ứng với chip của máy tính MacOS của bạn.

![picture 5](/assets/images/1309048bf1162c14cdb44e8a4844d8b3137b64f2409f27daafb2810f261c3961.png)

Hmm, xem nào, viết đến đây thì thấy là Flutter họ đã hướng dẫn sẵn việc cấu hình môi trường rồi. Mà từ trên xuống dưới bạn cũng đã hiểu chức năng của file `~/.zshrc`, biến môi trường `PATH` và shell ZSH trong MacOS rồi.

Bước này bạn có thể tự thao tác theo hướng dẫn đó nhé.

Gợi ý là: mình hay đặt `flutter` trong thư mục `/opt` nên đoạn cấu hình trong `~/.zshrc` của mình sẽ thế này:

```bash
# Flutter
export PATH="${PATH}:/opt/flutter/bin"
export PATH="${PATH}:/opt/flutter/bin/cache/dart-sdk/bin"
```

### Cài đặt Chrome

Flutter sử dụng Chrome làm debugger, vì vậy bạn cần cài đặt Google Chrome.

Hoặc bạn có thể sử dụng bất kỳ trình duyệt nào có nhân Chrome, nhưng sẽ cần chỉ định đường dẫn đến trình duyệt đó.

Ví dụ mình làm với `Microsoft Edge`, cũng là sửa file `~/.zshrc` nhé:

```bash
export CHROME_EXECUTABLE="/Applications/Microsoft Edge.app/Contents/MacOS/Microsoft Edge"
```

Với các trình duyệt khác bạn cũng làm tương tự.
