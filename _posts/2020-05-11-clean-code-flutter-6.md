---
layout: post
title: Flutter TDD Clean Architecture Course [6] - Repository Implementation
image: https://2.bp.blogspot.com/-a9nR2A_99yg/VMXJ2cBXE2I/AAAAAAAAB_s/O_gXstMAZ9I/s1600/tdd.png
---

*Sau phần trước , bây giờ chúng tôi có tất cả các "hợp đồng"  của **Repository**. Các phụ thuộc này là  **local** và **remote Data Source**  và cũng là lớp **NetworkInfo** , để tìm hiểu xem người dùng có trực tuyến hay không. Việc mock các phụ thuộc này sẽ cho phép chúng tôi triển khai lớp Kho lưu trữ  bằng cách sử dụng phát triển theo hướng kiểm tra.*

# Implementing Repository

Trong phần trước, chúng tôi hiện có một tệp thử nghiệm với các phụ thuộc được mock ...

```
number_trivia_repository_impl_test.dart
```
```
class MockRemoteDataSource extends Mock
    implements NumberTriviaRemoteDataSource {}

class MockLocalDataSource extends Mock implements NumberTriviaLocalDataSource {}

class MockNetworkInfo extends Mock implements NetworkInfo {}

void main() {
  NumberTriviaRepositoryImpl repository;
  MockRemoteDataSource mockRemoteDataSource;
  MockLocalDataSource mockLocalDataSource;
  MockNetworkInfo mockNetworkInfo;

  setUp(() {
    mockRemoteDataSource = MockRemoteDataSource();
    mockLocalDataSource = MockLocalDataSource();
    mockNetworkInfo = MockNetworkInfo();
    repository = NumberTriviaRepositoryImpl(
      remoteDataSource: mockRemoteDataSource,
      localDataSource: mockLocalDataSource,
      networkInfo: mockNetworkInfo,
    );
  });
}
```

... Và cũng là một **Kho lưu trữ** đơn giản, chưa được impl.

```
number_trivia_repository_impl.dart
```
```
class NumberTriviaRepositoryImpl implements NumberTriviaRepository {
  final NumberTriviaRemoteDataSource remoteDataSource;
  final NumberTriviaLocalDataSource localDataSource;
  final NetworkInfo networkInfo;

  NumberTriviaRepositoryImpl({
    @required this.remoteDataSource,
    @required this.localDataSource,
    @required this.networkInfo,
  });

  @override
  Future<Either<Failure, NumberTrivia>> getConcreteNumberTrivia(int number) {
    // TODO: implement getConcreteNumberTrivia
    return null;
  }

  @override
  Future<Either<Failure, NumberTrivia>> getRandomNumberTrivia() {
    // TODO: implement getRandomNumberTrivia
    return null;
  }
}

```

Trước tiên, hãy bắt đầu với việc triển khai phương thức **getConcreteNumberTrivia**

## getConcreteNumberTrivia

Công việc của  **Kho lưu trữ**  là lấy  dữ liệu mới từ API  khi có kết nối Internet (và sau đó lưu trữ cục bộ) hoặc để lấy dữ liệu được  lưu trong bộ nhớ cache  khi người dùng ngoại tuyến.

Do đó, lấy được trạng thái mạng của thiết bị là điều đầu tiên thực hiện trong phương pháp này. Hãy viết một bài test theo dòng Arrange - Act - Assert. Chúng tôi sẽ  mock  NetworkInfo  và xác minh xem thuộc tính isConnected của nó đã được gọi chưa.

```
number_trivia_repository_impl_test.dart
```
```
group('getConcreteNumberTrivia', () {
  // DATA FOR THE MOCKS AND ASSERTIONS
  // We'll use these three variables throughout all the tests
  final tNumber = 1;
  final tNumberTriviaModel =
      NumberTriviaModel(number: tNumber, text: 'test trivia');
  final NumberTrivia tNumberTrivia = tNumberTriviaModel;

  test('should check if the device is online', () {
    //arrange
    when(mockNetworkInfo.isConnected).thenAnswer((_) async => true);
    // act
    repository.getConcreteNumberTrivia(tNumber);
    // assert
    verify(mockNetworkInfo.isConnected);
  });
});

```

