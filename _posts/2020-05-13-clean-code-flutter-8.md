---
layout: post
title: Flutter TDD Clean Architecture Course [8] - Local Data Source
image: https://2.bp.blogspot.com/-a9nR2A_99yg/VMXJ2cBXE2I/AAAAAAAAB_s/O_gXstMAZ9I/s1600/tdd.png
---

*Sự phụ thuộc tiếp theo của **Kho lưu trữ** là **local Data Source** được sử dụng để lưu trữ dữ liệu được lấy từ API từ xa. Chúng tôi sẽ triển khai nó bằng **shared_preferences**.*

Có rất nhiều tùy chọn để lựa chọn khi nói đến việc truy xuất dữ liệu cục bộ. Chúng tôi đang sử dụng **shared_preferences** vì chúng tôi không lưu trữ nhiều dữ liệu - chỉ một **NumberTrivia** sẽ được chuyển đổi thành JSON. Như thường lệ, chúng ta hãy thiết lập thử nghiệm đầu tiên dưới một vị trí thử nghiệm được ánh xạ.

```
number_trivia_local_data_source_test.dart
```
```
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:matcher/matcher.dart';
import 'package:shared_preferences/shared_preferences.dart';

import '../../../../fixtures/fixture_reader.dart';

class MockSharedPreferences extends Mock implements SharedPreferences {}

void main() {
  NumberTriviaLocalDataSourceImpl dataSource;
  MockSharedPreferences mockSharedPreferences;

  setUp(() {
    mockSharedPreferences = MockSharedPreferences();
    dataSource = NumberTriviaLocalDataSourceImpl(
      sharedPreferences: mockSharedPreferences,
    );
  });
}
```

Theo những gì chúng tôi vừa quy định trong thử nghiệm, chúng tôi sẽ tạo  lớp **implementation** bên dưới lớp abstract.

```
number_trivia_local_data_source.dart
```
```
...
class NumberTriviaLocalDataSourceImpl implements NumberTriviaLocalDataSource {
  final SharedPreferences sharedPreferences;

  NumberTriviaLocalDataSourceImpl({@required this.sharedPreferences});

  @override
  Future<NumberTriviaModel> getLastNumberTrivia() {
    // TODO: implement getLastNumberTrivia
    return null;
  }

  @override
  Future<void> cacheNumberTrivia(NumberTriviaModel triviaToCache) {
    // TODO: implement cacheNumberTrivia
    return null;
  }
}
```

Trước tiên hãy tập trung vào phương thức **getLastNumberTrivia** . Khi gọi nó sẽ return về **NumberTriviaModel** từ shared preferences tất nhiên, với điều kiện là nó đã được lưu trữ trước đó. Làm thế nào model sẽ được lưu trữ bên trong preferences? Và làm thế nào chúng ta có thể kiểm tra tất cả những điều này?

# Objects và Shared Preferences

Lưu trữ đối tượng phức tạp như **NumberTrivia**  trong shared preferences chỉ có thể sử dụng định dạng string. Do đó, giá trị được trả về từ đối tượng SharedPreferences được giả định sẽ là một chuỗi JSON, cũng giống như phản hồi API là một chuỗi JSON. Từ các phần trước, bạn đã biết cách làm việc tốt nhất với JSON trong các bài kiểm tra - **fixtures**!

Chúng ta có thể sử dụng **fixtures** chúng ta đã tạo trong các phần trước không? Vâng, KHÔNG. Sử dụng chúng chắc chắn sẽ không phá vỡ bất cứ điều gì, nhưng điều đó không nhất thiết phải áp dụng cho các ứng dụng phức tạp hơn. Tại sao? **trivia.json**  và  **trivia_double.json**  chứa một số thuộc tính và các giá trị mà chỉ cho trường hợp là API.

```
trivia.json
```
```
{
  "text": "Test Text",
  "number": 1,
  "found": true,
  "type": "trivia"
}
```

Bất cứ khi nào **NumberTriviaModel** được chuyển đổi thành JSON (điều này cũng sẽ xảy ra trong **local Data Source** ), phương thức toJson return ra Map chứa các trường **text**  và **number**  . Map JSON này sẽ được lưu trong preferences: 

```
number_trivia_model.dart
```
```
Map<String, dynamic> toJson() {
  return {
    'text': text,
    'number': number,
  };
}
```

Do đó, hãy tạo ra một fixture khác để giả định một **NumberTriviaModel** . Tập tin mới được đặt trong **test/fixtures** có tên  là  **trivia_cached.json** .

```
trivia_cached.json
{
  "text": "Test Text",
  "number": 1
}
```

# getLastNumberTrivia

Bắt đầu với thử nghiệm, chúng tôi xác định chức năng đầu tiên của phương th này. Nó *sẽ trả về NumberTrivia từ SharedPreferences khi có dữ liệ trong cache*.

```
number_trivia_local_data_source_test.dart
```
```
group('getLastNumberTrivia', () {
  final tNumberTriviaModel =
      NumberTriviaModel.fromJson(json.decode(fixture('trivia_cached.json')));

  test(
    'should return NumberTrivia from SharedPreferences when there is one in the cache',
    () async {
      // arrange
      when(mockSharedPreferences.getString(any))
          .thenReturn(fixture('trivia_cached.json'));
      // act
      final result = await dataSource.getLastNumberTrivia();
      // assert
      verify(mockSharedPreferences.getString('CACHED_NUMBER_TRIVIA'));
      expect(result, equals(tNumberTriviaModel));
    },
  );
});
```

