---
layout: post
title: Dart Extension Methods Tutorial
---
Từ bản phát hành Dart 2.6, các nhà phát triển đã ra mắt một tính năng thú vị đó là **Extension**. Ta hãy đi tìm hiểu nó.

# Set up Dart 2.6

Có thể khi đọc bài viết này, Dart 2.6 đã sẵn sàng cho việc phát hành chính thức. Nếu thế thì thật tuyệt. Nếu không thì bạn hãy tự cài đặt bằng tay nhé.

```
pubspec.yaml
environment:
  sdk: '>=2.6.0 <3.0.0'

```

# Tại sao lại là Extension ?

Hầu như mọi ngôn ngữ nào cũng hỗ trợ extension, mang lại nhiều lợi ích cho coder. Chúng có thể biến các lớp utility với một loạt các static method thành một *tác phẩm nghệ thuật tuyệt đẹp* của riêng bạn.

Ví dụ chúng ta có đoạn mã sau:

```
main.dart
```
```
class StringUtil {
  static bool isValidEmail(String str) {
    final emailRegExp = RegExp(r"^[a-zA-Z0-9.]+@[a-zA-Z0-9]+\.[a-zA-Z]+");
    return emailRegExp.hasMatch(str);
  }
}

// Usage
main() {
  StringUtil.isValidEmail('someString');
}
```

Có vẻ sử dụng StringUtil class là dư thừa. Nếu chúng ta có thể viết như sau thì sao:

``'someString'.isValidEmail;``

# Extension giải quyết vấn đề

Thay vì định nghĩa một lớp util, bạn có thể định nghĩa một `extension` nó sẽ được applied (`on`) một type nhất định. Sau đó sử dụng `this` để có được `instance` hiện tại như thể bạn đang ở trong một regular class thông thường.

```
main.dart
```
```
extension StringExtensions on String {
  bool get isValidEmail {
    final emailRegExp = RegExp(r"^[a-zA-Z0-9.]+@[a-zA-Z0-9]+\.[a-zA-Z]+");
    return emailRegExp.hasMatch(this);
  }
}

// Usage
main() {
  'someString'.isValidEmail;
}
```

# Nhiều extensions

Hãy tạo hai String Extensions giống hệt nhau để nối chuỗi với nhau bằng khoảng trắng:

```
main.dart
```
```
extension StringExtensions on String {
  String concatWithSpace(String other) {
    return '$this $other';
  }

  /// DOCUMENTATION IS SUPPORTED: Concatenates two strings with a space in between.
  String operator &(String other) => '$this $other';
}
```

Sử dụng nó lúc này thật đơn giản:

```
main.dart
```
```
main() {
  'one'.concatWithSpace('two');
  'one' & 'two';
}
```

# Vấn đề về thừa kế

Giả sử bạn muốn thêm extensions vào `int`. Tất nhiên việc này có thể:

```
main.dart
```
```
extension IntExtensions on int {
  int addTen() => this + 10;
}
```

Nhưng sau đó bạn lại muốn áp dụng nó cho `double`. Vậy bạn lại phải copy code thật mất thời gian.

Bạn lại chợt nhận ra rằng `int` và `double` đều là lớp con của `num`, vậy bạn sẽ định nghĩa một extension cho `num`:

```
main.dart
```
```
extension NumExtensions on num {
  num addTen() => this + 10;
}
```

Có vẻ như bạn đã giải quyết vấn đề. Nhưng... đó chỉ là bạn chưa chạy thử nó. Vì method trong extension có giá trị trả về là `num` nên nếu bạn gọi đến method đó, bạn cũng chỉ nhận được `num`  mà thôi:

```
main.dart
```
```
main() {
  int anInt = 1.addTen();
  // Run-time error!
  // Putting a 'num' which is really a 'double' into an 'int' variable
  int shouldBeDouble = 1.0.addTen();
}
```

*Việc định nghĩa extension cho các base class có thể dẫn đến lỗi run-time, chẳng hạn: `TypeError`: "type `'double'` is not a subtype of type `'int'`"*

# Generic extensions

Ta sẽ giải quyết vấn đề kế thừa bằng generic type:

```
main.dart
```
```
extension NumGenericExtensions<T extends num> on T {
  T addTen() => this + 10;
}
```

Generics sẽ giúp câu lệnh `int anInt = 1.addTen();` hoạt động đúng. Nhưng hãy lưu ý là câu lệnh sau lại lỗi:


```
main.dart
```
```
main() {
  // Compile-time error!
  int shouldBeDouble = 1.0.addTen();
}
```

# Bạn đã học được gì?

Extension là một tính năng mạnh mẽ của Dart:
- Bạn đã học cách tạo extension **properties**, **method** and **operators**
- Cách giải quyết việc copy code bằng cách định nghĩa một extension trên lớp cơ sở có thể không phải luôn là một lựa chọn tốt nhất. Trong hầu hết các trường hợp bạn nên sử dụng **generic extensions**