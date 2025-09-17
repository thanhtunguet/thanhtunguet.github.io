---
title: "Overcoming the Drawbacks of Official Flutter JSON Serialization with json_entity"
date: 2024-05-24 00:01:00 +0700
categories: ["Software Development", "Mobile"]
tags: ["Mobile Development", "Flutter", Programming]
pin: false
---

As a Flutter developer, one of the common challenges you face is efficient and maintainable JSON serialization. The official Flutter JSON serialization mechanism offers a solution, but it comes with its own set of drawbacks. In this blog post, we'll discuss these drawbacks and introduce `json_entity`, an alternative JSON serialization library developed by the TrueSight team, which aims to address these issues.

## Drawbacks of Official Flutter JSON Serialization

### 1. File Explosion

**Issue:**
With the official JSON serialization approach, each model source file generates a corresponding `.g.dart` file for the generated code. In large projects with numerous model files, this can lead to a significant increase in the number of files, making the project structure cumbersome and harder to manage.

**Example:**
Imagine working on a project with 500 model files. You would end up with 1000 files (500 model files + 500 generated files). This proliferation of files can clutter the project, making navigation and management a nightmare. It also complicates version control, code reviews, and collaborative development.

### 2. Re-running Code Generators

**Issue:**
Every time you make changes to a model, you need to re-run the code generator to update the generated files. For large projects, this process can be time-consuming and disrupt the development workflow.

**Example:**
In a project with 500 models, a minor change in one model requires re-running the code generator for all models. This can take a significant amount of time, especially in CI/CD pipelines, slowing down the entire development process. Developers often find themselves waiting for the code generation to complete, hampering productivity.

### 3. Increased IDE Resource Consumption

**Issue:**
The sheer number of generated files in large projects can lead to increased resource consumption by the IDE. This can result in slower code completion, suggestions, and overall reduced development efficiency.

**Example:**
When your project contains thousands of files, the IDE needs to index and manage all these files. This can slow down auto-completion and other IDE features, making coding sluggish and frustrating. The increased memory usage can also lead to frequent crashes or the need for high-end hardware, adding to development costs.

## Introducing json_entity

To address these challenges, the TrueSight team has developed `json_entity`, a library designed to streamline JSON serialization and enhance overall project management. Here's a look at how `json_entity` can help overcome the drawbacks mentioned above.

### Key Features of json_entity

- **Convenient JSON Serialization with Auto Mapper:** Simplifies the process of mapping JSON to Dart objects and vice versa.
- **Custom Filters for Backend Integration:** Provides an easy way to create and manage filters for querying backend data.
- **Built-in HTTP Request Management:** Comes with an HTTP repository pattern to handle API requests efficiently.
- **Integration with go_router for Routing:** Supports the go_router package for managing routes in your application.

### Getting Started with json_entity

To install `json_entity`, run the following command:

```bash
flutter pub add json_entity go_router intl
```

### JSON Serialization with json_entity

#### Defining a JsonModel

In `json_entity`, each model extends the `JsonModel` class. Here's an example of defining an `AppUser` model:

```dart
class AppUser extends JsonModel {
  @override
  List<JsonField> get fields => [
    username,
    password,
    email,
    isAdmin,
    dateOfBirth,
    age,
    level,
    manager,
    members,
  ];

  JsonString username = JsonString('username', isRequired: false, helper: 'Username of the user');
  JsonString password = JsonString('password', isRequired: false, helper: 'Password of the user');
  JsonString email = JsonString('email', isRequired: false, helper: 'Email of the user');
  JsonBoolean isAdmin = JsonBoolean('isAdmin', isRequired: false, helper: 'Is the user an admin');
  JsonDate dateOfBirth = JsonDate('dateOfBirth', isRequired: false, helper: 'User\'s date of birth');
  JsonInteger age = JsonInteger('age', isRequired: false, helper: 'Age of the user');
  JsonDouble level = JsonDouble('level', isRequired: false, helper: 'Level of the user');
  JsonObject<AppUser> manager = JsonObject('manager', isRequired: false, helper: 'Manager of the user');
  JsonList<AppUser> members = JsonList<AppUser>('members', isRequired: false, helper: 'Members that this user manages');
}
```

#### Mapping Data from JSON

