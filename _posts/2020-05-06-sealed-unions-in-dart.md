---
layout: post
title: Sealed Unions in Dart – Không bao giờ code lại nhiều câu lệnh If
---

Kiểm tra một subtype của một đối tượng được sử dụng trong nhiều lĩnh vực, ví dụ như trong *quản lý state với BLoC*  hoặc  *Redux*. Hãy xem xét về tất cả những events và state. Đã bao nhiêu lần bạn quên kiểm tra một trường hợp trong câu lệnh `if`  hoặc  `switch` ? Đơn giản là không có cách nào khác ngoài việc ghi nhớ các class khác nhau trong đầu bạn, để đảm bảo rằng mọi trường hợp đang được xét tới. 

Không còn như vậy nữa! Thực sự  có  một cách để khiến bạn  không bao giờ  quên về việc kiểm tra các trường hợp nhất định. Mặc dù nhiều ngôn ngữ khác có tính năng này được tích hợp trong chúng, đối với chúng tôi, các nhà phát triển của Dart có một gói cho nó -  *seal_unions*.


# Adding Dependencies

Tên gói [sealed_unions](https://pub.dev/packages/sealed_unions) xuất phát từ việc nối tên của hai khái niệm lập trình tương ứng - *sealed classes* và *tagged unions*. Bất kể bạn quyết định gọi nó như thế nào, gói này cung cấp một cách để làm cho code của bạn mạnh mẽ hơn.

Đầu tiên và quan trọng nhất, hãy thêm **seal_unions** vào dự án. Để đơn giản, hướng dẫn này sẽ được viết dưới dạng một ứng dụng đơn giản nhưng tất cả những thứ này cũng hoạt động với Flutter và tất cả các loại gói quản lý state.

```
pubspec.yaml
```
```
dependencies:
  sealed_unions: ^3.0.2+2
```

# Code mà không có Sealed Unions

Chúng tôi sẽ chứng minh gói này trong một lớp state ứng dụng. Hãy tưởng tượng chúng ta đang xây dựng một ứng dụng dự báo thời tiết đơn giản có ba trạng thái riêng biệt:

1. WeatherInitial  - điều này sẽ nhắc người dùng tìm kiếm thời tiết ở một thành phố cụ thể.
2. WeatherLoading  - hiển thị một animation loading.
3. WeatherLoaded  - hiển thị nhiệt độ trong thành phố đã chọn.

Tất cả các trạng thái này là các  lớp con của  lớp cơ sở `WeatherState`. Code Dart thông thường cho các trạng state mà không có sự trợ giúp của  gói **sealed_unions** sẽ trông như thế này (được đơn giản hóa cho ngắn gọn):

```
weather_state.dart
```
```
abstract class WeatherState {}

class WeatherInitial extends WeatherState {}

class WeatherLoading extends WeatherState {}

class WeatherLoaded extends WeatherState {
  final int temperature;

  WeatherLoaded(this.temperature);
}
```

Để kiểm tra chính xác trạng thái nào, ví dụ, được phát ra bởi một Bloc, chúng tôi sẽ có một câu lệnh `if` mà từ đó các Widget Flutter sẽ được return lại.

```
main.dart
```
```
// Imagine this returns Flutter widgets
String widgetBuilder(WeatherState state) {
  if (state is WeatherInitial) {
    return "Some initial widget";
  } else if (state is WeatherLoading) {
    return "Circular progress indicator";
  } else if (state is WeatherLoaded) {
    return "The temperature is ${state.temperature}";
  }
}
```

Có cách nào để thực thi mà có thể kiểm tra cho tất cả state có thể không? Ngoài ra, nếu chúng ta thêm state `WeatherError`, liệu chúng ta có được thông báo bằng cách nào đó rằng chúng ta nên xử lý nó theo phương pháp trên không? Không, chúng ta phải dựa vào chúng ta để nhớ lại những thứ không phải là một thực hành lập trình tốt.

# Sealed Unions to the Rescue

Code sẽ trông như thế nào khi chúng ta thêm unions vào? Hãy xem nào! Các lớp con hầu như không thay đổi, nhưng chúng sẽ không còn extend `WeatherState` và chúng sẽ được private package:

```
weather_state.dart
```
```
abstract class WeatherState {}

class _WeatherInitial {}

class _WeatherLoading {}

class _WeatherLoaded {
  final int temperature;

  _WeatherLoaded(this.temperature);
}
```

*Người dùng gói BLoC hãy cẩn thận! Hãy chắc chắn để sử dụng **equatable** cho tất cả các States.*

Với đoạn mã trên, chúng tôi hoàn toàn phá vỡ mọi mối quan hệ giữa các states riêng lẻ. Tất nhiên, chúng tôi sẽ ngay lập tức sửa lỗi này bằng cách thêm gói **sealed_unions** vào. Đáng buồn thay, vì nó đi với bất kỳ thư viện nào thay thế cho chức năng thiếu của Dart, nó sẽ không được áp dụng mà không có một số lượng boilerplate nhất định 😢

Trong cùng một tệp, chúng tôi sẽ sửa đổi lớp `WeatherState` "base" để thể hiện **Union** gồm 3 loại:

```
main.dart
import 'package:sealed_unions/sealed_unions.dart';

// All the possible types of WeatherState have to be specified here
// The package supports Union9 at max (9 types)
class WeatherState
    extends Union3Impl<_WeatherInitial, _WeatherLoading, _WeatherLoaded> {
  // PRIVATE low-level factory
  // Used for instantiating individual "subclasses"
  static final Triplet<_WeatherInitial, _WeatherLoading, _WeatherLoaded>
      _factory =
      const Triplet<_WeatherInitial, _WeatherLoading, _WeatherLoaded>();

  // PRIVATE constructor which takes in the individual weather states
  WeatherState._(
    Union3<_WeatherInitial, _WeatherLoading, _WeatherLoaded> union,
  ) : super(union);

  // PUBLIC factories which hide the complexity from outside classes
  factory WeatherState.initial() =>
      WeatherState._(_factory.first(_WeatherInitial()));

  factory WeatherState.loading() =>
      WeatherState._(_factory.second(_WeatherLoading()));

  factory WeatherState.loaded(int temperature) =>
      WeatherState._(_factory.third(_WeatherLoaded(temperature)));
}

class _WeatherInitial {}

class _WeatherLoading {}

class _WeatherLoaded {
  final int temperature;

  _WeatherLoaded(this.temperature);
}
```

Vâng, trên đó là một chút boilerplate, tôi biết. Đó là phí chúng tôi phải trả cho func thiếu của Dart. Mặc dù các trạng thái riêng lẻ không còn exrends từ `WeatherState`, chúng vẫn có thể được thể hiện chính xác dưới dạng `WeatherState` vì sử dụng loại **Union** .

Thứ quan trọng nhất đối với chúng tôi là các factories **initial**, **loading** and **loaded** ở phía dưới. Chúng là cách duy nhất để khởi tạo các lớp trạng thái khác nhau , vì chúng không thể được khởi tạo trực tiếp từ bên ngoài vì chúng là private đối với các file khác.

*Bạn có thể làm cho unions lên đến 9 loại. Trong trường hợp như vậy, bạn sẽ extends **Union9Impl** class  và tạo **Nonet**  thay vì **Triplet***.

## Câu lệnh chuyển đổi cho Unions

Bây giờ đến phần của lý do mà chúng ta có mặt ở đây - một loại câu lệnh "if" hoặc "switch" có thể khiến bạn không thể kiểm tra hết các trường hợp. Với **Union**, điều đó có thể được thực hiện bằng cách **joining**. Với **Union3** mà chúng tôi đang sử dụng, chúng tôi phải cung cấp chính xác 3 "trường hợp" để trả lại:

```
main.dart
```
```
main(List<String> arguments) {
  // Instantiating the _WeatherLoades state with a factory
  final fakeWidget = widgetBuilder(WeatherState.loaded(42));
}

// Imagine this returns Flutter widgets
String widgetBuilder(WeatherState state) {
  return state.join(
    (initial) => "Some initial widget",
    (loading) => "Circular progress indicator",
    (loaded) => "The temperature is ${loaded.temperature}",
  );
}
```

Không có cách nào chúng tôi có thể xử lý ít hơn 3 trường hợp - code đơn giản là không biên dịch. Nếu chúng tôi thêm trạng thái `WeatherError` , chúng tôi sẽ chuyển sang `Union4`  (thay vì `Union3` ) và chúng tôi sẽ ngay lập tức gặp lỗi cho đến khi chúng tôi xử lý "trường hợp lỗi" trong phương thức **join**.

# Bạn đã học được gì

Chỉ vì Dart thiếu chức năng, so với các ngôn ngữ khác như Kotlin và Swift, không có nghĩa là chúng tôi phải giải quyết ít hơn. Với sự trợ giúp của  gói **seal_unions** , chúng tôi có thể có được chức năng tương tự như được cung cấp bởi  các lớp sealed của Kotlin hoặc enum của Swift. Seal_unions đòi hỏi một ít code boilerplate và tùy thuộc vào việc bạn chọn nó hoặc error code.
