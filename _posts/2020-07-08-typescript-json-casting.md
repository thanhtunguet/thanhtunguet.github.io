---
title: "TypeScript JSON Casting: Strong Typing with Decorators"
date: 2020-07-08 00:01:00 +0700
categories: ["Software Development", "Web"]
tags: ["Programming", "TypeScript", "JSON", "Decorators", "Strong Typing"]
pin: false
---

# TypeScript JSON Casting: Strong Typing with Decorators

Working with JSON in TypeScript can be challenging due to its dynamic nature. This guide explores how to achieve strong typing for JSON deserialization using decorators, making your code more robust and maintainable.

## The Problem: JSON Is Dynamically Typed

When working with JSON data from APIs, you often encounter type mismatches. For example:

```json
{
  "id": "123",
  "code": 456
}
```

In TypeScript, you might define a class:

```ts
class User {
  public id: number;
  public code: string;
}
```

However, when you deserialize the JSON:

```ts
const user = Object.assign(new User(), responseJson);
console.log(typeof user.id); // 'string' — not a number!
```

Despite using TypeScript, the runtime behavior is pure JavaScript, leading to potential bugs.

## Common (but Imperfect) Solutions

1. **Manual Casting**: Tedious and error-prone.
2. **Boilerplate Constructors**: Adds unnecessary complexity.
3. **External Libraries**: Tools like `class-transformer` can help but may have inconsistent behavior.
4. **Runtime Type Checking**: Requires additional logic, increasing code complexity.

## A Better Approach: Using Decorators

Decorators in TypeScript provide a clean and efficient way to enforce strong typing during JSON deserialization.

### Step 1: Define a Decorator

Create a decorator to map JSON fields to class properties:

```ts
function JsonProperty(key: string): PropertyDecorator {
  return (target: Object, propertyKey: string | symbol) => {
    Reflect.defineMetadata(key, propertyKey, target);
  };
}
```

### Step 2: Implement a Deserialization Function

Write a function to handle the mapping:

```ts
function deserialize<T>(cls: new () => T, json: any): T {
  const instance = new cls();
  for (const key of Object.keys(json)) {
    const propertyKey = Reflect.getMetadata(key, cls.prototype);
    if (propertyKey) {
      instance[propertyKey] = json[key];
    }
  }
  return instance;
}
```

### Step 3: Use the Decorator in Your Class

Apply the decorator to your class properties:

```ts
class User {
  @JsonProperty('id')
  public id: number;

  @JsonProperty('code')
  public code: string;
}

const user = deserialize(User, { id: "123", code: 456 });
console.log(user.id); // 123 (number)
console.log(user.code); // "456" (string)
```

## Benefits of This Approach

- **Type Safety**: Ensures correct types at runtime.
- **Clean Code**: Reduces boilerplate and improves readability.
- **Reusable Logic**: The deserialization function can be used across multiple classes.

## Conclusion

By leveraging TypeScript decorators, you can achieve strong typing for JSON deserialization, making your code more reliable and easier to maintain. This approach is particularly useful for large projects where type safety is critical.