To map data from JSON:

```dart
final json = await requestFromAPI();
AppUser user = AppUser();
user.fromJSON(json);
```

#### Converting to JSON

To convert a model to JSON:

```dart
AppUser user = AppUser();
user.toJSON(); // Returns the JSON representation of the user as a Dart Map object.
user.toString(); // Returns the JSON representation of the user as a Dart String.
```

## Conclusion

While the official Flutter JSON serialization mechanism is functional, it has notable drawbacks, especially in large projects. `json_entity` offers an alternative that addresses these issues by reducing the number of files, eliminating the need for frequent code generator runs, and improving IDE performance. `json_entity` enhances the development experience and boosts productivity.

If you are working on a large Flutter project and facing challenges with JSON serialization, give `json_entity` a try. It might be the solution you've been looking for to streamline your development workflow and improve your project's maintainability.

# Khắc Phục Những Hạn Chế của Cơ Chế JSON Serialization Chính Thức của Flutter với `json_entity`

Là một lập trình viên Flutter, một trong những thách thức phổ biến mà bạn gặp phải là việc serial hóa JSON một cách hiệu quả và dễ bảo trì. Cơ chế JSON serialization chính thức của Flutter cung cấp một giải pháp, nhưng nó cũng có những hạn chế riêng. Trong bài viết này, chúng ta sẽ thảo luận về những hạn chế này và giới thiệu `json_entity`, một thư viện JSON serialization thay thế được phát triển bởi đội ngũ TrueSight, nhằm giải quyết những vấn đề này.

## Hạn Chế của Cơ Chế JSON Serialization Chính Thức của Flutter

### 1. Sự Phát Sinh Nhiều File

**Vấn Đề:**
Với cơ chế JSON serialization chính thức, mỗi file nguồn của mô hình sẽ tạo ra một file `.g.dart` tương ứng cho mã được tạo ra. Trong các dự án lớn với nhiều file mô hình, điều này có thể dẫn đến việc tăng đáng kể số lượng file, làm cho cấu trúc dự án trở nên cồng kềnh và khó quản lý hơn.

**Ví Dụ:**
Hãy tưởng tượng làm việc trên một dự án với 500 file mô hình. Bạn sẽ kết thúc với 1000 file (500 file mô hình + 500 file được tạo ra). Sự gia tăng số lượng file này có thể làm rối cấu trúc dự án, làm cho việc điều hướng và quản lý trở thành một cơn ác mộng. Nó cũng làm phức tạp việc kiểm soát phiên bản, đánh giá mã và phát triển hợp tác.

### 2. Chạy Lại Các Trình Tạo Mã

**Vấn Đề:**
Mỗi khi bạn thay đổi một mô hình, bạn cần phải chạy lại trình tạo mã để cập nhật các file được tạo ra. Đối với các dự án lớn, quá trình này có thể tốn nhiều thời gian và làm gián đoạn quy trình phát triển.

**Ví Dụ:**
Trong một dự án với 500 mô hình, một thay đổi nhỏ trong một mô hình đòi hỏi phải chạy lại trình tạo mã cho tất cả các mô hình. Điều này có thể mất một khoảng thời gian đáng kể, đặc biệt là trong các pipeline CI/CD, làm chậm toàn bộ quá trình phát triển. Các lập trình viên thường phải chờ đợi cho quá trình tạo mã hoàn tất, làm giảm năng suất.

### 3. Tăng Tiêu Thụ Tài Nguyên Của IDE

**Vấn Đề:**
Số lượng file được tạo ra nhiều trong các dự án lớn có thể dẫn đến tăng tiêu thụ tài nguyên của IDE. Điều này có thể dẫn đến việc hoàn thành mã chậm hơn, gợi ý mã chậm hơn và tổng thể hiệu suất phát triển giảm.

**Ví Dụ:**
Khi dự án của bạn chứa hàng ngàn file, IDE cần phải lập chỉ mục và quản lý tất cả các file này. Điều này có thể làm chậm auto-completion và các tính năng khác của IDE, làm cho việc lập trình trở nên chậm chạp và bực bội. Việc tăng sử dụng bộ nhớ cũng có thể dẫn đến các vụ sập IDE thường xuyên hoặc cần phần cứng cao cấp, tăng thêm chi phí phát triển.

