---
layout: post
title: Flutter TDD Clean Architecture Course [14] - User Interface
image: https://2.bp.blogspot.com/-a9nR2A_99yg/VMXJ2cBXE2I/AAAAAAAAB_s/O_gXstMAZ9I/s1600/tdd.png
---

Cuối cùng chúng tôi đã ở đây! Phần này sẽ là việc xây dựng giao diện người dùng và chia nó thành nhiều phần có thể đọc và bảo trì Widget.

# Tạo một Page

Feature **number trivia** sẽ có một trang duy nhất, nó là một `StatelessWidget` được đặt tên là `NumberTriviaPage`. Bây giờ chúng ta hãy tạo một phiên bản trống của nó:

```
.../presentation/pages/number_trivia_page.dart
```
```
class NumberTriviaPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Number Trivia'),
      ),
    );
  }
}
```
Chúng tôi sẽ xóa mọi widget trong file `main.dart` và tạo ra một MaterialApp nguyên sơ, với theme màu xanh lá cây cơ bản (Reso Coder style). Tất nhiên, home widget sẽ là `NumberTriviaPage`.

```
main.dart
```
```
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Number Trivia',
      theme: ThemeData(
        primaryColor: Colors.green.shade800,
        accentColor: Colors.green.shade600,
      ),
      home: NumberTriviaPage(),
    );
  }
}
```

# Getting the Presentation Logic Holder

Trong Clean Architecture, con đường giao tiếp duy nhất giữa các UI widget và phần còn lại của ứng dụng là **presentation logic holder**. Ứng  dụng Number Trivia sử dụng Bloc, nhưng như tôi đang nói có lẽ là lần thứ 100, bạn có thể sử dụng mọi thứ từ Change Notifier đến MobX...