Tôi không nghĩ rằng tôi cần phải viết nó nữa, nhưng bài kiểm tra này sẽ thất bại . Chúng ta phải thực hiện các chức năng cần thiết. Với các thử nghiệm khác trong phần này, tôi sẽ chỉ cho bạn thấy code production ngay sau code thử nghiệm.

```
number_trivia_repository_impl.dart
```
```
@override
Future<Either<Failure, NumberTrivia>> getConcreteNumberTrivia(int number) {
  networkInfo.isConnected;
  return null;
}
```

*"Nhưng ... đoạn code trên không làm gì hữu ích với giá trị của  isConnected !"
Đúng rồi! Bạn không bao giờ nên viết nhiều code sản xuất hơn là đủ để vượt qua bài kiểm tra. Chúng tôi sẽ làm cho phương pháp này làm một cái gì đó hữu ích sau khi thử nghiệm thêm.*

Như bạn đã biết, chức năng của phương thức sẽ khác nhau dựa trên việc người dùng có trực tuyến hay không. Do đó, chúng tôi sẽ "phân nhánh" các bài kiểm tra thành hai loại - trực tuyến và ngoại tuyến . Hãy bắt đầu với  nhánh trực tuyến .

#### Online Behavior

```
test.dart
```
```
group('device is online', () {
  // This setUp applies only to the 'device is online' group
  setUp(() {
    when(mockNetworkInfo.isConnected).thenAnswer((_) async => true);
  });

  test(
    'should return remote data when the call to remote data source is successful',
    () async {
      // arrange
      when(mockRemoteDataSource.getConcreteNumberTrivia(tNumber))
          .thenAnswer((_) async => tNumberTriviaModel);
      // act
      final result = await repository.getConcreteNumberTrivia(tNumber);
      // assert
      verify(mockRemoteDataSource.getConcreteNumberTrivia(tNumber));
      expect(result, equals(Right(tNumberTrivia)));
    },
  );
});
```

```
impl.dart
```
```
@override
Future<Either<Failure, NumberTrivia>> getConcreteNumberTrivia(
  int number,
) async {
  networkInfo.isConnected;
  return Right(await remoteDataSource.getConcreteNumberTrivia(number));
}

```

**Right** của **Either**  chứa giá trị "thành công" trả lại một entity **NumberTrivia**. Việc thực hiện vẫn không nhìn giống như này lắm, nhưng chúng ta đang dần đến đó.

