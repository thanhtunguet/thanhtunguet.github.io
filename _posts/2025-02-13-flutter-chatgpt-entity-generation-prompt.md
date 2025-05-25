---
title: Generating Entity Classes for supa_architecture Using ChatGPT
date: 2025-02-13 00:01:00 +0700
categories: [Mobile Development, Flutter]
tags: [Mobile Development, Flutter]
pin: false
---

#### **Introduction**  
When working with Flutter applications, dealing with JSON serialization and deserialization can be tedious. The [`supa_architecture`](https://pub.dev/packages/supa_architecture) package simplifies this process by providing structured data types for JSON handling. However, manually creating entity classes for complex JSON structures can still be time-consuming.  

In this guide, we’ll explore how to **leverage ChatGPT** to automatically generate entity classes from JSON, streamlining development and reducing errors.  

---

## **Why Use `supa_architecture` for JSON Handling?**  
### **What is `supa_architecture`?**  
The `supa_architecture` package provides a **consistent, structured approach** to managing JSON data by using specialized types for different data fields. It simplifies serialization, deserialization, and data validation without requiring developers to manually implement constructors or JSON conversion methods.  

### **Key Features**  
- **Automatic JSON serialization** (`fromJson`, `toJson` handled internally).  
- **Typed JSON fields** (`JsonString`, `JsonInteger`, `JsonBoolean`, etc.).  
- **Seamless nested object handling** using `JsonObject<T>` and `JsonList<T>`.  
- **Null safety and default value management** are built into field types.  

---

## **Using ChatGPT to Generate Entity Classes**  

Instead of manually defining classes, **ChatGPT can generate them instantly** based on a given JSON structure. Here’s how to do it:  

### **Step 1: Provide a JSON Object**  
Let’s say you need to create an entity class for a `Product`. Your JSON might look like this:  

```json
{
    "id": 1,
    "name": "Macbook Pro 2021",
    "code": "MAC_PRO_2021",
    "statusId": 1
}
```

### **Step 2: Use the Right Prompt in ChatGPT**  
To generate the entity class correctly, use the following **optimized prompt**:  

{% raw %}
### **Act as a Flutter Developer to Convert JSON to an Entity Class**

I am working with a Flutter package named **`supa_architecture`** to handle JSON data conversion into entity classes. Your task is to convert a given JSON object into a Dart entity class using this package.

### **Package Overview**
The `supa_architecture` package provides specialized JSON data types to simplify serialization:

- **`JsonString`** → Handles string fields.  
- **`JsonInteger` / `JsonDouble` / `JsonNumber`** → Handles numerical values (`int`, `double`, `num`).  
- **`JsonDate`** → Handles `DateTime` values.  
- **`JsonBoolean`** → Handles `bool` values.  
- **`JsonObject<T>`** → Handles **nested objects**.  
- **`JsonList<T>`** → Handles **lists of objects**.  
- **`JsonModel`** → The **base class** for all entity models (includes implicit constructor and serialization methods like `fromJson` and `toJson`).

### **Entity Class Structure**
When generating an entity class:
1. **Import the package** → `import 'package:supa_architecture/supa_architecture.dart';`
2. **Extend `JsonModel`** → This base class already handles JSON serialization, so **do not** implement constructors or serialization methods.
3. **Define fields** using `supa_architecture` types (see the table above).
4. **Override the `fields` getter** to list all JSON fields that should be serialized/deserialized.
5. **Follow standard naming conventions**:
   - Use **camelCase** for field names.
   - Use **PascalCase** for entity class names.

Each field should be declared in the format:
```dart
JsonType fieldName = JsonType("jsonFieldName");
```
For example:
```dart
JsonString name = JsonString("name");
JsonInteger age = JsonInteger("age");
JsonDate birthday = JsonDate("birthday");
JsonObject<User> manager = JsonObject<User>("manager");
JsonList<User> members = JsonList<User>("members");
```

### **Handling Nested JSON**
- For **nested objects**, predict the class name and use `JsonObject<T>`.
- For **lists of objects**, predict the class name and use `JsonList<T>`.
- **Ignore third-level nested structures** (only map the first two levels).

### **Handling Unknown or Unclear Data Types**
- If a field is `null`, try to predict its data type based on its name.
- If the type **cannot be confidently determined**, list the field names at the end of the output as a comment:  
  ```dart
  // Warning: Uncertain type for field "exampleField"
  ```

### **Example Output**
Given this JSON:
```json
{
    "id": 1,
    "name": "Macbook Pro 2021",
    "code": "MAC_PRO_2021",
    "statusId": 1
}
```
The generated Dart entity class should be:
```dart
import 'package:supa_architecture/supa_architecture.dart';

class Product extends JsonModel {
    @override
    List<JsonField> get fields => [id, name, code, statusId];

    JsonInteger id = JsonInteger("id");
    JsonString name = JsonString("name");
    JsonString code = JsonString("code");
    JsonInteger statusId = JsonInteger("statusId");
}
```

### **Final Instructions**
- Ensure the output is properly formatted.
- If any field type **may be incorrect**, add a comment listing uncertain fields.

Now, generate the Dart entity class for the following JSON:

```json
{
    "id": 1,
    "name": "Macbook Pro 2021",
    "code": "MAC_PRO_2021",
    "statusId": 1
}
```

{% endraw %}

### **Step 3: Review and Use the Generated Entity**  
ChatGPT will generate the following Dart entity class:  

```dart
import 'package:supa_architecture/supa_architecture.dart';

class Product extends JsonModel {
    @override
    List<JsonField> get fields => [id, name, code, statusId];

    JsonInteger id = JsonInteger("id");
    JsonString name = JsonString("name");
    JsonString code = JsonString("code");
    JsonInteger statusId = JsonInteger("statusId");
}
```

### **Step 4: Verify and Integrate the Entity**  
- **Ensure field names match your JSON keys.**  
- **Check for potential misinterpretations of data types.**  
- **For nested objects, create separate entity classes and reference them using `JsonObject<T>` or `JsonList<T>`.**  

---

## **Handling More Complex JSON Structures**  

### **Example: Nested Objects and Lists**  
If your JSON contains nested objects and lists:  

```json
{
    "id": 1,
    "name": "Macbook Pro 2021",
    "category": {
        "id": 10,
        "title": "Laptops"
    },
    "tags": [
        {"id": 101, "name": "Apple"},
        {"id": 102, "name": "Macbook"}
    ]
}
```
  
### **Expected Output from ChatGPT**
```dart
import 'package:supa_architecture/supa_architecture.dart';

class Product extends JsonModel {
    @override
    List<JsonField> get fields => [id, name, category, tags];

    JsonInteger id = JsonInteger("id");
    JsonString name = JsonString("name");
    JsonObject<Category> category = JsonObject<Category>("category");
    JsonList<Tag> tags = JsonList<Tag>("tags");
}

class Category extends JsonModel {
    @override
    List<JsonField> get fields => [id, title];

    JsonInteger id = JsonInteger("id");
    JsonString title = JsonString("title");
}

class Tag extends JsonModel {
    @override
    List<JsonField> get fields => [id, name];

    JsonInteger id = JsonInteger("id");
    JsonString name = JsonString("name");
}
```

---

## **Best Practices for Using ChatGPT with `supa_architecture`**
✅ **Use a well-structured prompt** to ensure the AI follows the correct format.  
✅ **Manually verify** the generated entity to check for potential misinterpretations.  
✅ **Define missing or uncertain fields explicitly** in case ChatGPT cannot infer the type.  
✅ **Handle third-level nested objects separately** instead of including them in the root entity.  

---

## **Conclusion**
Using ChatGPT to generate `supa_architecture` entity classes significantly speeds up development and reduces human error in JSON serialization. By providing a **well-structured prompt**, developers can quickly convert any JSON object into a robust, maintainable Dart entity class.  

🚀 **Try this workflow today and make JSON handling in Flutter effortless!**  

---

## Raw prompt here

[https://raw.githubusercontent.com/thanhtunguet/thanhtunguet.github.io/refs/heads/main/prompts/2025-02-13-use-chatgpt-to-generate-entity-for-supa-architecture_raw_prompt.md](https://raw.githubusercontent.com/thanhtunguet/thanhtunguet.github.io/refs/heads/main/prompts/2025-02-13-use-chatgpt-to-generate-entity-for-supa-architecture_raw_prompt.md)

### **Further Reading**
- 📚 [supa_architecture Package Documentation](https://pub.dev/packages/supa_architecture)  
- 💡 [Best Practices for JSON Serialization in Flutter](https://flutter.dev/docs/development/data-and-backend/json)  
- 🛠 [Automating Code Generation in Flutter](https://medium.com/flutter)  
