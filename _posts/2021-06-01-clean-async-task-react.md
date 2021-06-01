---
title: Clean các task bất đồng bộ trong React
author: thanhtunguet
date: 2021-06-01 11:33:00 +0700
categories: [React]
tags: [React, Asynchronous tasks]
math: true
mermaid: true
---

# Một warning thường gặp

Nếu bạn đã làm việc với React, chắc hẳn bạn đã quen thuộc với dòng thông báo này, dù cho bạn có thể chưa hiểu rõ nó là gì:

```
Warning: Can't perform a React state update on an unmounted component.
This is a no-op, but it indicates a memory leak in your application.
To fix, cancel all subscriptions and asynchronous tasks in the componentWillUnmount
method.
```

Đây là một lỗi khá phổ biến mà người lập trình React dễ gặp phải. Bài này chúng ta cùng phân tích và đưa ra phương án giải quyết cho nguyên nhân phổ biến gây ra lỗi này: không clean các asynchronous hay subscription khi unmount React component

# Phân tích

Warning trên gói gọn trong 3 câu đã nêu lên toàn bộ vấn đề: nguyên nhân, hệ quả và cách khắc phục.

## Can't perform a React state update on an unmounted component

Warning này xuất hiện khi chúng ta xử lý các tác vụ bất đồng bộ hoặc subscription trong component và trigger việc cập nhật state sau khi tác vụ kết thúc mà không thực hiện clean chúng khi component được unmount.

Ví dụ một số hành động:

- setTimeout, setInterval, ...
- Lấy dữ liệu từ API

## This is a no-op, but it indicates a memory leak in your application

Rõ ràng, sau khi unmount thì bản thân component không còn tồn tại, việc gọi lệnh setState khi đó không những trở nên không còn ý nghĩa, mà ngược lại làm cho ứng dụng làm một việc mà chúng ta không thể kiểm soát được. Ở đây React team đã chỉ ra rằng nó sẽ gây ra sự **thất thoát bộ nhớ** (memory leak) trong ứng dụng.

Tuy nó không ảnh hưởng đến luồng hoạt động của ứng dụng, nhưng rõ ràng, việc này đi ngược lại một nguyên lý cơ bản trong lập trình và cả trong cuộc sống: _bày ra thứ gì thì phải dọn dẹp thứ đó khi xong việc_. Chắc hẳn trong chúng ta không ai muốn trở thành người vứt rác bừa bãi, vì hẳn là nó rất khó coi. Vậy chúng ta đi tìm cách dọn dẹp thôi nào.

## To fix, cancel all subscriptions and asynchronous tasks in the componentWillUnmount method

Dịch nôm: dọn dẹp các tác vụ bất đồng bộ và huỷ bỏ các ~~đăng ký~~ subscription trong method `componentWillUnmount`.

### Thế nào là asynchronous task?

Các tác vụ bất đồng bộ (asynchronous) ở đây được hiểu là các function, method dạng asynchronous: async function, promise, observable, ... Chúng có đặc điểm là không trả về kết quả ngay lập tức mà có độ trễ sau một khoảng thời gian. Ứng dụng sẽ không đợi kết quả của chúng mà tiếp tục thực hiện các lệnh khác cho đến khi có kết quả trả về mới quay lại xử lý.

Ví dụ:

- Lấy dữ liệu từ API
- Truy cập database
- Kết nối bluetooth
  ...

### Thế nào là subscription?

Subscription thường xuất hiện trong mô hình publisher - subscriber. Khi đó, component đóng vai trò subscriber và subscribe tới một publisher (có thể là database, message queue broker, ...) hoặc Observable. Việc này tất nhiên cũng tốn tài nguyên, và nếu nó nằm trong component lifecycle, nó cũng cần được huỷ bỏ (cancel) khi unmount.

Ví dụ:

- Kết quả việc gọi `subscribe` một observable, là subscription
- Kết nối tới 1 topic/channel trong RabbitMQ, Redis hay MQTT là subscription
- Listen UDP/TCP socket là subscription
- Thêm một event handler là một subscription

# Dọn dẹp asynchronous task và cancel subscription

## Nguyên tắc chung

- Sau khi có kết quả, trước khi thực hiện việc thay đổi state, cần kiểm tra component đã được update hay unmount chưa.

