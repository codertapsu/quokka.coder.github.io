---
layout: post
title: Flutter TDD Clean Architecture Course [9] - Remote Data Source
image: https://2.bp.blogspot.com/-a9nR2A_99yg/VMXJ2cBXE2I/AAAAAAAAB_s/O_gXstMAZ9I/s1600/tdd.png
---

Phần còn lại cuối cùng của **lớp dữ liệu** mà chúng tôi hiện có hợp đồng là **remote Data Source**. Đây là nơi tất cả các giao tiếp với Numbers API sẽ xảy ra, chúng tôi sẽ sử dụng gói **http** . Tất cả điều này sẽ được thực hiện khi phát triển dựa trên thử nghiệm.

# Thiết lập Test

Tệp **number_trivia_remote_data_source.dart**  hiện chỉ chứa  hợp đồng  của **Data Source**. Chúng tôi sẽ đặt việc thực hiện vào cùng một tệp, bên dưới lớp abstract. Như thường lệ, hãy tạo một tệp thử nghiệm ánh xạ vị trí của tệp đã nói ở trên - **test / features / number_trivia / data / datasources.**.

Để thiết lập tệp thử nghiệm, chúng tôi sẽ tạo một Client giả định từ gói **http** và chuyển nó vào **remote Data Source** implementation.

```
number_trivia_remote_data_source_test.dart
```
```
import 'package:http/http.dart' as http;
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:matcher/matcher.dart';

import '../../../../fixtures/fixture_reader.dart';

class MockHttpClient extends Mock implements http.Client {}

void main() {
  NumberTriviaRemoteDataSourceImpl dataSource;
  MockHttpClient mockHttpClient;

  setUp(() {
    mockHttpClient = MockHttpClient();
    dataSource = NumberTriviaRemoteDataSourceImpl(client: mockHttpClient);
  });
}
```

Để làm cho tệp thử nghiệm hài lòng, chúng tôi sẽ tạo một implementation của **remote Data Source**.

```
number_trivia_remote_data_source.dart
```
```
...
class NumberTriviaRemoteDataSourceImpl implements NumberTriviaRemoteDataSource {
  final http.Client client;

  NumberTriviaRemoteDataSourceImpl({@required this.client});

  @override
  Future<NumberTriviaModel> getConcreteNumberTrivia(int number) {
    // TODO: implement getConcreteNumberTrivia
    return null;
  }

  @override
  Future<NumberTriviaModel> getRandomNumberTrivia() {
    // TODO: implement getRandomNumberTrivia
    return null;
  }
}
```

Tương tự như cách có rất nhiều sự chồng chéo giữa các phương thức *concrete* và *random* của **Kho lưu trữ**được triển khai trong phần 6 , điều tương tự cũng sẽ đúng với **remote Data Source**. Hãy bắt đầu với  phương thức **getConcittleNumberTrivia** , viết các bài kiểm tra và thực hiện từng chút một.

# getConcreteNumberTrivia

