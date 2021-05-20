---
title: Tạo template cho project của mình
author: thanhtunguet
date: 2021-05-16 21:10:05 +0700
categories: ["React Native"]
tags: ["React Native", "Template"]
math: true
mermaid: true
---

# Câu chuyện về cái template

Hừm, chợt nhận ra là mỗi khi có một dự án mới, mình khởi tạo project React Native và phải cấu hình đau cả tay những tính năng cần thiết mà mọi người trong team phát triển quen thuộc. Cuối tuần này ngồi cũng mày mò cấu hình được một cái template để dùng lại.

Nó cũng khá đơn giản, mình xem lại câu lệnh khi khởi tạo project mới:

```sh
react-native init ProjectName --template react-native-template-typescript
```

thì bỗng dưng nhớ là hình như đây là tên một repo của React Native Community. Mình thử vào GitHub và gõ thì thấy:

![Abc](/assets/img/Screen Shot 2021-05-16 at 22.14.24.png)

Ồ, có vẻ đúng. Vậy ra cái template mà mình download mỗi khi khởi tạo project mới là từ cái repo này. Nghía qua một chút thì thấy nó có phần cấu hình bên ngoài và một cái thư mục `template` với cấu trúc đúng như một project mới, chuẩn bài rồi.

# Customize template cho riêng mình

Rồi, và thế là mình cài cắm thêm một số thứ như sau:

### react-native-sass-transformer

Dùng để viết styles dưới dạng scss, rất quen thuộc với web developer.

### react-native-svg-transformer

Có thể dùng `require` để gọi trực tiếp file svg thành component, không cần tạo component bằng file tsx.

### Kotlit support cho Android

Cấu hình sẵn Kotlin cho Android. Khi thêm một thư viện nào đó dùng Kotlin thì không cần phải cấu hình lại nữa.

### Swift support cho iOS

Tương tự với Android, ở iOS mình cũng cấu hình sẵn Swift.

### Chỉnh sửa tsconfig

Mình cấu hình sẵn file `tsconfig.json` với các options phù hợp cho dự án của công ty mà mình dùng. Ngoài ra, mình cũng cấu hình luôn alias cho các thư mục và file quan trọng: `assets`, `src`, `app.json`, `package.json`

```json
{
  "paths": {
    "assets/*": ["./assets/*"],
    "src/*": ["./src/*"],
    "app.json": ["./app.json"],
    "package.json": ["./package.json"]
  }
}
```

### Babel

Liên quan đến Typescript, mình cũng cấu hình sẵn một số plugins cho Babel để transpile code theo cách mình muốn, bao gồm việc resolve path tương thích với `tsconfig`

```js
module.exports = {
  presets: ["module:metro-react-native-babel-preset"],
  plugins: [
    "macros",
    [
      "module-resolver",
      {
        root: ["."],
        alias: {
          assets: "./assets/",
          src: "./src",
          "app.json": "./app.json",
          "package.json": "./package.json",
        },
      },
    ],
    ["@babel/plugin-proposal-decorators", { legacy: true }],
  ],
};
```

### Custom Eslint rules

Mình cấu hình sẵn một số rules cần thiết phù hợp với team:

```js
module.exports = {
  rules: {
    semi: ["error", "always"],
    "comma-dangle": ["error", "always-multiline"],
    "@typescript-eslint/consistent-type-imports": [
      "error",
      {
        prefer: "type-imports",
      },
    ],
  },
};
```

### Fastlane

Fastlane được cài sẵn để cập nhật phiên bản giữa `package.json` và project Android và iOS một cách đồng bộ.

# Sử dụng template như thế nào?

Ủa thế template có rồi, giờ dùng như nào đây nhỉ?

Xem lại câu lệnh, thì hình như _naming convention_ cũng tương tự như những CLI khác. Mình thử:

```sh
react-native init ProjectName thanhtunguet/react-native-template-typescript
```

thì thấy được luôn. Vậy chuẩn nốt rồi.

# Kết luận

Vậy là mình đã có một template dành riêng cho các dự án mà mình lead. Việc cấu hình cũng đơn giản và cài đặt khá tiện phải không nào?

# Template repository

[thanhtunguet/react-native-template-typescript](https://github.com/thanhtunguet/react-native-template-typescript)
