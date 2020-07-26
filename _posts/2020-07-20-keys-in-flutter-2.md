---
layout: post
title: Flutter - Sử dụng Keys trong Flutter [2]
---

[Nguồn](https://medium.com/flutter/keys-what-are-they-good-for-13cb51742e7d)

Ở phần trước, Quokka đã đề cập đến vấn đề trong một số trường hợp không có Keys, và khái quát mô hình hoạt động của Widget - Element. Trong phần này, chúng ta sẽ đến với việc sử dụng Keys như nào và các loại Keys mà Flutter hỗ trợ.

# Đặt Key ở đâu?

**Nếu bạn cần thêm Key vào ứng dụng của mình, bạn nên thêm chúng ở đầu của widget chứa state mà bạn cần giữ.**


Một lỗi phổ biến mà Quokka từng thấy là mọi người nghĩ rằng họ chỉ cần đặt một Key trên StatefulWidget đầu tiên. Với ví dụ dưới đây, Quokka đã bọc các Widget đầy màu sắc của mình bằng `PaddingWidget`, nhưng Quokka để Key trong `StatefulColorfulTile`.

```
class PositionedTiles extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => PositionedTilesState();
}

class PositionedTilesState extends State<PositionedTiles> {
  // Stateful tiles now wrapped in padding (a stateless widget) to increase height 
  // of widget tree and show why keys are needed at the Padding level.
  List<Widget> tiles = [
    Padding(
      padding: const EdgeInsets.all(8.0),
      child: StatefulColorfulTile(key: UniqueKey()), // here
    ),
    Padding(
      padding: const EdgeInsets.all(8.0),
      child: StatefulColorfulTile(key: UniqueKey()), // here
    ),
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

class StatefulColorfulTile extends StatefulWidget {
  StatefulColorfulTile({Key key}) : super(key: key);
 
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

Bây giờ khi Quokka nhấp button, các ô sẽ thay đổi thành các màu ngẫu nhiên hoàn toàn khác nhau!

Đây là giao diện của `WidgetTree` và `ElementTree` với sau khi Padding được thêm vào:

![](https://miro.medium.com/max/600/1*uC-SRZpRkOZCEr_rGisF9g.gif)

Khi chúng ta trao đổi vị trí của các children, Flutter xem xét `ElementTree` ở tầng đầu tiên với `PaddingElement`, mọi thứ chính xác.

![](https://miro.medium.com/max/600/1*vD86ZINBC-1Ctx9kudEGaw.gif)

Ở tầng thứ hai, Flutter thông báo rằng Key của `TileElement` không khớp với Key của Widget, do đó, nó hủy kích hoạt `TileElement` đó, drop các kết nối tới đó. Loại Key Quokka đang sử dụng trong ví dụ này là `LocalKeys`. Điều đó nghĩa là khi kết hợp Widget với các Element, Flutter chỉ tìm kiếm các key khớp trong một tầng cụ thể trong tree.
Vì nó không thể tìm thấy một `TileWidget` nào ở tầng đó khớp giá trị Key, nên nó tạo một Element mới và khởi tạo một State mới, làm cho Widget có màu khác!

![](https://miro.medium.com/max/600/1*JI1Ex87QRMTCJwBWmbNI5A.gif)

Nếu Quokka thêm Key ở tầng của `PaddingWidget`:

```
class PositionedTiles extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => PositionedTilesState();
}