Tóm lại, chúng tôi đang làm việc với API Numbers. Đối với số 42, nó trông như thế này: [http://numbersapi.com/42](http://numbersapi.com/42). 

Vấn đề là, việc thực hiện một yêu cầu GET trên loại URL đó sẽ cho chúng ta một phản hồi văn bản đơn giản  chỉ chứa chuỗi câu đố. Nhưng chúng tôi muốn nhận được phản hồi JSON !

Có thể lấy dữ liệu từ API theo `application/json` header bằng hai cách:

1. Nối một tham số truy vấn vào URL, làm cho nó trông như thế này: [ http://numbersapi.com/42?json]( http://numbersapi.com/42?json)
2. Gửi `Content-Type: application / json` header cùng với yêu cầu GET.


Chúng tôi sẽ chọn tùy chọn thứ hai. Vì việc lấy URL và tiêu đề rất quan trọng, chúng tôi sẽ tạo một bài kiểm tra dành riêng cho chúng.

```
test.dart
```
```
group('getConcreteNumberTrivia', () {
  final tNumber = 1;

  test(
    'should preform a GET request on a URL with number being the endpoint and with application/json header',
    () {
      //arrange
      when(mockHttpClient.get(any, headers: anyNamed('headers'))).thenAnswer(
        (_) async => http.Response(fixture('trivia.json'), 200),
      );
      // act
      dataSource.getConcreteNumberTrivia(tNumber);
      // assert
      verify(mockHttpClient.get(
        'http://numbersapi.com/$tNumber',
        headers: {'Content-Type': 'application/json'},
      ));
    },
  );
}
```

Với thử nghiệm thất bại , chúng tôi sẽ chỉ viết một số mã sản xuất là đủ để thử nghiệm pass.

```
implementation.dart
```
```
@override
Future<NumberTriviaModel> getConcreteNumberTrivia(int number) {
  client.get(
    'http://numbersapi.com/$number',
    headers: {'Content-Type': 'application/json'},
  );
}
```

Có vẻ như  phần *arrange* của bài test này là không cần thiết. Rốt cuộc, chúng tôi không làm bất cứ điều gì với **Response** được return lại.

Trong khi lập luận này là đúng so với bây giờ, khi chúng tôi thêm chức năng cho việc thực hiện phương thức, không *arranging* các **mockHttpClient** để trả về một giá trị **Response** đối tượng sẽ gây ra tất cả các loại lỗi không mong muốn. Đó là bởi vì một phương thức trên một mock mà trước đây không được thiết lập sẽ trả về `null` .

```
test.dart
```
```
final tNumberTriviaModel =
    NumberTriviaModel.fromJson(json.decode(fixture('trivia.json')));
...
test(
  'should return NumberTrivia when the response code is 200 (success)',
  () async {
    // arrange
    when(mockHttpClient.get(any, headers: anyNamed('headers'))).thenAnswer(
      (_) async => http.Response(fixture('trivia.json'), 200),
    );
    // act
    final result = await dataSource.getConcreteNumberTrivia(tNumber);
    // assert
    expect(result, equals(tNumberTriviaModel));
  },
);
```

Chúng tôi kiểm tra xem dữ liệu trong **result** của việc gọi phương thức có phải là những gì chúng tôi mong đợi hay không bằng cách so sánh nó với **tNumberTriviaModel** , một trường hợp được xây dựng từ cùng một JSON fixture như được trả về bởi **mockHttpClient** .

```
implementation.dart
```
```
@override
Future<NumberTriviaModel> getConcreteNumberTrivia(int number) async {
  final response = await client.get(
    'http://numbersapi.com/$number',
    headers: {'Content-Type': 'application/json'},
  );

  return NumberTriviaModel.fromJson(json.decode(response.body));
}
```

Cuối cùng, chúng ta không được quên tính đến một trường hợp khi có sự cố xảy ra. Nếu mã phản hồi là `404 NOT FOUND`  hoặc bất kỳ mã lỗi nào khác (bất kỳ thứ gì khác 200 ), một `ServerException`  sẽ được ném ra.

Tương tự như cách chúng tôi đã kiểm tra nếu phương thức đưa ra một ngoại lệ trong phần  trước , bây giờ chúng tôi cũng làm như vậy. Để giữ cho code cleans, chúng ta lưu trữ phương thức bên trong một biến **call** và gọi nó từ bên trong một hàm bậc cao hơn được truyền vào phương thức **expect** .

```
test.dart
```
```
test(
  'should throw a ServerException when the response code is 404 or other',
  () async {
    // arrange
    when(mockHttpClient.get(any, headers: anyNamed('headers'))).thenAnswer(
      (_) async => http.Response('Something went wrong', 404),
    );
    // act
    final call = dataSource.getConcreteNumberTrivia;
    // assert
    expect(() => call(tNumber), throwsA(TypeMatcher<ServerException>()));
  },
);
```

```
implementation.dart
```
```
@override
Future<NumberTriviaModel> getConcreteNumberTrivia(int number) async {
  final response = await client.get(
    'http://numbersapi.com/$number',
    headers: {'Content-Type': 'application/json'},
  );

  if (response.statusCode == 200) {
    return NumberTriviaModel.fromJson(json.decode(response.body));
  } else {
    throw ServerException();
  }
}
```

Tất nhiên, ngoại lệ này được xử lý trong **Kho lưu trữ**. Bây giờ chúng ta đã thực hiện đầy đủ phương thức **getConcreteNumberTrivia** .

# DRY ngay cả trong các thử nghiệm bên trong

DRY (**D**on't **R**epeat **Y**ourself) nguyên tắc là cốt lõi của bất kỳ loại lập trình, cho dù bạn đang làm TDD hay không. Sao chép code trong các thử nghiệm cũng tệ như sao chép code sản xuất. Như bạn có thể thấy, phần sắp xếp của hai bài kiểm tra đầu tiên hoàn toàn giống nhau - return  200 SUCCESS . Chúng ta không thể chuyển thiết lập của mock sang  phương thức **setUp**  cho toàn bộ nhóm thử nghiệm  , bởi vì  phần *arrange* của thử nghiệm cuối cùng sẽ return  404 NOT FOUND .

```
test.dart
```
```
void main() {
  ...

  void setUpMockHttpClientSuccess200() {
    when(mockHttpClient.get(any, headers: anyNamed('headers'))).thenAnswer(
      (_) async => http.Response(fixture('trivia.json'), 200),
    );
  }

  void setUpMockHttpClientFailure404() {
    when(mockHttpClient.get(any, headers: anyNamed('headers'))).thenAnswer(
      (_) async => http.Response('Something went wrong', 404),
    );
  }
  
  // Change the code below to use these methods...
}
```

# getRandomNumberTrivia

Tương tự như khi chúng tôi triển khai **Kho lưu trữ** , sự khác biệt giữa các phương thức *concrete* và  *random* sẽ là tối thiểu. Trên thực tế, tất cả những gì cần thiết để có được một số ngẫu nhiên là thay đổi url từ `http://numbersapi.com/some_number`  thành `http://numbersapi.com/random` .

Vì sự giống nhau này, tôi hy vọng bạn sẽ tha thứ cho tôi một lần nữa vì đã không hoàn toàn tuân theo kinh điển của TDD. Chúng tôi sắp sao chép, dán và sửa đổi một số mã! Các sửa đổi rất đơn giản - chỉ cần đổi tên bất kỳ tham chiếu nào của *concrete* thành *random* và bạn đã thiết lập:

```
test.dart
```
```
group('getRandomNumberTrivia', () {
  final tNumberTriviaModel =
      NumberTriviaModel.fromJson(json.decode(fixture('trivia.json')));

  test(
    'should preform a GET request on a URL with *random* endpoint with application/json header',
    () {
      //arrange
      setUpMockHttpClientSuccess200();
      // act
      dataSource.getRandomNumberTrivia();
      // assert
      verify(mockHttpClient.get(
        'http://numbersapi.com/random',
        headers: {'Content-Type': 'application/json'},
      ));
    },
  );

  test(
    'should return NumberTrivia when the response code is 200 (success)',
    () async {
      // arrange
      setUpMockHttpClientSuccess200();
      // act
      final result = await dataSource.getRandomNumberTrivia();
      // assert
      expect(result, equals(tNumberTriviaModel));
    },
  );

  test(
    'should throw a ServerException when the response code is 404 or other',
    () async {
      // arrange
      setUpMockHttpClientFailure404();
      // act
      final call = dataSource.getRandomNumberTrivia;
      // assert
      expect(() => call(), throwsA(TypeMatcher<ServerException>()));
    },
  );
});
```

Đối với code sản xuất, theo nghĩa đen, chúng tôi sẽ sao chép và dán mã từ **getConcreteNumberTriviaMethod** và chỉ thay đổi "**$number**" được trong URL thành "**random**".

```
implementation.dart
```
```
@override
Future<NumberTriviaModel> getRandomNumberTrivia() async {
  final response = await client.get(
    'http://numbersapi.com/random',
    headers: {'Content-Type': 'application/json'},
  );

  if (response.statusCode == 200) {
    return NumberTriviaModel.fromJson(json.decode(response.body));
  } else {
    throw ServerException();
  }
}
```

Tất cả các bài kiểm tra đang pass ! Tất nhiên, loại trùng lặp này là hoàn toàn không thể tha thứ được. Hãy tạo một phương thức helper! Nó sẽ nhận được URL để thực hiện yêu cầu GET dưới dạng đối số.

```
implementation.dart
```
```
@override
Future<NumberTriviaModel> getConcreteNumberTrivia(int number) =>
    _getTriviaFromUrl('http://numbersapi.com/$number');

@override
Future<NumberTriviaModel> getRandomNumberTrivia() =>
    _getTriviaFromUrl('http://numbersapi.com/random');

Future<NumberTriviaModel> _getTriviaFromUrl(String url) async {
  final response = await client.get(
    url,
    headers: {'Content-Type': 'application/json'},
  );

  if (response.statusCode == 200) {
    return NumberTriviaModel.fromJson(json.decode(response.body));
  } else {
    throw ServerException();
  }
}
```

Mặc dù nó rất khó khả thi, nhưng việc tái cấu trúc này vẫn có thể làm hỏng thứ gì đó. Chỉ sau khi kiểm tra lại code và thấy nó trôi qua , chúng ta mới có thể ngủ ngon vào tối nay...

# Tiếp theo

Chúng tôi đang đi khá nhanh. Chúng ta vừa hoàn thành toàn bộ **lớp dữ liệu**  và với **domain**  đã được triển khai, chỉ còn lại **presentation layer** . Chúng tôi sẽ một lần nữa giữ truyền thống và bắt đầu từ trung tâm của *layer*, đó là **presentation logic holder** . Trong trường hợp *Number Trivia App*, chúng tôi sẽ sử dụng **Bloc**.

