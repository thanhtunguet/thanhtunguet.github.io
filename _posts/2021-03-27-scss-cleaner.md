---
title: Công cụ tối ưu SCSS trong dự án React Native
author: thanhtunguet
date: 2021-03-27 11:33:00 +0700
categories: [Tools]
tags: [Tools]
math: true
mermaid: true
---

Trong các dự án React Native có sử dụng [react-native-sass-transformer](https://www.npmjs.com/package/react-native-sass-transformer),
do không có các IDE được implement riêng cho kiểu phát triển này nên khi thay đổi giao diện app,
developer dễ bỏ sót các phần code sass dư thừa.
Điều này dẫn tới bundle size tăng lên một cách không cần thiết,
dẫn tới nhu cầu có một công cụ có khả năng kiểm tra và loại bỏ các phần code thừa này.

[Xuống cuối trang](#frame)

Đây là một công cụ đơn giản, sử dụng để kiểm tra và loại bỏ các phần code sass thừa theo cấu trúc được quy ước sẵn.

### Cấu trúc

Các component trong dự án cần được viết theo mẫu sau:

#### File tsx

```tsx
import React from "react";
import styles from "./DemoComponent.scss";
import { View } from "react-native";

function DemoComponent() {
  return <View style={[styles.container]} />;
}

export default DemoComponent;
```

#### File scss:

```scss
.container {
  width: 100%;
  height: 100%;
  flex: 1;
  background-color: #ffffff;
}
```

Trong đó:

- Object stylesheet cần được import chính xác với tên `styles`
- File scss cần được đặt cùng thư mục và cùng tên với file tsx
- Chỉ hỗ trợ scss

### Ưu điểm

Có khả năng tìm và loại bỏ những đoạn code scss thừa (mixin, class)

### Nhược điểm

Do phụ thuộc vào scss-parser, một số file scss có thể chưa được đọc đúng và báo lỗi. Cần kiểm tra lại bằng tay.

<a href="https://thanhtunguet.github.io/scss-cleaner/" target="_blank"> Mở trong tab mới </a>

<iframe allowfullscreen="allowfullscreen"
        id="frame"
        width="100%"
        height="600"
        src="https://thanhtunguet.info/scss-cleaner/">
