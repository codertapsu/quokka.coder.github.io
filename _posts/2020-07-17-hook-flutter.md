---
layout: post
title: Ví dụ Hook Flutter
#image: /img/flutter/hook.png
---

# Vấn đề
Khi xây dựng ứng dụng với Flutter, Quokka thường bị hoa mắt thường xuyên 😖😖😖

Một phần vì cái trình bày rất dị của Dart. Một phần nữa là do cách quản lý code trong vòng đời của một Widget khá lộn xộn:
* Mã nằm trong các Widget rất khó để sử dụng lại
* Quokka thường hoàn thành một Widget với việc TUNG CODE MÙ các phương thức vòng đời trong Widget đó (Ví dụ như `initState()`, `disponse()`).

Quokka muốn tách mấy thứ khó chịu đó ra một khối riêng. Quokka tìm được một thứ khá hay đó là [Hook Flutter](https://pub.dev/packages/flutter_hooks).

Hook Flutter là một trong những cách để tách logic UI của bạn thành các "hook" độc lập và có thể kết hợp.
Nó là một bài thuốc có thể giúp Quokka đỡ hoa mắt hơn không? 🤔
**Quokka sẽ tiến hành demo thử.**
# Ví dụ
Quokka sẽ tạo một màn hình gồm: một ListView và một FloattingActionButton.
Khi trượt ListView đi xuống sẽ ẩn FAB, khi trượt lên thì hiển thị FAB.
### Mã khi không dùng Hook Flutter
`home_page.dart`

```dart
class HomePage extends StatefulWidget {
  const HomePage({
    Key key,
  }) : super(key: key);

  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage>
    with SingleTickerProviderStateMixin {
  ScrollController _scrollController;
  AnimationController _hideFabAnimController;

  @override
  void initState() {
  super.initState();
  _scrollController = ScrollController();
  _hideFabAnimController = AnimationController(
    vsync: this,
    duration: kThemeAnimationDuration,
    value: 1, // initially visible
  );
  _scrollController.addListener(() {
    switch (_scrollController.position.userScrollDirection) {
      // Scrolling up - forward the animation (value goes to 1)
      case ScrollDirection.forward:
        _hideFabAnimController.forward();
        break;
      // Scrolling down - reverse the animation (value goes to 0)
      case ScrollDirection.reverse:
        _hideFabAnimController.reverse();
        break;
      // Idle - keep FAB visibility unchanged
      case ScrollDirection.idle:
        break;
    }
  });
  	
  @override
  void dispose() { 
    _scrollController.dispose();
    _hideFabAnimController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
  ...
  }
``` 
Quokka chỉ demo phần thiết lập hiệu ứng còn `build` Quokka không code nữa nhé (Hoa mắt quá 😤)

#### Nhận xét:
- Code trong `init()` và `dispose()` còn khá nhiều và rắc rối.
- Nếu tách được phần thiết lập trên thành một khối riêng thì dễ quản lý hơn.

**Sau đây Quokka sẽ sử dụng Hook xem sao nhé ! 🤓**

### Mã khi dùng Hook Flutter

Hook hoạt động rất giống State Objects của StatefulWidgets. 

Nó chỉ khác ở chỗ: *Trong khi bạn không thể có nhiều States được liên kết với một StatefulWidget, bạn có thể có nhiều HookStates liên kết với một HookWidget.*

Nghe có vẻ khó hiểu? Bạn sẽ hiểu sau khi đọc mã thôi mà.

Có một loạt các Hook được định nghĩa trước mà chúng ta có thể sử dụng. Một trong số đó là `AnimationController`.

Quokka sẽ tận dụng Hook có sẵn của `AnimationController` để tạo một Hook cho `ScrollController`.

#### Import Hook


```
pubspec.yaml
```
```
dependencies:
  flutter:
    sdk: flutter
  flutter_hooks: ^0.7.0
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
#### Từ `Stateful` sang `HookWidget`
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Trong `home_page.dart` Quokka sẽ chuyển từ `StatefulWidget` sang `HookWidget`(`HookWidget` nó có cấu trúc tựa như `StatelessWidget`)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Từ đây chúng ta không cần `SingleTickerProviderStateMixin` và các mã trong `initState`, `dispose` : 


```
/home_page.dart
```
```
class HomePage extends HookWidget {

  // ERRORS HERE

  @override
  Widget build(BuildContext context) {
    final hideFabAnimController = useAnimationController(
        duration: kThemeAnimationDuration, initialValue: 1); // FAB ANIM CTRL HERE

    return Scaffold(
      appBar: AppBar(
        title: Text("Let's Scroll"),
      ),
      floatingActionButton: FadeTransition(
        opacity: hideFabAnimController,
        child: ScaleTransition(
          scale: hideFabAnimController,
          child: FloatingActionButton.extended(
            label: const Text('Useless Floating Action Button'),
            onPressed: () {},
          ),
        ),
      ),
      floatingActionButtonLocation: FloatingActionButtonLocation.centerFloat,
      body: ListView(
        controller: _scrollController,
        children: <Widget>[
          for (int i = 0; i < 5; i++)
            Card(child: FittedBox(child: FlutterLogo())),
        ],
      ),
    );
  }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Tạm thời bỏ qua lỗi không thể tránh khỏi từ việc cấu trúc chưa hoàn thành, giờ đây Quokka có được một `AnimationController` từ `build` mà không cần tất cả các thủ tục rườm rà như lúc nãy. (Phía trên Quokka đã thêm `return` Widget phần `build` vào rồi nhé )
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
#### Tạo một Custom Hook
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Custom ScrollController Hook sẽ cần phải khởi tạo một `ScrollController`, thêm một `listener` để nó cập nhật `AnimationController` (`AnimationController`được truyền vào như một tham số).
 Sau đó `return` lại `ScrollController` để Quokka có thể sử dụng nó từ UI. 

 (Nó thực tế sẽ gom tất cả các mã xử lý trong `State Object` của `StatefulWidget`)
```
hook / scroll_controll_for_animation.dart
```
```
ScrollController useScrollControllerForAnimation(
  AnimationController animationController,
) {
  final ScrollController scrollController = ScrollController();
  scrollController.addListener(() {
    switch (scrollController.position.userScrollDirection) {
      // Scrolling up - forward the animation (value goes to 1)
      case ScrollDirection.forward:
        animationController.forward();
        break;
      // Scrolling down - reverse the animation (value goes to 0)
      case ScrollDirection.reverse:
        animationController.reverse();
        break;
      case ScrollDirection.idle:
        break;
    }
  });
  return scrollController;
}
```
Tuyệt ! Quokka đã chuyển tất cả code logic rườm rà ở phần trước (trong `initState`) sang Function mới này. 
Bây giờ chúng ta có thể có được Hook này từ `build`:

```
home_page.dart
```
```
class HomePage extends HookWidget {
  @override
  Widget build(BuildContext context) {
    final hideFabAnimController = useAnimationController(
        duration: kThemeAnimationDuration, initialValue: 1);
    final scrollController =
        useScrollControllerForAnimation(hideFabAnimController);

    return Scaffold(
      appBar: AppBar(
        title: Text("Let's Scroll"),
      ),
      floatingActionButton: FadeTransition(
        opacity: hideFabAnimController,
        child: ScaleTransition(
          scale: hideFabAnimController,
          child: FloatingActionButton.extended(
            label: const Text('Useless Floating Action Button'),
            onPressed: () {},
          ),
        ),
      ),
      floatingActionButtonLocation: FloatingActionButtonLocation.centerFloat,
      body: ListView(
        controller: scrollController,
        children: <Widget>[
          for (int i = 0; i < 5; i++)
            Card(child: FittedBox(child: FlutterLogo())),
        ],
      ),
    );
  }
}
```

Chạy ứng dụng và Quokka đã thành công ! Yay ! Nhưng khoan ! 

Nếu bạn nhìn kỹ vào Hook mà Quokka đã tạo ra, có một điều thiếu: Quokka đã không gọi `scrollController.dispose()` 😱

#### Hook Class
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Khi Quokka mới bắt đầu với Flutter, Quokka được khuyên một điều: *whenever you instantiate something which needs to know about the lifecycle !*

**Nghĩa là khi khởi tạo thứ gì đó thì nên cẩn trọng xem xét vòng đời của nó trong `Widget`.**

Tất nhiên, các chức năng của Hook vẫn rất tuyệt cho những lần bạn không cần vòng đời của Widget (Đọc thêm về nó trong [tài liệu chính thức](https://github.com/rrousselGit/flutter_hooks#how-to-use)).

Bây giờ, hãy tạo một package-private `_ScrollControllerForAnimationHook` cùng với State của nó `_ScrollControllerForAnimationHookState.` 

`Hooks` thực sự giống với `StatefulWidgets`. Vì vậy nếu các đồng râm biết những điều cơ bản của Flutter thì mấy dòng code sau đây là ez game: 

```
hooks/scroll_controller_for_animation.dart
```
```
class _ScrollControllerForAnimationHook extends Hook<ScrollController> {
  final AnimationController animationController;

  const _ScrollControllerForAnimationHook({
    @required this.animationController,
  });

  @override
  _ScrollControllerForAnimationHookState createState() =>
      _ScrollControllerForAnimationHookState();
}

class _ScrollControllerForAnimationHookState
    extends HookState<ScrollController, _ScrollControllerForAnimationHook> {
  ScrollController _scrollController;

  @override
  void initHook() {
    _scrollController = ScrollController();
    _scrollController.addListener(() {
      switch (_scrollController.position.userScrollDirection) {
        case ScrollDirection.forward:
          // State has the "widget" property
          // HookState has the "hook" property
          hook.animationController.forward();
          break;
        case ScrollDirection.reverse:
          hook.animationController.reverse();
          break;
        case ScrollDirection.idle:
          break;
      }
    });
  }

  // Build doesn't return a Widget but rather the ScrollController
  @override
  ScrollController build(BuildContext context) => _scrollController;

  // This is what we came here for
  @override
  void dispose() => _scrollController.dispose();
}
```

Các Hook được cấu trúc theo nghĩa đen giống như một `StatefulWidget` và quan trọng nhất là Quokka đã có `dispose` !

#### Lưu ý: ####
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Bạn có thể tranh luận rằng Quokka vừa tạo ra rất nhiều boilerplate và diều đó **ĐÚNG** ! `Hook` không phải là một viên đạn bạc và khi sử dụng nó bạn phải cân nhắc. 

*Ví dụ: Chỉ sử dụng một hook trong một widget không phải là một gói thầu tốt về thời gian . Nếu bạn có một Hook mà bạn định sử dụng nó 10 lần, thì hãy tạo nó!*

---- Hết lưu ý 😂 ---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vậy, tại sao lại là `class package-private`? Quokka muốn giấu các mã xử lý chức năng trong Hook. Khi liên kết Hook đó với `HookWidget`, Quokka chỉ cần gọi `Hook.use.` :
```
hooks/scroll_controller_for_animation.dart
```
```
ScrollController useScrollControllerForAnimation(
  AnimationController animationController,
) {
  return Hook.use(_ScrollControllerForAnimationHook(
    animationController: animationController,
  ));
}
```
Chạy lại ứng dụng và trải nghiệm lại code nào ! Yay ! Yay !


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![quokka](https://media.giphy.com/media/246TAiZvGoewMQkcnj/giphy.gif)

# Kết luận

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Đối với Quokka, [Hook](https://pub.dev/packages/flutter_hooks) là một cách tuyệt vời để quản lý sự phức tạp trong mã UI. Chúng ta có thể chọn từ một loạt các Hook được xác định trước hoặc tạo riêng như Quokka đã làm trong bài viết này. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Như với bất kỳ công cụ nào, Hook có thể được sử dụng cho cả mục đích tốt và xấu. Vì vậy hãy chọn thời điểm sử dụng chúng một cách khôn ngoan! Sử dụng những cái được xác định trước luôn tốt, nhưng hãy suy nghĩ kỹ trước khi đầu tư thời gian vào việc tạo ra cái của riêng bạn !

Nguồn: [https://resocoder.com/2020/01/21/flutter-hooks-hide-fab-animation-100-widget-code-reuse/](https://resocoder.com/2020/01/21/flutter-hooks-hide-fab-animation-100-widget-code-reuse/)
