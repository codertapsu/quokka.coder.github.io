---
layout: post
title: Flutter TDD Clean Architecture Course [5] - Contracts of Data Sources
image: https://2.bp.blogspot.com/-a9nR2A_99yg/VMXJ2cBXE2I/AAAAAAAAB_s/O_gXstMAZ9I/s1600/tdd.png
---

**Repository** là bộ não của **data layer** của một ứng dụng. Nó xử lý dữ liệu từ **remote** và **local Data Sources**, quyết định **Nguồn dữ liệu** nào thích hợp và cũng là lúc cache dữ liệu được thực hiện.

Trong phần  trước , chúng ta đã đi qua cấu trúc cơ bản của **data layer** và hôm nay, đã đến lúc bắt đầu triển khai **data layer** ngay từ lõi của nó - **NumberTriviaRepository**, trong khi tạo hợp đồng  cho các phụ thuộc của nó.

# Implementing the Contract

Interface, abstract, ... hoặc bất cứ điều gì. Mỗi ngôn ngữ có spin riêng của nó. Điều quan trọng là chúng tôi đã có một  hợp đồng mà việc impl **Repository**  phải được thực hiện. Theo cách này, các **Ca sử dụng** liên lạc với **Repo** mà không cần phải biết gì về cách thức hoạt động của nó.

![](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/08/domain-layer-diagram.png?w=141&ssl=1)

Do đó, chúng ta hãy tạo một tệp mới trong *data/repositories*  cho tính năng **number_trivia** , nó sẽ chứa một lớp **NumberTriviaRepositoryImpl.**

```
number_trivia_repository_impl.dart
```
```
import 'package:dartz/dartz.dart';

import '../../../../core/error/failure.dart';
import '../../domain/entities/number_trivia.dart';
import '../../domain/repositories/number_trivia_repository.dart';

class NumberTriviaRepositoryImpl implements NumberTriviaRepository {
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

# Repository Dependencies

Trong phần này, chúng tôi sẽ chỉ tạo *hợp đồng*  cho tất cả các phụ thuộc **Kho lưu trữ** . Điều này sẽ cho phép chúng ta **mock** chúng một cách dễ dàng mà không cần bận tâm đến việc triển khai chúng - điều đó sẽ đến trong các phần tiếp theo.

**Data Sources** có thực sự đầy đủ? Rốt cuộc, chúng tôi sẽ lưu trữ **NumberTrivia** mới nhất vào local để đảm bảo người dùng nhìn thấy thứ gì đó ngay cả khi anh ta ngoại tuyến. Điều này có nghĩa, chúng ta cũng cần có cách để tìm hiểu về trạng thái hiện tại của kết nối mạng. Vì chúng tôi muốn giữ mã của mình độc lập với thế giới bên ngoài nhất có thể, chúng tôi  sẽ không đưa  bất kỳ thư viện bên thứ 3 nào để kết nối trực tiếp vào **Kho lưu trữ** . Thay vào đó, chúng tôi sẽ tạo một lớp **NetworkInfo** .

## Network Info

Lớp này sẽ nằm trong tệp `network_info.dart` trong  thư mục *core / platform* . Đó là bởi vì nó liên quan đến nền tảng cơ bản - trên Android, quá trình nhận thông tin mạng có thể khác với trên iOS. Tất nhiên, chúng tôi sẽ sử dụng gói bên thứ 3 để thống nhất quy trình.

![](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/09/network_info-file.png?w=277&ssl=1)

```
network_info.dart
```
```
abstract class NetworkInfo {
  Future<bool> get isConnected;
}
```

## Remote Data Source

Public interface của **NumberTriviaRemoteDataSource**  sẽ  gần  giống với một trong các **Kho lưu trữ**  - nó sẽ có các phương thức **getConcittleNumberTrivia**  và  **getRandomNumberTrivia** (như chúng ta đã thảo luận trong phần trước) nhưng loại trả về sẽ khác nhau.

Chúng tôi đang ở ranh giới giữa thế giới bên ngoài và ứng dụng của chúng tôi, vì vậy chúng tôi muốn đơn giản hóa điều này. Sẽ không có `<Failure, NumberTrivia>` , mà thay vào đó, chúng tôi sẽ trả lại một **NumberTriviaModel** đơn giản  (được chuyển đổi từ JSON). Lỗi sẽ được xử lý bằng cách **throwing Exceptions**. Chăm sóc dữ liệu "dumb" này và chuyển đổi nó thành **Either** sẽ là trách nhiệm của **Kho lưu trữ** .

```
number_trivia_remote_data_source.dart
```
```
import '../models/number_trivia_model.dart';

abstract class NumberTriviaRemoteDataSource {
  /// Calls the http://numbersapi.com/{number} endpoint.
  ///
  /// Throws a [ServerException] for all error codes.
  Future<NumberTriviaModel> getConcreteNumberTrivia(int number);

  /// Calls the http://numbersapi.com/random endpoint.
  ///
  /// Throws a [ServerException] for all error codes.
  Future<NumberTriviaModel> getRandomNumberTrivia();
}
```

## Adding Exceptions and Failures

Như bạn có thể thấy trong tài liệu, "both of" phương pháp này sẽ đưa ra  **ServerException**  khi phản hồi mà không có  `code 200` , nhưng, ví dụ nếu là `code 404 NOT FOUND`  thì sao? Vấn đề là, hiện tại chúng tôi không có bất kỳ **ServerException** nào  , vì vậy hãy tạo nó để sử dụng nó trong các test **Kho lưu trữ**.

Các **ServerException**  có thể có khả năng chia sẻ trên nhiều tính năng, vì vậy chúng ta sẽ đặt nó bên trong `core/error/exception.dart`. Trong khi chúng tôi làm việc đó, chúng tôi cũng sẽ tạo ra một **CacheException** sẽ được `thrown` bởi **local Data Source.**

![](https://i2.wp.com/resocoder.com/wp-content/uploads/2019/09/exception-file.png?w=239&ssl=1)

```
exception.dart
``
``
class ServerException implements Exception {}

