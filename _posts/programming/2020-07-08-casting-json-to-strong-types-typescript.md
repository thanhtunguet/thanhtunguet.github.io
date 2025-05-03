---
title: "Casting JSON to Strong Types in TypeScript Using Decorators"
date: 2020-07-08 00:01:00 +0700
categories: ["Programming", "Javascript"]
tags: [Programming, Javascript, "Node.js"]
pin: false
---

> *"Can we get Java/C#-like runtime typing in JavaScript land?"*

If you’ve worked with strongly typed languages like Java or C#, you’re used to **type-safe deserialization** — when JSON is mapped to class instances, all fields automatically cast to their correct types. You write `new User(json)` and everything *just works*. But in JavaScript and TypeScript, things aren't quite that clean.

---

## ❗ The Problem: JSON Is Dynamically Typed

Imagine you're working with JSON data from a backend API:

```json
{
  "id": "123",
  "code": 456
}
```

In TypeScript, you might define:

```ts
class User {
  public id: number;
  public code: string;
}
```

You assume `user.id` is a number, but when you do:

```ts
const user = Object.assign(new User(), responseJson);
console.log(typeof user.id); // ❌ 'string' — not a number!
```

Despite using TypeScript, this is **pure JavaScript at runtime**. There's no built-in mechanism to **cast or enforce types** on instance properties when deserializing from JSON.

---

## 🧪 Common (but brittle) Solutions

* Manually casting every property on assignment
* Writing boilerplate constructors
* Using external libraries (e.g. `class-transformer`) with inconsistent behavior
* Introducing runtime type checking logic manually

But wouldn’t it be great if you could write something like this?

```ts
@AutoModel()
class User {
  @Field(Number)
  public id: number;

  @Field(String)
  public code: string;
}
```

…and let your class automatically cast JSON properties correctly?

---

## 🧠 A Better Way: Decorators + Metadata

We leverage **TypeScript decorators** and the `reflect-metadata` package to associate transformation logic with each property.

You provide a casting function (like `Number`, `String`, or even a custom model), and the `@Field`, `@ObjectField`, `@List`, etc., decorators store a descriptor that:

* Intercepts `.get` and `.set`
* Reads/writes from metadata
* Applies the proper cast automatically

So setting `user.id = '123'` behind the scenes actually does:

```ts
Reflect.defineMetadata('__RAW_VALUE__', Number('123'), this, 'id');
```

This gives you full control over the data type *and* storage format — and you can even support things like nested models and lists.

---

## 👨‍🏫 Why JavaScript Prototype Inheritance Makes This Tricky

Here’s the catch: **decorators define property descriptors on the class’s prototype**, not the instance. That’s just how JavaScript’s prototype chain works.

Compare this with Java or C#:

| Feature                | JavaScript (Prototype-Based) | Java/C# (Class-Based) |
|------------------------|------------------------------|-----------------------|
| Instance field type    | Not enforced                 | Enforced at runtime   |
| Property assignment    | Dynamic                      | Type-safe             |
| Decorators/annotations | Prototype-bound              | Instance-aware        |

So when you define a decorated field in JS:

```ts
@Field(Number)
public id: number;
```

The descriptor for `id` is added to the prototype. But at runtime, when you assign:

```ts
user.id = '123';
```

…the instance has no idea that `id` was supposed to be a `Number`. Worse, if `Reflect.getMetadata(...)` expects per-instance data, this setup breaks completely.

---

## ✨ The Magic: Re-Applying Descriptors at Runtime

Here’s where your genius comes in.

You created the `@AutoModel()` class decorator, which *wraps the class constructor* and re-applies all property descriptors to the **instance**:

```ts
return class extends constructor {
  constructor(...rest: any[]) {
    super(...rest);

    const basePrototype = BasePrototype.getOrCreate(constructor);

    Object.entries(basePrototype.propertyDescriptors).forEach(
      ([prop, desc]) => {
        Object.defineProperty(this, prop, desc);
      }
    );
  }
};
```

This fixes everything.

✅ Each instance gets its own set of runtime-aware property descriptors
✅ Decorators behave consistently
✅ Metadata is preserved per instance
✅ JSON can be safely deserialized, even for nested models and lists

---

## 🏗️ Your Decorator System in Action

You now have a suite of decorators to handle all runtime typing needs:

### 🧩 Primitive Fields

```ts
@Field(Number)
public id: number;
```

### 🧱 Object Fields

```ts
@ObjectField(() => Role)
public role: Role;
```

### 🧾 List of Objects

```ts
@ObjectList(() => Permission)
public permissions: Permission[];
```

### 🕒 Moment Date Fields

```ts
@MomentField()
public createdAt: Moment;
```

Every field is automatically cast when setting a value, whether during JSON deserialization or manual assignment.

---

## 🧵 Final Thoughts

This pattern bridges the gap between static and dynamic typing in TypeScript — using:

* Decorators to describe field-level type transformations
* `reflect-metadata` to track values safely
* A runtime "re-patching" step to bring everything together per instance

It’s elegant, powerful, and brings **strong typing to JavaScript object instantiation**, just like in Java/C#. This is a game-changer for building robust data models in frontend applications.
