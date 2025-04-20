---
title: Một số quy tắc lập trình Flutter với Supa
date: 2025-02-16 00:01:00 +0700
categories: [Mobile Development, Flutter]
tags: [Mobile Development, Flutter]
pin: false
---

## Giới thiệu

### Flutter và các ưu điểm chính

- Flutter cung cấp rất đầy đủ các công cụ, thiết kế và widget để tạo giao diện dễ dàng nhanh chóng
- Có sẵn các loại widget, bố cục, chỉ cần tùy chỉnh (customize) giao diện
- Khả năng tùy chỉnh theme rất tốt, có sẵn, thống nhất cho toàn ứng dụng

## Cấu trúc ứng dụng

Cấu trúc ứng dụng như sau:

```
- blocs: chứa các bloc logic dùng chung
- config
  - get_it.dart: cấu hình getIt, phục vụ cho dependency injection
  - supa_subsystem.dart: cấu hình danh sách các subsystem trong hệ thống
  - supa_locale.dart: cấu hình các loại ngôn ngữ được hỗ trợ
- extensions: mở rộng các class có sẵn (chỉ bao gồm tiện ích, không có nghiệp vụ)
- modules: mỗi thư mục con là code của một phân hệ, độc lập, có thể clone thành app riêng
- notifications: chứa giao diện thông báo và các notification handlers để xử lý từng loại thông báo cụ thể
- router: chứa các cấu hình của go_router, định nghĩa routing cho toàn app, có chia nhỏ các file routing cho từng phân hệ
- services: chứa các logic service dùng chung: location, media, etc.
- themes: chứa cấu hình theme cho ứng dụng, dễ dàng tùy chỉnh giao diện bằng theme ở đây, hướng tới việc làm tính năng multi-theme
- widgets: chứa các widget dùng chung
```

## Đa ngôn ngữ (Localization)

Do các công cụ localization có sẵn của Flutter không thuận tiện, Supa phát triển riêng một thư viện với cơ chế quản lý localization riêng là `supa_l10n_manager` như sau:

### Quy cách, cấu trúc thư mục

Các file localization sẽ được lưu trữ dạng JSON phẳng với quy tắc sau: `subsystem.namespace.key1.key2`

Trong đó:

- subsystem là tên phân hệ. Ví dụ: `spend`, `work`, `general`, ...
- namespace có thể là tên entity, tên màn hình để gom nhóm các key vào cùng một nhóm. Ví dụ: `home`, `user`, `product`, ...
- `key1`, `key2`: Hỗ trợ tối đa 2 lớp key con (Tuy nhiên nên hạn chế key2)
- Do JSON có cấu trúc phẳng, vì vậy phải tránh trường hợp ghi đè giữa key 3 lớp và key 4 lớp với cùng prefix. Ví dụ: nếu có một key là `work.home.inspection`, thì sẽ không được phép tồn tại các key `work.home.inspection.noChildKeyHere` do khi quét mã nguồn, các key con sẽ bị ghi đè dẫn đến lỗi dịch trong runtime.


### Chức năng quét mã nguồn sinh key:

  Package hỗ trợ lệnh để quét toàn bộ mã nguồn (Các file Dart) để sinh ra các file JSON tương ứng mà không cần làm thủ công.

  Người dùng cần "đánh dấu" các key được sinh ra khi viết code theo cú pháp sau: `translate('subsystem.namespace.key1.key2')`

  Do package cung cấp cơ chế quét và nhận diện cú pháp đơn giản, nên bắt buộc cần đánh dấu các translation key bằng việc gọi trực tiếp cùng `translate`. Nếu không, package sẽ không thể nhận diện được.

  Ví dụ sai:

  ```dart
  final key = 'subsystem.namespace.key1.key2';
  final translatedText = translate(key); /// Key is not marked correctly
  ```

  Ví dụ đúng:

  ```dart
  final translatedText = translate('subsystem.namespace.key1.key2');
  ```

  Lưu ý: nếu có 2 lớp cần dịch, hãy viết thành 2 statement như sau:


  ```dart
  final translatedDateTime = translate('system.namespace.formattedDate', {'date': formattedDate});
  final translatedWarningText = translate('system.namespace.deadlineExceededWarning', {'deadline': translatedDateTime});
  ```

  Để quét và sinh ra các file JSON, dùng câu lệnh sau:


  ```sh
  dart run supa_l10n_manager extract --locale vi
  ```

  Trong đó: bạn có thể thay `vi` bằng locale code tương ứng của ngôn ngữ muốn tạo.

