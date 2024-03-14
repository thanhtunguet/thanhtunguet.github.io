---
title: Hướng dẫn cài đặt môi trường cho MacOS phục vụ lập trình Mobile (Phần 1)
date: 2023-03-28 10:10:00 +0700
categories: ["Mobile Development", MacOS]
tags: ["Mobile Development", MacOS, Setup]
pin: false
---

## Đặt vấn đề

Cài đặt môi trường lập trình là một công việc không quá khó, nhưng đòi hỏi thời gian và các công đoạn phức tạp để có thể hoàn thiện được môi trường phục vụ đầy đủ cho lập trình, đặc biệt là lập trình Mobile cross-platform với Flutter và React Native. Bài viết này hướng dẫn và giải thích cho các bạn từng bước cài đặt để các bạn hiểu hơn về nền tảng MacOS, nền tảng lập trình Android & iOS với React Native hay với Flutter.

## Hiểu hơn về máy Mac và MacOS

### Kiến trúc Chip

Máy tính Mac của Apple sử dụng các loại kiến trúc chip khác nhau để cung cấp hiệu suất và tính năng tốt nhất cho người dùng. Các loại kiến trúc chip này bao gồm:

### Apple Silicon (ARM)

Apple Silicon là tên chung cho các loại chip được thiết kế bởi Apple và chạy trên các máy tính Mac mới nhất. Các chip này có khả năng tính toán cao hơn và tiêu thụ năng lượng thấp hơn so với các chip Intel trước đây. Apple đã chuyển đổi từ sử dụng chip Intel sang Apple Silicon từ năm 2020.

### Intel-Core (x86-64 hay amd64)

Trước khi chuyển sang Apple Silicon, các máy tính Mac sử dụng các chip Intel với kiến trúc x86-64. Các chip Intel này cung cấp hiệu suất và tính năng tốt nhất cho các ứng dụng đòi hỏi nhiều sức mạnh tính toán, chẳng hạn như đồ họa và làm phim.

### Nắm rõ loại chip của máy bạn

Kể từ năm 2020 trở đi, trên thị trường đang duy trì 2 dòng máy Mac là Mac Intel và Mac M-Series, tướng ứng với 2 kiến trúc là x86-64 và ARM. Kể từ 2021, đa số các phần mềm đều đã cập nhật hỗ trợ cho dòng máy Mac M với các tùy chọn sau:

- Mỗi kiến trúc một bản dựng tương ứng

![picture 11](/assets/img/1bf577b6d241c9878e0fcd561bdced5ba5830e6e3eb72f14bc916ad788c785bd.png)  

- Universal dùng chung

![picture 13](/assets/img/cfde445227764c9e27c09433e3e5f91fe3a3e16944a27e533cf17ada750ece9f.png)  

Bạn cần nắm rõ loại Chip của máy để lựa chọn bản dựng tương ứng cho các phần mềm mình sử dụng. Để kiểm tra loại Chip, từ Apple Menu (Logo quả táo), chọn **About this Mac**:

![picture 10](/assets/img/ba8094b2008eee21b821bb7175442f27a4458915e43467db3c8999d88d6ce386.png)  

## Cài đặt Command-line tools và Homebrew

Command-line tools và Homebrew là các công cụ nền tảng để cài đặt các package khác nên sẽ cần được thực hiện đầu tiên.

### Command-line tools

`Command-line tools` là một bộ công cụ cung cấp cho người dùng các công cụ dòng lệnh để phát triển ứng dụng trên macOS. Được phát triển bởi Apple, `command-line tools` cung cấp cho người dùng các công cụ quản lý gói, mã nguồn (Git), biên dịch và xây dựng ứng dụng, và nhiều công cụ khác để giúp người dùng phát triển các ứng dụng trên macOS.

Hầu hết các môi trường lập trình trên MacOS đều yêu cầu gói này. Ngoài ra, chúng ta cần sử dụng `git` và `curl` để cài các gói khác, bao gồm cả Homebrew. Do đó, ta cần thực hiện cài Command line tools đầu tiên khi bắt đầu thiết lập môi trường cho máy Mac của mình.

Để sử dụng `Command-line tools`, người dùng cần cài đặt XCode trên macOS của mình. Sau khi cài đặt XCode, người dùng có thể cài đặt XCode `command-line tools` thông qua Terminal trên macOS bằng cách sử dụng lệnh:

```
xcode-select --install

```

Việc cài đặt `command-line tools` sẽ giúp người dùng có thể sử dụng các công cụ dòng lệnh để phát triển ứng dụng trên macOS.

### Homebrew

Homebrew là một trình quản lý gói cho MacOS, giúp bạn cài đặt các ứng dụng và thư viện một cách dễ dàng thông qua Terminal. Để cài đặt Homebrew, bạn có thể sử dụng lệnh sau trên Terminal:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"

```

Sau khi cài đặt xong, bạn có thể sử dụng lệnh sau để cài đặt các gói trên Homebrew:

```bash
brew install {tên gói}
```

Ví dụ, để cài đặt Chrome trên Homebrew, bạn có thể sử dụng lệnh:

```bash
brew install google-chrome

