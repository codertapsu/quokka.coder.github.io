---
layout: post
title: Xây dựng ứng dụng cập nhật ca nhiễm COVID-19
image: https://pbs.twimg.com/profile_images/935545553181483008/NqTOIvxr.jpg
---

# Lan man
Hôm nay là những ngày đầu tháng 4 năm 2020. Tình hình dịch Corona ở Việt Nam cũng như thế giới đang có nhiều biến động phức tạp.

Cũng như bao DEV khác, Quokka đang làm việc online ở nhà. Có vẻ như thằng Corona này không ảnh hưởng nhiều đến anh em làm DEV lắm.

Cũng như mọi ngày, Bug chất một đống ...

Sau một ngày cày cuốc nữa, Quokka cũng hoàn thành xong Task của mình. Hôm nay có vẻ hơi sớm. Cũng rảnh không có gì làm, Quokka sẽ demo xây dựng ứng dụng Flutter: **Cập nhật các ca nhiễm Corona bằng mô hình quản lý Bloc nhé !**

À, Quokka sẽ sử dụng lib có sẵn: [bloc_flutter](https://bloclibrary.dev/#/)
Anh em mới làm quen Flutter hoặc tìm hiểu sâu hơn thì vào đó đào bới nhé :D

# Demo
![quokka](https://media.giphy.com/media/MaaWOZeBZUJb6fFonl/giphy.gif)
- Quokka tạo ra màn hình hiển thị như trên. Khi vuốt cuộn thì refresh lại trang.

Về cơ bản, một ứng dụng triển khai theo Bloc đầy đủ cần tổ chức theo các lớp:
- Model
- Data (gồm: Data Provider và Repository)
- Business Logic (Bloc)
- Presentation (UI)

Do ứng dụng Quokka chuẩn bị build là nhỏ, nên Quokka sẽ bỏ phần `Repository` nhé ! Ở các project lớn hơn, các bạn hãy thêm `Repository` cho dễ quản lý.

# Build Code

## 1. Setup
- Quokka setup lib vào `pubspec.yaml`:

```
../pubspec.yaml
```
```
  flutter_bloc: ^3.2.0
  equatable: ^1.1.1
  http: ^0.12.0+4
```
- Sau đó thực hiện lệnh cài đặt: `flutter packages get`

## 2. REST API
- Quokka sử dụng [API](https://coronavirus-19-api.herokuapp.com/countries)
- Dùng Json Format để dễ nhìn hơn nào:
![quokka](/img/flutter/post_corona_demo/json_data.png)
- Qua hình trên Quokka thấy dữ liệu ở đây là một mảng đối tượng JSON.
- OK. Quokka Sử dụng [công cụ này](https://javiercbk.github.io/json_to_dart/) để tạo Model:

```
.../lib/model/country.dart
```
```
import 'package:equatable/equatable.dart';

class CountCountry extends Equatable{
  String country;
  int cases;
  int todayCases;
  int deaths;
  int todayDeaths;
  int recovered;
  int mildCondition;
  int active;
  int critical;
  int closedCase;
  String ratioMildCondition;
  String ratioRecovered;
  String ratioCritical;
  String ratioDeaths;

  CountCountry.fromJson(Map<String, dynamic> json){
    country = json['country'];
    cases = json['cases'];
    todayCases = json['todayCases'];
    deaths = json['deaths'];
    todayDeaths = json['todayDeaths'];
    recovered = json['recovered'];
    active = json['active'];
    critical = json['critical'];
    //// custom property ////
    mildCondition = active - critical; // Mild Condition
    closedCase = deaths + recovered; // All Closed Case
    ratioMildCondition= (mildCondition / active * 100).toStringAsFixed(2); // ratio Mild Condition per Active 
    ratioRecovered= (recovered / closedCase * 100).toStringAsFixed(2); // ratio Recovered per Closed Case 
    ratioCritical= (critical / active * 100).toStringAsFixed(2); // ratio Critical Condition per Active 
    ratioDeaths= (deaths / closedCase * 100).toStringAsFixed(2); // ratio Deaths per Closed Case  
  }

  Map<String, dynamic> toJson() {
    final Map<String, dynamic> data = new Map<String, dynamic>();
    data['country'] = this.country;
    data['cases'] = this.cases;
    data['todayCases'] = this.todayCases;
    data['deaths'] = this.deaths;
    data['todayDeaths'] = this.todayDeaths;
    data['recovered'] = this.recovered;
    data['active'] = this.active;
    data['critical'] = this.critical;
    return data;
  }

  @override
  // TODO: implement props
  List<Object> get props => [country, cases, todayCases, deaths, todayDeaths, recovered, active, critical];

  @override
  // TODO: implement stringify
  bool get stringify => true;
}
```
- Ở trong model phía trên, Quokka có thêm một số thuộc tính được gán giá trị từ những thuộc tính có sẵn đã Parse từ JSON Data (Chỗ Quokka cmt ấy ^^).

*OK. Vậy là xong phần model. Sang xây dựng Data Provider nào !*

## 3. Data Provider
*"Trách nhiệm của Data Provider là cung cấp dữ liệu thô. Các Data Provider nên chung chung và linh hoạt."*

- Tạo package data_provider và file corona_api trong đó:
```
.../lib/data_provider/corona_api.dart
```
```
class CoronaApi{
  final String _url = 'https://coronavirus-19-api.herokuapp.com'; // base URL

  @override
  Future<List<CountCountry>> getCountList() async {
    final urlGetCountries = '$_url/countries'; // get countries URL

    try{
      var response = await http.get(urlGetCountries,);
      if(response.statusCode != 200){
      	// failed
        throw Exception('>>> Fetch corona error with status: ${response.statusCode}');
      }else{
      	// success
        var data = List<CountCountry>();
        final responseJson = json.decode(response.body);
        for(dynamic x in responseJson) {
          final country = CountCountry.fromJson(x);
          data.add(country);
        }
        return data;
      }
    }catch(e){
      throw Exception('>>> Fetch corona error : ${e.toString()}');
    }
  }
}
```
- Như đã đề cập, lớp Data Provider này có nhiệm vụ lấy dữ liệu *"thô"* từ các nguồn, và return lại dữ liệu bằng `Future`.
- Quokka đã dùng lib [http](https://pub.dev/packages/http) để lấy dữ liệu từ url API bằng method GET.
- Bắt lỗi, xử lý và return data khi thành công.

*Vậy là xong lớp Data Provider. Chuyển qua lớp Bloc xem Quokka xử lý tiếp nhé !*

## 4. Business Logic (Bloc)
*Trách nhiệm của lớp Bloc là xử lý logic tương ứng với các event được tương tác từ lớp UI, mỗi lần tương tác UI sẽ cung cấp một State mới.*

*Lớp Bloc có thể phụ thuộc vào một hoặc nhiều Repositories để lấy dữ liệu cần thiết, từ đó xây dựng State của ứng dụng (ở đây Quokka dùng thẳng Data Provider luôn nên không có Repository).*

**Trong mô hình Bloc, việc khó nhất là xử lý ở lớp này. Nên các bạn để ý nhé !**

Bloc hoạt động theo cơ chế sử dụng Stream để:
- Nhận vào một Event
- Xử lý logic tương ứng với event nhận vào
- Đổ ra đầu kia một State mới tương ứng với kết quả xử lý

Với mỗi màn hình, các event và state có thể là giống hoặc khác nhau. Điều khó ở đây là Event và State là gì và làm sao xác định được event và state nào sẽ sử dụng?

Theo Quokka:
- Trong mỗi lớp Bloc, chúng ta sẽ xác định các Event trước. Event chính là yêu cầu của UI đặt ra cho Bloc xử lý. Như là front-end và back-end trong web vậy ! (*Ví dụ: fetchData, click button,...*)
- State chính là các trạng thái của UI sau khi lấy data từ provider và xử lý *(Ví dụ: trạng thái thành công, lỗi, data trống...)*

Áp dụng vào dự án đang làm, Quokka xác định được:
- Event:
: `FetchData`: Dùng để lấy dữ liệu lúc bắt đầu mở trang, và khi refresh lại trang.

- State:
: `Loading`: Trạng thái của màn hình khi đang xử lý data
: `Loaded`: Trạng thái màn hình khi đã lấy thành công data
: `Error`: Trạng thái màn hình khi gặp lỗi trong lúc xử lý data


*Okay. Xác định được các đối tượng rồi. Code thôi !*

#### Tạo package bloc và các file:
```
../lib/bloc/
	|__ bloc.dart
	|__ corona_bloc.dart
	|__ corona_event.dart
	|__ corona_state.dart
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
#### Thêm event và state đã xác định ở trên vào các file tương ứng:

- Thêm events:

```
../lib/bloc/
	|__ corona_event.dart
```
```
import 'package:equatable/equatable.dart';

class CoronaEvent extends Equatable{
  @override
  // TODO: implement props
  List<Object> get props => [];

}

class FetchCoronaData extends CoronaEvent{}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
- Thêm states:

```
../lib/bloc/
	|__ corona_state.dart
```
```
import 'package:demo_bloc_corona/model/country.dart';
import 'package:equatable/equatable.dart';

class CoronaState extends Equatable{
  @override
  // TODO: implement props
  List<Object> get props => [];

}

class CoronaLoading extends CoronaState{}

class CoronaLoaded extends CoronaState{
  final List<CountCountry> coronaData;

  CoronaLoaded({this.coronaData}) : assert(coronaData!=null);

  @override
  // TODO: implement props
  List<Object> get props => [coronaData];
}

class CoronaError extends CoronaState{}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
#### Xử lý trong file bloc:

```
../lib/bloc/
	|__ corona_bloc.dart
```
```
import 'package:demo_bloc_corona/data_provider/corona_api.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'bloc.dart';
import 'package:rxdart/rxdart.dart';

class CoronaBloc extends Bloc<CoronaEvent, CoronaState>{
  final _apiCorona= CoronaApi();

  @override
  // TODO: implement initialState
  CoronaState get initialState => CoronaLoading();

  @override
  Stream<CoronaState> transformEvents(Stream<CoronaEvent> events, Stream<CoronaState> Function(CoronaEvent) next) {
    // TODO: implement transformEvents
    return super.transformEvents(
      events.debounceTime(const Duration(milliseconds: 300)),
      next,
    );
  }

  @override
  Stream<CoronaState> mapEventToState(CoronaEvent event) async*{
    if(event is FetchCoronaData){
      yield CoronaLoading();
      try {
        final corona = await _apiCorona.getCountList();
        yield CoronaLoaded(coronaData: corona);
      }catch(e){
        yield CoronaError();
      }
    }
  }

}
```

- `initialState` là state khi Bloc được khởi tạo
- Như Quokka đã nói, trong method `mapEventToState` có nhận vào một tham số event. Dựa vào đây Quokka đã lấy dữ liệu từ api rồi đổ ra state  `CoronaLoaded` khi thành công hoặc `CoronaError` khi có lỗi.
- `transformEvents` là một method xử lý *"hậu Events"*. Quokka sử dụng `debounce` để phòng trường hợp bị spam API.

*Sau khi xây dựng xong các file liên quan. Quokka export hết chúng vào file bloc.dart:*
```
../lib/bloc/
	|__ bloc.dart
```
```
export 'corona_bloc.dart';
export 'corona_event.dart';
export 'corona_state.dart';
```
*Okay. Vậy là hoàn thành lớp Bloc rồi nhé! Giờ qua xử lý UI nè :D*

## 5. Build UI
*Bình thường khi tạo dự án Flutter mới, máy sẽ tự tạo cho bạn một ứng dụng đếm số mẫu đúng không nào ?
Quokka sẽ xóa hết code đó đi và tổ chức lại main cũng như phần view nhé*

- Quokka sẽ tạo một package `view` dành riêng cho lớp UI
- Tách code hàm main ra file `main.dart`
- Tách code của `MaterialApp` ra file `app.dart`

Lúc đó cấu trúc dự án sẽ như thế này:

```
../lib/
	|__ bloc
	|__ data_provider
	|__ model
	|__ views
	|__ app.dart
	|__ main.dart
```  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
- **File `main.dart`:** File này chứa các code xử lý, gọi và khởi tạo các dịch vụ hoặc đơn giản là khai báo biến toàn cục. 

Ví dụ: Trong Bloc Flutter Doc có `SimpleBlocDelegate ` nhằm log lại các state của ứng dụng. Quokka sẽ khởi tạo ở đây. Xem trong [Github](https://github.com/quokkacoder/flutter_ncovid_bloc/tree/master/demo_bloc_corona) nhé

*Lưu ý là thêm code xử lý vào trước lệnh runApp nhé !*

```
```
```
import 'package:flutter/material.dart';
import 'app.dart';

void main() {
  // add something here
  runApp(MyApp());
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
- **File `app.dart`:** File này nhằm mục đích khởi tạo một `MaterialApp` (là một Flutter Wiget hỗ trợ Material design). Thường thì Quokka hay cấu hình theme, và khởi bloc dùng cho toàn ứng dụng trong file này.
Dưới đây, Quokka đã khởi tạo CoronaBloc cho tree HomePage đồng thời add vào event `FetchCoronaData` để tiến hành lấy data:

```
import 'package:demo_bloc_corona/views/home.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'bloc/bloc.dart';

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Corona Demo',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: BlocProvider(
        create: (context) => CoronaBloc()..add(FetchCoronaData()),
        child: HomePage(),
      )
    );
  }
}

```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
- **Package `views`:** Quokka sẽ nhét tất cả UI của mobile vào trong package này. Trong dự án này, các phần tử trong package  được tổ chức như sau:
```
../lib/views/
	|__ has_data
	|__ error_widget.dart
	|__ home.dart
```  

- Trong đó:
: - package `has_data` chứa các widget được xây dựng khi Bloc đổ ra state là `CoronaLoaded`(Xử lý dữ liệu thành công).
: - `errror_widget.dart` là widget được build khi state là `CoronaError` (xử lý bị lỗi).
: - `home.dart` là file chứa các thành phần không thay đổi và chứa `BlocBuilder` (widget chứa state để build các trường hợp tương ứng).

*Bỏ qua code của error_widget và childrend nhỏ. Quokka sẽ trình bày code các phần chính khi bắt state và gửi event. Các bạn có thể xem chi tiết trong [Github]()*


#### Trong file `home.dart` tiến hành bắt `State`:

```
import 'package:demo_bloc_corona/bloc/bloc.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'error_widget.dart';
import 'has_data/has_data.dart';

class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(child: BlocBuilder<CoronaBloc, CoronaState>(
        builder: (context, state) {
          if (state is CoronaError) {
            return HomeErrorWidget();
          } else if (state is CoronaLoaded) {
            final c =
                state.coronaData.where((c) => c.country == 'World').toList();
            return HasCoronaData(c[0]);
          }

          return Center(child: CircularProgressIndicator());
        },
      )),
    );
  }
}
```

Lưu ý:
: Tại đây Quokka có dòng code `final c = state.coronaData.where((c) => c.country == 'World').toList();` nó có mục đích là lấy ra giá trị của đối tượng có country = World (vì Quokka chỉ muốn lấy dữ liệu của Wolrd thôi).
: Sau khi thực thi thì `c` sẽ có kiểu dữ liệu là một `List`. Và vì chắc chắn rằng list này chỉ có một phần tử nên Quokka đã truyền c[0] vào widget `HasCoronaData` để tiếp tục build các child widget khác

#### Cuối cùng là xử lý refresh lại trang trong `has_data`

- Quokka sử dụng `RefreshIndicator` Widget để hỗ trợ việc khi vuốt cuộn thì sẽ gửi event lấy lại data từ api. Lưu ý `RefreshIndicator` phải có child là một Widget có scroll:

```
class HasCoronaData extends StatelessWidget {
  final CountCountry _world;

  HasCoronaData(this._world);

  CoronaBloc _coronaBloc;

  @override
  Widget build(BuildContext context) {
    _coronaBloc = BlocProvider.of<CoronaBloc>(context);

    return RefreshIndicator(
      onRefresh: ()async{
        _coronaBloc.add(FetchCoronaData());
      },
      child: SingleChildScrollView(
        physics: const AlwaysScrollableScrollPhysics(),
        padding: const EdgeInsets.symmetric(vertical: 16, horizontal: 8),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.center,
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            _header(),
            Divider(
              color: Colors.blue[900],
            ),
            SizedBox(
              height: 8,
            ),
            overall(_world),
            SizedBox(
              height: 16,
            ),
            detail(_world),
          ],
        ),
      ),
    );
  }

  _header() {
    final widget = Text(
      'COVID-19 CORONAVIRUS PANDEMIC',
      style: TextStyle(
          fontWeight: FontWeight.bold, fontSize: 18, color: Colors.blue[900]),
    );
    return widget;
  }

}
```
- Ở code trên Quokka có `_coronaBloc = BlocProvider.of<CoronaBloc>(context);` được sử dụng như một dependency injection (DI) để lấy `singleton instance` của `CoronaBloc` ở trong tree `HomePage`.
- Từ `_coronaBloc` Quokka `add` vào event `FetchCoronaData` để gọi lại api.
- Dữ liệu từ api sẽ được cập nhật qua `state`. Và nếu chúng thay đổi, UI cũng sẽ cập nhật theo.

*Hmmm...Đọc code xử lý các bạn hiểu chứ?*

**Mục đích của Quokka là muốn các bạn nắm được cách tổ chức code theo mô hình Bloc. Nên muốn thêm các child widget khác thì các bạn vào [Github](https://github.com/quokkacoder/flutter_ncovid_bloc/tree/master/demo_bloc_corona) clone code về test, xem và chạy thử nha <3**

*Quokka sẽ dừng code tại đây vì bài viết khá dài rồi, đọc sẽ rất khó theo dõi*

# Tổng kết
- Qua dự án nhỏ này Quokka rút ra được:
: Bloc sử dụng Stream để:
: - Nhận vào một Event
: - Xử lý logic tương ứng với event nhận vào
: - Đổ ra đầu kia một State mới tương ứng với kết quả xử lý
- Điểm trừu tượng nhất là xác định **Event** và **State**
- Điểm khó nhất là xử lý **Bloc**
- Nên tách `main` và `app` ra từng file riêng biệt để dễ theo dõi

*Có thể bài viết sẽ không đầy đủ nên có khó khăn gì khi tiếp cận thì các bạn hãy liên hệ với Quokka bằng cách ấn nút góc phía trên bên phải nhé :D* 

**Cảm ơn mọi người đã theo dõi ! Quokka lại vào hang đây...**