Bất cứ khi nào câu đố được lấy thành công từ API, chúng ta nên lưu trữ cục bộ. Đó là những gì chúng tôi sẽ triển khai tiếp theo (vẫn trong nhóm trực tuyến ) - bạn luôn có thể nhận được [toàn bộ dự án từ GitHub](https://github.com/ResoCoder/flutter-tdd-clean-architecture-course).

```
test.dart
```
```
test(
  'should cache the data locally when the call to remote data source is successful',
  () async {
    // arrange
    when(mockRemoteDataSource.getConcreteNumberTrivia(tNumber))
        .thenAnswer((_) async => tNumberTriviaModel);
    // act
    await repository.getConcreteNumberTrivia(tNumber);
    // assert
    verify(mockRemoteDataSource.getConcreteNumberTrivia(tNumber));
    verify(mockLocalDataSource.cacheNumberTrivia(tNumberTrivia));
  },
);

```

```
impl.dart
```
```
@override
Future<Either<Failure, NumberTrivia>> getConcreteNumberTrivia(
  int number,
) async {
  networkInfo.isConnected;
  final remoteTrivia = await remoteDataSource.getConcreteNumberTrivia(number);
  localDataSource.cacheNumberTrivia(remoteTrivia);
  return Right(remoteTrivia);
}
```

Cuối cùng, khi chúng tôi  trực tuyến  và **remote Data Source** `throws` một `ServerException` , chúng tôi nên chuyển đổi nó thành  **ServerFailure** và return lại nó từ phương thức. Trong trường hợp như vậy, không có gì nên được lưu trữ cục bộ (do đó sử dụng `verifyZeroInteractions`).

```
test.dart
```
```
test(
  'should return server failure when the call to remote data source is unsuccessful',
  () async {
    // arrange
    when(mockRemoteDataSource.getConcreteNumberTrivia(tNumber))
        .thenThrow(ServerException());
    // act
    final result = await repository.getConcreteNumberTrivia(tNumber);
    // assert
    verify(mockRemoteDataSource.getConcreteNumberTrivia(tNumber));
    verifyZeroInteractions(mockLocalDataSource);
    expect(result, equals(Left(ServerFailure())));
  },
);

```

```
impl.dart
```
```
@override
Future<Either<Failure, NumberTrivia>> getConcreteNumberTrivia(
  int number,
) async {
  networkInfo.isConnected;
  try {
    final remoteTrivia =
        await remoteDataSource.getConcreteNumberTrivia(number);
    localDataSource.cacheNumberTrivia(remoteTrivia);
    return Right(remoteTrivia);
  } on ServerException {
    return Left(ServerFailure());
  }
}
```

#### Offline Behavior

Các thử nghiệm ở trên là đủ khi thiết bị trực tuyến , giờ là lúc để thực hiện hành vi ngoại tuyến . Một lần nữa hãy tạo một nhóm thử nghiệm mới cùng với thử nghiệm đầu tiên . Các **Repository** nên return dữ liệu cache cuối cùng tại local khi không trực tuyến.

```
test.dart
```
```
group('device is offline', () {
  setUp(() {
    when(mockNetworkInfo.isConnected).thenAnswer((_) async => false);
  });

  test(
    'should return last locally cached data when the cached data is present',
    () async {
      // arrange
      when(mockLocalDataSource.getLastNumberTrivia())
          .thenAnswer((_) async => tNumberTriviaModel);
      // act
      final result = await repository.getConcreteNumberTrivia(tNumber);
      // assert
      verifyZeroInteractions(mockRemoteDataSource);
      verify(mockLocalDataSource.getLastNumberTrivia());
      expect(result, equals(Right(tNumberTrivia)));
    },
  );
});
```

```
impl.dart
```
```
@override
Future<Either<Failure, NumberTrivia>> getConcreteNumberTrivia(
  int number,
) async {
  // Finally doing something with the value of isConnected 😉
  if (await networkInfo.isConnected) {
    try {
      final remoteTrivia =
          await remoteDataSource.getConcreteNumberTrivia(number);
      localDataSource.cacheNumberTrivia(remoteTrivia);
      return Right(remoteTrivia);
    } on ServerException {
      return Left(ServerFailure());
    }
  } else {
    final localTrivia = await localDataSource.getLastNumberTrivia();
    return Right(localTrivia);
  }
}
```

Chúng tôi cũng có để xử lý các trường hợp khi các **local Data Source** ném một **CacheException**  bằng cách return một **CacheFailure** qua **Left**  "error" trng **Either**. Như việc ghi trong tài liệu của  **getLastNumberTrivia** method, **CacheException** sẽ xảy ra bất cứ khi nào có gì hiện bên trong bộ nhớ cache.

```
test.dart
```
```
test(
  'should return CacheFailure when there is no cached data present',
  () async {
    // arrange
    when(mockLocalDataSource.getLastNumberTrivia())
        .thenThrow(CacheException());
    // act
    final result = await repository.getConcreteNumberTrivia(tNumber);
    // assert
    verifyZeroInteractions(mockRemoteDataSource);
    verify(mockLocalDataSource.getLastNumberTrivia());
    expect(result, equals(Left(CacheFailure())));
  },
);
```

```
impl.dart
```
```
@override
Future<Either<Failure, NumberTrivia>> getConcreteNumberTrivia(
  int number,
) async {
  if (await networkInfo.isConnected) {
    try {
      final remoteTrivia =
          await remoteDataSource.getConcreteNumberTrivia(number);
      localDataSource.cacheNumberTrivia(remoteTrivia);
      return Right(remoteTrivia);
    } on ServerException {
      return Left(ServerFailure());
    }
  } else {
    try {
      final localTrivia = await localDataSource.getLastNumberTrivia();
      return Right(localTrivia);
    } on CacheException {
      return Left(CacheFailure());
    }
  }
}
```

## getRandomNumberTrivia

Cách chúng tôi sẽ xây dựng  **getRandomNumberTrivia**  sẽ gần như giống hệt với  **getConcreteNumberTrivia**. Chúng tôi thậm chí có thể cấu trúc lại một số phần của các thử nghiệm thành các phương thức được đặt tên - cụ thể là các group trực tuyến  và  ngoại tuyến  . Được cấu trúc lại bằng hai phương thức mới, code kiểm tra hiện tại sẽ trông như thế này (được cắt ngắn để lấy ngắn gọn, [lấy mã đầy đủ trên GitHub](https://github.com/ResoCoder/flutter-tdd-clean-architecture-course) ).

```
test.dart
```
```
void main() {
  ...

  void runTestsOnline(Function body) {
    group('device is online', () {
      setUp(() {
        when(mockNetworkInfo.isConnected).thenAnswer((_) async => true);
      });

      body();
    });
  }

  void runTestsOffline(Function body) {
    group('device is offline', () {
      setUp(() {
        when(mockNetworkInfo.isConnected).thenAnswer((_) async => false);
      });

      body();
    });
  }

  group('getConcreteNumberTrivia', () {
    ...

    runTestsOnline(() {
      test(
        ...
    });

    runTestsOffline(() {
      test(
        ...
    });
  });
}
```

Các luật bán chính thức của TDD nói rằng bạn nên luôn luôn viết và thực hiện các bài kiểm tra từng cái một. Tuy nhiên, tôi thích thực tế, đặc biệt là khi nói về coding trong các hướng dẫn như thế này.

Vì phương thức **getRandomNumberTrivia** sẽ chỉ khác nhau trong một *single call*  đến **remote Data Source** , nên chúng tôi sẽ sao chép tất cả các thử nghiệm chúng tôi hiện có cho phương thức **concrete**  và sửa đổi một chút để hoạt động với phương thức **random**.

```
test.dart
```
```
group('getRandomNumberTrivia', () {
  final tNumberTriviaModel =
      NumberTriviaModel(number: 123, text: 'test trivia');
  final NumberTrivia tNumberTrivia = tNumberTriviaModel;

  test('should check if the device is online', () {
    //arrange
    when(mockNetworkInfo.isConnected).thenAnswer((_) async => true);
    // act
    repository.getRandomNumberTrivia();
    // assert
    verify(mockNetworkInfo.isConnected);
  });

  runTestsOnline(() {
    test(
      'should return remote data when the call to remote data source is successful',
      () async {
        // arrange
        when(mockRemoteDataSource.getRandomNumberTrivia())
            .thenAnswer((_) async => tNumberTriviaModel);
        // act
        final result = await repository.getRandomNumberTrivia();
        // assert
        verify(mockRemoteDataSource.getRandomNumberTrivia());
        expect(result, equals(Right(tNumberTrivia)));
      },
    );

    test(
      'should cache the data locally when the call to remote data source is successful',
      () async {
        // arrange
        when(mockRemoteDataSource.getRandomNumberTrivia())
            .thenAnswer((_) async => tNumberTriviaModel);
        // act
        await repository.getRandomNumberTrivia();
        // assert
        verify(mockRemoteDataSource.getRandomNumberTrivia());
        verify(mockLocalDataSource.cacheNumberTrivia(tNumberTrivia));
      },
    );

    test(
      'should return server failure when the call to remote data source is unsuccessful',
      () async {
        // arrange
        when(mockRemoteDataSource.getRandomNumberTrivia())
            .thenThrow(ServerException());
        // act
        final result = await repository.getRandomNumberTrivia();
        // assert
        verify(mockRemoteDataSource.getRandomNumberTrivia());
        verifyZeroInteractions(mockLocalDataSource);
        expect(result, equals(Left(ServerFailure())));
      },
    );
  });

  runTestsOffline(() {
    test(
      'should return last locally cached data when the cached data is present',
      () async {
        // arrange
        when(mockLocalDataSource.getLastNumberTrivia())
            .thenAnswer((_) async => tNumberTriviaModel);
        // act
        final result = await repository.getRandomNumberTrivia();
        // assert
        verifyZeroInteractions(mockRemoteDataSource);
        verify(mockLocalDataSource.getLastNumberTrivia());
        expect(result, equals(Right(tNumberTrivia)));
      },
    );

    test(
      'should return CacheFailure when there is no cached data present',
      () async {
        // arrange
        when(mockLocalDataSource.getLastNumberTrivia())
            .thenThrow(CacheException());
        // act
        final result = await repository.getRandomNumberTrivia();
        // assert
        verifyZeroInteractions(mockRemoteDataSource);
        verify(mockLocalDataSource.getLastNumberTrivia());
        expect(result, equals(Left(CacheFailure())));
      },
    );
  });
});
```

Theo tinh thần của TDD, chúng tôi sẽ không thực hiện bất kỳ tái cấu trúc sớm nào. Trước tiên chúng ta hãy triển khai code một cách thô sơ, mặc dù chúng ta đã biết nó sẽ bị trùng lặp.

```
impl.dart
```
```
@override
Future<Either<Failure, NumberTrivia>> getRandomNumberTrivia() async {
  if (await networkInfo.isConnected) {
    try {
      final remoteTrivia = await remoteDataSource.getRandomNumberTrivia();
      localDataSource.cacheNumberTrivia(remoteTrivia);
      return Right(remoteTrivia);
    } on ServerException {
      return Left(ServerFailure());
    }
  } else {
    try {
      final localTrivia = await localDataSource.getLastNumberTrivia();
      return Right(localTrivia);
    } on CacheException {
      return Left(CacheFailure());
    }
  }
}
```

Sự khác biệt duy nhất theo nghĩa đen giữa **concrete**  và **random** chỉ là gọi code này được tái cấu trúc. Hầu hết logic có thể được chia sẻ giữa các phương thức **concrete**  và **random**. Chúng tôi sẽ xử lý một cuộc gọi khác nhau đến **local Data Source** với hàm bậc cao hơn. Phiên bản cuối cùng của **NumberTriviaRepositoryImpl** sẽ trông như sau:

```
impl.dart
```
```
typedef Future<NumberTrivia> _ConcreteOrRandomChooser();

class NumberTriviaRepositoryImpl implements NumberTriviaRepository {
  final NumberTriviaRemoteDataSource remoteDataSource;
  final NumberTriviaLocalDataSource localDataSource;
  final NetworkInfo networkInfo;

  NumberTriviaRepositoryImpl({
    @required this.remoteDataSource,
    @required this.localDataSource,
    @required this.networkInfo,
  });

  @override
  Future<Either<Failure, NumberTrivia>> getConcreteNumberTrivia(
    int number,
  ) async {
    return await _getTrivia(() {
      return remoteDataSource.getConcreteNumberTrivia(number);
    });
  }

  @override
  Future<Either<Failure, NumberTrivia>> getRandomNumberTrivia() async {
    return await _getTrivia(() {
      return remoteDataSource.getRandomNumberTrivia();
    });
  }

  Future<Either<Failure, NumberTrivia>> _getTrivia(
    _ConcreteOrRandomChooser getConcreteOrRandom,
  ) async {
    if (await networkInfo.isConnected) {
      try {
        final remoteTrivia = await getConcreteOrRandom();
        localDataSource.cacheNumberTrivia(remoteTrivia);
        return Right(remoteTrivia);
      } on ServerException {
        return Left(ServerFailure());
      }
    } else {
      try {
        final localTrivia = await localDataSource.getLastNumberTrivia();
        return Right(localTrivia);
      } on CacheException {
        return Left(CacheFailure());
      }
    }
  }
}
```

# Cái gì tiếp theo

Bây giờ chúng tôi đã có **Kho lưu trữ** được triển khai đầy đủ, chúng tôi sẽ bắt đầu làm việc trên các phần cấp thấp của **lớp dữ liệu**  bằng cách triển khai **Data Sources** và **Network Info**.



