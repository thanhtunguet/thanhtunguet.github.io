---
title: "Introducing supa_l10n_manager: A Scalable Flutter Localization Solution"
date: 2025-02-10 00:01:00 +0700
categories: ["Software Development", "Mobile"]
tags: [Mobile Development, Flutter]
pin: false
---

Localization in Flutter can be a challenging task, especially when managing translations across large apps with modularized structures. Developers often use **`flutter_localizations`** (the default approach) or **`slang_flutter`**, but both have trade-offs.

That's where **`supa_l10n_manager`** comes in. It provides a **namespace-based, scalable localization solution** with a **CLI tool** to automate translation management. In this blog, we’ll compare common approaches and explain why `supa_l10n_manager` is an excellent alternative.

---

## **Why Do We Need a Better Localization Approach?**
Localization in Flutter typically involves **storing translations**, **loading them efficiently**, and **retrieving keys dynamically**. However, traditional approaches have limitations:

| Approach                  | Pros ✅                                                                                                                                                 | Cons ❌                                                                       |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------- |
| **flutter_localizations** | ✅ Official support <br> ✅ Works well for small apps                                                                                                    | ❌ Requires manual `.arb` file management <br> ❌ Not ideal for large projects |
| **slang_flutter**         | ✅ Generates Dart code for translations <br> ✅ Type-safe access to keys                                                                                 | ❌ Requires code generation <br> ❌ Can lead to compile-time issues            |
| **supa_l10n_manager**     | ✅ Namespace-based JSON structure <br> ✅ CLI automation (merge/extract) <br> ✅ No code generation needed <br> ✅ Works well with large, modular projects | ❌ No compile-time key validation                                             |

Let’s explore each approach and why `supa_l10n_manager` is a great alternative.

---

## **Option 1: Using `flutter_localizations` (Default Approach)**

Flutter provides built-in localization support using **`.arb` files** and `flutter_localizations`. While this works for **basic translations**, it quickly becomes **hard to manage** as your app grows.

### **Setup:**
1. Enable localization in `pubspec.yaml`:
   ```yaml
   dependencies:
     flutter_localizations:
       sdk: flutter
   ```
2. Add `.arb` files inside `lib/l10n/`:
   ```
   lib/l10n/
     app_en.arb
     app_vi.arb
   ```
3. Use generated messages in Dart:
   ```dart
   import 'package:flutter_gen/gen_l10n/app_localizations.dart';

   Text(AppLocalizations.of(context)!.hello);
   ```

### **Pros & Cons**
✅ Official Flutter solution  
✅ Works well for small projects  
❌ **Requires manual `.arb` file management**  
❌ **No namespace-based organization**  
❌ **Not optimized for modular projects**  

---

## **Option 2: Using `slang_flutter` (Code-Generated Approach)**

`slang_flutter` is a **code-generation-based solution** that provides **type-safe translations**.

### **Setup:**
1. Add `slang_flutter` to `pubspec.yaml`:
   ```yaml
   dependencies:
     slang_flutter: ^3.19.0
   ```
2. Create a translation JSON file:
   ```
   assets/i18n/
     en.json
     vi.json
   ```
3. Run the code generator:
   ```bash
   dart run build_runner build
   ```
4. Use translations with type-safe access:
   ```dart
   context.t.user.login.username
   ```

### **Pros & Cons**
✅ Type-safe translations  
✅ IDE autocomplete support  
❌ Requires **code generation**  
❌ **Slower build times** in large projects  

---

## **Option 3: Using `supa_l10n_manager` (Namespace-Based JSON Management)**

`flutter_localizations` is too rigid, and `slang_flutter` requires code generation. What if you need **dynamic JSON-based localization** without extra build steps?

This is where `supa_l10n_manager` excels.

### **Key Features**
✅ **Namespace-based JSON storage**  
✅ **Automatic key extraction from Dart code**  
✅ **Merge translations into a single JSON file per locale**  
✅ **Works with Flutter’s `LocalizationsDelegate`**  
✅ **No build runner or code generation required**  

---

## **How `supa_l10n_manager` Works**

### **1. Install the Package**
```yaml
dependencies:
  supa_l10n_manager:
    git:
      url: https://github.com/thanhtunguet/supa_l10n_manager.git
```
Run:
```bash
flutter pub get
```

---

### **2. Organize Your Translations by Namespace**
Instead of a **single large JSON file**, we use **separate namespace files**:

```
assets/i18n/
  en/
    user.json
    product.json
  vi/
    user.json
    product.json
```

Example: **`assets/i18n/en/user.json`**
```json
{
  "login.username": "Username",
  "login.password": "Password"
}
```

This is **automatically loaded as a nested structure**:
```dart
{
  "user": {
    "login": {
      "username": "Username",
      "password": "Password"
    }
  }
}
```

---

### **3. Load Translations in Your App**
Before using translations, load the locale:
```dart
import 'package:supa_l10n_manager/localization.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Localization.load('en');
  runApp(MyApp());
}
```

---

### **4. Retrieve Translations with `translate()`**
```dart
import 'package:supa_l10n_manager/translator.dart';

String username = translate('user.login.username');
print(username); // Output: "Username"
```

Dynamic values:
```dart
print(translate('dashboard.welcome', {'name': 'John'}));
// Output: "Welcome, John!"
```

---

### **5. Use the CLI Tool for Automation**
#### **Merge JSON Files**
```bash
dart run bin/cli.dart merge
```
Combines `user.json` and `product.json` into a single `en.json`.

---

#### **Extract Missing Keys from Dart Code**
```bash
dart run bin/cli.dart extract --locale en --source lib
```
Finds all `translate('key')` usages and **adds missing keys to the JSON files**.

---

## **Why Choose `supa_l10n_manager`?**
| Feature                             | flutter_localizations | slang_flutter | supa_l10n_manager |
| ----------------------------------- | --------------------- | ------------- | ----------------- |
| **Namespace-based organization**    | ❌ No                  | ❌ No          | ✅ Yes             |
| **Auto key extraction**             | ❌ No                  | ❌ No          | ✅ Yes             |
| **Code generation required**        | ❌ No                  | ✅ Yes         | ❌ No              |
| **Works with large apps**           | ❌ Hard to manage      | ✅ Scalable    | ✅ Modular         |
| **Dynamic translations at runtime** | ❌ No                  | ❌ No          | ✅ Yes             |

---

## **Final Thoughts**

### **Choose `flutter_localizations` if...**
✔ You’re working on a **small app**  
✔ You don’t need **runtime flexibility**  

### **Choose `slang_flutter` if...**
✔ You want **type-safe translations**  
✔ You don’t mind **code generation overhead**  

### **Choose `supa_l10n_manager` if...**
✔ You need **modular, namespace-based translations**  
✔ You want **automatic key extraction**  
✔ You want **dynamic translations at runtime**  

---

## **🚀 Get Started Today!**
`flutter_localizations` is great but **hard to scale**, and `slang_flutter` is **too restrictive**. If you need **flexibility, automation, and modularization**, then `supa_l10n_manager` is the **best localization solution** for your Flutter app.  

🔗 **Try it now** → [GitHub Repository](https://github.com/thanhtunguet/supa_l10n_manager)  

Happy coding! 🚀
