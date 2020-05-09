---
layout: post
title: Flutter TDD Clean Architecture Course [1]
image: /img/hello_world.jpeg
---

Giữ code clean và  tested là 2 điều quan trọng nhất đối với một developer. Trong Flutter, điều này càng đúng hơn với một số framework. Một mặt, đó là một cách để tạo ra ứng dụng một cách nhanh chóng. Mặt khác, các dự án bắt đầu "khủng hoảng" khi bạn áp dụng Business Logic ở mọi nơi. Ngay cả các pattern quản lý State như BLoC cũng không đủ để mở rộng dự án.

# The Secret to Maintainable Apps

Đây là nơi chúng ta có thể employ **clean architecture** và **test driven development**. Theo đề xuất, chúng ta nên tách code thành các lớp độc lập và phụ thuộc vào **abstractions** thay vì **implementations**

Vậy làm sao để thực hiện điều đó ? Dưới đây là một mô hình "onion", mũi tên ngang thể hiện sự phụ thuộc. Ví dụ Entities không phụ thuộc vào bất cứ điều gì, Use Case phụ thuộc vào Entities...

![mohinhcuhanh](https://i1.wp.com/resocoder.com/wp-content/uploads/2019/08/CleanArchitecture.jpg?w=772&ssl=1)

Các nguyên tắc như SOLID, YAGNI nghe có vẻ hay. Nhưng chúng sẽ không có ích gì nếu bạn không biết cách clean code.

# Xây dựng project

Chúng ta sẽ tạo một ứng dụng Number Trivia: Mô tả những con số. 
Project này gồm: lấy data từ API, local cache, xử lý lỗi, input validation...
Đối với state management, tôi sử dụng BLoC. Clean architecture không phải là một kỹ thuật quản lý state cụ thể. Bạn sẽ nhận ra kiến trúc ứng dụng của bạn tốt sẽ làm cho quản lý state gần như không quan trọng. Tôi sẽ không chọn StatefulWidget nhưng mọi thứ khác đều ổn.

# Clean Architecture và Flutter

Để cho các bạn nhìn nhận rõ hơn về Flutter, tôi sẽ giới thiệu cho bạn về ***Reso Coder Flutter Clean Architecture Proposed ™*** 

![](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/08/Clean-Architecture-Flutter-Diagram.png?w=556&ssl=1)

# Giải thích và tổ chức project

Mỗi feature của ứng dụng sẽ được chia thành 3 lớp: presentation, domain và data. Ứng dụng mà tôi xây dựng chỉ có một tính năng.

![](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/08/number_trivia-feature.png?w=269&ssl=1)

## Presentation

Bạn sẽ cần các widget để hiển thị something lên màn hình. Các widget này có thể gửi các event đến Bloc và listen các state. 

![presentation](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/08/presentation-layer-diagram.png?w=287&ssl=1)

Lưu ý rằng *Presentation Logic Holder* (Bloc) không tự làm được gì nhiều. Nó sẽ ủy thác (delegates) tất cả công việc cho **Use Case**. Nhiều nhất Presentation Layer xử lý cơ bản về input conversion và validation.

### Áp dụng vào Number Trivia App

Chúng ta cần một page và các widgets được đặt tên *NumberTriviaPage* với một *NumberTriviaBloc*

## Domain

Domain là lớp mà bên trong nó không dễ bị thay đổi khi thay đổi nguồn dữ liệu hoặc chuyển đổi ứng dụng sang Angular Dart. Nó sẽ chỉ chứa business logic (Use Case) và business object (entities). Nó nên độc lập hoàn toàn với mọi lớp khác. 

**Use Case** là các lớp gói  tất cả các business logic của một trường hợp cụ thể của ứng dụng. Ví dụ như Get Concrete NumberTrivia hoặc Get Random NumberTrivia.

Nhưng làm sao để lớp **Domain** độc lập hoàn toàn khi mà nó lấy dữ liệu từ **Repositories** trong **Data Layer** ? Bạn có nhận ra màu sắc của Repository có hiệu ứng gradient không ? Có nghĩa rằng nó thuộc về cả 2 lớp cùng một lúc. Chúng ta có thể thực hiện điều này với [*dependency invesion*](https://en.wikipedia.org/wiki/Dependency_inversion_principle).

Dependency insersion là nguyên tắc cuối cùng trong SOLI**D**. Về cơ bản, nó nói rằng ranh giới giữa các lớp nên được  xử lý bằng các interface (abstract trong Dart).

![Repositories are on the edge between data and domain](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/08/domain-layer-diagram.png?w=141&ssl=1)

Chúng ta sẽ tạo một abstract class **Repository** định nghĩa những điều mà Repo cần phải làm ("hợp đồng") ở trong **Domain Layer**. Việc hoàn thành "hợp đồng" đấy sẽ do **Repository** trong **Data Layer** thực hiện. 

### Áp dụng vào Number Trivia App

Sẽ không có nhiều Business Logic để thực thi trong ứng dụng, vì chúng ta chỉ hiển thị thông tin. Đối với các Business Object, sẽ có một **Entity** duy nhất được gọi là **NumberTrivia** (gồm một số và text mô tả con số)

![Domain folder structure](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/08/domain-layer.png?w=366&ssl=1) 

## Data

**Data Layer** bao gồm một **Repository implementation** ("hợp đồng" từ **Domain Layer**) và **Data source** - một cái dùng để lấy dữ liệu từ remote data (API) và cái kia để caching dữ liệu đã lấy được. **Repository** là nơi bạn quyết định return dữ liệu mới hoặc dữ liệu cache, khi nào thì cache...

Có thể nhận thấy rằng **Data source** không return **Entities** mà thay vào đó là **Models**. Lý do đằng sau việc này là việc chuyển đổi dữ liệu thô (ví dụ JSON) thành các Object Dart yêu cầu một số JSON conversion code. Tôi không muốn các mã chuyển đổi đấy ở trong **Domain Entities** - Điều gì sẽ xảy ra nếu tôi chuyển sang  XML ?

![](https://i0.wp.com/resocoder.com/wp-content/uploads/2019/08/data-layer-diagram.png?w=329&ssl=1)

Do đó, tôi tạo **Model** chúng extends **Entities** và  một số func như toJson, fromJson... hoặc thêm  một số trường như ID Database...

### Áp dụng vào Number Trivia App

**RemoteDataSource** sẽ thực hiện các yêu cầu HTTP GET tới API

**LocalDataSource** sẽ chỉ lưu dữ liệu bằng cách sử dụng shared preferences

2 nguồn dữ liệu này sẽ được kết hợp trong **NumberTriviaRepository**, đây sẽ là nguồn dữ liệu chính cho ứng dụng.

Cơ chế caching rất đơn giản. Nếu có kết nối mạng thì lấy dữ liệu từ api và cache nó. Nếu không có kết nối mạng, return dữ liệu mới nhất đã được lưu trong bộ nhớ cache.

![](https://i1.wp.com/resocoder.com/wp-content/uploads/2019/08/data-layer.png?w=416&ssl=1)

# Tiếp theo...

Tôi đã có kiến trúc nền tảng cho project của mình. Tiếp theo, tôi sẽ triển khai lớp ổn định nhất - **domain**. Tôi cũng sẽ thêm các package cần thiết cho project, bao gồm các gói dành cho TDD [mockito](https://pub.dev/packages/mockito). Chúng tôi sẽ phát triển theo hướng TDD.






