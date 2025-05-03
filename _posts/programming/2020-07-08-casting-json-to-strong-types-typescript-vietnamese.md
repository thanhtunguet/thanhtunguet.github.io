---
title: "Giải thích kĩ thuật thư viện react3l: chuyển đổi kiểu JSON chuẩn xác"
date: 2020-07-08 00:02:00 +0700
categories: ["Programming", "Javascript"]
tags: [Programming, Javascript, "Node.js"]
pin: false
---

Trong các ngôn ngữ lập trình mạnh về kiểu dữ liệu như Java hoặc C#, việc ánh xạ dữ liệu từ backend sang các object với kiểu dữ liệu chính xác là một việc hiển nhiên và được hỗ trợ sẵn thông qua cơ chế serialization/deserialization. Nhưng trong JavaScript/TypeScript — vốn sử dụng hệ thống prototype-based inheritance — việc đảm bảo dữ liệu từ backend có đúng kiểu tại runtime lại không hề đơn giản. Nếu bạn từng gặp lỗi `undefined is not a function` hoặc `x.toLowerCase is not a function` khi thao tác trên một chuỗi tưởng chừng là string, bạn đã từng đối mặt với vấn đề này.

## Vấn đề: TypeScript không đảm bảo kiểu tại runtime

TypeScript cung cấp type-checking trong quá trình biên dịch (compile-time), nhưng khi dữ liệu được nhận từ backend thông qua JSON, mọi thứ chỉ đơn giản là `object literal` — không có prototype, không có method, không có constructor, và dĩ nhiên, không có loại (type) nào được áp dụng. Đây là nguyên nhân khiến các kiểu như `Date`, `Moment`, `Custom Model`, hoặc kể cả `number`, `string`, `boolean` đôi khi hoạt động sai khi bạn cố gắng thao tác trên chúng ngay sau khi deserialize từ JSON.

## Javascript không giống Java: Vấn đề với prototype inheritance

Trong JavaScript, kế thừa được thực hiện thông qua prototype chain, không phải class-based inheritance như Java/C#. Khi bạn khai báo một class trong TypeScript và khởi tạo bằng `new`, instance sẽ được gắn với prototype của class, bao gồm các method và getter/setter đã khai báo.

Tuy nhiên, khi bạn deserialize một object từ JSON (ví dụ dùng `JSON.parse()` hoặc lấy từ `fetch()`), object đó **không** có prototype. Nó chỉ là một plain object — mất toàn bộ logic getter/setter, method, decorator, v.v. Đây là lý do bạn **không thể** tin tưởng dữ liệu từ backend trong TypeScript nếu bạn không làm gì thêm để xử lý.

## Giải pháp: "Magic" decorator + runtime casting

Thư viện `react3l` ra đời để giải quyết chính vấn đề trên: làm sao để ánh xạ dữ liệu từ backend và tự động đảm bảo kiểu dữ liệu tại runtime bằng một cách rõ ràng, dễ hiểu và dễ maintain.

### Ý tưởng chính:

1. **Decorators** dùng để annotate các field với kiểu cần cast.
2. **Prototype registry** lưu lại `PropertyDescriptor` ban đầu cho mỗi field (gồm getter/setter).
3. **AutoModel decorator** sẽ re-apply lại các property descriptors này trong constructor để đảm bảo prototype hoạt động đúng sau khi deserialize.

Ví dụ:

```ts
@AutoModel()
class User extends Model {
  @Field(Number)
  public id: number;

  @Field(String)
  public name: string;

  @MomentField()
  public createdAt: Moment;

  @ObjectField(Role)
  public role: Role;

  @ObjectList(Permission)
  public permissions: Permission[];
}
```

Bạn chỉ cần:

```ts
const user = Object.assign(new User(), rawJson);
```

Là toàn bộ logic getter/setter sẽ được "khôi phục" để cast giá trị đúng kiểu, ví dụ: `user.createdAt` luôn là `Moment`, `user.role` là instance của `Role`, `user.permissions` là array các instance `Permission`, v.v.

### Tại sao cần bước **re-apply** trong `AutoModel`?

Decorators trong TypeScript được áp dụng tại thời điểm **khai báo class**, chứ không phải mỗi lần bạn khởi tạo instance. Vì thế khi bạn gán dữ liệu JSON vào instance bằng `Object.assign()`, các setter ban đầu sẽ bị ghi đè mất. `AutoModel` sẽ lấy lại các descriptor từ prototype registry và re-apply lại cho từng field, đảm bảo behavior getter/setter được khôi phục.

Đây là điểm **then chốt** giúp kỹ thuật “magic” này hoạt động đúng và bền vững.

## Tổng kết

Bằng cách kết hợp `reflect-metadata`, `PropertyDescriptor`, và hệ thống decorators của TypeScript, thư viện này mang lại khả năng casting dữ liệu mạnh mẽ tại runtime mà vẫn giữ được code rõ ràng, dễ maintain và dễ mở rộng. Đây là cách đưa tính năng “serialization mạnh như Java/C#” vào TypeScript — và nó hoàn toàn có thể sử dụng tốt trong production.
