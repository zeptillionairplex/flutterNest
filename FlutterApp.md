안녕하세요! 30년 경력의 개발자 입장에서 Flutter로 간단한 쇼핑몰 앱을 만들고자 하시는군요. 다양한 상품이 나열된 페이지, 상품을 터치하면 상세 페이지로 이동하는 기능, 그리고 장바구니 기능을 구현하는 것이 목표라고 하셨는데, 필요한 내용을 단계별로 안내해 드리겠습니다.

### 1. Flutter 프로젝트 생성
```bash
flutter create shopping_mall_app
cd shopping_mall_app
```

### 2. 필요한 패키지 추가
`pubspec.yaml` 파일에 다음 패키지를 추가하세요.
```yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.0.5
```

### 3. 데이터 모델 정의
`lib/models/product.dart` 파일에 상품의 데이터 모델을 정의하세요.
```dart
class Product {
  final String id;
  final String title;
  final String description;
  final double price;
  final String imageUrl;

  Product({
    required this.id,
    required this.title,
    required this.description,
    required this.price,
    required this.imageUrl,
  });
}
```

### 4. 가짜 데이터 생성
`lib/data/dummy_products.dart` 파일에 가짜 데이터를 생성하세요.
```dart
import '../models/product.dart';

final List<Product> dummyProducts = [
  Product(
    id: 'p1',
    title: 'Red Shirt',
    description: 'A red shirt - it is pretty red!',
    price: 29.99,
    imageUrl: 'https://via.placeholder.com/150',
  ),
  Product(
    id: 'p2',
    title: 'Blue Pants',
    description: 'A pair of blue pants.',
    price: 59.99,
    imageUrl: 'https://via.placeholder.com/150',
  ),
  // 더 많은 상품을 추가할 수 있습니다.
];
```

### 5. ProductProvider 생성
`lib/providers/products_provider.dart` 파일에 상품 관리 로직을 구현하세요.
```dart
import 'package:flutter/material.dart';
import '../models/product.dart';
import '../data/dummy_products.dart';

class ProductsProvider with ChangeNotifier {
  List<Product> _items = dummyProducts;

  List<Product> get items => [..._items];

  Product findById(String id) {
    return _items.firstWhere((product) => product.id == id);
  }
}
```

### 6. 장바구니 모델 및 Provider 생성
`lib/models/cart_item.dart`에 장바구니 항목 모델을 정의하세요.
```dart
class CartItem {
  final String id;
  final String title;
  final int quantity;
  final double price;

  CartItem({
    required this.id,
    required this.title,
    required this.quantity,
    required this.price,
  });
}
```

`lib/providers/cart_provider.dart` 파일에 장바구니 관리 로직을 구현하세요.
```dart
import 'package:flutter/material.dart';
import '../models/cart_item.dart';

class CartProvider with ChangeNotifier {
  Map<String, CartItem> _items = {};

  Map<String, CartItem> get items => {..._items};

  int get itemCount => _items.length;

  double get totalAmount {
    double total = 0.0;
    _items.forEach((key, cartItem) {
      total += cartItem.price * cartItem.quantity;
    });
    return total;
  }

  void addItem(String productId, String title, double price) {
    if (_items.containsKey(productId)) {
      _items.update(
        productId,
        (existingCartItem) => CartItem(
          id: existingCartItem.id,
          title: existingCartItem.title,
          quantity: existingCartItem.quantity + 1,
          price: existingCartItem.price,
        ),
      );
    } else {
      _items.putIfAbsent(
        productId,
        () => CartItem(
          id: DateTime.now().toString(),
          title: title,
          quantity: 1,
          price: price,
        ),
      );
    }
    notifyListeners();
  }

  void removeItem(String productId) {
    _items.remove(productId);
    notifyListeners();
  }

  void clear() {
    _items = {};
    notifyListeners();
  }
}
```

