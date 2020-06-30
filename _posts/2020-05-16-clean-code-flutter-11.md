---
layout: post
title: Flutter TDD Clean Architecture Course [11] - Bloc Implementation 1/2
image: https://2.bp.blogspot.com/-a9nR2A_99yg/VMXJ2cBXE2I/AAAAAAAAB_s/O_gXstMAZ9I/s1600/tdd.png
---

**presentation logic holder** chúng ta sẽ sử dụng trong *Number Trivia App* là BLoC. Chúng ta đã thiết lập Event và State cho nó trong phần trước. Bây giờ là lúc để bắt đầu kết hợp tất cả lại với nhau để thực hiện phát triển dựa trên thử nghiệm với Dart's Streams.

# Setup

Chắc chắn, chúng ta có Event và State đã có thể sử dụng được từ trong `NumberTriviaBloc` nhưng chúng ta cũng phải suy nghĩ về những phụ thuộc mà nó sẽ có.

Từ một Bloc (hoặc bất kỳ khác presentation logic holder​) là ranh giới giữa lớp *domain* và *presentation*, nó sẽ phụ thuộc vào hai *use case* mà chúng tôi đã có. Sau đó, tất nhiên, nó cũng sẽ sử dụng InputConverter  được tạo trong phần trước.

[](https://i1.wp.com/resocoder.com/wp-content/uploads/2019/09/usecase-to-ploc-diagram-small.png?w=149&ssl=1)

Trước tiên, hãy tạo ra constructor cùng với các trường thuộc tính. Chúng tôi sẽ làm một số **thủ thuật** xây dựng với `null` kiểm tra và phân công trường bên ngoài của dấu ngoặc nhọn:

```
number_trivia_bloc.dart
```
```
class NumberTriviaBloc extends Bloc<NumberTriviaEvent, NumberTriviaState> {
  final GetConcreteNumberTrivia getConcreteNumberTrivia;
  final GetRandomNumberTrivia getRandomNumberTrivia;
  final InputConverter inputConverter;

  NumberTriviaBloc({
    // Changed the name of the constructor parameter (cannot use 'this.')
    @required GetConcreteNumberTrivia concrete,
    @required GetRandomNumberTrivia random,
    @required this.inputConverter,
    // Asserts are how you can make sure that a passed in argument is not null.
    // We omit this elsewhere for the sake of brevity.
  })  : assert(concrete != null),
        assert(random != null),
        assert(inputConverter != null),
        getConcreteNumberTrivia = concrete,
        getRandomNumberTrivia = random;

  @override
  NumberTriviaState get initialState => Empty();

  @override
  Stream<NumberTriviaState> mapEventToState(
    NumberTriviaEvent event,
  ) async* {
    // TODO: Add Logic
  }
}
```


Các tập tin thử nghiệm sẽ như thường lệ, ở trong một ví trí được ánh xạ, có nghĩa là *test / features / number_trivia / presentation / bloc*. Hãy thiết lập nó với Mock:

```
number_trivia_bloc_test.dart
```
```
class MockGetConcreteNumberTrivia extends Mock
    implements GetConcreteNumberTrivia {}

class MockGetRandomNumberTrivia extends Mock implements GetRandomNumberTrivia {}

class MockInputConverter extends Mock implements InputConverter {}

void main() {
  NumberTriviaBloc bloc;
  MockGetConcreteNumberTrivia mockGetConcreteNumberTrivia;
  MockGetRandomNumberTrivia mockGetRandomNumberTrivia;
  MockInputConverter mockInputConverter;

  setUp(() {
    mockGetConcreteNumberTrivia = MockGetConcreteNumberTrivia();
    mockGetRandomNumberTrivia = MockGetRandomNumberTrivia();
    mockInputConverter = MockInputConverter();

    bloc = NumberTriviaBloc(
      concrete: mockGetConcreteNumberTrivia,
      random: mockGetRandomNumberTrivia,
      inputConverter: mockInputConverter,
    );
  });
}
```

*Nếu bạn đang tự hỏi về việc import, có khá nhiều trong số chúng. Kiểm tra repo [GitHub](https://github.com/ResoCoder/flutter-tdd-clean-architecture-course) cho toàn bộ dự án cùng với import.*

# Initial State

Thử nghiệm đầu tiên khá đơn giản và thực tế, nó đã được thực hiện! 😱 Đúng vậy, chúng ta đang phá vỡ nguyên tắc TDD ở đây, vì các code được tạo ra bởi các Bloc extendsion cho VS Mã.

```
test.dart
```
```
test('initialState should be Empty', () {
  // assert
  expect(bloc.initialState, equals(Empty()));
});
```

Và như bạn dự đoán,  `initialState` return `Empty()`.

```
implementation.dart
```
```
@override
NumberTriviaState get initialState => Empty();
```

# Event-Driven Testing

Tất cả logic của Bloc được thực thi trong `mapEventToState()` method. Điều này có nghĩa là để kiểm tra Bloc, chúng ta phải giả định các widget UI  bằng cách gửi Events thích hợp ngay từ thử nghiệm.

Trong phần này, chúng tôi sẽ bắt đầu thử nghiệm với sự kiện `GetTriviaForConcreteNumber`, vì vậy chúng tôi sẽ tạo một thử nghiệm có cùng tên. Chúng ta cũng hãy thiết lập các biến chúng ta sẽ kiểm tra trong một group:

```
test.dart
```
```
group('GetTriviaForConcreteNumber', () {
  // The event takes in a String
  final tNumberString = '1';
  // This is the successful output of the InputConverter
  final tNumberParsed = int.parse(tNumberString);
  // NumberTrivia instance is needed too, of course
  final tNumberTrivia = NumberTrivia(number: 1, text: 'test trivia');
}
```

## Ensuring Validation & Conversion

Điều quan trọng nhất xảy ra khi a `GetTriviaForConcreteNumber` được gửi đi, là đảm bảo String nhận được từ UI là một số nguyên dương hợp lệ . Thông qua sự phụ thuộc, chúng tôi đã có logic cần thiết cho việc xác thực và chuyển đổi này - nó nằm trong `InputConverter`. Vì điều này, chúng tôi có thể tuân thủ nguyên tắc *single responsibility*, giả định rằng **InputConverter** được thực hiện thành công và mock nó như bình thường .

Thử nghiệm đầu tiên sẽ chỉ xác minh rằng `InputConverter` method thực tế đã được gọi.

```
test.dart
test(
  'should call the InputConverter to validate and convert the string to an unsigned integer',
  () async {
    // arrange
    when(mockInputConverter.stringToUnsignedInteger(any))
        .thenReturn(Right(tNumberParsed));
    // act
    bloc.dispatch(GetTriviaForConcreteNumber(tNumberString));
    await untilCalled(mockInputConverter.stringToUnsignedInteger(any));
    // assert
    verify(mockInputConverter.stringToUnsignedInteger(tNumberString));
  },
);
```

Như thường lệ, chạy thử nghiệm và nó sẽ thất bại. Chúng tôi sẽ thực hiện nó trong bước tiếp theo.

*Chúng tôi `await untilCalled()` bởi vì logic bên trong Bloc được kích hoạt thông qua `Stream<Event>`, tất nhiên, nó không đồng bộ. Vì vậy, chúng ta không chờ đợi cho đến khi `stringToUnsignedInteger` đã được gọi, xác minh sẽ luôn luôn thất bại , vì chúng ta muốn xác minh trước khi mã đã có cơ hội để thực thi.*

```
implementation.dart
```
```
@override
Stream<NumberTriviaState> mapEventToState(
  NumberTriviaEvent event,
) async* {
  // Immediately branching the logic with type checking, in order
  // for the event to be smart casted
  if (event is GetTriviaForConcreteNumber) {
    inputConverter.stringToUnsignedInteger(event.numberString);
  }
}
```

## Invalid Input Failure

Nếu chuyển đổi thành công , code sẽ tiếp tục với việc lấy dữ liệu từ use case `GetConcreteNumberTrivia`, sẽ được kiểm tra kỹ lưỡng trong các thử nghiệm tiếp theo. Tuy nhiên, trước tiên, hãy giải quyết những gì xảy ra khi chuyển đổi không thành công. Trong trường hợp đó,  trách nhiệm của `NumberTriviaBloc` là phải cho UI biết điều gì đã xảy ra bằng cách phát ra một `Error` State.

`Error` class cần một thông báo lỗi để được thông qua. Chúng ta sẽ bỏ qua một chút và tạo hằng số cho tất cả các message, chỉ cần như vậy mà chúng ta sẽ không được pass ngay từ đầu. Sẽ có một thông điệp cho mỗi `Failure` có thể xảy ra bên trong các phụ thuộc của `NumberTriviaBloc`. Đặt code này vào đầu tệp:

```
number_trivia_bloc.dart
```
```
const String SERVER_FAILURE_MESSAGE = 'Server Failure';
const String CACHE_FAILURE_MESSAGE = 'Cache Failure';
const String INVALID_INPUT_FAILURE_MESSAGE =
    'Invalid Input - The number must be a positive integer or zero.';
```

Để đưa logic được mô tả ở trên vào một thử nghiệm, chúng tôi sẽ sử dụng một cách thử nghiệm khác, so với những gì chúng tôi đã sử dụng, nó phù hợp với `Streams`.

Cho đến bây giờ, tất cả các phương thức chúng tôi đã kiểm tra cho một giá trị đều tự trả về giá trị đó . Ví dụ, gọi `InputConverter.stringToUnsignedInteger()` return `Either<Failure, int>`. Ngay cả các phương thức return một `Future` là cách dễ dàng để xử lý - chỉ `await` nó và bạn đã thiết lập.

Với Bloc, bạn gọi `dispatch` với một `Event` để thực hiện logic, nhưng `dispatch` chính nó trả về `void`. Các giá trị thực tế được  phát ra từ một nơi hoàn toàn khác  - từ `Stream` bên trong một `State` (trường của `Bloc`). Tôi biết đoạn này có lẽ quá trừu tượng để tự hiểu, mọi thứ sẽ trở nên rõ ràng hơn với một bài kiểm tra:

```
test.dart
```
```
test(
  'should emit [Error] when the input is invalid',
  () async {
    // arrange
    when(mockInputConverter.stringToUnsignedInteger(any))
        .thenReturn(Left(InvalidInputFailure()));
    // assert later
    final expected = [
      // The initial state is always emitted first
      Empty(),
      Error(message: INVALID_INPUT_FAILURE_MESSAGE),
    ];
    expectLater(bloc.state, emitsInOrder(expected));
    // act
    bloc.dispatch(GetTriviaForConcreteNumber(tNumberString));
  },
);
```

Chúng tôi tạo một danh sách các `States` mà chúng tôi dự kiến ​​sẽ được phát ra và sau đó thiết lập một khung kiểm tra mà đôi khi trong tương lai (`expectLater`) Stream sẽ phát ra các giá trị từ `List` theo thứ tự chính xác với `emitsInOrder`. Sau đó, chúng tôi gọi `bloc.dispatch` để khởi động mọi thứ.

*Thay vì **arrange -> act -> assert**, chúng tôi **arrange -> assert later -> act**. Thông thường không cần thiết phải gọi `expectLater` trước khi thực sự `dispatch` tham gia sự kiện vì phải mất một thời gian trước khi Stream phát ra giá trị đầu tiên. Tôi thích sai về mặt an toàn.*

Nó sẽ được thực hiện sau đây, nơi bạn sẽ thấy sức mạnh thực sự của `Either`. Sử dụng nó `fold` method, chúng tôi chỉ cần có để xử lý cả **failure** và **success** và không giống như với exception, không có cách nào đơn giản.

```
implementation.dart
```
```
@override
Stream<NumberTriviaState> mapEventToState(
  NumberTriviaEvent event,
) async* {
  if (event is GetTriviaForConcreteNumber) {
    final inputEither =
        inputConverter.stringToUnsignedInteger(event.numberString);

    yield* inputEither.fold(
      (failure) async* {
        yield Error(message: INVALID_INPUT_FAILURE_MESSAGE);
      },
      // Although the "success case" doesn't interest us with the current test,
      // we still have to handle it somehow. 
      (integer) => throw UnimplementedError(),
    );
  }
}
```

Mặc dù chúng ta đang ném một `UnimplementedError` từ `Right()` trường hợp này chứa các số nguyên đã được chuyển đổi, điều này sẽ không gây ra bất kỳ rắc rối trong hai bài kiểm tra chúng tôi hiện có.

# Tiếp theo

Trong phần này, chúng tôi bắt đầu thực hiện `NumberTriviaBloc` phát triển dựa trên thử nghiệm với `Streams`. Chúng tôi cũng đã thấy lý do để sử dụng `Either` trong các hành động. Trong phần tiếp theo, chúng ta sẽ hoàn thành Bloc, làm cho nó xử lý cả sự kiện *concrete* và  *random*.


