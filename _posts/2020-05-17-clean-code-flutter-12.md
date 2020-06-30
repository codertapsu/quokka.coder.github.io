---
layout: post
title: Flutter TDD Clean Architecture Course [12] - Bloc Implementation 2/2
image: https://2.bp.blogspot.com/-a9nR2A_99yg/VMXJ2cBXE2I/AAAAAAAAB_s/O_gXstMAZ9I/s1600/tdd.png
---

Chúng tôi đã bắt đầu thực hiện `NumberTriviaBloc` ở phần trước và bạn đã học những điều cơ bản khi thực hiện TDD với `Streams`. Trong phần này, chúng ta hãy hoàn thành việc triển khai Bloc để chúng ta có thể chuyển sang dependency injection tiếp theo.

# Continuing Where We Left Off

Chúng tôi đã triển khai một phần quan trọng của logic, nó chạy bất cứ khi nào `GetTriviaForConcreteNumber` event được truyền đến trong Bloc - chuyển đổi đầu vào. Tuy nhiên, sau khi chúng tôi đã chuyển đổi và xác thực thành công int mà người dùng muốn xem một số câu đố số đáng kinh ngạc, chúng tôi hiện chỉ cần ném một UnimplementedException. Hãy thay đổi điều đó!

## Testing với Use Case

Điều đầu tiên nên xảy ra là `GetConcreteNumberTrivia` use case nên được thực thi với một đối số `number`  thích hợp . Ví dụ: nếu `numberString` của event là `"42"`, integer được chuyển vào trường hợp sử dụng dĩ nhiên là 42.

Chú ý rằng bây giờ chúng tôi phải thử cả `InputConverter` và `GetConcreteNumberTrivia` use case trong phần *arrange* của thử nghiệm.

```
number_trivia_bloc_test.dart
```
```
test(
  'should get data from the concrete use case',
  () async {
    // arrange
    when(mockInputConverter.stringToUnsignedInteger(any))
        .thenReturn(Right(tNumberParsed));
    when(mockGetConcreteNumberTrivia(any))
        .thenAnswer((_) async => Right(tNumberTrivia));
    // act
    bloc.dispatch(GetTriviaForConcreteNumber(tNumberString));
    await untilCalled(mockGetConcreteNumberTrivia(any));
    // assert
    verify(mockGetConcreteNumberTrivia(Params(number: tNumberParsed)));
  },
);
```

Một lần nữa chúng ta sẽ triển khai code vừa đủ để bài kiểm tra pass. Bây giờ chúng ta có thể bỏ qua tất cả các cảnh báo trong IDE một cách an toàn khi nói rằng phương thức này không trả về `Stream`.

```
number_trivia_bloc.dart
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
      (integer) {
        getConcreteNumberTrivia(Params(number: integer));
      },
    );
  }
}
```

## Tái cấu trúc lại bài kiểm tra

Mock `InputConverter` để return về giá trị thành công

```
when(mockInputConverter.stringToUnsignedInteger(any))
    .thenReturn(Right(tNumberParsed));
```

sẽ xảy ra trong hầu hết các bài kiểm tra. Đó là một ý tưởng tốt để đưa nó vào phương thức riêng của mình `setUpMockInputConverterSuccess` và sử dụng nó trong tất cả các thử nghiệm, bao gồm cả thử nghiệm từ phần trước.

```
number_trivia_bloc_test.dart
```
```
group('GetTriviaForConcreteNumber', () {
  ...

  void setUpMockInputConverterSuccess() =>
      when(mockInputConverter.stringToUnsignedInteger(any))
          .thenReturn(Right(tNumberParsed));
  // Use in tests below
  ...
}
```

## Successful Execution

Tất nhiên, chỉ đơn giản là thực hiện use case là không đủ. Chúng ta cần cho UI biết những gì đang diễn ra để nó có thể hiển thị một cái gì đó cho người dùng. Những States nào Bloc nên phát ra khi use case thực hiện trơn tru?

Vâng, trước khi gọi UC, đó là một ý tưởng tốt để hiển thị một số phản hồi trực quan cho người dùng. Mặc dù hiển thị a `CircularProgressIndicator` là một nhiệm vụ cho UI, chúng ta phải thông báo cho UI để hiển thị nó bằng cách phát ra `Loading` state.

