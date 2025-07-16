---
title: "Hướng Dẫn Dùng Gemini Để Chuyển Đổi Đề Thi Trắc Nghiệm Sang Excel"
date: 2025-07-16 08:00:00 +0700
categories: ["Artificial Intelligence", "LLMs"]
tags: [AI, Gemini, Prompt Engineering, Automation, Vietnamese]
pin: false
---

### Hướng Dẫn Chuyển Đổi Đề Thi Trắc Nghiệm Từ Word Sang Excel Siêu Tốc Với Gemini

Trong công tác đào tạo và giảng dạy, việc biên soạn và quản lý ngân hàng câu hỏi là một công việc thường xuyên. Các bộ đề thi trắc nghiệm thường được soạn thảo trên Word, nhưng khi cần nhập liệu lên các hệ thống quản lý học tập (LMS), tạo bài kiểm tra trên Google Forms hay các nền tảng khác, chúng ta lại cần định dạng file Excel (bảng tính).

Quá trình chuyển đổi thủ công từ Word sang Excel không chỉ tốn thời gian mà còn dễ xảy ra sai sót. May mắn thay, với sự trợ giúp của các mô hình AI như Gemini, bạn có thể tự động hóa công việc này chỉ trong vài giây.

Bài viết này sẽ hướng dẫn bạn cách sử dụng một "câu lệnh thần kỳ" (prompt) để yêu cầu Gemini thực hiện việc chuyển đổi một cách chính xác và chuyên nghiệp.

#### Bước 1: Chuẩn Bị "Câu Lệnh Thần Kỳ" (System Prompt)

Để Gemini hiểu rõ vai trò và yêu cầu của bạn, hãy ra lệnh cho nó bằng một bộ chỉ dẫn (prompt) thật chi tiết. Đây chính là chìa khóa để AI trả về kết quả đúng như mong đợi. Bạn chỉ cần sao chép và sử dụng lại câu lệnh dưới đây mỗi khi có nhu cầu.

**Hãy sao chép toàn bộ nội dung trong khung dưới đây:**

```text
Bạn là chuyên viên nhập liệu hỗ trợ các tác vụ liên quan đến đào tạo:

- Biên soạn câu hỏi, đề thi
- Chuyển đổi câu hỏi, đề thi từ dạng file Word (doc, docx) sang dạng bảng tính (Excel)

Thông thường, 1 bài quiz là tập hợp nhiều câu hỏi trắc nghiệm với 4 đáp án (A, B, C, D) trong đó có một đáp án đúng.

Nếu người dùng yêu cầu bạn chuyển đổi tệp câu hỏi từ định dạng Word sang Excel, bạn hãy trả kết quả dạng bảng tính để người dùng có thể dễ dàng sao chép (Copy) sang Google Sheets hoặc Excel.

Tệp kết quả bao gồm các cột sau:

- STT (Số thứ tự)
- Câu hỏi
- Đáp án A
- Đáp án B
- Đáp án C
- Đáp án D
- Đáp án đúng
- Ghi chú (mặc định để trống)

Dựa vào các yêu cầu trên, hãy chuyển đổi trực tiếp và nhanh chóng mà không hỏi lại người dùng, trừ khi được người dùng yêu cầu làm rõ!
```

**Tại sao câu lệnh này hiệu quả?**
*   **Xác định vai trò:** "Bạn là chuyên viên nhập liệu..." giúp Gemini hiểu rõ vai trò cần thực hiện.
*   **Mô tả nhiệm vụ:** Nêu rõ bối cảnh và định dạng của dữ liệu đầu vào (câu hỏi trắc nghiệm từ Word).
*   **Chỉ định định dạng đầu ra:** Yêu cầu rõ ràng kết quả phải là "dạng bảng tính" và liệt kê chi tiết 8 cột cần có.
*   **Tăng tốc độ:** Câu lệnh "hãy chuyển đổi trực tiếp và nhanh chóng mà không hỏi lại" giúp bạn nhận được kết quả ngay lập tức.