```

Ngoài ra, bạn cũng có thể tìm kiếm các gói trên Homebrew bằng lệnh:

```bash
brew search {tên gói}
```

Lưu ý: bạn cần cài đặt Command-line tools trước do trong CLT đã có sẵn curl phục vụ cho lệnh cài Homebrew ở dưới.

## Cài đặt ZSH plugins

ZSH là một shell (môi trường dòng lệnh) được phát triển để thay thế cho Bash, được sử dụng phổ biến trên các hệ điều hành như Linux và MacOS. ZSH có nhiều tính năng tiên tiến hơn so với Bash, bao gồm cả khả năng hoàn thiện câu lệnh tự động và cấu hình dễ dàng.

Oh-my-zsh là một plugin cho ZSH, cung cấp cho người dùng các tính năng bổ sung và giao diện dễ dàng sử dụng. Oh-my-zsh bao gồm nhiều chủ đề khác nhau để thay đổi giao diện của terminal và hơn 150 plugins khác nhau để cung cấp các tính năng khác nhau cho ZSH.

ZSH hiện chính là Shell mặc định cho Terminal của MacOS.

Bước này không bắt buộc. Tuy nhiên mình khuyến khích các bạn cài để có một terminal hữu ích và dễ dùng hơn, với 2 plugin `zsh-syntax-highlighting` và `zsh-autosuggestions`, bạn sẽ sử dụng Terminal dễ hơn rất nhiều do được hỗ trợ highlight câu lệnh và gợi ý các câu lệnh sẵn có hoặc thường xuyên sử dụng dựa theo lịch sử dùng lệnh của bạn.

Ví dụ: với lệnh `adb reverse tcp:8090 tcp:8090` là lệnh mình thường xuyên sử dụng để mở Objectbox Admin khi lập trình Flutter. Khi cài đầy đủ plugin, mình sẽ được hỗ trợ tự động complete câu lệnh như thế này:

![picture 6](/assets/images/b45a08485a797ba8ae0742ac59e160bda7cded41c34c135c27dc9705f3b6c166.png)

### Cài đặt Oh-my-zsh

Sử dụng lệnh sau để cài đặt `Oh-my-zsh`:

```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

Sau khi cài đặt xong, `oh-my-zsh` sẽ tự động mở ra một session mới ngay trong terminal hiện tại. Bạn có thể giữ nguyên session này để cài đặt tiếp 2 plugin kế tiếp hoặc thoát ra và mở một terminal mới.

### Cài đặt zsh-syntax-highlighting và zsh-autosuggestions

Bước 1: Sử dụng Git để clone 2 plugin về

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

Bước 2: Cấu hình `oh-my-zsh` để tự động load 2 plugin này mỗi khi bạn mở terminal mới:

- Mở file `~/.zshrc`

  ```bash
  nano ~/.zshrc
  ```

- tìm đến đoạn:

  ```bash
  plugins=(git)
  ```

- và sửa thành:
  ```bash
  plugins=(
    git
    zsh-autosuggestions
    zsh-syntax-highlighting
  )
  ```

Bước 3: Khởi động lại terminal

Hoặc nhập lệnh:

```bash
source ~/.zshrc
```

### Lưu ý quan trọng

Khi cài `oh-my-zsh`, plugin này sẽ ghi đè file `~/.zshrc` là file cấu hình cho zsh của terminal.
Vì vậy, bước này cũng cần cài trước khi cài các môi trường khác.

Link tham khảo: [https://www.freecodecamp.org/news/jazz-up-your-zsh-terminal-in-seven-steps-a-visual-guide-e81a8fd59a38/](https://www.freecodecamp.org/news/jazz-up-your-zsh-terminal-in-seven-steps-a-visual-guide-e81a8fd59a38/)

## Version manager cho các ngôn ngữ lập trình phổ biến

Một xu hướng khá phổ biến của các ngôn ngữ lập trình gần đây là `version manager`. Các ngôn ngữ lập trình phổ biến thường có các phiên bản mới định kỳ và bổ sung các tính năng / thay đổi thường xuyên. Những sự thay đổi này thường được thực hiện nhanh hơn so với các thư viện / framework. Do đó, mỗi framework, thư viện hay dự án thường sẽ yêu cầu các phiên bản khác nhau cho mỗi ngôn ngữ.

Ví dụ:

Với React Native, từ bản 0.70 về trước có thể chạy được với Node.js `v16`, nhưng từ 0.71 sẽ yêu cầu Node.js `v18`.

Vì vậy, khi cài đặt môi trường lập trình, khuyến khích các bạn cài tất cả các công cụ version manager tương ứng.

### NVM

Ta sẽ bắt đầu với `NVM` cho Node.js

```bash
brew install nvm
```

Thêm đoạn sau vào cuối file `~/.zshrc`:

```bash
# Node version manager
export NVM_DIR=~/.nvm
source $(brew --prefix nvm)/nvm.sh
```

Sau khi cài đặt `NVM`, bạn sẽ cần những câu lệnh sau đây:

#### Cài đặt một phiên bản Major mới nhất:

```bash
nvm install 16
```

NVM sẽ cài đặt phiên bản mới nhất của `v16`

- Cài đặt một phiên bản cụ thể `major.minor.patch`

```bash
nvm install 16.15
```

Các phiên bản này tuân thủ định dạng `semantic versioning`.

#### Chuyển đổi phiên bản cho session shell hiện tại:

```bash
nvm use 16
```

hoặc

```bash
nvm use 16.15
```

Lưu ý: Câu lệnh này cần chỉ định một phiên bản đã được cài đặt trước đó.

#### Thiết lập phiên bản mặc định cho Node.js

Bước này là cực kỳ quan trọng đối với các bạn lập trình React Native hoặc Web front-end.

Do Xcode hoặc các IDE sẽ lấy cấu hình mặc định của shell tương ứng để thực hiện các lệnh bundle code, nghĩa là chúng sẽ sử dụng phiên bản mặc định được chỉ định của Node.js

```bash
nvm alias default <version>
```