Sau khi UC trả về `Right` trong `Either` (trường hợp thành công), Bloc sẽ chuyển giao thu được `NumberTrivia` cho UI trong `Loaded` State.

```
test.dart
test(
  'should emit [Loading, Loaded] when data is gotten successfully',
  () async {
    // arrange
    setUpMockInputConverterSuccess();
    when(mockGetConcreteNumberTrivia(any))
        .thenAnswer((_) async => Right(tNumberTrivia));
    // assert later
    final expected = [
      Empty(),
      Loading(),
      Loaded(trivia: tNumberTrivia),
    ];
    expectLater(bloc.state, emitsInOrder(expected));
    // act
    bloc.dispatch(GetTriviaForConcreteNumber(tNumberString));
  },
);
```

Việc triển khai sẽ bắt đầu trở nên tồi tệ hơn, nhưng tất nhiên chúng ta sẽ cấu trúc lại nó sau khi chúng ta viết xong bài kiểm tra.

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
      (integer) async* {
        yield Loading();
        final failureOrTrivia = await getConcreteNumberTrivia(
          Params(number: integer),
        );
        yield failureOrTrivia.fold(
          (failure) => throw UnimplementedError(),
          (trivia) => Loaded(trivia: trivia),
        );
      },
    );
  }
}
```

## Unsuccessful Execution

`Either` type làm cho chúng ta xử lý trường hợp `Failure` , nhưng cho đến bây giờ, chúng tôi chỉ đơn giản là ném một `UnimplementedError` khi điều này xảy ra. Chúng ta hãy tạo một thử nghiệm cho trường hợp khi trường hợp sử dụng trả về `ServerFailure`.

```
test.dart
```
```
test(
  'should emit [Loading, Error] when getting data fails',
  () async {
    // arrange
    setUpMockInputConverterSuccess();
    when(mockGetConcreteNumberTrivia(any))
        .thenAnswer((_) async => Left(ServerFailure()));
    // assert later
    final expected = [
      Empty(),
      Loading(),
      Error(message: SERVER_FAILURE_MESSAGE),
    ];
    expectLater(bloc.state, emitsInOrder(expected));
    // act
    bloc.dispatch(GetTriviaForConcreteNumber(tNumberString));
  },
);
```

```
implementation.dart
```
```
(integer) async* {
  yield Loading();
  final failureOrTrivia = await getConcreteNumberTrivia(
    Params(number: integer),
  );
  yield failureOrTrivia.fold(
    (failure) => Error(message: SERVER_FAILURE_MESSAGE),
    (trivia) => Loaded(trivia: trivia),
  );
},
```

Đơn giản phải không? `ServerFailure` không phải là loại duy nhất được `Failure` trả về bởi UC. Một `CacheFailure` cũng có thể xảy ra và chúng ta phải xử lý điều đó.

*Điều rất quan trọng để xử lý tất cả các kiểu con có thể có Failure trong Bloc. Rốt cuộc, chúng tôi muốn hiển thị một thông báo lỗi có ý nghĩa cho người dùng và điều đó chỉ có thể bằng cách phát ra Error State với một thông báo tùy chỉnh.*

Kiểm tra cho `CacheFailure` sẽ rất giống với thử nghiệm trước, nhưng đủ khác nhau để tái cấu trúc không thực sự cần thiết.

```
test.dart
```
```
test(
  'should emit [Loading, Error] with a proper message for the error when getting data fails',
  () async {
    // arrange
    setUpMockInputConverterSuccess();
    when(mockGetConcreteNumberTrivia(any))
        .thenAnswer((_) async => Left(CacheFailure()));
    // assert later
    final expected = [
      Empty(),
      Loading(),
      Error(message: CACHE_FAILURE_MESSAGE),
    ];
    expectLater(bloc.state, emitsInOrder(expected));
    // act
    bloc.dispatch(GetTriviaForConcreteNumber(tNumberString));
  },
);
```

Chúng tôi sắp có được một số code trông thực sự khó chịu. Đừng lo lắng, chúng tôi sẽ tái cấu trúc nó chỉ trong một thời gian ngắn.

```
implementation.dart
```
```
(integer) async* {
  yield Loading();
  final failureOrTrivia = await getConcreteNumberTrivia(
    Params(number: integer),
  );
  yield failureOrTrivia.fold(
    (failure) => Error(
      message: failure is ServerFailure
          ? SERVER_FAILURE_MESSAGE
          : CACHE_FAILURE_MESSAGE,
    ),
    (trivia) => Loaded(trivia: trivia),
  );
},
```

## Tái cấu trúc toán tử Ternary

Cách chúng tôi quyết định msg của `Error` state để lại rất nhiều mong muốn. Toán tử ternary thực sự là một lựa chọn không an toàn - điều gì xảy ra nếu có một số `Failure` kiểu khác mà chúng ta không biết. Ý tôi là, điều đó không nên xảy ra với Clean Architecture, nhưng ... Bất kể thực tế nó là loại gì của `Failure`, trừ khi nó là `ServerFailure`, thì `CACHE_FAILURE_MESSAGE` sẽ luôn được gán cho nó.

Chúng ta hãy tạo một `_mapFailureToMessage` method helper riêng, trong đó chúng ta sẽ cố gắng xử lý các msg phù hợp hơn. Chúng tôi vẫn đang trong giới hạn cở bản của Dart mà không hỗ trợ của [​sealed classes](https://resocoder.com/2019/09/16/sealed-unions-in-dart-never-write-an-if-statement-again-kind-of/) , vì vậy chúng tôi vẫn cần phải xử lý những lỗi không mong muốn từ `Failure`.
Phương thức `mapEventToState` bây giờ trông như thế này:

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
      (integer) async* {
        yield Loading();
        final failureOrTrivia = await getConcreteNumberTrivia(
          Params(number: integer),
        );
        yield failureOrTrivia.fold(
          (failure) => Error(message: _mapFailureToMessage(failure)),
         (trivia) => Loaded(trivia: trivia),
        );
      },
    );
  }
}

String _mapFailureToMessage(Failure failure) {
  // Instead of a regular 'if (failure is ServerFailure)...'
  switch (failure.runtimeType) {
    case ServerFailure:
      return SERVER_FAILURE_MESSAGE;
    case CacheFailure:
      return CACHE_FAILURE_MESSAGE;
    default:
      return 'Unexpected Error';
  }
}
```

