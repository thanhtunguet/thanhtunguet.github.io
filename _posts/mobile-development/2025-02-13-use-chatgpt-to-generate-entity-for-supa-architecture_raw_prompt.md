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

### Raw prompt here

[https://github.com/thanhtunguet/thanhtunguet.github.io/blob/main/_posts/mobile-development/2025-02-13-use-chatgpt-to-generate-entity-for-supa-architecture_raw_prompt.md](https://github.com/thanhtunguet/thanhtunguet.github.io/blob/main/_posts/mobile-development/2025-02-13-use-chatgpt-to-generate-entity-for-supa-architecture_raw_prompt.md)