Làm theo quy định của bài kiểm tra, chúng tôi thực hiện phương pháp để làm cho nó pass qua. Vì kiểu trả về của phương thức là **Future** và **SharedPreferences** có lẽ là thư viện lưu trữ cục bộ đồng bộ duy nhất ngoài đó, chúng tôi sẽ sử dụng một **Future.value** factory để trả về một **Future** đã hoàn thành  .

```
number_trivia_local_data_source.dart
```
```
@override
Future<NumberTriviaModel> getLastNumberTrivia() {
  final jsonString = sharedPreferences.getString('CACHED_NUMBER_TRIVIA');
  // Future which is immediately completed
  return Future.value(NumberTriviaModel.fromJson(json.decode(jsonString)));
}
```

Bài kiểm tra sẽ pass ngay bây giờ và hãy nhảy ngay vào giai đoạn tái cấu trúc. Tôi chắc chắn không thích truyền qua các chuỗi như 'CACHED_NUMBER_TRIVIA' , vì vậy hãy tạo một hằng số có cùng tên và sử dụng nó trong suốt quá trình production và tệp test .

```
number_trivia_local_data_source.dart
```
```
const CACHED_NUMBER_TRIVIA = 'CACHED_NUMBER_TRIVIA';

class NumberTriviaLocalDataSourceImpl implements NumberTriviaLocalDataSource {
...
}
```

Chúng ta không thể tin rằng sẽ luôn có một phiên bản mới nhất được lưu trong bộ nhớ cache của **NumberTrivia**. Điều gì xảy ra nếu người dùng khởi chạy ứng dụng lần đầu tiên trong khi KHÔNG có kết nối Internet? Trong trường hợp như vậy,  **Kho lưu trữ**  sẽ ngay lập tức chuyển sang  **SharedPreferences**  và chúng sẽ trả về  `null` .

Chào người dùng đầu tiên với sự cố ứng dụng chắc chắn sẽ không phải là trải nghiệm người dùng tốt! Để ngăn chặn bất kỳ sự cố nào xảy ra, chúng tôi sẽ kiểm soát việc ném **CacheException**. Nếu bạn nhớ từ phần trước, ngoại lệ này được bắt gặp trong **Kho lưu trữ**  return về một **Left(CacheFailure())**.

```
test.dart
```
```
test('should throw a CacheException when there is not a cached value', () {
  // arrange
  when(mockSharedPreferences.getString(any)).thenReturn(null);
  // act
  // Not calling the method here, just storing it inside a call variable
  final call = dataSource.getLastNumberTrivia;
  // assert
  // Calling the method happens from a higher-order function passed.
  // This is needed to test if calling a method throws an exception.
  expect(() => call(), throwsA(TypeMatcher<CacheException>()));
});
```

Thực hiện các thử nghiệm là đơn giản như thêm một câu lệnh `if`:

```
implementation.dart
```
```
@override
Future<NumberTriviaModel> getLastNumberTrivia() {
  final jsonString = sharedPreferences.getString('CACHED_NUMBER_TRIVIA');
  if (jsonString != null) {
    return Future.value(NumberTriviaModel.fromJson(json.decode(jsonString)));
  } else {
    throw CacheException();
  }
}
```

# cacheNumberTrivia

Phương thức put dữ liệu vào preferences chỉ nên *gọi SharedPreferences để lưu trữ dữ liệu*. Chúng tôi thực sự không thể kiểm tra nếu dữ liệu hiện diện bên trong preferences (ít nhất là trong một unit test). Điều tốt nhất tiếp theo là sử dụng sức mạnh của các mock để xác minh xem trường hợp giả định đã được gọi với các đối số phù hợp chưa.

Xét cho cùng, chuỗi JSON được tạo bởi phương thức `toJson` của Model và chuỗi được lưu trữ bên trong các preferences phải hoàn toàn giống nhau.

```
test.dart
```
```
group('cacheNumberTrivia', () {
  final tNumberTriviaModel =
      NumberTriviaModel(number: 1, text: 'test trivia');

  test('should call SharedPreferences to cache the data', () {
    // act
    dataSource.cacheNumberTrivia(tNumberTriviaModel);
    // assert
    final expectedJsonString = json.encode(tNumberTriviaModel.toJson());
    verify(mockSharedPreferences.setString(
      CACHED_NUMBER_TRIVIA,
      expectedJsonString,
    ));
  });
});

```

```
implementation.dart
```
```
@override
Future<void> cacheNumberTrivia(NumberTriviaModel triviaToCache) {
  return sharedPreferences.setString(
    CACHED_NUMBER_TRIVIA,
    json.encode(triviaToCache.toJson()),
  );
}
```

# Tiếp theo

Trong phần này, chúng tôi đã triển khai lớp **NumberTriviaLocalDataSource** thực hiện TDD. Phần còn lại cuối cùng của **lớp dữ liệu** là **remote Data Source** mà chúng ta sẽ triển khai tiếp theo. Điều này có nghĩa là chúng tôi sẽ thực hiện phát triển dựa trên thử nghiệm với gói **http** .