### 7. 메인 파일에서 Provider 설정
`lib/main.dart` 파일을 다음과 같이 수정하세요.
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import './providers/products_provider.dart';
import './providers/cart_provider.dart';
import './screens/products_overview_screen.dart';
import './screens/product_detail_screen.dart';
import './screens/cart_screen.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => ProductsProvider()),
        ChangeNotifierProvider(create: (_) => CartProvider()),
      ],
      child: MaterialApp(
        title: 'Simple Shopping Mall',
        theme: ThemeData(
          primarySwatch: Colors.blue,
          accentColor: Colors.orangeAccent,
        ),
        home: ProductsOverviewScreen(),
        routes: {
          ProductDetailScreen.routeName: (ctx) => ProductDetailScreen(),
          CartScreen.routeName: (ctx) => CartScreen(),
        },
      ),
    );
  }
}
```

### 8. 제품 개요 화면 구현
`lib/screens/products_overview_screen.dart` 파일을 생성하여 상품 목록을 표시하세요.
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/products_provider.dart';
import '../widgets/products_grid.dart';
import '../widgets/badge.dart';
import '../providers/cart_provider.dart';
import '../screens/cart_screen.dart';

class ProductsOverviewScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Shopping Mall'),
        actions: [
          Consumer<CartProvider>(
            builder: (_, cart, ch) => Badge(
              child: ch!,
              value: cart.itemCount.toString(),
            ),
            child: IconButton(
              icon: Icon(Icons.shopping_cart),
              onPressed: () {
                Navigator.of(context).pushNamed(CartScreen.routeName);
              },
            ),
          ),
        ],
      ),
      body: ProductsGrid(),
    );
  }
}
```

### 9. 상품 그리드 위젯 구현
`lib/widgets/products_grid.dart` 파일을 생성하여 상품 그리드를 구현하세요.
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/products_provider.dart';
import './product_item.dart';

class ProductsGrid extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final productsData = Provider.of<ProductsProvider>(context);
    final products = productsData.items;

    return GridView.builder(
      padding: const EdgeInsets.all(10.0),
      itemCount: products.length,
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        childAspectRatio: 3 / 2,
        crossAxisSpacing: 10,
        mainAxisSpacing: 10,
      ),
      itemBuilder: (ctx, i) => ChangeNotifierProvider.value(
        value: products[i],
        child: ProductItem(),
      ),
    );
  }
}
```

### 10. 상품 항목 위젯 구현
`lib/widgets/product_item.dart` 파일을 생성하여 각 상품 항목을 구현하세요.
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../models/product.dart';
import '../providers/cart_provider.dart';
import '../screens/product_detail_screen.dart';

class ProductItem extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final product = Provider.of<Product>(context, listen: false);
    final cart = Provider.of<CartProvider>(context, listen: false);

    return ClipRRect(
      borderRadius: BorderRadius.circular(10),
      child: GridTile(
        child: GestureDetector(
          onTap: () {
            Navigator.of(context).pushNamed(
              ProductDetailScreen.routeName,
              arguments: product.id,
            );
          },
          child: Image.network(
            product.imageUrl,
            fit: BoxFit.cover,
          ),
        ),
        footer: GridTileBar(
          backgroundColor: Colors.black87,
          leading: IconButton(
            icon: Icon(Icons.shopping_cart),
            color: Theme.of(context).accentColor,
            onPressed: () {
              cart.addItem(product.id, product.title, product.price);
              ScaffoldMessenger.of(context).hideCurrentSnackBar();
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(
                  content: Text('Added item to cart!'),
                  duration: Duration(seconds: 2),
                ),
              );
            },
          ),
          title: Text(
            product.title,
            textAlign: TextAlign.center,
          ),
        ),
      ),
    );
  }
}
```

### 11. 상품 상세 페이지 구현
`lib/screens/product_detail_screen.dart` 파일을 생성하여 상품 상세 페이지를 구현하세요.
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/products_provider.dart';

class ProductDetailScreen extends StatelessWidget {
  static const routeName = '/product-detail';

