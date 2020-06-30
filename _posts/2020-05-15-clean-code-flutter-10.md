---
layout: post
title: Flutter TDD Clean Architecture Course [10] - Bloc Scaffolding & Input Conversion
image: https://2.bp.blogspot.com/-a9nR2A_99yg/VMXJ2cBXE2I/AAAAAAAAB_s/O_gXstMAZ9I/s1600/tdd.png
---

**Presentation layer** chứa UI dưới dạng **Widgets** và cả các bộ *presentation logic holders*, có thể được triển khai dưới dạng ChangeNotifier, Bloc, Reducer, ViewModel, MobX Store... do bạn quyết định! Trong trường hợp *Number Trivia App* của chúng tôi  , chúng tôi sẽ sử dụng  gói **flutter_bloc** để giúp chúng tôi triển khai mẫu BLoC.

# Setting Up the IDE

Trước khi tạo các tệp và lớp cần thiết cho **Bloc** , ta nên giao việc lặp đi lặp lại cho [VS Code extension](https://marketplace.visualstudio.com/items?itemName=FelixAngelov.bloc) hoặc [IntelliJ plugin](https://plugins.jetbrains.com/plugin/12129-bloc-code-generator) (nhấp vào liên kết!). Mặc dù bạn hoàn toàn có thể tự tạo tất cả các file, nhưng tốt hơn hết là chỉ cần nhấp vào nút và để tiện ích mở rộng thực hiện công việc cho bạn.

Ngoài việc thêm một cách đơn giản để tạo file Bloc , extension/plugin này còn thêm các đoạn code tiện dụng để sử dụng từ các Widget khi xây dựng giao diện người dùng.

# Events, States, Bloc and More

**Bloc**, cũng được viết là **BLoC**  là tên viết tắt của ***B**usiness **L**ogic **C**omponent*. Theo Clean Architecture, nó cũng được gọi là một PLoC  ( **P**resentation  **Lo**gic  **C**omponent) nhưng tôi nghĩ chúng tôi sẽ theo quy ước đặt tên ban đầu 😬. Rốt cuộc, tất cả business logic đều nằm trong **domain layer**.

Nếu bạn không quen thuộc với **Bloc** , bạn thực sự nên xem hướng dẫn bên dưới để có được lời giải thích đầy đủ, chuyên sâu. Đó là phiên bản cũ hơn của gói, nhưng bản chất vẫn giữ nguyên.

[https://resocoder.com/2019/06/12/bloc-library-updated-painless-state-management-for-flutter/](https://resocoder.com/2019/06/12/bloc-library-updated-painless-state-management-for-flutter/)

Tóm lại, Bloc là một state management trong đó dữ liệu chỉ truyền theo một hướng. Tất cả có thể được tách thành ba bước cốt lõi:

1. Các **Event** (chẳng hạn như "get concrete number trivia") được gửi từ các Widget UI
2. **Bloc**  nhận **Event** và thực hiện business logic phù hợp (gọi **Use Cases** , trong trường hợp áp dụng Clean Architecture). 
3. Các **State** ( chẳng hạn như chứa một NumberTrivia instance) được phát ra  từ **Bloc** return lại UI Widgets , nó sẽ hiển thị dữ liệu mới.

![Data flowing in one way without any side effects](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/04/Bloc-Diagram.png?resize=1020%2C169&ssl=1)

## Creating the Files

Trong VS Code và với extension được cài đặt, nhấp chuột phải vào thư mục **bloc** và chọn "Bloc: New Bloc" từ menu.

![](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/09/creating-bloc-files.png?w=349&ssl=1)

Đặt tên cho các file "number_trivia":

![](https://i1.wp.com/resocoder.com/wp-content/uploads/2019/09/bloc-name.png?w=509&ssl=1)

Cuối cùng chọn để sử dụng **Equatable**  để làm cho Events và States có giá trị bình đẳng ngay từ đầu.

![](https://i2.wp.com/resocoder.com/wp-content/uploads/2019/09/bloc-equatable.png?w=392&ssl=1)

Phần extendsion bây giờ sẽ tạo ra 3 tệp, từ đó mỗi tệp chứa một lớp cơ bản cho Bloc, Event và State tương ứng. Tệp thứ 4 được gọi đơn giản là bloc.dart là cái gọi là "barrel file", nó bao gồm tất cả các tệp khác. Điều này làm cho việc import dễ dàng hơn trong các phần khác của presentation layer.

## Events

File `number_trivia_event.dart` hiện chỉ chứa một lớp abstract cơ sở, từ đó tất cả Events custom của chúng tôi sẽ thừa hưởng. Những loại Event nào  mà  Widgets có thể gửi đến Bloc ? Chà, bằng cách nhìn vào UI, chỉ có 2 nút - một nút để hiển thị các câu đố cho một số cụ thể, một nút khác cho một số ngẫu nhiên.

![](https://i1.wp.com/resocoder.com/wp-content/uploads/2019/09/app-ui.png?resize=576%2C1024&ssl=1)

Vì vậy, đó là một ý tưởng tốt để có hai Events. Bạn đã đoán đúng -  *GetTriviaForConcreteNumber*  và  *GetTriviaForRandomNumber* . Đừng lo lắng, vì bây giờ sẽ có một số khác biệt trong cách chúng ta xử lý các Events đó trong Khối. Bạn sẽ thấy tại sao chỉ trong một giây.

Các **random Events** sẽ chỉ là một empty class. Các **concrete Event** có chứa một trường cho number. Điều gì nên là type của các trường? Nó có thể gây sốc cho một số bạn, nhưng loại trường number sẽ là một **String** .

```
number_trivia_event.dart
```
```
import 'package:equatable/equatable.dart';
import 'package:meta/meta.dart';

@immutable
abstract class NumberTriviaEvent extends Equatable {
  NumberTriviaEvent([List props = const <dynamic>[]]) : super(props);
}

class GetTriviaForConcreteNumber extends NumberTriviaEvent {
  final String numberString;

  GetTriviaForConcreteNumber(this.numberString) : super([numberString]);
}

class GetTriviaForRandomNumber extends NumberTriviaEvent {}
```

Các Events được gửi từ Widgets . Cái Widget mà người dùng viết một số sẽ là  `TextField`. Một giá trị được giữ trong `TextField` luôn là một String .

Chuyển đổi một chuỗi  thành một số int trực tiếp trong giao diện người dùng hoặc thậm chí bên trong lớp Event sẽ đi ngược lại với những gì chúng tôi đã cố gắng thực hiện với Clean Architecture - khả năng maintain, đọc và kiểm tra. Ồ, và chúng tôi cũng sẽ vi phạm nguyên tắc **S**OLID đầu tiên về **separation of concerns.**

*Không bao giờ đặt bất kỳ **logic business** hoặc **presentation logic** nào vào UI. Các ứng dụng Flutter đặc biệt dễ bị ảnh hưởng bởi vì mã UI cũng được viết bằng Dart.*

## InputConverter

Chúng tôi sẽ phá vỡ truyền thống của mình một chút và tạo một lớp để thực hiện chuyển đổi - **InputConverter** , mà không tạo hợp đồng lớp abstract trước. Cá nhân tôi cảm thấy rằng việc tạo hợp đồng cho các lớp  đơn giản như lớp này là không cần thiết. Thêm vào đó, vì mọi lớp trong Dart có thể được triển khai như một interface, việc mock InputConverter trong khi kiểm tra Bloc trong phần tiếp theo sẽ vẫn dễ dàng như mock một lớp trừu tượng.

Nó sẽ ở bên trong **presentation layer** rất giống với **NumberTriviaModel** ở bên trong **data layer**. Mục đích của trình chuyển đổi sẽ giống như của model - không để **domain layer** bị vướng vào thế giới bên ngoài. Các số không phải là String, giống như NumberTrivia không phải là JSON!

Chúng tôi sẽ tạo một tệp cho nó trong *core / util*. Nếu bạn muốn nghiêm ngặt trong việc tách code thành các lớp, dĩ nhiên, bạn có thể đặt nó dưới *core / presentation / util*.

[Location of the file](https://i1.wp.com/resocoder.com/wp-content/uploads/2019/09/input-converter-file.png?w=290&ssl=1)

Nó sẽ có một phương thức duy nhất gọi là  **stringToUnsignedInteger**. Đó là bởi vì ngoài việc chuyển đổi chuỗi, nó cũng sẽ đảm bảo rằng số được nhập không âm .

Để làm cho việc kiểm tra dễ dàng hơn, hãy tạo một phương thức trống cùng với **Failure** sẽ được trả về nếu số đó không hợp lệ.

```
input_converter.dart
```
```
import 'package:dartz/dartz.dart';

import '../error/failure.dart';

class InputConverter {
  Either<Failure, int> stringToUnsignedInteger(String str) {
    // TODO: Implement
  }
}

class InvalidInputFailure extends Failure {}
```

Trong tệp thử nghiệm ở vị trí được ánh xạ thông thường, sẽ không có gì để mock, vì **InputConverter**  không có bất kỳ phụ thuộc nào. Thử nghiệm đầu tiên sẽ xử lý trường hợp khi mọi thứ diễn ra suôn sẻ và trên thực tế String đầu vào là một số nguyên không dấu (nguyên dương).

```
input_converter_test.dart
```
```
import 'package:clean_architecture_tdd_prep/core/util/input_converter.dart';
import 'package:dartz/dartz.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  InputConverter inputConverter;

  setUp(() {
    inputConverter = InputConverter();
  });

  group('stringToUnsignedInt', () {
    test(
      'should return an integer when the string represents an unsigned integer',
      () async {
        // arrange
        final str = '123';
        // act
        final result = inputConverter.stringToUnsignedInteger(str);
        // assert
        expect(result, Right(123));
      },
    );
  });
}
```

Chúng tôi sẽ chỉ đơn giản là trả về chuỗi đã được phân tích cú pháp trong **Right** của **Either**.

```
input_converter.dart
```
```
Either<Failure, int> stringToUnsignedInteger(String str) {
  return Right(int.parse(str));
}
```

Tất nhiên, String hoàn toàn không phải là một số, nhưng thay vào đó, nó chứa các ký tự như 'abc' hoặc ngay cả khi nó chứa các số thập phân, phương thức sẽ trả về một **InvalidInputFailure** .

```
test.dart
```
```
test(
  'should return a failure when the string is not an integer',
  () async {
    // arrange
    final str = 'abc';
    // act
    final result = inputConverter.stringToUnsignedInteger(str);
    // assert
    expect(result, Left(InvalidInputFailure()));
  },
);
```

```
implementation.dart
```
```
Either<Failure, int> stringToUnsignedInteger(String str) {
  try {
    final integer = int.parse(str);
    if (integer < 0) throw FormatException();
    return Right(integer);
  } on FormatException {
    return Left(InvalidInputFailure());
  }
}
```

Đây là tất cả các **InputConverter**  sẽ làm trong *Number Trivia App*. Chúng ta sẽ sử dụng nó từ trong  **Bloc** trong phần tiếp theo.

## States 

**States** được  xuất ra bởi **Bloc** để kiểm soát UI. Đã có một lớp cụ thể được tạo trong file `number_trivia_state.dart` . Chỉ cần đổi tên thành **Empty** .

Trong trường hợp của chúng tôi, sẽ có bốn trạng thái - **Empty**, **Loading**, **Loaded** và **Error**. Tương tự như cách **Events** mang dữ liệu từ giao diện người dùng đến **Bloc**, **States** mang dữ liệu từ  **Bloc** đến giao diện người dùng. **Loaded** state sẽ chứa một **NumberTrivia** entity để hiển thị dữ liệu, và các **Error** state sẽ chứa một thông báo lỗi.

```
number_trivia_state.dart
```
```
import 'package:equatable/equatable.dart';
import 'package:meta/meta.dart';

import '../../domain/entities/number_trivia.dart';

@immutable
abstract class NumberTriviaState extends Equatable {
  NumberTriviaState([List props = const <dynamic>[]]) : super(props);
}

class Empty extends NumberTriviaState {}

class Loading extends NumberTriviaState {}

class Loaded extends NumberTriviaState {
  final NumberTrivia trivia;

  Loaded({@required this.trivia}) : super([trivia]);
}

class Error extends NumberTriviaState {
  final String message;

  Error({@required this.message}) : super([message]);
}
```

# Tiếp theo

Chúng tôi đã tạo Events và States cho Bloc , cùng với InputConverter class chứa presentation logic để chuyển đổi một String sang một int .

Tất nhiên, sắp tới trong phần tiếp theo là triển khai Bloc ! Điều này có nghĩa là, chúng tôi sẽ thực hiện phát triển dựa trên test-driven với Stream , bởi vì đó là những gì mẫu BLoC được xây dựng trên đầu trang. 

