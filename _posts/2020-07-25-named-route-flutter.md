---
layout: post
title: Named Route trong Flutter 
subtitle: Lại là mình - Quokka đây !
tags: [flutter, route, navigation_name]
---

Nguồn: 

[https://viblo.asia/p/flutter-navigator-and-routing-Do754beeZM6](https://viblo.asia/p/flutter-navigator-and-routing-Do754beeZM6)

[https://flutter.dev/docs/cookbook/navigation/named-routes](https://flutter.dev/docs/cookbook/navigation/named-routes)

# Vấn đề
Sau bài viết về Hook để quản lý State trong Widget, Quokka lại tiếp tục code một vài dự án nhỏ để luyện Flutter.

Qua một thời gian ở trong hang tu luyện kha khá, Quokka quyết định viết luôn một dự án lớn tầm cỡ hơn.

Mọi chuyện đều ổn cả cho đến khi Quokka nhận thấy mình có nhiều màn hình quản lý hơn dự kiến, và rất khó quản lý chúng. Quokka quyết định cắp sách đi học [Sư phụ](https://flutter.dev).

[Sư phụ](https://flutter.dev) mới đưa ra cho mình một trang bí kíp hầu như ai cũng biết chỉ có Quokka chưa biết đó là [Giữ phím Ctrl rồi Click chuột vào đây nè !](https://flutter.dev/docs/cookbook/navigation/named-routes)

Qua đó Quokka mạo muội viết lại Blog này cho chính Quokka cũng như cho những ai chưa biết như Quokka có thể đọc và dễ hiểu hơn.


# Tìm hiểu các khái niệm
1. **Route:** Route là một abstraction của một màn hình ("screen", "page") của ứng dụng. Navigator là một widget chịu trách nhiệm quản lý các route đó
2. **Navigator:** Nhiệm vụ của Navigator là tạo một Widget để lưu trữ, duy trì một stack-based lịch sử các child widget. Navigator có thể push hoặc pop một route để giúp người dùng duy chuyển giữa các màn hình khác nhau
3. **Material Page Route:** Là một modal route cung cấp cái hiệu ứng chuyển trang tương thích với từng nền tảng khác nhau (android & ios)

# Giới thiệu
Hẳn khi học Flutter cơ bản, mọi người đều biết về [Navigator](https://api.flutter.dev/flutter/widgets/Navigator-class.html) rồi. (*Tương lai Quokka sẽ có một bài viết riêng về Navigator*).

Quokka biết là khi điều hướng đến màn hình mới thì dùng `Navigator.push()` và đóng màn hình hiện tại thì `Navigator.pop()`.

Tuy nhiên, nhiều khi Quokka cần điều hướng đến cùng một màn hình trong nhiều phần của ứng dụng, hoặc là điều hướng đến nhiều màn hình trong nhiều trường hợp phải xét trong một Widget. Khi đó thì cách tiếp cận phía trên phải copy code lại nhiều lần.

#### Giải pháp: 
- Định nghĩa *Named Route* và sử dụng nó để điều hướng.

- Khi làm việc với các *Named Route*, sử dụng `Navigator.pushNamed()` để điều hướng màn hình mới và `Navigator.pop()` để đóng màn hình.

# Ví dụ
Okay ! Chuyển qua làm ví dụ cho dễ hiểu nè:
- Quokka sẽ tạo 2 màn hình đơn giản là **Page 1** và **Page 2**.
- **Page 1** sẽ có một *Button* mà ấn vào thì chuyển sang **Page 2**
- **Page 2** có một *Button* back lại **Page 1**

Quokka sẽ code demo sử dụng cách thường và cách dùng Named Routing rồi mình so sánh nhé !

#### Quokka tạo cấu trúc các file như hình:
![quokka](https://raw.githubusercontent.com/quokkacoder/quokkacoder.github.io/master/img/flutter/post_route/file.PNG)
#### Code trong các file như sau:

```
main.dart
```
```
void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Named Routing',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Page1(),
    );
  }
}

```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

```
../lib/page1.dart
```
```
class Page1 extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Page 1'),
      ),
      body: Center(
        child: RaisedButton(
            onPressed: ()=> _handlingPush(context),
            child: Text('Launch screen')
        ),
      ),
    );
  }

  _handlingPush(BuildContext context) {
  	// todo push screen here
  }
}

```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

```
../lib/page2.dart
```
```
import 'package:flutter/material.dart';

class Page2 extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Page 2'),
      ),
      body: Center(
        child: RaisedButton(
            onPressed: ()=> _handlingBack(context),
            child: Text('Go back')
        ),
      ),
    );
  }

  _handlingBack(BuildContext context) {
  	// todo go back here
  }
}
```
- Trong `Page1.dart` Quokka sẽ xử lý chuyển màn hình tại hàm `_handlingPush(context)`

- Tương tự trong `Page2.dart` Quokka sẽ xử lý đóng màn hình này tại hàm `_handlingBack(context)`

*Sau đây Quokka sẽ demo chuyển đổi giữa 2 màn hình bằng 2 cách: Simple Navigator và Named Route nhé !*

## 1. Sử dụng Simple Navigator
Như đã đề cập, đây là cách cơ bản nhất để điều hướng giữa các màn hình trong Flutter, đó là dùng method `Navigator.push` để đến màn hình mới và `Navigator.pop` để back lại màn hình trước đó

Method `Navigator.push` nhận vào 2 tham số (BuildContext, Route<T>) . Ở đây chúng ta sẽ sử dụng `MaterialPageRoute` để thay thế hiệu ứng chuyển cảnh với hiệu ứng của từng nền tảng (platform-adaptive transition)

*OKay ! Thực hiện thôi nào !*

- Trong `_handlingPush(context)` :
```
_handlingPush(BuildContext context) {
	Navigator.push(context, MaterialPageRoute(builder: (_) => Page2()));
}
```

- Trong `_handlingBack(context)` :
```
_handlingBack(BuildContext context) {
   Navigator.pop(context);
}
```

*Chạy ứng dụng lên thôi nào !*

## 2. Sử dụng Named Route
Một cách khác đó là sử dụng `Navigator.pushNamed` để navigate tới màn hình mới và `Navigator.pop` để back về màn hình trước .

`Navigator.pushNamed` nhận vào tối thiểu 2 tham số `(BuildContext, String, {Object})` và một tùy chọn `Argument`. String ở đây chính là tên chúng ta định nghĩa cho từng route.

Trong trường hợp này, chúng ta sẽ không sử dụng `MaterialPageRoute`, điều đó khiến cho hiệu ứng chuyển cảnh không được áp dụng với từng nền tảng. Do đó Quokka sẽ cấu hình `MaterialPageRoute` trong tham số có sẵn của `MaterialApp` đó là `onGenerateRoute`.

Quokka sẽ tạo ra một file mới là : `route.dart`. Chúng ta cấu hình các named router trong file này như sau:

```
../lib/route.dart
```
```
RouteFactory routers(){
  return (RouteSettings settings){
    final args = settings.arguments as Map<String, dynamic>;

    switch (settings.name) {
      case '/':
        return MaterialPageRoute(builder: (context)=> Page1());
        break;
      case '/page2':
        return MaterialPageRoute(builder: (context)=> Page2());
        break;
      default:
        return MaterialPageRoute(builder: (context) => NotFoundPage(settings.name));
    }
  };
}
```

**Phân tích:**
- Khi chúng ta truyền giá trị named (là một chuỗi) vào trong `Navigator.pushNamed(context, named, {args})`, route sẽ lấy được giá trị này gán vào trong RouteSettings
- Trong hàm routes đã tạo, Quokka đã sử dụng RouteSettings để lấy named mà Navigator đã truyền vào để *"kích hoạt"*  `MaterialPageRoute` chuyển tới màn hình tương ứng
- Giá trị `args` là tham số cần truyền giữa các màn hình được định dạng theo kiểu `Map<String, dynamic>`
- Ở trên tại sao named router của Page1, Quokka lại xét chuỗi `'/'`? Quokka sẽ giải thích phía dưới. Nhưng hãy cứ hiểu màn hình nào xuất hiện đầu thì xét và gán `'/'` cho named router ở màn hình đó. 
- Trường hợp truyền sai giá trị named, Quokka sẽ return lại màn hình `NotFoundPage` để cảnh báo:

```
../lib/not_found_page.dart
```
```
class NotFoundPage extends StatelessWidget {
  final String _named;

  const NotFoundPage(this._named);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Not found page'),
      ),
      body: Center(
        child: Text('Sorry not found page: $_named'),
      ),
    );
  }
}

```

Rồi okey. Như đã nói ở trên chúng ta sẽ truyền routers đã cấu hình vào `onGenerateRoute` trong `MaterialApp`:

```
../lib/main.dart
```
```
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Named Routes',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      initialRoute: '/', // add init route here
      onGenerateRoute: routers(), // add named routers here
    );
  }
}
```
Trong `MaterialApp` có tham số `initialRoute`. Khi khởi chạy, ứng dụng sẽ trỏ vào route này đầu tiên.
Mặc định của giá trị này ta truyền vào là `'/'`, tương ứng với named router của Page1 mà Quokka đã cấu hình ở hàm `routes()`.


Khi dùng `initialRoute` thì tham số `home` sẽ không cần thiết nữa. Quokka đã xóa nó !

Về cơ bản, Quokka đã định hướng được các *tuyến đường* cho các màn hình. Bây giờ Quokka sẽ thay thế code xử lý chuyển hướng trong button ở các màn hình:

- Hàm `_handlingPush(context)` ở `..lib/page1.dart`:
```
  _handlingPush(BuildContext context) {
    Navigator.pushNamed(context, '/page2');
  }
```

- Hàm `_handlingBack(context)` ở `../lib/page2.dart` vẫn giữ nguyên.

*OK. Chạy lại ứng dụng thôi !*

## 3. Một vài lưu ý khi sử dụng Named Routing
- Để dễ quản lý Named Routers, Quokka khuyên bạn nên gán mặc định chuỗi named trong mỗi màn hình. Hoặc liệt kê các chuỗi named của các màn hình vào cùng 1 file cho dễ quản lý.
- `MaterialPageRoute` có thể được thay thế bằng nhiều loại Animation chuyển màn khác mà bạn tự custom hoặc import ở đâu đó.
- Về việc truyền giá trị giữa 2 màn hình, các bạn sử dụng `Argument` theo dạng Map<String, dynamic> để truyền  và lấy giá trị.

*Ví dụ:*

- Truyền giá trị:
```
 Navigator.pushNamed(context, '/page2', arguments: {'param1': 'from page 1'});
```
- Lấy giá trị trong routes() và truyền vào màn hình mới:
```
case '/page2':
        return MaterialPageRoute(builder: (context)=> Page2(args['param1']));
        break;
```


# Kết luận:
- Quokka thấy làm việc với Named Route là cần thiết, và dễ dàng quản lý cấu trúc của dự án, nhất là các dự án lớn bao gồm nhiều màn hình.
- Cấu hình Named Route cho phép bạn quản lý Dependences Injection dễ dàng hơn (có thời gian Quokka sẽ demo cho mọi người)


Cảm ơn mọi người đã theo dõi <3