Bất kể phương pháp quản lý State ưa thích của bạn là gì, bạn vẫn cần cung cấp presentation logic holder của mình trong toàn bộ tree widget. Đối với điều đó, rõ ràng bạn có thể sử dụng  gói [provider](https://pub.dev/packages/provider) ! Vì chúng tôi đang sử dụng Bloc được tích hợp với Provider , nên chúng tôi sẽ sử dụng sản phẩm lai đặc biệt `BlocProvider` có một số tính năng dành riêng cho Bloc.

`NumberTriviaBloc` đã được cung cấp cho toàn bộ body các trang của `Scaffold`. Đây là nơi chúng ta sẽ nhận được các **instance** đã đăng ký từ **service locator**, nó sẽ lần lượt khởi động tất cả các **lazy singletons** đã đăng ký trong phần trước.

```
number_trivia_page.dart
```
```
...
import '../../../../injection_container.dart';
import '../bloc/bloc.dart';

class NumberTriviaPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Number Trivia'),
      ),
      body: BlocProvider(
        builder: (_) => sl<NumberTriviaBloc>(),
        child: Container(),
      ),
    );
  }
}
```  

![](https://i1.wp.com/resocoder.com/wp-content/uploads/2019/09/app-ui.png?resize=576%2C1024&ssl=1)

# Xây dựng nội dung

Trước khi thực hiện với các `State` được phát ra bởi `NumberTriviaBloc`, trước tiên chúng ta hãy xây dựng một bản phác thảo UI cơ bản bằng `Placeholder` widget.

Thư mục gốc của giao diện người dùng sẽ là **phần đệm** và **trung tâm** `Column` bao gồm hai phần cơ bản:

- Nửa trên sẽ là đầu ra . Sẽ có một message chứa câu chuyện, một số lỗi hoặc thậm chí là trạng thái ban đầu "Bắt đầu tìm kiếm!". `CircularLoadingIndicator` cũng sẽ được hiển thị trong phần trên cùng. Phần này của UI sẽ được xây dựng lại bất cứ khi nào State thay đổi.
- Phần dưới cùng sẽ là đầu vào . Nó sẽ có một `TextField` và hai `RaisedButtons`.

## Phác thảo với Placeholders

Thiết kế bố cục UI với `Placeholders` là một cách hoàn hảo để đặt một cách thích hợp khoảng cách và kích thước của các widget mà không phải suy nghĩ về việc triển khai chúng. Tại thời điểm này, nó cũng sẽ là tốt nhất để trích xuất body của `Scaffold` thành helper của nó (`buildBody` method).

![](https://i1.wp.com/resocoder.com/wp-content/uploads/2019/09/ui-with-placeholders.png?resize=576%2C1024&ssl=1)

```
number_trivia_page.dart
```
```
class NumberTriviaPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Number Trivia'),
      ),
      body: buildBody(context),
    );
  }

  BlocProvider<NumberTriviaBloc> buildBody(BuildContext context) {
    return BlocProvider(
      builder: (_) => sl<NumberTriviaBloc>(),
      child: Center(
        child: Padding(
          padding: const EdgeInsets.all(10),
          child: Column(
            children: <Widget>[
              SizedBox(height: 10),
              // Top half
              Container(
                // Third of the size of the screen
                height: MediaQuery.of(context).size.height / 3,
                // Message Text widgets / CircularLoadingIndicator
                child: Placeholder(),
              ),
              SizedBox(height: 20),
              // Bottom half
              Column(
                children: <Widget>[
                  // TextField
                  Placeholder(fallbackHeight: 40),
                  SizedBox(height: 10),
                  Row(
                    children: <Widget>[
                      Expanded(
                        // Search concrete button
                        child: Placeholder(fallbackHeight: 30),
                      ),
                      SizedBox(width: 10),
                      Expanded(
                        // Random button
                        child: Placeholder(fallbackHeight: 30),
                      )
                    ],
                  )
                ],
              )
            ],
          ),
        ),
      ),
    );
  }
}
```

# Nửa trên - Hiển thị dữ liệu

Nửa trên của màn hình phải hiển thị các widget khác nhau tùy thuộc vào đầu ra `State` từ `NumberTriviaBloc`. `Loading` tất nhiên sẽ hiển thị một **progress indicator**, `Error` tất nhiên sẽ hiển thị một thông báo lỗi, v.v. Xây dựng các widget khác nhau theo trạng thái hiện tại của `Bloc` là có thể với `BlocBuilder`.

`initialState` của `NumberTriviaBloc` là `Empty`, vì vậy trước tiên hãy xử lý cái đó bằng cách return một `Text` đơn giản bên trong `Container` để đảm bảo nó chiếm 1/3 chiều cao màn hình.

```
number_trivia_page.dart
```
```
...
// Top half
BlocBuilder<NumberTriviaBloc, NumberTriviaState>(
  builder: (context, state) {
    if (state is Empty) {
      return Container(
        // Third of the size of the screen
        height: MediaQuery.of(context).size.height / 3,
        child: Center(
          child: Text('Start searching!'),
        ),
      );
    }
    // We're going to also check for the other states
  },
),
...
```

## Hiển thị Message

Ngoài các text nhỏ, `Empty` state được xử lý. Tuy nhiên, trước tiên hãy dọn dẹp mớ hỗn độn mà chúng ta đã tạo bằng cách trích xuất `Container` vào `MessageDisplay` widget của chính nó . Chúng tôi sẽ sử dụng widget này để hiển thị các thông báo lỗi khi `Error` state được phát ra, vì vậy chúng tôi phải đảm bảo `String message` có thể truyền qua constructor.

Ngoài ra, chúng tôi sẽ thêm một chút style để làm cho văn bản lớn hơn và cũng có thể cuộn bằng cách sử dụng `SingleChildScrollView`. Nếu không, tin nhắn dài sẽ bị cắt vì `Container` có chiều cao hạn chế.

```
number_trivia_page.dart
...
BlocBuilder<NumberTriviaBloc, NumberTriviaState>(
  builder: (context, state) {
    if (state is Empty) {
      return MessageDisplay(
        message: 'Start searching!',
      );
    }
  },
),

...

class MessageDisplay extends StatelessWidget {
  final String message;

  const MessageDisplay({
    Key key,
    @required this.message,
  })  : assert(message != null),
        super(key: key);

  @override
  Widget build(BuildContext context) {
    return Container(
      // Third of the size of the screen
      height: MediaQuery.of(context).size.height / 3,
      child: Center(
        child: SingleChildScrollView(
          child: Text(
            message,
            style: TextStyle(fontSize: 25),
            textAlign: TextAlign.center,
          ),
        ),
      ),
    );
  }
}
```

![](https://i1.wp.com/resocoder.com/wp-content/uploads/2019/09/message-display-widget.png?resize=576%2C1024&ssl=1)

Sử dụng lại `MessageDisplay` cho các messages từ `Error` state là tốt. Chúng tôi sẽ mở rộng `BlocBuilder` với câu lệnh `else if` bổ sung :

```
number_trivia_page.dart
```
```
BlocBuilder<NumberTriviaBloc, NumberTriviaState>(
  builder: (context, state) {
    if (state is Empty) {
      return MessageDisplay(
        message: 'Start searching!',
      );
    } else if (state is Error) {
      return MessageDisplay(
        message: state.message,
      );
    }
  },
),
```

Chúng tôi sẽ không sử dụng `MessageDisplay` cho `Loaded` state để hiển thị NumberTrivia, bởi vì chúng ta muốn có nhiều hiển thị độc đáo trên đỉnh.

## LoadingWidget

Lấy dữ liệu từ API remote mất một thời gian . Đó là lý do tại sao Bloc phát ra một `Loading` state và trách nhiệm của UI là hiển thị một loading indicator. Chúng ta đã biết rằng việc đưa các widget trực tiếp vào` BlocBuilder` sẽ khó đọc, vì vậy chúng ta sẽ trích xuất `Container` thành một `LoadingWidget`.

```
number_trivia_page.dart
```
```
...
BlocBuilder<NumberTriviaBloc, NumberTriviaState>(
  builder: (context, state) {
    if (state is Empty) {
      return MessageDisplay(
        message: 'Start searching!',
      );
    } else if (state is Loading) {
      return LoadingWidget();
    } else if (state is Error) {
      return MessageDisplay(
        message: state.message,
      );
    }
  },
),

...

class LoadingWidget extends StatelessWidget {
  const LoadingWidget({
    Key key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Container(
      height: MediaQuery.of(context).size.height / 3,
      child: Center(
        child: CircularProgressIndicator(),
      ),
    );
  }
}
```

Mỗi widget tùy chỉnh ở nửa trên của màn hình sẽ chiếm chính xác 1/3 chiều cao của toàn bộ màn hình bằng cách sử dụng `MediaQuery`. Fixating chiều cao rất hữu ích để ngăn chặn nửa dưới từ "nhảy xung quanh" khi chiều dài của message/trivia thay đổi.

## Hiển thị Trivia

Chúng tôi đang thiếu một "phản ứng state" cho state quan trọng nhất của tất cả chúng - `Loaded` state chứa `NumberTrivia` entity mà người dùng quan tâm.

`TriviaDisplay` widget sẽ cực kỳ giống với `MessageDisplay`, nhưng tất nhiên, nó sẽ đưa vào một` NumberTrivia` object thông qua constructor. Ngoài ra, `TriviaDisplay` sẽ được làm bằng một `Column` hiển thị hai `Text` widget. Một cái sẽ hiển thị **số** trong một font chữ lớn, cái còn lại sẽ hiển thị câu chuyện thực tế.

```
number_trivia_page.dart
```
```
...
BlocBuilder<NumberTriviaBloc, NumberTriviaState>(
  builder: (context, state) {
    if (state is Empty) {
      return MessageDisplay(
        message: 'Start searching!',
      );
    } else if (state is Loading) {
      return LoadingWidget();
    } else if (state is Loaded) {
      return TriviaDisplay(
        numberTrivia: state.trivia,
      );
    } else if (state is Error) {
      return MessageDisplay(
        message: state.message,
      );
    }
  },
),

...

class TriviaDisplay extends StatelessWidget {
  final NumberTrivia numberTrivia;

  const TriviaDisplay({
    Key key,
    this.numberTrivia,
  })  : assert(numberTrivia != null),
        super(key: key);

  @override
  Widget build(BuildContext context) {
    return Container(
      height: MediaQuery.of(context).size.height / 3,
      child: Column(
        children: <Widget>[
          // Fixed size, doesn't scroll
          Text(
            numberTrivia.number.toString(),
            style: TextStyle(
              fontSize: 50,
              fontWeight: FontWeight.bold,
            ),
          ),
          // Expanded makes it fill in all the remaining space
          Expanded(
            child: Center(
              // Only the trivia "message" part will be scrollable
              child: SingleChildScrollView(
                child: Text(
                  numberTrivia.text,
                  style: TextStyle(fontSize: 25),
                  textAlign: TextAlign.center,
                ),
              ),
            ),
          )
        ],
      ),
    );
  }
}
```

## Tái cấu trúc nhỏ

Chúng tôi đã làm rất nhiều để giữ cho code có thể duy trì bằng cách tạo các custom widget. Hiện nay, mặc dù chung nằm tất cả bên trong file `number_trivia_page.dart`, vì vậy chúng ta hãy di chuyển chúng thành các file riêng của mình nằm dưới thư mục `widgets`.

![Number Trivia feature - presentation folder](https://i2.wp.com/resocoder.com/wp-content/uploads/2019/09/widgets-folder.png?w=280&ssl=1)

Vì Dart không hỗ trợ "package imports" như Kotlin hay Java, nên chúng tôi phải tự giúp mình thoát khỏi rất nhiều import riêng lẻ với *barrel file*. Nó chỉ đơn giản là `export` tất cả các file khác có trong folder:

```
widgets.dart
```
```
export 'loading_widget.dart';
export 'message_display.dart';
export 'trivia_display.dart';
```

Và sau đó, bên trong `number_trivia_page.dart`  nó đủ để import chỉ barrel file:

`import '../widgets/widgets.dart';`

# Nửa dưới - Nhận đầu vào

Tất cả các Widget chúng tôi đã thực hiện đến thời điểm này sẽ không hiển thị, trừ khi người dùng có thể tiến hành lấy `NumberTrivia` theo **random** hoặc **concrete**. Bởi vì chúng tôi đang sử dụng Bloc, điều này sẽ xảy ra bằng cách gửi các **events** .

Hiện tại, nửa dưới của giao diện người dùng có đầy đủ `Placeholder` nhưng ngay cả trước khi thay thế chúng bằng widget thực, hãy tách toàn bộ nửa dưới Column thành một `TriviaControls` **stateful** widget.

```
number_trivia_page.dart
```
```
class TriviaControls extends StatefulWidget {
  const TriviaControls({
    Key key,
  }) : super(key: key);

  @override
  _TriviaControlsState createState() => _TriviaControlsState();
}

class _TriviaControlsState extends State<TriviaControls> {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        // Placeholders here...
      ],
    );
  }
}
```

Tại sao `stateful`? `TriviaControls` sẽ có một `TextField` và để gửi dữ liệu `String` được nhập vào Bloc mỗi khi nhấn nút, widget này sẽ cần giữ `String` đó như state cục bộ.

Ngoài `dispatching`, `GetTriviaForConcreteNumber` event khi button **concrete** được nhấn, event này sẽ được phát đi khi `TextField` được **submitted** bằng cách nhấn một nút trên bàn phím.

```
number_trivia_page.dart
class _TriviaControlsState extends State<TriviaControls> {
  final controller = TextEditingController();
  String inputStr;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        TextField(
          controller: controller,
          keyboardType: TextInputType.number,
          decoration: InputDecoration(
            border: OutlineInputBorder(),
            hintText: 'Input a number',
          ),
          onChanged: (value) {
            inputStr = value;
          },
          onSubmitted: (_) {
            dispatchConcrete();
          },
        ),
        SizedBox(height: 10),
        Row(
          children: <Widget>[
            Expanded(
              child: RaisedButton(
                child: Text('Search'),
                color: Theme.of(context).accentColor,
                textTheme: ButtonTextTheme.primary,
                onPressed: dispatchConcrete,
              ),
            ),
            SizedBox(width: 10),
            Expanded(
              child: RaisedButton(
                child: Text('Get random trivia'),
                onPressed: dispatchRandom,
              ),
            ),
          ],
        )
      ],
    );
  }

  void dispatchConcrete() {
    // Clearing the TextField to prepare it for the next inputted number
    controller.clear();
    BlocProvider.of<NumberTriviaBloc>(context)
        .dispatch(GetTriviaForConcreteNumber(inputStr));
  }

  void dispatchRandom() {
    controller.clear();
    BlocProvider.of<NumberTriviaBloc>(context)
        .dispatch(GetTriviaForRandomNumber());
  }
}
```

Dĩ nhiên, chúng ta hãy một lần nữa đặt widget này vào file `trivia_controls.dart` và sau đó thêm đó thêm nó vào `export` trong `widgets.dart`. 

# Sửa đổi lần cuối

Nó hoạt động! Người dùng cuối cùng có thể nhập một số và nhận được một mẩu chuyện cụ thể hoặc người dùng chỉ cần nhấn nút random và nhận được một mẩu chuyện ngẫu nhiên. Có một lỗi UI và nó trở nên rõ ràng ngay lập tức khi chúng ta mở bàn phím:

![](https://i1.wp.com/resocoder.com/wp-content/uploads/2019/09/keyboard-overflow.png?resize=576%2C1024&ssl=1)

Điều này có thể được sửa chữa rất dễ dàng với một `SingleChildScrollView` bọc toàn bộ `body` của `Scaffold`. Cùng với đó, bất cứ khi nào bàn phím xuất hiện và body bị co lại ở chiều cao , nó sẽ hiện thị thanh cuộn cuộn và overflow sẽ không xảy ra.

```
number_trivia_page.dart
```
```
class NumberTriviaPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Number Trivia'),
      ),
      body: SingleChildScrollView(
        child: buildBody(context),
      ),
    );
  }
  ...
}
```

![](https://resocoder.com/wp-content/uploads/2019/09/005-confetti.svg)

# Chúng ta đã xong ?

Trong 14 phần của series Clean Architecture, chúng tôi đã chuyển từ một ý tưởng sang một ứng dụng hoạt động. Bạn đã học cách kiến ​​trúc ứng dụng của mình thành các lớp độc lập, cách thực hiện TDD, khám phá sức mạnh của hợp đồng (contract), học cách dependency injection và nhiều hơn nữa.

*Number Trivia App*, chúng tôi đã xây dựng nó với chức năng đơn giản, điều quan trọng ở đây là những nguyên tắc chúng ta thực hiện. Những gì bạn học được trong khóa học này có thể áp dụng cho tất cả các loại trường hợp và tôi tin rằng sau khi theo dõi đến đây, bạn đã là một nhà phát triển tốt hơn trước. Thực hành các nguyên tắc bạn đã học ở đây và xây dựng một thứ gì đó thật tuyệt vời nhé!