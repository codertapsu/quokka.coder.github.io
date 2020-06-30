---
layout: post
title: Flutter TDD Clean Architecture Course [7] - Network Info
image: https://2.bp.blogspot.com/-a9nR2A_99yg/VMXJ2cBXE2I/AAAAAAAAB_s/O_gXstMAZ9I/s1600/tdd.png
---

*Bây giờ chúng tôi đã triển khai **Kho lưu trữ**, chúng tôi sẽ triển khai các phụ thuộc của nó, bắt đầu với  lớp **NetworkInfo** được sử dụng để tìm hiểu xem thiết bị hiện có được kết nối với mạng hay không. Phần này là nơi cuối cùng chúng tôi sẽ thực hiện phát triển dựa trên thử nghiệm với các gói của bên thứ 3, có nghĩa là chúng tôi sẽ mock các lớp bên thứ 3.*

Như mọi khi, chúng tôi đã biết inteface của **NetworkInfo** trông như thế nào, vì trước tiên chúng tôi đã xác định hợp đồng của nó . Nó có một thuộc tính duy nhất được gọi là `isConnected` .

```
network_info.dart
```
```
abstract class NetworkInfo {
  Future<bool> get isConnected;
}
```

# Chuyển đổi Packages

Trong phần thứ hai tôi đã nói với bạn rằng chúng ta sẽ sử dụng gói **connectivity** để tìm hiểu về trạng thái mạng. Mặc dù gói đó hữu ích để xác định xem thiết bị đang chạy trên dữ liệu di động  hay trên  WiFi , tôi phát hiện ra rằng nó không tuyệt vời để kiểm tra truy cập Internet thực tế. Vì nó viết ngay trên trang gói:

*"Lưu ý rằng trên Android, điều này không đảm bảo kết nối với Internet. Chẳng hạn, ứng dụng có thể có quyền truy cập WiFi nhưng có thể là VPN hoặc WiFi khách sạn không có truy cập Internet."*

