---
layout: post
title: Flutter TDD Clean Architecture Course [13] - Dependency Injection
image: https://2.bp.blogspot.com/-a9nR2A_99yg/VMXJ2cBXE2I/AAAAAAAAB_s/O_gXstMAZ9I/s1600/tdd.png
---

Chúng tôi có tất cả các phần cụ thể của kiến ​​trúc. Trước khi chúng ta có thể sử dụng chúng bằng cách xây dựng một UI, chúng ta phải kết nối chúng lại với nhau. Vì mỗi lớp được tách rời khỏi các phụ thuộc của nó bằng cách truyền chúng thông qua constructor , nên bằng cách nào đó chúng ta phải truyền chúng vào.

Chúng tôi cũng đã làm điều này trong các thử nghiệm với **Mock** các classes. Tuy nhiên, bây giờ đã đến lúc để truyền dữ liệu thực trong các class sản xuất bằng cách sử dụng **service locator**.

# Injecting Dependencies

Hầu như tất cả các lớp chúng tôi tạo ra trong khóa học này cho đến bây giờ có một số phụ thuộc. Ngay cả trong một ứng dụng nhỏ như *Number Trivia App* mà chúng tôi đang xây dựng, có khá nhiều thứ để cấu hình. Có nhiều **service locators** phù hợp với Flutter và như bạn có thể nhớ từ phần 2, chúng tôi đang sử dụng [get_it](https://pub.dev/packages/get_it).

Bây giờ chúng ta sẽ lướt qua tất cả các class từ đầu theo các luồng được gọi. Trong trường hợp của chúng tôi, nó bắt đầu từ `NumberTriviaBloc` và kết thúc với các phụ thuộc bên ngoài như `SharedPreferences`.

![Thiết lập service locator là dễ nhất khi lướt theo các class theo luồng cuộc gọi](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/08/Clean-Architecture-Flutter-Diagram.png?w=556&ssl=1)

Chúng ta hãy thiết lập mọi thứ trong một file mới có tên là `injection_container.dart`  nằm ngay trong root folder **lib**. Về bản chất, chúng ta sẽ điền vào các constructor với các đối số thích hợp. Cấu trúc cơ bản của tệp như sau:

```
injection_container.dart
final sl = GetIt.instance;

void init() {
  //! Features - Number Trivia

  //! Core

  //! External

}
```

*Nếu ứng dụng của bạn có nhiều features, bạn có thể muốn tạo nhiều tệp injection_container hơn với `init` function cho mỗi tính năng để giữ cho mọi thứ có tổ chức.
Sau đó, bạn sẽ gọi các `init()` function dành riêng cho tính năng này từ bên trong main.*

Các `init()` function sẽ được gọi ngay khi ứng dụng bắt đầu từ `main.dart` . Ở tại đó function của tất cả các class và "hợp đồng" sẽ được đăng ký và sau đó chúng cũng được injected bằng cách sử dụng **singleton** instance của `GetIt` được lưu trữ trong `sl`(viết tắt của **service locator**).

**get_it** package hỗ trợ tạo **singleton** và instance **factories**. Kể từ khi chúng tôi không nắm giữ bất kỳ State nào trong các class, chúng ta sẽ phải đăng ký tất cả mọi thứ như một `singleton`, có nghĩa là chỉ có một instance của một class sẽ được tạo ra cho mỗi lifetime của ứng dụng. Sẽ chỉ có một ngoại lệ cho quy tắc này là `NumberTriviaBloc`, theo "call flow", chúng tôi sẽ đăng ký trước.

## Registering a Factory

Quá trình đăng ký rất đơn giản. Chỉ cần khởi tạo class như bình thường và truyền `sl()` vào mọi tham số của constructor. Như bạn có thể thấy, `GetIt` class có `call()` method để tạo một cú pháp dễ dàng hơn, rất giống các use case của chúng ta cũng có một `call()` method.

```
injection_container.dart
```
```
//! Features - Number Trivia
//Bloc
sl.registerFactory(
  () => NumberTriviaBloc(
    concrete: sl(),
    random: sl(),
    inputConverter: sl(),
  ),
);
```

**Presentation logic holders** như Bloc không nên được đăng ký là `singletons`. Chúng rất gần với UI và nếu ứng dụng của bạn có nhiều màn hình điều hướng, có lẽ bạn muốn thực hiện một số dọn dẹp (như đóng `Streams` của một Bloc) từ `dispose()` method của `StatefulWidget`.

Có một `singleton` cho các lớp với loại xử lý này sẽ dẫn đến việc cố gắng sử dụng **presentation logic holder** (chẳng hạn như Bloc) với **closed** của Stream , thay vì tạo một instance mới với **opened** của Stream bất cứ khi nào bạn cố gắng lấy một đối tượng của nó từ GetIt.

Sử dụng **type inference**, các cuộc gọi đến `sl()` sẽ xác định đối tượng cần truyền qua như là đối số của các constructor nhất định. Tất nhiên, điều này chỉ có thể khi loại câu hỏi cũng được đăng ký. Rõ ràng là bây giờ chúng ta cần phải đăng ký use case `GetConcreteNumberTrivia` và `GetRandomNumberTrivia` và cả `InputConverter`. Những điều này sẽ không được đăng ký là factory , thay vào đó, chúng sẽ là singleton.

## Registering Singletons

GetItcung cấp cho chúng tôi hai lựa chọn khi nói đến **singletons**. Chúng ta có thể `registerSingleton` hoặc `registerLazySingleton`. Sự khác biệt duy nhất giữa chúng là một **non-lazy singleton** luôn được đăng ký ngay sau khi ứng dụng bắt đầu, trong khi một **lazy singleton** chỉ được đăng ký khi nó được yêu cầu như một sự phụ thuộc cho một số lớp khác.

Trong trường hợp của chúng tôi, lựa chọn giữa đăng ký **lazy** và đăng ký bình thường sẽ không tạo ra sự khác biệt , vì *Number Trivia App* sẽ chỉ có một màn hình với một Bloc và một "dependency tree" , có nghĩa là ngay cả những lazy singleton cũng sẽ được đăng ký **ngay lập tức**. Chúng tôi sẽ chọn `registerLazySingleton`.

## Bloc Dependencies

Để giữ cho việc đăng ký có tổ chức, tất cả mọi thứ diễn ra dưới một comment đặc biệt. Tạo các chức năng riêng biệt cũng có thể, nhưng tôi cảm thấy như thế thực sự có thể code khó đọc hơn nếu bạn chỉ có một vài câu lệnh đăng ký.

```
injection_container.dart
```
```
//! Features - Number Trivia
...
// Use cases
sl.registerLazySingleton(() => GetConcreteNumberTrivia(sl()));
sl.registerLazySingleton(() => GetRandomNumberTrivia(sl()));

//! Core
sl.registerLazySingleton(() => InputConverter());
```

## Repository Registration

Trong khi `InputConverter` là một lớp độc lập, cả hai trường hợp sử dụng đều yêu cầu một `NumberTriviaRepository`. Chú ý rằng chúng phụ thuộc vào "hợp đồng" và không phải về việc thực hiện cụ thể. Tuy nhiên, chúng ta không thể khởi tạo một  hợp đồng (đó là một `abstract class`). Thay vào đó, chúng ta phải khởi tạo việc triển khai **repository**. Điều này có thể bằng cách chỉ định một loại tham số trong method `registerLazySingleton`.

```
injection_container.dart
```
```
//! Features - Number Trivia
...
// Repository
sl.registerLazySingleton<NumberTriviaRepository>(
  () => NumberTriviaRepositoryImpl(
    remoteDataSource: sl(),
    localDataSource: sl(),
    networkInfo: sl(),
  ),
);
```

Điều độc đáo này thể hiện sự hữu ích của **loose coupling**. Tùy thuộc vào abstraction thay vì implementation không chỉ cho phép thử nghiệm, mà nó cũng cho phép chuyển đổi `NumberTriviaRepository` thực hiện cho một chương trình khác mà không cần bất kỳ thay đổi nào đối với các lớp phụ thuộc.

## Data Sources & NetworkInfo

Repository cũng phụ thuộc vào "hợp đồng", vì vậy chúng tôi một lần nữa sẽ chỉ định một kiểu tham số bằng tay.

```
injection_container.dart
```
```
//! Features - Number Trivia
...
// Data sources
sl.registerLazySingleton<NumberTriviaRemoteDataSource>(
  () => NumberTriviaRemoteDataSourceImpl(client: sl()),
);

sl.registerLazySingleton<NumberTriviaLocalDataSource>(
  () => NumberTriviaLocalDataSourceImpl(sharedPreferences: sl()),
);

//! Core
...
sl.registerLazySingleton<NetworkInfo>(() => NetworkInfoImpl(sl()));
```

## Phụ thuộc bên ngoài

Chúng tôi đã chuyển toàn bộ cuộc gọi xuống vương quốc của các thư viện bên thứ 3. Chúng ta cần phải đăng ký `http.Client`, `DataConnectionChecker` và `SharedPreferences`. Cuối cùng là một chút khó khăn.

Không giống như tất cả các lớp khác, `SharedPreferences` không thể được khởi tạo đơn giản bằng một lệnh gọi constructor thông thường. Thay vào đó, chúng ta phải gọi `SharedPreferences.getInstance()` - đó là một **asynchronous method** (phương thức không đồng bộ) ! Bạn có thể nghĩ rằng chúng ta có thể làm điều này một cách đơn giản:

`sl.registerLazySingleton(() async => await SharedPreferences.getInstance());`

Tuy nhiên, hàm bậc cao hơn sẽ trả về `Future<SharedPreferences>`, đây không phải là điều chúng ta muốn. Chúng tôi muốn đăng ký một instance đơn giản `SharedPreferences` thay thế.

Đối với điều đó, chúng ta cần phải `await` gọi ra `getInstance()` bên ngoài để đăng ký. Điều này sẽ yêu cầu chúng tôi thay đổi `init()` method:

```
injection_container.dart
```
```
Future<void> init() async {
  ...
  //! External
  final sharedPreferences = await SharedPreferences.getInstance();
  sl.registerLazySingleton(() => sharedPreferences);
  sl.registerLazySingleton(() => http.Client());
  sl.registerLazySingleton(() => DataConnectionChecker());
}
```

# Initializing

Nơi tốt nhất để khởi tạo **service locator** là bên trong hàm `main()`

```
main.dart
```
```
import 'injection_container.dart' as di;

void main() async {
  await di.init();
  runApp(MyApp());
}
```

*Nó quan trọng để `await` `Future` mặc dù nó chỉ chứa `void`. Chúng tôi chắc chắn không muốn giao diện người dùng được xây dựng lên trước bất kỳ sự phụ thuộc được đăng ký.*

# Tiếp theo

Tiêm phụ thuộc là liên kết bị thiếu giữa production code và test code, trong đó chúng tôi sắp xếp các phụ thuộc theo cách thủ công với các lớp được Mock. Bây giờ chúng ta đã triển khai thực hiện **service locator**, không có gì có thể ngăn cản chúng tôi tiến hành viết widget Flutter. Chúng tôi sẽ sử dụng tất cả các phần mà chúng tôi đã viết, để hiển thị một giao diện người dùng đầy đủ chức năng. 

