---
layout: post
title: Flutter TDD Clean Architecture Course [2]
image: /img/hello_world.jpeg
---

Trong phần đầu tiên, bạn đã học các khái niệm cốt lõi của **Clean Architecture** trong Flutter. Chúng ta cũng đã tạo một loạt các empty folder cho các lớp **presentation**, **domain** và **data** trong project *Number Trivia App*. Bây giờ đã đến lúc bắt đầu thêm code vào các folder đó, tất nhiên  là sử dụng TDD.

# Bắt đầu từ đâu?

Bất cứ khi nào xây dựng một ứng dụng có UI, bạn nên thiết kế UX/UI trước. Tôi đã hoàn thành bài tập về nhà đó cho bạn và ứng dụng đã có mẫu ở phần trước.

Quá trình coding sẽ diễn ra từ bên trong, từ lớp ổn định nhất của Architecture outwards. Có nghĩa là chúng ta sẽ triển khai **domain layer** và bắt đầu bằng **Entity**. Trước khi làm điều đó, chúng ta phải thêm một số package dependencies vào **pubspec.yaml**. Tôi không muốn đụng đến tập tin này nhiều lần, nên tôi sẽ thêm mọi thứ vào luôn. 

```
pubspec.yaml
```
```
dependencies:
  flutter:
    sdk: flutter
  # Service locator
  get_it: ^2.0.1
  # Bloc for state management
  flutter_bloc: ^0.21.0
  # Value equality
  equatable: ^0.4.0
  # Functional programming thingies
  dartz: ^0.8.6
  # Remote API
  connectivity: ^0.4.3+7
  http: ^0.12.0+2
  # Local cache
  shared_preferences: ^0.5.3+4

dev_dependencies:
  flutter_test:
    sdk: flutter
  mockito: ^4.1.0
``` 

# Entity

