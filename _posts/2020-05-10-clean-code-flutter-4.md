---
layout: post
title: Flutter TDD Clean Architecture Course [4] - Data Layer Overview & Models
image: https://2.bp.blogspot.com/-a9nR2A_99yg/VMXJ2cBXE2I/AAAAAAAAB_s/O_gXstMAZ9I/s1600/tdd.png
---

*Mặc dù **domain layer** là nơi an toàn của một ứng dụng độc lập với các lớp khác, nhưng **data layer** là nơi ứng dụng kết nối với thế giới của API và thư viện của bên thứ 3. Nó bao gồm các low-level **Data Sources, Repositories** là nguồn dữ liệu thật và cuối cùng là  **Models***

# Tiến ra ngoài

Bạn có thể nhận thấy rằng chúng tôi luôn bắt đầu làm việc từ các phần bên trong của ứng dụng và làm việc theo cách của chúng tôi đến vùng ngoài. Sau tất cả, chúng tôi đã bắt đầu với độc lập hoàn toàn **domain**  và cũng đã tạo ra  Entity . Bây giờ, chúng ta sẽ bắt đầu với **Model** và sau đó tiếp tục triển khai **Repository**  và low-level **Data Sources** . Tại sao?

Toàn bộ Kiến trúc TDD được xác định theo một nguyên tắc cơ bản - các lớp bên ngoài phụ thuộc vào các lớp bên trong, được biểu thị bằng các  mũi tên dọc  --->  tại củ hành dưới đây.