  @override
  Widget build(BuildContext context) {
    final productId = ModalRoute.of(context)!.settings.arguments as String;
    final loadedProduct = Provider.of<ProductsProvider>(
      context,
      listen: false,
    ).findById(productId);

    return Scaffold(
      appBar: AppBar(
        title: Text(loadedProduct.title),

알겠습니다! `lib/screens/product_detail_screen.dart` 파일을 계속해서 완성하겠습니다.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/products_provider.dart';
import '../providers/cart_provider.dart';

class ProductDetailScreen extends StatelessWidget {
  static const routeName = '/product-detail';

  @override
  Widget build(BuildContext context) {
    final productId = ModalRoute.of(context)!.settings.arguments as String;
    final loadedProduct = Provider.of<ProductsProvider>(
      context,
      listen: false,
    ).findById(productId);
    final cart = Provider.of<CartProvider>(context, listen: false);

    return Scaffold(
      appBar: AppBar(
        title: Text(loadedProduct.title),
        actions: [
          IconButton(
            icon: Icon(Icons.shopping_cart),
            onPressed: () {
              cart.addItem(loadedProduct.id, loadedProduct.title, loadedProduct.price);
              ScaffoldMessenger.of(context).hideCurrentSnackBar();
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(
                  content: Text('Item added to cart!'),
                  duration: Duration(seconds: 2),
                ),
              );
            },
          ),
        ],
      ),
      body: SingleChildScrollView(
        child: Column(
          children: [
            Container(
              height: 300,
              width: double.infinity,
              child: Image.network(
                loadedProduct.imageUrl,
                fit: BoxFit.cover,
              ),
            ),
            SizedBox(height: 10),
            Text(
              '\$${loadedProduct.price}',
              style: TextStyle(
                color: Colors.grey,
                fontSize: 20,
              ),
            ),
            SizedBox(height: 10),
            Container(
              padding: EdgeInsets.symmetric(horizontal: 10),
              width: double.infinity,
              child: Text(
                loadedProduct.description,
                textAlign: TextAlign.center,
                softWrap: true,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Nest.js 서버 실행 및 Flutter 앱 연동
이제 Nest.js 서버를 실행하고 Flutter 앱과 연동하여 전체 시스템을 완성합니다.

1. **Nest.js 서버 실행**  
   백엔드 서버를 실행합니다.

   ```bash
   npm run start
   ```

2. **Flutter 앱에서 API URL 수정**  
   `lib/providers/cart_provider.dart` 및 `lib/providers/products_provider.dart` 파일에서 `baseUrl`을 Nest.js 서버의 실제 URL로 수정하세요. 예를 들어, `localhost` 대신 실제 IP 주소나 도메인을 사용해야 합니다.

3. **Flutter 앱 실행**  
   Flutter 앱을 실행합니다.

   ```bash
   flutter run
   ```

### 최종 프로젝트 구조
Flutter와 Nest.js 프로젝트 구조는 다음과 같습니다.

**Flutter 프로젝트 구조:**

```
lib
├── data
│   └── dummy_products.dart
├── models
│   ├── cart_item.dart
│   └── product.dart
├── providers
│   ├── cart_provider.dart
│   └── products_provider.dart
├── screens
│   ├── cart_screen.dart
│   ├── product_detail_screen.dart
│   └── products_overview_screen.dart
└── widgets
    ├── badge.dart
    ├── cart_item.dart
    ├── product_item.dart
    └── products_grid.dart
```

**Nest.js 프로젝트 구조:**

```
src
├── app.module.ts
├── cart
│   ├── cart-item.entity.ts
│   ├── cart.controller.ts
│   ├── cart.module.ts
│   └── cart.service.ts
├── main.ts
└── products
    ├── product.entity.ts
    ├── products.controller.ts
    ├── products.module.ts
    └── products.service.ts
```

### 결론
이렇게 하면 Flutter 앱과 Nest.js 백엔드 서버가 연동되어 간단한 쇼핑몰 앱을 구축할 수 있습니다. 필요한 확장 기능을 추가하여 더 풍부한 기능을 구현해 보세요.