Vì chúng tôi không thể chỉ dựa vào thông tin mà nền tảng (Android / iOS) cung cấp, thay vào đó chúng tôi có thể dựa vào điều gì? Trên thực tế kết nối với một cái gì đó, tất nhiên! [data_connection_checker](https://pub.dev/packages/data_connection_checker) package chỉ là hoàn hảo cho điều đó.

```
pubspec.yaml
```
```
dependencies:
  # Swap the connectivity package for this
  data_connection_checker: ^0.3.4
```

Nó mở một socket đến một số địa chỉ nhất định và xác định trạng thái kết nối thực dựa trên việc nó có thực sự có thể kết nối hay không. Ngoài ra, nó hoàn toàn độc lập với nền tảng - nó cũng có thể hoạt động trên web!

*Các địa chỉ được sử dụng trỏ đến các máy chủ DNS của CloudFlare , Google và OpenDNS . Chúng ta chỉ cần nói rằng ba dịch vụ này kết hợp có thời gian hoạt động 100%, vì vậy không phải lo lắng về việc liệu trạng thái trực tuyến của thiết bị sẽ được xác định một cách chính xác.*

Bây giờ chúng tôi biết rằng chúng tôi sẽ không sử dụng  gói **connectivity** vì nhận thông tin kết nối từ nền tảng là không đáng tin cậy, tối sẽ đổi tên folder **core/platform** tảng thư mục  thành **core/network** . Chúng tôi cũng cần sửa lỗi import trong quá trình  thực hiện và kiểm tra **Kho lưu trữ** .

# Implementation

Trước tiên chúng ta hãy tạo một tệp thử nghiệm tại một vị trí được ánh xạ như bình thường. Mặc dù sẽ không có nhiều logic để thực hiện, nhưng điều quan trọng là không từ bỏ TDD ngay cả trong những trường hợp như vậy. Lỗi có thể được ẩn ngay cả trong một mã dường như vô hại.

![](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/09/network_info_test-location.png?w=306&ssl=1)

Lớp **NetworkInfo**  sẽ đưa một instance **DataConnectionChecker** vào constructor của nó và tất nhiên, chúng ta sẽ tạo ra một bản mock. Thử nghiệm thực tế của thuộc tính `isConnected` sẽ khác một chút so với các thử nghiệm chúng tôi đã viết cho đến bây giờ ...

```
network_info_test.dart
```
```
import 'package:clean_architecture_tdd_prep/core/network/network_info.dart';
import 'package:data_connection_checker/data_connection_checker.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

class MockDataConnectionChecker extends Mock implements DataConnectionChecker {}

void main() {
  NetworkInfoImpl networkInfo;
  MockDataConnectionChecker mockDataConnectionChecker;

  setUp(() {
    mockDataConnectionChecker = MockDataConnectionChecker();
    networkInfo = NetworkInfoImpl(mockDataConnectionChecker);
  });

  group('isConnected', () {
    test(
      'should forward the call to DataConnectionChecker.hasConnection',
      () async {
        // arrange
        final tHasConnectionFuture = Future.value(true);

        when(mockDataConnectionChecker.hasConnection)
            .thenAnswer((_) => tHasConnectionFuture);
        // act
        // NOTICE: We're NOT awaiting the result
        final result = networkInfo.isConnected;
        // assert
        verify(mockDataConnectionChecker.hasConnection);
        // Utilizing Dart's default referential equality.
        // Only references to the same object are equal.
        expect(result, tHasConnectionFuture);
      },
    );
  });
}
```

Gọi `NetworkInfo().isConnected` thực sự chỉ là một nickname để gọi `DataConnectionChecker().hasConnection`. Chúng tôi chỉ đơn giản là ẩn thư viện bên thứ 3 đằng sau một interface class của chúng tôi. Chúng ta có thể kiểm tra xem việc call đến thuộc tính có được "chuyển tiếp" hay không bằng cách kiểm tra xem đối tượng **Future** được return bởi `isConnected` có giống hệt như giá trị được return bởi `hasConnection` hay không .

*Có vẻ như việc "chuyển tiếp" như vậy chỉ là một sự lãng phí công sức. Hoàn toàn ngược lại!
Hãy tưởng tượng bạn muốn trao đổi gói **data_connection_checker** cho một cái gì đó khác. Nếu bạn sử dụng nó trực tiếp bên trong **Kho lưu trữ** bạn cần thay đổi RẤT NHIỀU code kiểm tra kết nối.
Bằng cách ẩn nó đằng sau một interface mà bạn kiểm soát, sẽ không có nhiều code để thay đổi!*

Việc thực hiện rất đơn giản. Chúng tôi sẽ không tạo một tệp riêng cho nó, nhưng chúng tôi sẽ đặt nó ngay bên dưới định nghĩa lớp abstract.

```
network_info.dart
```
```
import 'package:data_connection_checker/data_connection_checker.dart';

abstract class NetworkInfo {
  Future<bool> get isConnected;
}

class NetworkInfoImpl implements NetworkInfo {
  final DataConnectionChecker connectionChecker;

  NetworkInfoImpl(this.connectionChecker);

  @override
  Future<bool> get isConnected => connectionChecker.hasConnection;
}
```

# Tiếp theo

Mặc dù phần này có thể không dài nhất, nhưng có rất nhiều thứ ở đây. Chúng tôi đã triển khai lớp **NetworkInfo**  với cách tiếp cận thử nghiệm thú vị và bạn đã học được tại sao việc tạo ra các lớp dường như "vô dụng" chỉ để ẩn code bên thứ 3 dưới sự ổn định của interface.

Vẫn có **Data Sources**  để thực hiện. Trong phần tiếp theo, chúng ta sẽ làm việc với **local Data Source**, có nghĩa là thực hiện TDD với gói **shared_preferences** .