!['Dependencies flow inwardx'](https://i1.wp.com/resocoder.com/wp-content/uploads/2019/08/CleanArchitecture.jpg?w=772&ssl=1)

# Tổng quan về Data Layer

**Repository implementation** sẽ được thực thi dựa trên "hợp đồng" sau đây từ **domain layer**.

```
number_trivia_repository.dart
```
```
abstract class NumberTriviaRepository {
  Future<Either<Failure, NumberTrivia>> getConcreteNumberTrivia(int number);
  Future<Either<Failure, NumberTrivia>> getRandomNumberTrivia();
}
```

Trước khi tạo **Repository implementation**, chúng ta sẽ phải tạo **remote** và **local Data Sources**

Một lần nữa, chúng ta sẽ không cần phải tạo ra các triển khai của chúng ngay lập tức. Nó sẽ đủ để thực hiện các hợp đồng của chúng ,  mà chúng phải hoàn thành, sử dụng các **abstract classes** . Sau đó chúng tôi sẽ mô phỏng các **data sources** trong khi viết các test cho  **repository** trong các phần sau của khóa học này. Nhưng trước tiên...

![](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/08/data-layer-diagram.png?w=329&ssl=1)

# Sự cần thiết cúa Model

Các kiểu trả về của các **data sources**  sẽ rất giống với các kiểu trong **repository**  nhưng có hai sự khác biệt lớn. Chúng sẽ  không  trả lại lỗi bằng cách sử dụng  Failure , thay vào đó họ sẽ ném ra các ngoại lệ. Ngoài ra, thay vì return các **NumberTrivia entities**, chúng sẽ return các **NumberTriviaModel**.

Tất cả điều này xảy ra bởi vì **data sources** nằm ở ranh giới tuyệt đối giữa thế giới tốt đẹp và ấm cúng trong code của chúng ta và thế giới bên ngoài đáng sợ của API hay thư viện của bên thứ 3.

**Models** là các thực thể với một số chức năng bổ sung được thêm vào đầu. Trong trường hợp của chúng tôi, đó sẽ là to/from JSON. API sẽ respond với dữ liệu theo định dạng JSON, vì vậy chúng ta cần có cách để chuyển đổi nó thành các đối tượng trong Dart.

Bạn có thể nhận thấy rằng  **NumberTriviaModel**  sẽ chứa một số *logic* chuyển đổi JSON  . Từ này sẽ bắt đầu lóe sáng lên trong đầu bạn bởi vì nó có nghĩa là employing sự phát triển dựa trên TDD.

# Implementing Model với TDD

Bản thân file cho model sẽ nằm trong thư mục **models** cho tính năng **number_trivia** . Như mọi khi, bài test đi kèm của nó sẽ ở cùng một vị trí, chỉ liên quan đến **test** folder.

![](https://i2.wp.com/resocoder.com/wp-content/uploads/2019/09/number_trivia_model.png?w=339&ssl=1)

Vì mối quan hệ giữa **Model** và **Entity** là rất quan trọng, chúng tôi sẽ kiểm tra xem nó có là một giấc ngủ ngon không.

```
number_trivia_model_test.dart
```
```
import 'package:clean_architecture_tdd_prep/features/number_trivia/data/models/number_trivia_model.dart';
import 'package:clean_architecture_tdd_prep/features/number_trivia/domain/entities/number_trivia.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  final tNumberTriviaModel = NumberTriviaModel(number: 1, text: 'Test Text');

  test(
    'should be a subclass of NumberTrivia entity',
    () async {
      // assert
      expect(tNumberTriviaModel, isA<NumberTrivia>());
    },
  );
}
```

Để thực hiện biên dịch và vượt qua thử nghiệm ở trên, **NumberTriviaModel** sẽ extends **NumberTrivia**:

```
number_trivia_model.dart
```
```
import 'package:meta/meta.dart';

import '../../domain/entities/number_trivia.dart';

class NumberTriviaModel extends NumberTrivia {
  NumberTriviaModel({
    @required String text,
    @required int number,
  }) : super(
          text: text,
          number: number,
        );
}
```

Logic chuyển đổi đầu tiên của chúng tôi sẽ triển khai sẽ là phương thức `fromJson` sẽ trả về một **NumberTriviaModel** instance có cùng dữ liệu như trong chuỗi JSON.

Chúng tôi sẽ không nhận chuỗi JSON từ ""live" Number API. Thay vào đó, chúng tôi sẽ tạo một **test fixture** là một tệp JSON thông thường được sử dụng để thử nghiệm. Đó là bởi vì chúng tôi muốn có một chuỗi JSON có thể dự đoán được để kiểm tra - ví dụ: nếu API số đang được bảo trì thì sao? Chúng tôi không muốn bất kỳ lực lượng bên ngoài nào gây rối với kết quả kiểm tra của chúng tôi.

*"**test fixture** là thứ được sử dụng để liên tục kiểm tra một số vật phẩm, thiết bị hoặc phần mềm." - Wikipedia*

## Tạo Fixtures

Nội dung của **fixture** sẽ bắt chước phản hồi JSON từ API. Hãy xem phản hồi từ điểm cuối ngẫu nhiên ( http://numbersapi.com/random/trivia?json ) trông như thế nào:

```
response.json
```
```
{
 "text": "418 is the error code for \"I'm a teapot\" in the Hyper Text Coffee Pot Control Protocol.",
 "number": 418,
 "found": true,
 "type": "trivia"
}
```

Chúng tôi phải đảm bảo biết về tất cả các trường hợp khác nhau để trả lời. Ví dụ, số không phải luôn luôn là một số nguyên đẹp. Đôi khi, nó có thể là thứ mà Dart coi là số thực , mặc dù nó thực sự chỉ là một số nguyên.

```
response.json
```
```
{
 "text": "4e+185 is the number of planck volumes in the observable universe.",
 "number": 4e+185,
 "found": true,
 "type": "trivia"
}
```

Bây giờ chúng tôi sẽ thực hiện các phản hồi này và "đóng băng chúng tại chỗ" bằng cách tạo hai fixtures -  `trivia.json`  và  `trivia_double.json` . Tất cả chúng sẽ đi vào một thư mục gọi là **fixtures** được lồng ngay bên trong  thư mục **test**  .

Mặc dù các tệp fixture JSON này phải có cùng các trường với các phản hồi thực tế, chúng tôi sẽ làm cho các giá trị của chúng đơn giản hơn để giúp chúng tôi dễ dàng viết các bài kiểm tra với chúng.

![](https://i2.wp.com/resocoder.com/wp-content/uploads/2019/09/fixtures-folder.png?w=265&ssl=1)

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

Mặc dù API đã trả lời số `4e + 185` ( double trong mắt của Dart), chúng ta có thể đạt được điều tương tự với số  1.0 - thực tế nó vẫn là một số nguyên , nhưng Dart sẽ xử lý nó như một số double .

```
trivia_double.json
{
  "text": "Test Text",
  "number": 1.0,
  "found": true,
  "type": "trivia"
}
```

**Reading Fixture Files**

Bây giờ chúng ta đã đưa các phản hồi JSON giả vào các tệp. Để sử dụng JSON chứa bên trong chúng, chúng ta phải có cách lấy nội dung của các tệp này dưới dạng String . Vì vậy, chúng tôi sẽ tạo ra một hàm cấp cao nhất được gọi là  fixture bên trong tệp  fixture_reader.dart (nó đi vào  thư mục fixture  ).

```
fixture_reader.dart
```
```
import 'dart:io';

String fixture(String name) => File('test/fixtures/$name').readAsStringSync();
```

**fromJson**

Với fixtures, cuối cùng chúng ta có thể bắt đầu với  phương thức fromJson . Theo tinh thần của TDD, chúng tôi sẽ viết các bài kiểm tra trước. Như là một tùy chỉnh trong Dart,  `fromJson` luôn lấy  `Map <String, Dynamic>`  làm đối số và xuất ra một type, trong trường hợp này là  NumberTriviaModel .

```
number_trivia_model_test.dart
```
```
void main() {
  final tNumberTriviaModel = NumberTriviaModel(number: 1, text: 'Test Text');
  ...
  group('fromJson', () {
    test(
      'should return a valid model when the JSON number is an integer',
      () async {
        // arrange
        final Map<String, dynamic> jsonMap =
            json.decode(fixture('trivia.json'));
        // act
        final result = NumberTriviaModel.fromJson(jsonMap);
        // assert
        expect(result, tNumberTriviaModel);
      },
    );
  });
}
```

Như bạn có thể thấy, chúng tôi đã nhận được Map JSON   từ tệp trivia.json . Thử nghiệm này sẽ thất bại, trên thực tế, nó sẽ không được biên dịch! Hãy thực hiện phương thức `fromJson` .

```
number_trivia_model.dart
```
```
class NumberTriviaModel extends NumberTrivia {
  ...
  factory NumberTriviaModel.fromJson(Map<String, dynamic> json) {
    return NumberTriviaModel(
      text: json['text'],
      number: json['number'],
    );
  }
}
```

Sau khi chạy thử, nó passes ! Sooo, rất tuyệt phải không? Chưa. Chúng tôi cũng phải kiểm tra tất cả các trường hợp khác, như khi số bên trong JSON sẽ được coi là double (khi giá trị là  4e + 185  hoặc 1.0 ) Hãy thực hiện kiểm tra cho điều đó.

```
number_trivia_model_test.dart
```
```
group('fromJson', () {
  ...
  test(
    'should return a valid model when the JSON number is regarded as a double',
    () async {
      // arrange
      final Map<String, dynamic> jsonMap =
          json.decode(fixture('trivia_double.json'));
      // act
      final result = NumberTriviaModel.fromJson(jsonMap);
      // assert
      expect(result, tNumberTriviaModel);
    },
  );
});
```

Ồ không, thử nghiệm  thất bại: "type 'double' is not a subtype of type 'int'". Hmm, có thể đó là vì trường number  là kiểu  int ? Điều gì sẽ xảy ra nếu chúng ta cast double thành int? Nó sẽ làm việc sau đó?

```
number_trivia_model.dart
```
```
...
factory NumberTriviaModel.fromJson(Map<String, dynamic> json) {
  return NumberTriviaModel(
    text: json['text'],
    // The 'num' type can be both a 'double' and an 'int'
    number: (json['number'] as num).toInt(),
  );
}
...
```

Sure, điều này đã sửa lỗi cast và bây giờ thử nghiệm vượt qua !

**toJson**

Bây giờ chúng ta hãy hướng đến phương thức chuyển đổi thứ hai -  `toJson`. Theo quy ước, đây là một instance method trả về Map<String, dynamic>. Viết bài kiểm tra trước ...

```
number_trivia_model_test.dart
```
```
...
final tNumberTriviaModel = NumberTriviaModel(number: 1, text: 'Test Text');
...
group('toJson', () {
  test(
    'should return a JSON map containing the proper data',
    () async {
      // act
      final result = tNumberTriviaModel.toJson();
      // assert
      final expectedJsonMap = {
        "text": "Test Text",
        "number": 1,
      };
      expect(result, expectedJsonMap);
    },
  );
});
...
```

Trước cả khi chạy nó, một lần nữa chúng ta sẽ thực hiện   phương thức `toJson` để loại bỏ các lỗi "method not present".

```
number_trivia_model.dart
```
```
class NumberTriviaModel extends NumberTrivia {
  ...
  Map<String, dynamic> toJson() {
    return {
      'text': text,
      'number': number,
    };
  }
}
```

Và tất nhiên, bài kiểm tra đã pass . Vì không có bất kỳ trường hợp khác nào khi chuyển đổi  sang JSON (xét cho cùng, chúng tôi đang kiểm soát các loại dữ liệu ở đây), thử nghiệm duy nhất này là đủ cho phương thức toJson .

# Tiếp theo
Bây giờ chúng ta đã có Model có thể được chuyển đổi to/from JSON, chúng ta sẽ bắt đầu làm việc với các  **Repository implementation**  và  **Data Source contracts**.