class PositionedTilesState extends State<PositionedTiles> {
  List<Widget> tiles = [
    Padding(
      // Place the keys at the *top* of the tree of the items in the collection.
      key: UniqueKey(), 
      padding: const EdgeInsets.all(8.0),
      child: StatefulColorfulTile(),
    ),
    Padding(
      key: UniqueKey(),
      padding: const EdgeInsets.all(8.0),
      child: StatefulColorfulTile(),
    ),
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

class StatefulColorfulTile extends StatefulWidget {
  StatefulColorfulTile({Key key}) : super(key: key);
 
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

Flutter thông báo và cập nhật các kết nối một cách chính xác, giống như nó đã làm trong ví dụ trước.

![](https://miro.medium.com/max/600/1*FkCvw_LCfQ2x02wj7cmrpA.gif)

# Vậy nên sử dụng loại Key nào?

API Flutter đã cung cấp cho chúng ta nhiều loại Key khác nhau để lựa chọn. Loại key bạn nên sử dụng tùy thuộc vào đặc điểm của các items cần đến Key. Ở đây Quokka sẽ nói về 4 loại khác nhau của các Key: `ValueKey`, `ObjectKey`, `UniqueKey`, và `UniqueKey`

Hãy xem ứng dụng *Todo List* sau đây, bạn có thể sắp xếp lại các item trong danh sách TODO của mình dựa trên mức độ ưu tiên và sau đó xóa chúng khi bạn hoàn thành.

![](https://miro.medium.com/max/320/1*wHJZnNPhMkePFEw1ihrbEA.gif)

Trong trường hợp này, giả sử task của các work là duy nhất, bạn nên sử dụng `ValueKey`, trong đó task là giá trị của ValueKey:

```
return TodoItem(
  key: ValueKey(todo.task),
  todo: todo,
  onDismissed: (direction) => _removeTodo(context, todo),
);
```

Trong một trường hợp khác, Quokka có một ứng dụng sổ địa chỉ của người dùng. Trong trường hợp này, mỗi widget con lưu trữ một sự kết hợp dữ liệu phức tạp hơn. Bất kỳ trường nào, như tên hoặc ngày sinh có thể giống với đối tượng khác, nhưng sự kết hợp của chúng là duy nhất. Vì vậy, một `ObjectKey` có lẽ là thích hợp nhất.

![](https://miro.medium.com/max/700/1*vZV_QjG1GEg7nJILMbhEkA.png)

Nếu bạn có nhiều widget trong collection của mình với cùng một giá trị hoặc nếu bạn muốn đảm bảo mỗi widget khác biệt với các widget khác, bạn có thể sử dụng `UniqueKey` như Quokka đã dùng nó trong ví dụ ở đầu bài. Và hãy cẩn thận nếu bạn khởi tạo `UniqueKey` trong `build` method, widget sử dụng Key đó sẽ nhận được một Key khác mỗi khi build method được thực thi lại. Khi đó thì việc sử dụng Key là vô nghĩa!

`PageStorageKeys` là các Key chuyên dụng lưu trữ vị trí cuộn của người dùng để ứng dụng có thể lưu giữ nó sau này.

![](https://miro.medium.com/max/320/1*KgQeq1LDIPVuE2dwNzZbRQ.gif)

`GlobalKeys` có hai cách sử dụng: chúng cho phép child widget thay đổi parent widget ở bất cứ đâu trong ứng dụng của bạn mà không bị mất trạng thái hoặc chúng có thể được sử dụng để truy xuất thông tin về một widget khác trong một phần hoàn toàn khác của cây widget. 

Một ví dụ về kịch bản đầu tiên đó là nếu bạn muốn hiển thị cùng một widget trên hai màn hình khác nhau, nhưng giữ trạng thái chúng giống nhau, bạn muốn sử dụng một `GlobalKey`. Trong kịch bản thứ hai, bạn muốn xác thực mật khẩu, nhưng không muốn chia sẻ thông tin trạng thái đó với các widget khác trong cây. `GlobalKeys` cũng có thể được sử dụng để kiểm tra, bằng cách sử dụng key để truy cập một widget cụ thể và thông tin truy vấn về trạng thái của nó.

![](https://miro.medium.com/max/320/1*JIPjn-gM6OIG_TfPJvtuVA.gif)

`GlobalKeys` có một chút giống như các biến toàn cầu. Thường có một cách tốt hơn để truy xuất state đó là sử dụng `InheritedWidgets` hoặc các pattern như Redux hoặc BLoC.

# Tóm tắt nhanh

Tóm lại, sử dụng Key khi bạn muốn duy trì state trên các cây widget. Điều này thường xảy ra khi bạn sửa đổi một bộ collection các widget cùng loại, như trong `List<Widget>`. Đặt key ở đầu cây widget bạn muốn giữ state và chọn loại key bạn sử dụng dựa trên dữ liệu mà widget có.