## Giới Thiệu `json_entity`

Để giải quyết những thách thức này, đội ngũ TrueSight đã phát triển `json_entity`, một thư viện thiết kế để đơn giản hóa JSON serialization và nâng cao quản lý dự án tổng thể. Dưới đây là cái nhìn về cách `json_entity` có thể giúp vượt qua những hạn chế đã nêu ở trên.

### Các Tính Năng Chính của `json_entity`

- **JSON Serialization Tiện Lợi Với Auto Mapper:** Đơn giản hóa quá trình ánh xạ JSON sang các đối tượng Dart và ngược lại.
- **Bộ Lọc Tuỳ Chỉnh Cho Tích Hợp Backend:** Cung cấp cách dễ dàng để tạo và quản lý các bộ lọc cho việc truy vấn dữ liệu backend.
- **Quản Lý Yêu Cầu HTTP Tích Hợp Sẵn:** Đi kèm với mô hình repository HTTP để xử lý các yêu cầu API một cách hiệu quả.
- **Tích Hợp Với `go_router` Cho Quản Lý Routing:** Hỗ trợ gói `go_router` để quản lý các tuyến đường trong ứng dụng của bạn.

### Bắt Đầu Với `json_entity`

Để cài đặt `json_entity`, chạy lệnh sau:

```bash
flutter pub add json_entity go_router intl
```

### JSON Serialization Với `json_entity`

#### Định Nghĩa Một JsonModel

Trong `json_entity`, mỗi mô hình mở rộng lớp `JsonModel`. Dưới đây là ví dụ về việc định nghĩa một mô hình `AppUser`:

```dart
class AppUser extends JsonModel {
  @override
  List<JsonField> get fields => [
    username,
    password,
    email,
    isAdmin,
    dateOfBirth,
    age,
    level,
    manager,
    members,
  ];

  JsonString username = JsonString('username', isRequired: false, helper: 'Tên đăng nhập của người dùng');
  JsonString password = JsonString('password', isRequired: false, helper: 'Mật khẩu của người dùng');
  JsonString email = JsonString('email', isRequired: false, helper: 'Email của người dùng');
  JsonBoolean isAdmin = JsonBoolean('isAdmin', isRequired: false, helper: 'Người dùng có phải là admin không');
  JsonDate dateOfBirth = JsonDate('dateOfBirth', isRequired: false, helper: 'Ngày sinh của người dùng');
  JsonInteger age = JsonInteger('age', isRequired: false, helper: 'Tuổi của người dùng');
  JsonDouble level = JsonDouble('level', isRequired: false, helper: 'Cấp độ của người dùng');
  JsonObject<AppUser> manager = JsonObject('manager', isRequired: false, helper: 'Quản lý của người dùng');
  JsonList<AppUser> members = JsonList<AppUser>('members', isRequired: false, helper: 'Các thành viên mà người dùng quản lý');
}
```

#### Ánh Xạ Dữ Liệu Từ JSON

Để ánh xạ dữ liệu từ JSON:

```dart
final json = await requestFromAPI();
AppUser user = AppUser();
user.fromJSON(json);
```

#### Chuyển Đổi Sang JSON

Để chuyển đổi một mô hình sang JSON:

```dart
AppUser user = AppUser();
user.toJSON(); // Trả về đại diện JSON của người dùng dưới dạng một đối tượng Map của Dart.
user.toString(); // Trả về đại diện JSON của người dùng dưới dạng một chuỗi Dart.
```

## Kết Luận

Trong khi cơ chế JSON serialization chính thức của Flutter có hiệu quả, nó có những hạn chế đáng chú ý, đặc biệt trong các dự án lớn. `json_entity` cung cấp một giải pháp thay thế giải quyết các vấn đề này bằng cách giảm số lượng file, loại bỏ nhu cầu chạy lại trình tạo mã thường xuyên, và cải thiện hiệu suất của IDE. `json_entity` nâng cao trải nghiệm phát triển và tăng năng suất.

Nếu bạn đang làm việc trên một dự án Flutter lớn và gặp phải những thách thức với JSON serialization, hãy thử sử dụng `json_entity`. Nó có thể là giải pháp bạn đang tìm kiếm.