# Getting the Random Trivia

Như chúng ta đã quen với các phần trước, chúng ta sẽ *cố tình phá vỡ các nguyên tắc TDD* để chúng ta không tự làm mình phát điên với việc thực hiện tẻ nhạt về việc code lặp lại. Không giống như **data source** hoặc **repository**, cả code kiểm tra và code sản xuất cho `GetTriviaForRandomNumber` sẽ đơn giản hơn. Đó là bởi vì nó không sử dụng `InputValidator`! Người dùng sẽ không cần nhập vào để có được một NumberTrivia ngẫu nhiên .

Bắt đầu với các thử nghiệm, chúng tôi sẽ sao chép và dán code từ nhóm **concrete** , xóa mọi thứ liên quan đến `InputConverter` và cũng thay đổi bất kỳ tham chiếu nào đến use case/event **concrete** để tham chiếu use case/event **random**.

```
test.dart
```
```
group('GetTriviaForRandomNumber', () {
  final tNumberTrivia = NumberTrivia(number: 1, text: 'test trivia');

  test(
    'should get data from the random use case',
    () async {
      // arrange
      when(mockGetRandomNumberTrivia(any))
          .thenAnswer((_) async => Right(tNumberTrivia));
      // act
      bloc.dispatch(GetTriviaForRandomNumber());
      await untilCalled(mockGetRandomNumberTrivia(any));
      // assert
      verify(mockGetRandomNumberTrivia(NoParams()));
    },
  );

  test(
    'should emit [Loading, Loaded] when data is gotten successfully',
    () async {
      // arrange
      when(mockGetRandomNumberTrivia(any))
          .thenAnswer((_) async => Right(tNumberTrivia));
      // assert later
      final expected = [
        Empty(),
        Loading(),
        Loaded(trivia: tNumberTrivia),
      ];
      expectLater(bloc.state, emitsInOrder(expected));
      // act
      bloc.dispatch(GetTriviaForRandomNumber());
    },
  );

  test(
    'should emit [Loading, Error] when getting data fails',
    () async {
      // arrange
      when(mockGetRandomNumberTrivia(any))
          .thenAnswer((_) async => Left(ServerFailure()));
      // assert later
      final expected = [
        Empty(),
        Loading(),
        Error(message: SERVER_FAILURE_MESSAGE),
      ];
      expectLater(bloc.state, emitsInOrder(expected));
      // act
      bloc.dispatch(GetTriviaForRandomNumber());
    },
  );

  test(
    'should emit [Loading, Error] with a proper message for the error when getting data fails',
    () async {
      // arrange
      when(mockGetRandomNumberTrivia(any))
          .thenAnswer((_) async => Left(CacheFailure()));
      // assert later
      final expected = [
        Empty(),
        Loading(),
        Error(message: CACHE_FAILURE_MESSAGE),
      ];
      expectLater(bloc.state, emitsInOrder(expected));
      // act
      bloc.dispatch(GetTriviaForRandomNumber());
    },
  );
});
```