#### Bước 2: Chuẩn Bị Dữ Liệu Câu Hỏi

Mở tệp Word chứa bộ câu hỏi của bạn và sao chép (Copy) toàn bộ nội dung.

Ví dụ, nội dung của bạn có thể trông như thế này:

```
Câu 1: Đâu là thủ đô của Việt Nam?
A. Đà Nẵng
B. Hà Nội
C. Hải Phòng
D. TP. Hồ Chí Minh
Đáp án đúng: B

Câu 2: Ngôn ngữ lập trình nào được Google phát triển cho Flutter?
A. Java
B. Swift
C. Kotlin
D. Dart
Đáp án đúng: D
```

#### Bước 3: Gửi Yêu Cầu Tới Gemini

Bây giờ, hãy kết hợp câu lệnh và dữ liệu của bạn. Mở Gemini và dán nội dung theo cấu trúc sau:

1.  Dán **"Câu Lệnh Thần Kỳ"** đã sao chép ở Bước 1.
2.  Xuống dòng hai lần (nhấn Enter 2 lần).
3.  Dán **dữ liệu câu hỏi** đã sao chép ở Bước 2.

Yêu cầu đầy đủ của bạn sẽ trông như thế này:

> Bạn là chuyên viên nhập liệu hỗ trợ các tác vụ liên quan đến đào tạo... (và các phần còn lại của câu lệnh)
>
> ---
>
> Câu 1: Đâu là thủ đô của Việt Nam?
> A. Đà Nẵng
> B. Hà Nội
> C. Hải Phòng
> D. TP. Hồ Chí Minh
> Đáp án đúng: B
>
> Câu 2: Ngôn ngữ lập trình nào được Google phát triển cho Flutter?
> A. Java
> B. Swift
> C. Kotlin
> D. Dart
> Đáp án đúng: D

Sau khi gửi yêu cầu, Gemini sẽ xử lý và trả về kết quả dưới dạng bảng Markdown.

#### Bước 4: Sao Chép Kết Quả vào Excel hoặc Google Sheets

Gemini sẽ trả về một bảng như sau:

| STT | Câu hỏi                                                    | Đáp án A | Đáp án B | Đáp án C  | Đáp án D        | Đáp án đúng | Ghi chú |
|:----|:-----------------------------------------------------------|:---------|:---------|:----------|:----------------|:------------|:--------|
| 1   | Đâu là thủ đô của Việt Nam?                                | Đà Nẵng  | Hà Nội   | Hải Phòng | TP. Hồ Chí Minh | B           |         |
| 2   | Ngôn ngữ lập trình nào được Google phát triển cho Flutter? | Java     | Swift    | Kotlin    | Dart            | D           |         |

Việc cuối cùng bạn cần làm là:
1.  Bôi đen và sao chép (Copy) toàn bộ bảng kết quả này.
2.  Mở một trang tính Excel hoặc Google Sheets mới.
3.  Dán (Paste) vào ô A1. Dữ liệu sẽ tự động được điền vào các cột tương ứng một cách hoàn hảo.

### Mẹo Nhỏ Để Đạt Kết Quả Tốt Nhất

*   **Định dạng câu hỏi đơn giản:** Khi sao chép từ Word, hãy đảm bảo định dạng câu hỏi và đáp án rõ ràng, nhất quán.
*   **Kiểm tra lại đáp án:** AI rất thông minh nhưng không phải lúc nào cũng hoàn hảo. Hãy luôn rà soát lại cột "Đáp án đúng" để đảm bảo tính chính xác.
*   **Chia nhỏ dữ liệu:** Nếu bạn có một bộ đề quá dài (vài trăm câu), hãy cân nhắc chia th��nh nhiều phần nhỏ để xử lý hiệu quả hơn.

Chúc bạn thành công và tiết kiệm được nhiều thời gian trong công việc!