class CacheException implements Exception {}
```

*Chúng tôi giữ cho nó đơn giản bằng cách không có bất kỳ trường nào bên trong các Ngoại lệ tùy chỉnh  . Nếu bạn muốn truyền đạt thêm thông tin về lỗi, vui lòng thêm  trường thông báo vào lớp trong các dự án của bạn.*

Vì chúng ta đang đối phó với các **Exceptions** , chúng ta cũng sẽ tránh được những **Failures** . Hãy nhớ rằng, **Kho lưu trữ** sẽ bắt các **Exceptions**  và return chúng bằng cách sử dụng **Either** là **Failures** . Vì lý do này,  các **Failures** thường ánh xạ chính xác đến  các loại **Exceptions**.

```
failure.dart
```
```
import 'package:equatable/equatable.dart';

abstract class Failure extends Equatable {
  Failure([List properties = const <dynamic>[]]) : super(properties);
}

// General failures
class ServerFailure extends Failure {}

class CacheFailure extends Failure {}
```

## Local Data Source

Cho đến nay, các phương thức chúng tôi tạo ra luôn là về việc lấy dữ liệu, cho dù là **Entity**  hay **Model** . Họ cũng được chia thành  các số cụ thể  hoặc ngẫu nhiên . Chúng tôi sẽ phá vỡ mô hình này với  **NumberTriviaLocalDataSource** .  

Ở đây, chúng tôi cũng sẽ cần đưa dữ liệu  vào  bộ đệm và chúng tôi cũng sẽ không quan tâm đến việc chúng tôi đang xử lý  các câu đố số cụ thể hay ngẫu nhiên . Đó là bởi vì chính sách cache (được triển khai bên trong **Kho lưu trữ** ) sẽ đơn giản - luôn lưu cache và truy xuất các câu đố cuối cùng nhận được từ **remote Data Source**.

```
number_trivia_local_data_source.dart
```
```
import '../models/number_trivia_model.dart';

abstract class NumberTriviaLocalDataSource {
  /// Gets the cached [NumberTriviaModel] which was gotten the last time
  /// the user had an internet connection.
  ///
  /// Throws [NoLocalDataException] if no cached data is present.
  Future<NumberTriviaModel> getLastNumberTrivia();

  Future<void> cacheNumberTrivia(NumberTriviaModel triviaToCache);
}
```

*Lưu ý rằng trong cả các **Data Sources** chúng ta viết code mà có vẻ là phụ thuộc vào một số lớp ngoài của ứng dụng.
URL cho API sẽ phải là một String , tuy nhiên, loại các số  tham số cho remote Data Source là một số nguyên ...
`Future<NumberTriviaModel> getConcreteNumberTrivia(int number);`
Code này cho phép bạn trao đổi gói http  cấp thấp cho một cái gì đó như  **chopper** mà không có bất kỳ vấn đề quan trọng nào.*

# Thiết lập Repository

Mặc dù chúng tôi sẽ không viết bất kỳ logic thực tế nào của  lớp **NumberTriviaRepositoryImpl** trong phần này (sắp tới trong phần tiếp theo!), nhưng ít nhất chúng tôi sẽ thiết lập nó để sẵn sàng làm việc với tất cả các phụ thuộc mà chúng tôi đã tạo ở trên. Vì chúng tôi đang làm TDD, chúng tôi cũng sẽ chuẩn bị tệp thử nghiệm cho phần tiếp theo.

Một lần nữa, theo tinh thần của TDD, chúng tôi sẽ viết thử nghiệm đầu tiên, mặc dù chúng tôi sẽ không thực sự kiểm tra bất cứ điều gì chỉ được nêu ra . Như thường lệ, tệp kiểm tra đi vào vị trí "phản chiếu" của tệp sản xuất.

![](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/09/repository-test-file.png?w=472&ssl=1)

Chúng tôi biết rằng **Kho lưu trữ** nên lấy **remote**, **local Data Sources** và **NetworkInfo** . Bởi vì chúng tôi đang chuẩn bị tệp thử nghiệm cho phần tiếp theo, hãy tạo Mocks cho các phụ thuộc này ngay lập tức.

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

Tiếp tục và thêm tất cả các trường và tham số hàm tạo cần thiết vào **NumberTriviaRepositoryImpl**

```
number_trivia_repository_impl.dart
```
```
import 'package:dartz/dartz.dart';
import 'package:meta/meta.dart';

import '../../../../core/error/failure.dart';
import '../../../../core/platform/network_info.dart';
import '../../domain/entities/number_trivia.dart';
import '../../domain/repositories/number_trivia_repository.dart';
import '../datasources/number_trivia_local_data_source.dart';
import '../datasources/number_trivia_remote_data_source.dart';

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

# Tiếp theo

Đã có khá nhiều thứ để tiêu hóa từ phần này. Chúng tôi đã tạo 3 hợp đồng cho các phụ thuộc của **Kho lưu trữ** . Bởi vì chúng tôi luôn triển khai mọi thứ "từ các phần bên trong ra", phần tiếp theo sẽ là tất cả về việc **Kho lưu trữ**  thực hiện công việc của mình.