### Chức năng merge

  Với mỗi subsystem, package sẽ sinh ra một file JSON tương ứng. Việc này đảm bảo để file đủ nhỏ và độc lập giữa các phân hệ, giúp dễ dàng xử lý xung đột trên Git và phân chia công việc trong nhóm trong quá trình lập trình.

  Ứng dụng sẽ sử dụng file một file JSON duy nhất cho mỗi ngôn ngữ, do đó, cần "gộp" các file lại với nhau. Để gộp các file trong cùng ngôn ngữ (locale), dùng câu lệnh sau:

  ```sh
  dart run supa_l10n_manager merge
  ```

### Công cụ chỉnh sửa nhanh

  Supa cung cấp thêm extension Chrome để chỉnh sửa nhanh các file JSON với các chức năng sau:

  - Dịch song song các ngôn ngữ trên giao diện web
  - Hỗ trợ AI dịch tự động cho các ngôn ngữ mới
  - Xuất key từ thiết kế Figma (có quy tắc đặt tên node)

  Link tải extension: [Localization Editor on Chrome Web Store](https://chromewebstore.google.com/detail/localization-editor/eepapdeoidmlaihgncfaphiccekeghjn?hl=vi)

### Hỗ trợ tham số

  `supa_l10n_manager` hỗ trợ truyền tham số khi thực hiện dịch từ translation key sang label hiển thị cho từng ngôn ngữ.

  Cú pháp:


  ```dart
  translate('subsystem.namespace.key1[.key2]', {
    'param1': value1,
    'param2': value2,
  });
  ```

  Lưu ý:
  - Tham số truyền vào dạng map
  - Các giá trị truyền vào phải là các tên biến đơn, hạn chế tên đặc biệt, lời gọi phương thức / hàm, lời gọi các thuộc tính nhiều lớp.

  Ví dụ đúng:

  ```dart
  translate('work.taskAssignment.taskAssignmentCount', {
    'count': count,
  });
  ```

  Ví dụ sai / nên hạn chế:

  ```dart
  translate('work.taskAssignment.taskAssignmentCount', {
    'count': widget.taskAssignment.list.countLength(), /// Multiple-level property access, method call, etc.
  });
  ```

### Hạn chế:
 
  Hiện tại còn một số hạn chế:
  - Chưa hỗ trợ dạng số nhiều (plural), chia động từ (word form) trong một số ngôn ngữ
  - Các translation key là string và được tạo tùy ý, nên cần nhập theo quy tắc để hạn chế các lỗi nhập liệu, chính tả, trùng lặp

## Giao diện (Theming)

Flutter Material hỗ trợ cấu trúc app và hầu hết các widget có cấu trúc sẵn / ổn định để có thể dựng rất nhanh bố cục ứng dụng.

Developer chỉ cần:

- Điều chỉnh theme cho phù hợp với thiết kế
- Tuân thủ atomic design, tạo các widget có khả năng tùy chỉnh và sử dụng lại để ổn định giao diện cũng như tăng khả năng dễ bảo trì

### Text theme

Flutter đã quy định sẵn text theme cho toàn bộ text sử dụng trong app:

| Loại     | Vị trí sử dụng                        |
|----------|---------------------------------------|
| Title    | Tiêu đề chính trang, header           |
| Headline | Phụ đề, section title                 |
| Body     | Nội dung chính, đoạn text             |
| Label    | Text trong button, label input        |
| Display  | Text highlight, trang trí (hero text) |

- Mỗi loại style đã cung cấp sẵn 3 kích thước: `small`, `medium`, `large`

- Trong trường hợp cần customize: sử dụng phương thức `.copyWith` để tùy chỉnh. Lưu ý: chỉ dùng cho font size và color, rất hạn chế sử dụng cho các thuộc tính khác.

- Các thuộc tính line height, letter spacing sẽ quy định sẵn và cố định theo text theme, không tự tùy chỉnh.

### Quy cách spacing

#### Quy ước chung

- `margin` và `padding` là các số chia hết cho 4, một số trường hợp đặc biệt có thể sử dụng 2 (1 nửa của 4), 6 (1 nửa của 12)

- Icon size bình thường cho các button là 24 với padding 8 các bên, tổng kích thước của icon button là 40x40

- Tuân thủ theo thiết kế Figma nhưng cần linh hoạt: các đối tượng co giãn kích thước theo nội dung (ListView, Text) cần sử dụng Flexbox với Expanded, Flexible để cân đối, tránh overflow do mobile có nhiều kích thước màn hình khác nhau.


#### Spacing cho các đối tượng

- Sử dụng `margin` theo chiều dọc cho các ListItem. Chia đôi mỗi cạnh. Ví dụ: nếu thiết kế Figma có spacing giữa 2 element ListItem là 16, vậy mỗi margin cạnh sẽ là 8

- Sử dụng `padding` theo chiều ngang cho ListView.

- Nếu giữa 2 ListItem có spacing là 16, do mỗi cạnh có margin là 8, nên element đầu tiên (first) và cuối (last) sẽ chỉ cách lề trên và dưới là 8. Cần bổ sung thêm padding 8 (1 nửa spacing) cho container là ListView ở ngoài.

- Container có `padding` chung là 16, hoặc 12 hoặc 8 trong 1 số trường hợp tùy thiết kế, nhưng cần thống nhất tránh việc không đồng nhất

- Container có `BorderRadius` là 4, với `elevation` là 1 và đổ bóng theo màu sắc của border `Theme.of(context).colorScheme.outline`

### Color scheme:

Flutter cung cấp sẵn color scheme trong theme `Theme.of(context).colorScheme` với các giá trị sau:

| Key            | Chức năng                                                             |
|----------------|-----------------------------------------------------------------------|
| primary        | Màu chính của app                                                     |
| secondary      | Màu phụ, trong các button không mạnh                                  |
| tertiary       | Màu phụ thứ 3, chưa xác định rõ cách thức áp dụng                     |
| error          | Màu của thông báo lỗi, thường là đỏ nhưng có mã hex tùy theo thiết kế |
| surface        | Màu nền                                                               |
| outline        | Màu border, shadow                                                    |
| outlineVariant | Màu border, shadow ở mức nhẹ hơn (mờ)                                 |

Ngoài ra, Supa tự extends thêm các getter sau:

| Getter  | Chức năng                                                |
|---------|----------------------------------------------------------|
| success | Màu xanh lá, dùng cho các trường hợp thành công / hợp lệ |
| info    | màu xanh biển nhạt, dùng để hiển thị thông tin           |
| warning | màu cam, dùng để hiển thị cảnh báo                       |

Mỗi màu trên sẽ có màu text tương ứng với prefix `on`. Ví dụ: `onPrimary`, `onSurface`

Lưu ý: luôn luôn dùng theme, không hard code màu sắc

## Một số quy tắc, cú pháp và cấu trúc

### Translation

- Đặt tên tiếng Anh, rõ ràng, ngắn gọn
- Đối với level 3 trở lên (sau tên phân hệ và namespace), có thể sử dụng một câu rút gọn bằng tiếng Anh để thể hiện ngữ nghĩa, đồng thời hỗ trợ việc dịch bằng AI cho nhanh. VIỆC NÀY QUAN TRỌNG DO MÌNH CÓ ÍT NGUỒN LỰC.

Ví dụ: `work.error.anErrorOccurredDuringProcess`: Có lỗi xảy ra trong quá trình xử lý. dễ dịch luôn, nhìn key cũng hiểu

- KHÔNG sử dụng các từ mập mờ, đánh số không rõ ràng. Ví dụ: `tab1`, `tab2`. Tab cần có tên.

### Viết widget

- Tuân thủ atomic design, thiết kế các widget theo từng level để dễ tái sử dụng, QUAN TRỌNG HƠN là TÍNH ĐỒNG NHẤT, và DỄ SỬA

- Ưu tiên làm nhanh ra sản phẩm nhưng cần:
  + Bố cục ổn định
  + Cấu trúc mạch lạc, không nhồi nhét vào 1 file.

- Có thể dùng ChatGPT refactor sau khi code.