- Với `useEffect`, mỗi khi dependency list thay đổi, effect sẽ được thực hiện lại với một bộ dữ liệu mới. Do đó, mọi asynchronous task cần được cancel ngay tại effect đó (trả về cleanup function)

- Với những asynchronous task được tạo / gọi ra ngoài effect, ta chỉ cần gom chúng lại và thực hiện cancel khi component unmount.

## Subscription có sẵn phương thức để cancel

Và việc của chúng ta là gọi nó đúng lúc.

### Gọi method unsubscribe

Ví dụ với RxJS Observable:

```tsx
function ExampleComponent() {
  React.useEffect(() => {
    const subscription = observable.subscribe((result: Result) => {
      // handle result
    });

    return function cleanup() {
      subscription.unsubscribe();
    };
  }, []);

  render(<>{/* Component JSX */}</>);
}
```

Ví dụ với React Navigation event handler

```tsx
function ExampleComponent() {
  React.useEffect(() => {
    const unsubscribe = navigation.addListener("focus", () => {
      doSomething();
    });

    return function cleanup() {
      unsubsribe();
    };
  }, []);

  render(<>{/* Component JSX */}</>);
}
```

Với những subscription không được tạo ra trong `useEffect`, chúng ta cần gom chúng lại và cancel _một lần duy nhất_ khi component được unmount. Ta có thể sử dụng một effect không có dependency list, hoặc dependency list không thay đổi trong suốt vòng đời component để thực hiện việc này.

Ví dụ với RxJS subscription:

```tsx
function ExampleComponent() {
  const [subscription] = React.useState<Subscription>(() => new Subscription());

  const handleCallback = React.useCallback(() => {
    subscription.add(
      getData().subscriber((result) => {
        doSomething();
      });
    );
  }, []);

  React.useEffect(() => {
    return function cleanup() {
      subscription.unsubscribe();
    };
  }, []);

  render(<>{/* Component JSX */}</>);
}
```

Trong ví dụ này, subscription là một state nhưng giá trị của nó không thay đổi trong suốt vòng đời của component. Vì vậy nó sẽ chỉ được `unsubscribe` khi component unmount.

## Các asynchronous task đơn giản: Promise, async - await, callback, timeout

Các asynchronous task này không có cơ chế để huỷ bỏ, nên ta cần tạo ra một flag đánh dấu trạng thái component đã unmount hay chưa và kiểm tra nó trước khi thực hiện thay đổi state.

Xem ví dụ sau:

```tsx
function ExampleComponent() {
  const [users, setUsers] = React.useState<User[]>([]);

  React.useEffect(() => {
    let unmounted = false;

    getUsers().then((users) => {
      if (!unmounted) {
        setUsers(users);
      }
    });

    return function cleanup() {
      unmounted = true;
    };
  }, []);

  render(<>{/* Component JSX */}</>);
}
```

Ở đây chúng ta dùng một biến boolean `unmounted` để kiểm tra trạng thái component đã unmount hay chưa, khởi tạo nó với giá trị false và set nó về true khi component unmount. Khi có kết quả bất đồng bộ trả về, ta cần kiểm tra giá trị `unmounted` để quyết định có thực hiện thay đổi state hay không.

### Các asynchronous task ở ngoài Effect thì sao?

Ở đây, chúng ta không có cơ chế nào để kiểm tra trạng thái component đã được unmount hay chưa. Vì vậy, ta cần một cách tiếp cận khác. Ta vừa nói về subscription là bản thân nó có sẵn phương thức để cancel, vậy sao mình không biến tác vụ này thành một dạng subscription?

Thật đơn giản, RxJS đã hỗ trợ tốt điều đó với Observable. Ta có thể tạo ra một observable, đẩy giá trị kết quả cho nó và thực hiện subscribe và cancel giống với phần subscription đã nói ở trên.

Xem ví dụ sau:

```tsx
const handleGetData = React.useCallback(() => {
  return new Observable(async (subscriber) => {
    await getData().then((result) => {
      subscriber.next(result);
    });
  });
}, []);

React.useEffect(() => {
  const subscription = handleGetData().subscribe((result) => {
    setResult(result);
  });

  return function cleanup() {
    subsription.unsubscribe();
  };
}, []);
```

# Kết luận

Việc thực hiện dọn dẹp các tác vụ bất đồng bộ và huỷ các subscription trong component lifecycle là cần thiết để có thể tạo ra các component hoạt động ổn định và tối ưu.