*Number Trivia App* sẽ hoạt động theo kiểu dữ liệu nào ? Đó là các **NumberTrivia** entities. Để xem trường nào mà lớp này phải có, chúng ta phải xem phản hồi từ API. Ví dụ: [http://numbersapi.com/42?json](http://numbersapi.com/42?json).

```
answer.json
```
```
{
  "text": "42 is the answer to the Ultimate Question of Life, the Universe, and Everything.",
  "number": 42,
  "found": true,
  "type": "trivia"
}
```

Chúng ta chỉ cần  quan tâm đến trường *text* và *number*. Nếu không tìm thấy số:

```
not_found.json
```
```
{
  "text": "123456 is an unremarkable number.",
  "number": 123456,
  "found": false,
  "type": "trivia"
}
```

**NumberTrivia** là  một trong số ít các lớp mà chúng ta sẽ không viết theo cách hướng TDD vì nó không có gì để kiểm tra. Nó sẽ extends **Equitable** để cho phép so sánh giá trị dễ dàng mà không cần các boilerplate vì mặc định Dart chỉ hỗ trợ referential equality.

```
number_trivia.dart
```
```
import 'package:equatable/equatable.dart';
import 'package:meta/meta.dart';

class NumberTrivia extends Equatable {
  final String text;
  final int number;

  NumberTrivia({
    @required this.text,
    @required this.number,
  }) : super([text, number]);
}
``` 

# Use Case

**Use Case** là nơi để business logic được thực thi. Sẽ không có nhiều logic trong project này, tất cả những gì mà **Use Case** phải làm là lấy dữ liệu từ **Repository**. Chúng ta sẽ có 2 trong số đó: **GetConcreteNumberTrivia** và **GetRandomNumberTrivia**.

![](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/08/domain-layer-diagram.png?w=141&ssl=1)

## Luồng dữ liệu và xử lý lỗi

Chúng ta nhận thấy rằng **Use Case** sẽ lấy entity **NumberTrivia** từ **Repository** và nó sẽ chuyển entity này đến **Presentation Layer**. Vì vậy, loại được return bởi **Use Case** phải là *Future<NumberTrivia>* để cho phép asynchrony, right ?

Còn lỗi thì sao ? Chúng ta sẽ catch exceptions càng sớm càng tốt trong **Repository** và  sau đó return về **Failure** objects từ method được đề cập.

OKay, let's recap, tóm tắt lại nào: **Repository** và **Use case** sẽ return cả 2 objects là **NumberTrivia** và **Failure** từ method của chúng. Nhưng làm sao có thể ? **functional programming** có hỗ trợ điều này.

#### The Either Type

[dartz package](https://pub.dev/packages/dartz) mà ta đã thêm vào như là một dependancy cung cấp **functional programming** (FP) cho Dart. Chúng ta sẽ dùng <L, R> trong package này để xử lý lỗi. 

Loại này có thể được sử dụng để đại diện cho 2 loại dữ liệu cùng một lúc và nó hoàn hảo để xử lý lỗi, trong đó **L** là **Error** và **R** là **NumberTrivia**. Theo cách này, các **Failure** không có "error folow" như exceptions. Chúng sẽ được sử dụng như mọi dữ liệu khác mà không cần **try/catch**. Chúng ta sẽ đề cập cách làm việc chi tiết của **Either** khi chúng ta cần nó ở phần sau.

#### Defining Failures

Trước khi chúng ta tiến hành viết **Use Case**, ta phải xác định **Failures**, vì nó sẽ là một phần  của kiểu return **Either**. **Failures** sẽ được sử dụng trên nhiều feature và layer nên hãy tạo nó trong **core** folder, dưới new **error** folder. 

![](https://i2.wp.com/resocoder.com/wp-content/uploads/2019/08/error-folder.png?w=201&ssl=1)

Ta sẽ tạo một base abstract **Failure** class mà từ đó bất kỳ failure cụ thể nào sẽ *derived*, giống như với các trường hợp exceptions thông thường và base **Exception** class.

```
failures.dart
```
```
import 'package:equatable/equatable.dart';

abstract class Failure extends Equatable {
  // If the subclasses have some properties, they'll get passed to this constructor
  // so that Equatable can perform value comparison.
  Failure([List properties = const <dynamic>[]]) : super(properties);
}
```

Tạm đến đây đã, chúng ta sẽ  xác định các **Failure** cụ thể hơn như **ServerFailure** trong các phần tiếp theo.

#### Repository Contract

Nhớ lại phần trước và cũng được đề cập ở trên, **Repository** thuộc cả lớp **domain** và **data**, **Use Case** sẽ lấy dữ liệu từ nó. Nói chính xác hơn, nó được xác định trong **domain** (còn gọi là "hợp đồng"), trong khi việc thực hiện là trong **data**.

Điều này cho phép hoàn toàn độc lập với lớp **domain**, và nó cũng phù hợp cho khả năng **testability**.

Tạo một *hợp đồng* của **Repository**, trong Dart nó sẽ là một **abstract class**, sẽ cho phép chúng ta viết các bài test theo TDD cho các **Use Case** mà không cần triển khai **Repository *implementation***.

Testing mà không cần implementation các lớp là có thể với package **mocking**.

Vậy "contract" sẽ như thế nào ? Nó sẽ có 2 method: một để lấy trivia data cụ thể, một nữa để lấy trivia data random. Chúng sẽ return lại **Future<Either<Failure, NumberTrivia>>**, đảm bảo việc xử lý lỗi diễn ra nhanh chóng và dễ dàng.

```
number_trivia_Vposeective.dart
```
```
import 'package:dartz/dartz.dart';

import '../../../../core/error/failure.dart';
import '../entities/number_trivia.dart';

abstract class NumberTriviaRepository {
  Future<Either<Failure, NumberTrivia>> getConcreteNumberTrivia(int number);
  Future<Either<Failure, NumberTrivia>> getRandomNumberTrivia();
}
```

##### GetConcreteNumberTrivia

Mặc dù phần này khá dài, nhưng tôi không muốn bạn bị *bỏ con giữa chợ*. Chúng ta sẽ viết một số test trong khi triển khai use case **GetConcittleNumberTrivia**. Trong phần tiếp theo, chúng ta sẽ thêm use case **GetRandomNumberTrivia**, hãy theo dõi nhé.

Như TDD, chúng ta sẽ viết test trước khi viết production code. Điều này đảm bảo rằng chúng ta sẽ không thêm một loạt những thứ "**ain't gonna need**" và mã sẽ không bị phá vỡ như hiệu ứng domino.

**Writing the Test**

Trong Dart, các Test sẽ ở trong **test** folder và đó là một custom để cho **test folders** map tới **lib folders**. Hãy tạo tất cả các folder giống như folders gốc và cũng bao gồm cả folder "use case" trong "domain".

![The test we're about to write goes into the usecases folder](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/08/test-folder-structure.png?w=252&ssl=1)

Tạo một file mới trong *use case test folder* đặt tên là **get_concrete_number_trivia_test.dart**, và cũng ở đó nhưng trong *lib folder* ta tạo một **get_concrete_number_trivia.dart**.

*Các tệp test được viết bằng TDD luôn map tới các tệp production và thêm "_test" vào cuối tên của chúng.*

Hãy set up test đầu tiên. Chúng ta biết rằng **Use Case** sẽ lấy dữ liệu của nó từ **NumberTriviaRepository**. Chúng ta sẽ mock nó, vì chúng ta chỉ có một lớp abstract cho nó và cũng bởi vì mock cho phép kiểm tra khi method được gọi.

```
get_concittle_number_trivia_test.dart
```
```
import 'package:clean_architecture_tdd_prep/features/number_trivia/domain/repositories/number_trivia_repository.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

class MockNumberTriviaRepository extends Mock
    implements NumberTriviaRepository {}
```

Để hoạt động với **NumberTriviaRepository** instance, use case **GetConcreteNumberTrivia** sẽ truyền nó qua một constructor. Tests trong Dart có một method tiện dụng là *setUp* chạy trước mỗi test riêng lẻ. Đó là nơi chúng ta sẽ khởi tạo các object.

**NOTE:** Mã chúng ta sẽ có lỗi vì chưa có lớp **GetConcreteNumberTrivia**

```
get_concittle_number_trivia_test .dart
```
```
import 'package:clean_architecture_tdd_prep/features/number_trivia/domain/repositories/number_trivia_repository.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

class MockNumberTriviaRepository extends Mock
    implements NumberTriviaRepository {}

void main() {
  GetConcreteNumberTrivia usecase;
  MockNumberTriviaRepository mockNumberTriviaRepository;

  setUp(() {
    mockNumberTriviaRepository = MockNumberTriviaRepository();
    usecase = GetConcreteNumberTrivia(mockNumberTriviaRepository);
  });
}
```

Mặc dù chúng ta chưa thực sự viết bất kỳ test nào, nhưng hiện tại ta có thể bắt đầu viết production code rồi. Tôi sẽ tạo một bộ khung cho **GetConcreteNumberTrivia** class, để mã setup trên sẽ không có lỗi:

```
get_concittle_number_trivia.dart
```
```
import '../repositories/number_trivia_repository.dart';

class GetConcreteNumberTrivia {
  final NumberTriviaRepository repository;

  GetConcreteNumberTrivia(this.repository);
}
```

Bây giờ đến lúc cho bài test thực tế. Vì bản chất của *Number Trivia App* rất đơn giản, thực tế sẽ không có nhiều logic trong **Use Case**, thực tế thì nó không có logic thực sự nào cả. Nó sẽ chỉ nhận dữ liệu từ **Repository**.

Do đó, test đầu tiên và duy nhất sẽ đảm bảo rằng **Repository** thực sự được gọi và dữ liệu  chỉ đơn giản passes không bị thay đổi qua **Use Case**.

```
get_concittle_number_trivia_test .dart
```
```
import 'package:clean_architecture_tdd_prep/features/number_trivia/domain/entities/number_trivia.dart';
import 'package:clean_architecture_tdd_prep/features/number_trivia/domain/repositories/number_trivia_repository.dart';
import 'package:clean_architecture_tdd_prep/features/number_trivia/domain/usecases/get_concrete_number_trivia.dart';
import 'package:dartz/dartz.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

class MockNumberTriviaRepository extends Mock
    implements NumberTriviaRepository {}

void main() {
  GetConcreteNumberTrivia usecase;
  MockNumberTriviaRepository mockNumberTriviaRepository;

  setUp(() {
    mockNumberTriviaRepository = MockNumberTriviaRepository();
    usecase = GetConcreteNumberTrivia(mockNumberTriviaRepository);
  });

  final tNumber = 1;
  final tNumberTrivia = NumberTrivia(number: 1, text: 'test');

  test(
    'should get trivia for the number from the repository',
    () async {
      // "On the fly" implementation of the Repository using the Mockito package.
      // Khi getConcreteNumberTrivia được gọi với bất kỳ arg nào, luôn luôn answer bằng
      // the Right "side" of Either có chứa test NumberTrivia object.
      when(mockNumberTriviaRepository.getConcreteNumberTrivia(any))
          .thenAnswer((_) async => Right(tNumberTrivia));
      //  giai đoạn "act" của test. Call the not-yet-existent method.
      final result = await usecase.execute(number: tNumber);
      // UseCase chỉ cần trả về bất cứ gì từ Repository
      expect(result, Right(tNumberTrivia));
      // Xác minh rằng method đã được gọi trong Repository
      verify(mockNumberTriviaRepository.getConcreteNumberTrivia(tNumber));
      // chỉ gọi phương thức trên và không có gì nữa.
      verifyNoMoreInteractions(mockNumberTriviaRepository);
    },
  );
}
```

Nếu chạy test bạn sẽ gặp lỗi, tất nhiên. Chúng ta sẽ tiếp tục implementation.

Tất cả những gì ta cần thêm vào use case **GetConcreteNumberTrivia** là *following function* nó sẽ thực hiện mọi thứ theo đúng bài test.

```
get_concrete_number_trivia.dart
```
```
import 'package:dartz/dartz.dart';
import 'package:meta/meta.dart';

import '../../../../core/error/failure.dart';
import '../entities/number_trivia.dart';
import '../repositories/number_trivia_repository.dart';

class GetConcreteNumberTrivia {
  final NumberTriviaRepository repository;

  GetConcreteNumberTrivia(this.repository);

  Future<Either<Failure, NumberTrivia>> execute({
    @required int number,
  }) async {
    return await repository.getConcreteNumberTrivia(number);
  }
}
```

Hãy chạy lại test, nó sẽ pass. Và với điều đó, chúng tôi vừa viết **Use Case** đầu tiên của *Number Trivia App* sử dụng TDD.

Trong phần tiếp theo, chúng ta sẽ cùng nhau refactor lại  đoạn mã trên. Tạo một base **Use Case** class để làm cho ứng dụng dễ dàng mở rộng và thêm use case **GetRandomNumberTrivia** 