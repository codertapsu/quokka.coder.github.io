---
layout: post
title: Flutter TDD Clean Architecture Course [3] - Domain Layer Refactoring
image: https://2.bp.blogspot.com/-a9nR2A_99yg/VMXJ2cBXE2I/AAAAAAAAB_s/O_gXstMAZ9I/s1600/tdd.png
---

*Ứng dụng Number Trivia  của chúng tôi đang được xây dựng. Trong phần trước, chúng tôi đã tạo một **Entity** ,  **Repository contract**  và **Use Case - GetConcreteNumberTrivia** bằng cách sử dụng phát triển theo hướng TDD. Hôm nay, chúng tôi sẽ thêm một **Use Case**  khác*

# Callable Classes

Bạn có biết rằng trong Dart, một phương thức có tên **call**  có thể được chạy bằng cách gọi  `object.call ()`  hoặc là `object ()` ? Đó là phương pháp hoàn hảo để sử dụng trong **Use Cases** ! Xét cho cùng , tên lớp của chúng đã là các động từ như  **GetConcreteNumberTrivia** , vì vậy sử dụng chúng làm "phương thức giả" hoàn toàn phù hợp.

Theo tinh thần của TDD, trước tiên chúng tôi sẽ modify test ( get_concittle_number_test.dart ) để không còn gọi  phương thức thực hiện :

``final result = await usecase(number: tNumber);``

Và vì mã thậm chí không biên dịch, chúng tôi có thể sửa đổi  lớp **GetConcittleNumberTrivia**:

