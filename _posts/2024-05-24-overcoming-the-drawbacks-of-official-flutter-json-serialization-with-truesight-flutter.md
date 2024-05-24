---
title: "Overcoming the Drawbacks of Official Flutter JSON Serialization with truesight_flutter"
date: 2024-05-24 00:01:00 +0700
categories: [Mobile Development, "Flutter"]
tags: ["Mobile Development", "Flutter"]
pin: false
---

As a Flutter developer, one of the common challenges you face is efficient and maintainable JSON serialization. The official Flutter JSON serialization mechanism offers a solution, but it comes with its own set of drawbacks. In this blog post, we'll discuss these drawbacks and introduce `truesight_flutter`, an alternative JSON serialization library developed by the TrueSight team, which aims to address these issues.

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

## Introducing truesight_flutter

To address these challenges, the TrueSight team has developed `truesight_flutter`, a library designed to streamline JSON serialization and enhance overall project management. Here's a look at how `truesight_flutter` can help overcome the drawbacks mentioned above.

### Key Features of truesight_flutter

- **Convenient JSON Serialization with Auto Mapper:** Simplifies the process of mapping JSON to Dart objects and vice versa.
- **Custom Filters for Backend Integration:** Provides an easy way to create and manage filters for querying backend data.
- **Built-in HTTP Request Management:** Comes with an HTTP repository pattern to handle API requests efficiently.
- **Integration with go_router for Routing:** Supports the go_router package for managing routes in your application.

### Getting Started with truesight_flutter

To install `truesight_flutter`, run the following command:

```bash
flutter pub add truesight_flutter go_router intl
```

### JSON Serialization with truesight_flutter

#### Defining a DataModel

In `truesight_flutter`, each model extends the `DataModel` class. Here's an example of defining an `AppUser` model:

```dart
class AppUser extends DataModel {
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

### Advanced Filters

Similar to JSON fields, `truesight_flutter` provides various filter classes for querying backend data:

```dart
class UserFilter extends DataFilter {
  StringFilter username = StringFilter('username');
  DateFilter dateOfBirth = DateFilter('dateOfBirth');
  IntFilter age = IntFilter('age');
}
```

### Managing HTTP Requests

`truesight_flutter` simplifies HTTP request management with the `HttpRepository` class:

```dart
@singleton
class UserRepository extends HttpRepository {
  @override
  InterceptorsWrapper interceptorWrapper = InterceptorsWrapper();

  @override
  String? get baseUrl => 'https://app.example.com';

  Future<AppUser> login(String username, String password) {
    return post("/login", data: {
      'username': username,
      'password': password,
    }).then((response) => response.body<AppUser>(AppUser));
  }
}
```

## Conclusion

While the official Flutter JSON serialization mechanism is functional, it has notable drawbacks, especially in large projects. `truesight_flutter` offers an alternative that addresses these issues by reducing the number of files, eliminating the need for frequent code generator runs, and improving IDE performance. By integrating features like auto-mapping, advanced filters, and streamlined HTTP request management, `truesight_flutter` enhances the development experience and boosts productivity.

If you are working on a large Flutter project and facing challenges with JSON serialization, give `truesight_flutter` a try. It might be the solution you've been looking for to streamline your development workflow and improve your project's maintainability.