Với tất cả những thử nghiệm thất bại vì những lý do rõ ràng, chúng ta sẽ sao chép impl cho `GetTriviaForRandomNumber` trong Bloc. Tất nhiên, chúng tôi không cần xử lý chuyển đổi đầu vào, vì vậy chúng ta sẽ chỉ copy code trong trường hợp các số nguyên đã được chuyển đổi.

Chỉ có hai thay đổi để thực hiện trong code được copy này - call use case `GetRandomNumberTrivia` và truyền qua `NoParams()` trong Use case (Không có tham số nào):

```
implementation.dart
```
```
@override
Stream<NumberTriviaState> mapEventToState(
  NumberTriviaEvent event,
) async* {
  if (event is GetTriviaForConcreteNumber) {
    ...
  } else if (event is GetTriviaForRandomNumber) {
    yield Loading();
    final failureOrTrivia = await getRandomNumberTrivia(
      NoParams(),
    );
    yield failureOrTrivia.fold(
      (failure) => Error(message: _mapFailureToMessage(failure)),
      (trivia) => Loaded(trivia: trivia),
    );
  }
}
```

Sau khi code impl này thực hiện, tất cả các thử nghiệm đã pass, đã đến lúc...

## Xóa trùng lặp

Thực tế không có nhiều trùng lặp để loại bỏ, nhưng vẫn có thể làm cho code của chúng ta **DRY** hơn một chút bằng cách: trích xuất sự phát ra của `Error` và `Loaded` states.

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
      (integer) async* {
        yield Loading();
        final failureOrTrivia = await getConcreteNumberTrivia(
          Params(number: integer),
        );
        yield* _eitherLoadedOrErrorState(failureOrTrivia);
      },
    );
  } else if (event is GetTriviaForRandomNumber) {
    yield Loading();
    final failureOrTrivia = await getRandomNumberTrivia(
      NoParams(),
    );
    yield* _eitherLoadedOrErrorState(failureOrTrivia);
  }
}

Stream<NumberTriviaState> _eitherLoadedOrErrorState(
  Either<Failure, NumberTrivia> either,
) async* {
  yield either.fold(
    (failure) => Error(message: _mapFailureToMessage(failure)),
    (trivia) => Loaded(trivia: trivia),
  );
}

String _mapFailureToMessage(Failure failure) {
  switch (failure.runtimeType) {
    case ServerFailure:
      return SERVER_FAILURE_MESSAGE;
    case CacheFailure:
      return CACHE_FAILURE_MESSAGE;
    default:
      return 'Unexpected Error';
  }
}
```

Kiểu tái cấu trúc này làm chúng tôi phải chạy lại các bài kiểm tra một lần nữa và tất nhiên, tất cả chúng vẫn vượt qua. Và với sự tái cấu trúc này này, chúng tôi vừa hoàn thành một phần khác của *​Number Trivia App* - presentation logic holder (tôi nên nói PLoC 😅)  trong hình thức của một BLoC.

# Tiếp theo

Chúng ta đang tiến gần đến đích. Bây giờ chúng ta đã thực hiện từng logic cụ thể , đã đến lúc kết hợp tất cả các phần lại với nhau bằng cách dependency injection và sau đó ... Chúng ta sẽ không còn gì để làm ngoài việc tạo UI!