``Future<Either<Failure, NumberTrivia>> call({ ...``  

# Thêm Use Case khác

Ngoài việc nhận được những câu chuyện cho một con số cụ thể , ứng dụng của chúng tôi cũng sẽ nhận được những  câu chuyện cho một số ngẫu nhiên . Điều này có nghĩa là chúng ta cần một UC khác - **GetRandomNumberTrivia** . **Numbers API** chúng tôi đang sử dụng thực sự có một thiết bị đầu cuối API khác nhau cho số cố định và ngẫu nhiên, vì vậy chúng tôi sẽ không tạo con số đó. Mặt khác, mã tạo số sẽ được thực thi chính xác bên trong **domain layer** trong UC **GetRandomNumberTrivia**  . Tạo số là một **business logic**.

## UseCase Base Class
Là những clean coder, chúng tôi chắc chắn code phải trực quan. Các phương thức và thuộc tính public của các lớp giống nhau nên có các tên được tiêu chuẩn hóa.

Khi nói đến **Use Case** , mỗi một trong số chúng  nên  có một  phương thức **call**  . Sẽ không có vấn đề gì nếu logic bên trong **Use Case** đưa chúng ta đến **NumberTrivia**  hoặc gửi tàu con thoi tới Mặt trăng LOL, interface phải  giống nhau để tránh mọi sự nhầm lẫn.

Một cách để thực thi một interface ổn định của một lớp là dựa vào " keyword của lập trình viên ". Đáng buồn thay, các lập trình viên không nổi tiếng vì nhớ mọi thứ. Heck, tôi thậm chí xem một số hướng dẫn cũ của tôi bởi vì tôi đã quên làm thế nào để làm một số điều mà tôi đã biết trước đây!

Một cách khác để ngăn chặn một lớp có một  phương thức **call**  và một  phương thức thực thi khác  là cung cấp một inteface rõ ràng (trong trường hợp của Dart là một abstract class ), không thể quên được xuất phát từ đó.  Ví dụ như một **UseCase** base class.

Đoạn mã sau sẽ đi vào  **core/usecases** , vì lớp này có thể được chia sẻ trên nhiều tính năng của một ứng dụng. Và tất nhiên, không có điểm nào trong việc kiểm tra một **abstract class**, vì vậy chúng ta có thể viết nó ngay lập tức:

```
usecase.dart
```
```
import 'package:dartz/dartz.dart';
import 'package:equatable/equatable.dart';

import '../error/failure.dart';

// Parameters have to be put into a container object so that they can be
// included in this abstract base class method definition.
abstract class UseCase<Type, Params> {
  Future<Either<Failure, Type>> call(Params params);
}

// This will be used by the code calling the use case whenever the use case
// doesn't accept any parameters.
class NoParams extends Equatable {}
```

## Extending the Base Class

Như bạn có thể thấy, chúng tôi đã thêm hai tham số cho  lớp **UseCase**  . Một là cho loại trả về "không có lỗi", trong trường hợp ứng dụng của chúng tôi sẽ là **NumberTrivia** entity . Tham số khác,  **Params** , sẽ gây ra một số thay đổi mã nhỏ trong UC GetConcreteNumberTrivia  đã có  .

Mỗi UseCase extend base class sẽ định nghĩa các tham số được truyền vào method **call** là một lớp riêng biệt trong cùng một file. Trong khi chúng ta ở đó, chúng ta cũng extend UseCase . Những thay đổi khác liên quan đến hoạt động của lớp sẽ chỉ đến sau khi cập nhật bài kiểm tra - chúng tôi đang thực hiện TDD!

```
get_concrete_number_trivia.dart
```
```
class GetConcreteNumberTrivia extends UseCase<NumberTrivia, Params> {
  ...
}

class Params extends Equatable {
  final int number;

  Params({@required this.number}) : super([number]);
}
```

Bây giờ chúng ta biết rằng **call** phải nhận một **Params** object, thay vì truyền integer là tham số trực tiếp. Vì vậy, vì chúng tôi đã viết một test trong phần trước, chúng tôi có thể sử dụng nó để tự tin vào mã của mình. Tất cả chúng ta phải làm là:

- Cập nhật test để sử dụng Params.
- Nó sẽ không được biên dịch.

```
get_concrete_number_trivia_test.dart
```
```
...
test(
  'should get trivia for the number from the repository',
  () async {
    // arrange
    when(mockNumberTriviaRepository.getConcreteNumberTrivia(any))
        .thenAnswer((_) async => Right(tNumberTrivia));
    // act
    final result = await usecase(Params(number: tNumber));
    // assert
    expect(result, Right(tNumberTrivia));
    verify(mockNumberTriviaRepository.getConcreteNumberTrivia(tNumber));
    verifyNoMoreInteractions(mockNumberTriviaRepository);
  },
);
```

- Cập nhật code. Sử dụng **Params**  làm tham số cho  phương thức **call**  .
- Chạy thử nghiệm - nó sẽ pass, ok.

```
get_concrete_number_trivia.dart
```
```
class GetConcreteNumberTrivia extends UseCase<NumberTrivia, Params> {
  ...
  @override
  Future<Either<Failure, NumberTrivia>> call(Params params) async {
    return await repository.getConcreteNumberTrivia(params.number);
  }
}
...
```

# GetRandomNumberTrivia

Việc thêm **Use Case** mới này bây giờ rất đơn giản - chúng tôi đã định nghĩa một interface mà mỗi **UseCase**  phải có. Ngoài ra, do tính chất đơn giản của  Ứng dụng **Number Trivia App** , UC mới này sẽ chỉ nhận dữ liệu từ repository.

Chúng tôi lại bắt đầu bằng cách viết test- tạo một tệp mới trong thư mục **"test /.../ usecase"**. Hầu hết code được sao chép từ test cho UC trước đó.

```
get_random_number_trivia_test.dart
```
```
import 'package:clean_architecture_tdd_prep/core/usecase/usecase.dart';
import 'package:clean_architecture_tdd_prep/features/number_trivia/domain/entities/number_trivia.dart';
import 'package:clean_architecture_tdd_prep/features/number_trivia/domain/repositories/number_trivia_repository.dart';
import 'package:clean_architecture_tdd_prep/features/number_trivia/domain/usecases/get_random_number_trivia.dart';
import 'package:dartz/dartz.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

class MockNumberTriviaRepository extends Mock
    implements NumberTriviaRepository {}

void main() {
  GetRandomNumberTrivia usecase;
  MockNumberTriviaRepository mockNumberTriviaRepository;

  setUp(() {
    mockNumberTriviaRepository = MockNumberTriviaRepository();
    usecase = GetRandomNumberTrivia(mockNumberTriviaRepository);
  });

  final tNumberTrivia = NumberTrivia(number: 1, text: 'test');

  test(
    'should get trivia from the repository',
    () async {
      // arrange
      when(mockNumberTriviaRepository.getRandomNumberTrivia())
          .thenAnswer((_) async => Right(tNumberTrivia));
      // act
      // Since random number doesn't require any parameters, we pass in NoParams.
      final result = await usecase(NoParams());
      // assert
      expect(result, Right(tNumberTrivia));
      verify(mockNumberTriviaRepository.getRandomNumberTrivia());
      verifyNoMoreInteractions(mockNumberTriviaRepository);
    },
  );
}
```

Tất nhiên, test này không thành công và việc triển khai  **GetRandomNumberTrivia**  như sau:

```
get_random_number_trivia.dart
```
```
import 'package:dartz/dartz.dart';

import '../../../../core/error/failure.dart';
import '../../../../core/usecase/usecase.dart';
import '../entities/number_trivia.dart';
import '../repositories/number_trivia_repository.dart';

class GetRandomNumberTrivia extends UseCase<NumberTrivia, NoParams> {
  final NumberTriviaRepository repository;

  GetRandomNumberTrivia(this.repository);

  @override
  Future<Either<Failure, NumberTrivia>> call(NoParams params) async {
    return await repository.getRandomNumberTrivia();
  }
}
```

Thử nghiệm bây giờ sẽ pass và cùng với đó, chúng tôi mới thực hiện đầy đủ **domain layer** của  Ứng dụng *Number Trivia App* . Trong phần tiếp theo, chúng tôi sẽ bắt đầu làm việc trên  **data layer** có chứa **Repository** implementation **Data Sources**
