---
layout: post
title: Flutter - Sử dụng Keys trong Flutter [1]
---

[Nguồn](https://medium.com/flutter/keys-what-are-they-good-for-13cb51742e7d)

Tham số **Key** có thể xuất hiện ở hầu hết mọi widget trong Flutter, nhưng việc sử dụng chúng lại ít phổ biến. Các key này sẽ giữ state của widget và *định vị* chúng trong `WidgetTree`. Trong thực tế, chúng hữu ích trong việc xác định vị trí cuộn của người dùng hoặc giữ trạng thái khi ta sửa đổi collection chứa các widget.

# Ví dụ về Key

Hầu hết, bạn không cần đến **Key** trong tất cả các Widget của mình ! Nói chung là không cần đến chúng cũng được, giống như từ khóa **new** hoặc việc khai báo các type ở cả bên phải và bên trái của một biến (Ví dụ: `Map<Foo, Bar> aMap = Map<Foo, Bar>()`). **Nhưng, nếu trong trường hợp thêm, xóa hoặc sắp xếp lại một collection các widget cùng loại có state thay đổi, bạn có thể phải dùng đến Key!**

Quokka sẽ viết một ứng dụng đơn giản với 2 Widget có màu ngẫu nhiên hoán đổi vị trí khi nhấn vào button:

![](https://miro.medium.com/max/320/1*edgczyvaQRgGRy8yhht0QQ.gif)

Trong phiên bản **Stateless** của ứng dụng, Quokka có hai `StatelessTiles`, mỗi ô có một màu được tạo ngẫu nhiên, trong một `Row` và tạo một **Stateful** `PositionedTiles` để lưu trữ vị trí của các ô này. Khi Quokka ấn vào `FloatingActionButton`, `PositionedTiles` sẽ hoán đổi vị trí của chúng trong list bằng phương thức `setState`:

```
class PositionedTiles extends StatefulWidget {
 @override
 State<StatefulWidget> createState() => PositionedTilesState();
}

class PositionedTilesState extends State<PositionedTiles> {
 List<Widget> tiles = [
   StatelessColorfulTile(),
   StatelessColorfulTile(),
 ];

 @override
 Widget build(BuildContext context) {
   return Scaffold(
     body: Row(children: tiles),
     floatingActionButton: FloatingActionButton(
         child: Icon(Icons.sentiment_very_satisfied), onPressed: swapTiles),
   );
 }

 swapTiles() {
   setState(() {
     tiles.insert(1, tiles.removeAt(0));
   });
 }
}

class StatelessColorfulTile extends StatelessWidget {
 Color myColor = UniqueColorGenerator.getColor();
 @override
 Widget build(BuildContext context) {
   return Container(
       color: myColor, child: Padding(padding: EdgeInsets.all(70.0)));
 }
}
```

Nhưng nếu Quokka thay `StatelessColorfulTile` bằng `StatefulColorfulTile` và lưu Color trong State. Chạy lại và nhấn button thì có vẻ như không có gì xảy ra? Why ?

![](https://miro.medium.com/max/320/1*T7TBQx9DhaQ16gbX68XxVw.gif)

```
List<Widget> tiles = [
   StatefulColorfulTile(),
   StatefulColorfulTile(),
];

...
class StatefulColorfulTile extends StatefulWidget {
 @override
 ColorfulTileState createState() => ColorfulTileState();
}

class ColorfulTileState extends State<ColorfulTile> {
 Color myColor;

 @override
 void initState() {
   super.initState();
   myColor = UniqueColorGenerator.getColor();
 }

 @override
 Widget build(BuildContext context) {
   return Container(
       color: myColor,
       child: Padding(
         padding: EdgeInsets.all(70.0),
       ));
 }
}
```

Xin nhắc lại, đoạn code hiển thị ở trên bị lỗi ở chỗ: nó không hiển thị màu sắc bị hoán đổi khi người dùng nhấn button. Cách khắc phục cho vấn đề này là thêm một tham số **Key** vào các widget state:

![](https://miro.medium.com/max/320/1*3XbdhaQ9_lPfILdViiipeQ.gif)

```
List<Widget> tiles = [
  StatefulColorfulTile(key: UniqueKey()), // Keys added here
  StatefulColorfulTile(key: UniqueKey()),
];

...
class StatefulColorfulTile extends StatefulWidget {
  StatefulColorfulTile({Key key}) : super(key: key);  // NEW CONSTRUCTOR
 
  @override
  ColorfulTileState createState() => ColorfulTileState();
}

class ColorfulTileState extends State<ColorfulTile> {
  Color myColor;

  @override
  void initState() {
    super.initState();
    myColor = UniqueColorGenerator.getColor();
  }

  @override
  Widget build(BuildContext context) {
    return Container(
        color: myColor,
        child: Padding(
          padding: EdgeInsets.all(70.0),
        ));
  }
}
```

Nhưng, điều này chỉ cần thiết nếu bạn có các Widget có Stateful trong subtree mà bạn đang sửa đổi. **Nếu toàn bộ subtree widget trong collection của bạn là Stateless , các Key không cần thiết**.
Đó là tất cả những gì bạn cần biết để sử dụng Key trong Flutter. Nhưng nếu bạn muốn hiểu chi tiết hơn thì tại sao lại như vậy?

# Sự khó chịu của Key đôi khi lại cần thiết

Có thể bạn đã biết, bên dưới một **Widget**, Flutter sẽ xây dựng một **Element** tương ứng. Khi xây dựng một `WidgetTree`, Flutter cũng xây dựng một `ElementTree`. Chỉ là ElementTree này đơn giản hơn, nó chỉ chứa thông tin về Type của từng Widget và tham chiếu đến các Element con. Bạn có thể nghĩ ElementTree giống như một bộ xương của ứng dụng Flutter.
Row widget trong ví dụ trên về cơ bản chứa một tập hợp các vị trí được đặt cho mỗi con của nó. Khi Quokka trao đổi thứ tự của các `TileWidget` trong Row, Flutter sẽ kiểm tra ElementTree xem cấu trúc *khung xương* cũ và mới có giống nhau không.

![](https://miro.medium.com/max/600/1*sHDIVXBu9RpJYN9Zdn8iBw.gif)

Bắt đầu với `RowElement`, và sau đó đến con của nó. Flutter sẽ kiểm tra xem widget mới có cùng type và key với element cũ hay không, và nếu giống, nó sẽ cập nhật tham chiếu của nó tới widget đó. Trong phiên bản stateless, các widget không có key, vì vậy Flutter chỉ kiểm tra type (xem hình).
Cấu trúc của `ElementTree` cho các `StatefulWidget` trông hơi khác. Có các widget và các element như trước đây, nhưng có thêm một cặp state objects đi cùng chứa thông tin về Color.

![](https://miro.medium.com/max/600/1*noTkKudlGuaAkiGaubEcNA.gif)

Trong trường hợp `TileStateful` không có Key, khi Quokka đổi thứ tự của hai Widget, Flutter duyệt `ElementTree`, kiểm tra Type của `RowWidget` và cập nhật tham chiếu. Sau đó `TileElement` kiểm tra xem widget tương ứng có cùng loại (`TileWidget`) hay không, sau đó nó cập nhật tham chiếu. Điều tương tự cũng xảy ra với child thứ hai. Bởi vì Flutter sử dụng `ElementTree` và State tương ứng của nó để xác định những gì thực sự sẽ hiển thị trên thiết bị, và vì thế, trong trường hợp này, các widget đã không được hoán đổi đúng cách!

![](https://miro.medium.com/max/600/1*7n-u4yexzRZDEtNvbrsG1g.gif)

Trong phiên bản đã sửa lỗi với `StatefulTiles`, Quokka đã thêm param Key cho các widget. Bây giờ chúng ta trao đổi các widget như trước, nhưng giờ Key của `TileElement` không khớp với khóa của `TileWidget` tương ứng. Điều này khiến Flutter hủy kích hoạt các element đó và xóa các tham chiếu đến các `TileElement` trong `ElementTree`, bắt đầu với element đầu tiên không khớp.

![](https://miro.medium.com/max/600/1*AcBxC8IF_irZpFARt-Nqyw.gif)

Sau đó, Flutter xét children element của `RowElement`, tìm và tham chiếu đến Widget có Key tương ứng. Flutter làm điều tương tự cho child thứ hai. Và thế là Flutter sẽ hiển thị những gì Quokka mong đợi, với các Widget hoán đổi vị trí và cập nhật màu sắc khi nhấn button.

Tóm lại, Key sẽ hữu ích nếu bạn tác động đến vị trí của các stateful widget trong collection như trong ví dụ trên. Thông thường, state phức tạp hơn nhiều. Việc phát animation, hiển thị dữ liệu mà người dùng đã nhập và vị trí scroll đều liên quan đến state.

# Tiếp theo

Ở phần tiếp theo, Quokka sẽ đề cập đến việc sử dụng Key ở đâu, và các loại Keys được sử dụng trong Flutter cũng như các trường hợp áp dụng của chúng.

Cảm ơn bạn vì đã đọc ! <3 
