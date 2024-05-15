쇼핑몰 앱을 Flutter로 만들 때, 서버와 통신하는 과정에서 DTO와 레포지토리 패턴을 적용하고, 상태 관리를 위해 Provider 패턴을 적용하는 방법을 살펴보겠습니다. 여기에서는 간단한 상품 목록을 가져오는 예제로 설명하겠습니다.

### 프로젝트 구조
```
lib/
├── data/
│   ├── models/
│   │   └── product_dto.dart
│   └── repositories/
│       └── product_repository.dart
├── providers/
│   └── product_provider.dart
└── main.dart
```

### 1. DTO (Data Transfer Object)
서버와 통신할 때 데이터를 전송하거나 수신하는 데 사용되는 객체입니다.

**`product_dto.dart`**
```dart
// lib/data/models/product_dto.dart
class ProductDTO {
  final int id;
  final String name;
  final double price;

  ProductDTO({
    required this.id,
    required this.name,
    required this.price,
  });

  factory ProductDTO.fromJson(Map<String, dynamic> json) {
    return ProductDTO(
      id: json['id'],
      name: json['name'],
      price: json['price'],
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'price': price,
    };
  }
}
```

### 2. Repository
데이터를 가져오는 로직을 캡슐화합니다. 여기서는 HTTP 요청을 통해 서버에서 데이터를 가져옵니다.

**`product_repository.dart`**
```dart
// lib/data/repositories/product_repository.dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import '../models/product_dto.dart';

class ProductRepository {
  final String baseUrl;

  ProductRepository({required this.baseUrl});

  Future<List<ProductDTO>> fetchProducts() async {
    final response = await http.get(Uri.parse('$baseUrl/products'));

    if (response.statusCode == 200) {
      List<dynamic> data = json.decode(response.body);
      return data.map((json) => ProductDTO.fromJson(json)).toList();
    } else {
      throw Exception('Failed to load products');
    }
  }
}
```

### 3. Provider
어플리케이션 상태 관리에 사용됩니다. 여기서는 상품 목록을 관리하는데 사용됩니다.

**`product_provider.dart`**
```dart
// lib/providers/product_provider.dart
import 'package:flutter/material.dart';
import '../data/models/product_dto.dart';
import '../data/repositories/product_repository.dart';

class ProductProvider with ChangeNotifier {
  final ProductRepository repository;
  List<ProductDTO> _products = [];
  bool _isLoading = false;

  ProductProvider({required this.repository});

  List<ProductDTO> get products => _products;
  bool get isLoading => _isLoading;

  Future<void> fetchProducts() async {
    _isLoading = true;
    notifyListeners();

    try {
      _products = await repository.fetchProducts();
    } catch (e) {
      // 에러 처리 로직
      print(e);
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }
}
```

### 4. Flutter UI
Flutter UI와 Provider를 연결하여 상품 목록을 표시합니다.

**`main.dart`**
```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'data/repositories/product_repository.dart';
import 'providers/product_provider.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(
          create: (_) => ProductProvider(
            repository: ProductRepository(baseUrl: 'https://example.com/api'),
          ),
        ),
      ],
      child: MaterialApp(
        title: 'Shopping Mall',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: ProductListPage(),
      ),
    );
  }
}

class ProductListPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final productProvider = Provider.of<ProductProvider>(context);

    return Scaffold(
      appBar: AppBar(
        title: Text('Product List'),
      ),
      body: productProvider.isLoading
          ? Center(child: CircularProgressIndicator())
          : ListView.builder(
              itemCount: productProvider.products.length,
              itemBuilder: (context, index) {
                final product = productProvider.products[index];
                return ListTile(
                  title: Text(product.name),
                  subtitle: Text('\$${product.price}'),
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.refresh),
        onPressed: () {
          productProvider.fetchProducts();
        },
      ),
    );
  }
}
```

### 전체 코드 구조

위의 모든 코드를 종합하여 프로젝트 구조를 다시 확인하고, 각 파일에 필요한 코드를 정리해 보겠습니다.

**프로젝트 디렉토리 구조**
```
lib/
├── data/
│   ├── models/
│   │   └── product_dto.dart
│   └── repositories/
│       └── product_repository.dart
├── providers/
│   └── product_provider.dart
└── main.dart
```

**`product_dto.dart`**
```dart
// lib/data/models/product_dto.dart
class ProductDTO {
  final int id;
  final String name;
  final double price;

  ProductDTO({
    required this.id,
    required this.name,
    required this.price,
  });

  factory ProductDTO.fromJson(Map<String, dynamic> json) {
    return ProductDTO(
      id: json['id'],
      name: json['name'],
      price: json['price'],
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'price': price,
    };
  }
}
```

**`product_repository.dart`**
```dart
// lib/data/repositories/product_repository.dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import '../models/product_dto.dart';

class ProductRepository {
  final String baseUrl;

  ProductRepository({required this.baseUrl});

  Future<List<ProductDTO>> fetchProducts() async {
    final response = await http.get(Uri.parse('$baseUrl/products'));

    if (response.statusCode == 200) {
      List<dynamic> data = json.decode(response.body);
      return data.map((json) => ProductDTO.fromJson(json)).toList();
    } else {
      throw Exception('Failed to load products');
    }
  }
}
```

**`product_provider.dart`**
```dart
// lib/providers/product_provider.dart
import 'package:flutter/material.dart';
import '../data/models/product_dto.dart';
import '../data/repositories/product_repository.dart';

class ProductProvider with ChangeNotifier {
  final ProductRepository repository;
  List<ProductDTO> _products = [];
  bool _isLoading = false;

  ProductProvider({required this.repository});

  List<ProductDTO> get products => _products;
  bool get isLoading => _isLoading;

  Future<void> fetchProducts() async {
    _isLoading = true;
    notifyListeners();

    try {
      _products = await repository.fetchProducts();
    } catch (e) {
      // 에러 처리 로직
      print(e);
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }
}
```

**`main.dart`**
```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'data/repositories/product_repository.dart';
import 'providers/product_provider.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(
          create: (_) => ProductProvider(
            repository: ProductRepository(baseUrl: 'https://example.com/api'),
          ),
        ),
      ],
      child: MaterialApp(
        title: 'Shopping Mall',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: ProductListPage(),
      ),
    );
  }
}

class ProductListPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final productProvider = Provider.of<ProductProvider>(context);

    return Scaffold(
      appBar: AppBar(
        title: Text('Product List'),
      ),
      body: productProvider.isLoading
          ? Center(child: CircularProgressIndicator())
          : ListView.builder(
              itemCount: productProvider.products.length,
              itemBuilder: (context, index) {
                final product = productProvider.products[index];
                return ListTile(
                  title: Text(product.name),
                  subtitle: Text('\$${product.price}'),
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.refresh),
        onPressed: () {
          productProvider.fetchProducts();
        },
      ),
    );
  }
}
```

### 코드 설명

1. **DTO (Data Transfer Object):** `product_dto.dart`에서 서버로부터 받은 JSON 데이터를 Flutter 애플리케이션에서 사용하기 쉽도록 `ProductDTO` 클래스로 변환합니다.

2. **Repository:** `product_repository.dart`에서 서버와 통신하는 로직을 캡슐화하여 `ProductRepository` 클래스로 제공합니다. 이 클래스는 서버 API 엔드포인트로부터 상품 목록을 가져오는 `

3. **Provider:** `product_provider.dart`에서 `ProductProvider` 클래스를 통해 앱의 상태를 관리합니다. 이 클래스는 `ChangeNotifier`를 상속하며, 내부적으로 `ProductRepository`를 통해 상품 목록을 가져옵니다. `fetchProducts` 메서드 호출 시 로딩 상태를 관리하고, 데이터가 업데이트되면 `notifyListeners`를 호출하여 UI 컴포넌트에 변경 사항을 알립니다.

4. **Flutter UI:** `main.dart`에서 Provider 패턴을 적용한 `ProductListPage` 위젯을 통해 상품 목록을 표시합니다. `ProductProvider`의 상태에 따라 로딩 중에는 로딩 스피너를, 데이터가 준비되면 상품 목록을 보여주는 UI를 구성합니다.

### 실행 방법
1. `main.dart`의 `ProductRepository`에서 `baseUrl` 값을 실제 서버 API URL로 변경합니다.
2. 필요한 패키지를 설치합니다.
   ```
   flutter pub add provider http
   ```
3. 앱을 실행합니다.

### 전체 코드
**`product_dto.dart`**
```dart
// lib/data/models/product_dto.dart
class ProductDTO {
  final int id;
  final String name;
  final double price;

  ProductDTO({
    required this.id,
    required this.name,
    required this.price,
  });

  factory ProductDTO.fromJson(Map<String, dynamic> json) {
    return ProductDTO(
      id: json['id'],
      name: json['name'],
      price: json['price'],
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'price': price,
    };
  }
}
```

**`product_repository.dart`**
```dart
// lib/data/repositories/product_repository.dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import '../models/product_dto.dart';

class ProductRepository {
  final String baseUrl;

  ProductRepository({required this.baseUrl});

  Future<List<ProductDTO>> fetchProducts() async {
    final response = await http.get(Uri.parse('$baseUrl/products'));

    if (response.statusCode == 200) {
      List<dynamic> data = json.decode(response.body);
      return data.map((json) => ProductDTO.fromJson(json)).toList();
    } else {
      throw Exception('Failed to load products');
    }
  }
}
```

**`product_provider.dart`**
```dart
// lib/providers/product_provider.dart
import 'package:flutter/material.dart';
import '../data/models/product_dto.dart';
import '../data/repositories/product_repository.dart';

class ProductProvider with ChangeNotifier {
  final ProductRepository repository;
  List<ProductDTO> _products = [];
  bool _isLoading = false;

  ProductProvider({required this.repository});

  List<ProductDTO> get products => _products;
  bool get isLoading => _isLoading;

  Future<void> fetchProducts() async {
    _isLoading = true;
    notifyListeners();

    try {
      _products = await repository.fetchProducts();
    } catch (e) {
      // 에러 처리 로직
      print(e);
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }
}
```

**`main.dart`**
```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'data/repositories/product_repository.dart';
import 'providers/product_provider.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(
          create: (_) => ProductProvider(
            repository: ProductRepository(baseUrl: 'https://example.com/api'),
          ),
        ),
      ],
      child: MaterialApp(
        title: 'Shopping Mall',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: ProductListPage(),
      ),
    );
  }
}

class ProductListPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final productProvider = Provider.of<ProductProvider>(context);

    return Scaffold(
      appBar: AppBar(
        title: Text('Product List'),
      ),
      body: productProvider.isLoading
          ? Center(child: CircularProgressIndicator())
          : ListView.builder(
              itemCount: productProvider.products.length,
              itemBuilder: (context, index) {
                final product = productProvider.products[index];
               return ListTile(
                  title: Text(product.name),
                  subtitle: Text('\$${product.price}'),
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.refresh),
        onPressed: () {
          productProvider.fetchProducts();
        },
      ),
    );
  }
}
```

### 전체 코드 구조 및 설명
프로젝트 구조와 각 파일의 내용을 다시 정리하면 아래와 같습니다.

**프로젝트 디렉토리 구조**
```
lib/
├── data/
│   ├── models/
│   │   └── product_dto.dart
│   └── repositories/
│       └── product_repository.dart
├── providers/
│   └── product_provider.dart
└── main.dart
```

**`product_dto.dart`**
```dart
// lib/data/models/product_dto.dart
class ProductDTO {
  final int id;
  final String name;
  final double price;

  ProductDTO({
    required this.id,
    required this.name,
    required this.price,
  });

  factory ProductDTO.fromJson(Map<String, dynamic> json) {
    return ProductDTO(
      id: json['id'],
      name: json['name'],
      price: json['price'],
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'price': price,
    };
  }
}
```

**`product_repository.dart`**
```dart
// lib/data/repositories/product_repository.dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import '../models/product_dto.dart';

class ProductRepository {
  final String baseUrl;

  ProductRepository({required this.baseUrl});

  Future<List<ProductDTO>> fetchProducts() async {
    final response = await http.get(Uri.parse('$baseUrl/products'));

    if (response.statusCode == 200) {
      List<dynamic> data = json.decode(response.body);
      return data.map((json) => ProductDTO.fromJson(json)).toList();
    } else {
      throw Exception('Failed to load products');
    }
  }
}
```

**`product_provider.dart`**
```dart
// lib/providers/product_provider.dart
import 'package:flutter/material.dart';
import '../data/models/product_dto.dart';
import '../data/repositories/product_repository.dart';

class ProductProvider with ChangeNotifier {
  final ProductRepository repository;
  List<ProductDTO> _products = [];
  bool _isLoading = false;

  ProductProvider({required this.repository});

  List<ProductDTO> get products => _products;
  bool get isLoading => _isLoading;

  Future<void> fetchProducts() async {
    _isLoading = true;
    notifyListeners();

    try {
      _products = await repository.fetchProducts();
    } catch (e) {
      // 에러 처리 로직
      print(e);
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }
}
```

**`main.dart`**
```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'data/repositories/product_repository.dart';
import 'providers/product_provider.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(
          create: (_) => ProductProvider(
            repository: ProductRepository(baseUrl: 'https://example.com/api'),
          ),
        ),
      ],
      child: MaterialApp(
        title: 'Shopping Mall',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: ProductListPage(),
      ),
    );
  }
}

class ProductListPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final productProvider = Provider.of<ProductProvider>(context);

    return Scaffold(
      appBar: AppBar(
        title: Text('Product List'),
      ),
      body: productProvider.isLoading
          ? Center(child: CircularProgressIndicator())
          : ListView.builder(
              itemCount: productProvider.products.length,
              itemBuilder: (context, index) {
                final product = productProvider.products[index];
                return ListTile(
                  title: Text(product.name),
                  subtitle: Text('\$${product.price}'),
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.refresh),
        onPressed: () {
          productProvider.fetchProducts();
        },
      ),
    );
  }
}
```

### 요약
- **DTO 패턴:** 서버에서 받은 데이터를 애플리케이션에서 쉽게 사용할 수 있는 형태로 변환합니다.
- **Repository 패턴:** 데이터를 가져오는 로직을 캡슐화하여 중앙 집중화합니다.  
- **Provider 패턴:** 앱 상태를 관리하기 위해 `ChangeNotifier`를 상속한 클래스를 사용하고, 이를 통해 UI와 데이터를 쉽게 연결합니다. `ProductProvider`는 `ProductRepository`를 이용해 상품 데이터를 가져오고, 필요한 경우 `notifyListeners`를 호출하여 UI를 갱신합니다.

### 각 패턴의 역할
1. **DTO 패턴**: 서버에서 받은 데이터를 애플리케이션에서 쉽게 다루기 위해 `ProductDTO` 클래스로 변환합니다.
2. **Repository 패턴**: 데이터 접근 로직을 중앙 집중화하고, `ProductRepository` 클래스를 통해 서버와 통신합니다.
3. **Provider 패턴**: 앱 상태 관리를 위해 `ProductProvider` 클래스를 사용하고, 이를 Flutter의 `Provider` 패키지와 함께 사용하여 UI와 데이터를 연결합니다.

### 실행 방법
1. `main.dart`의 `ProductRepository`에서 `baseUrl` 값을 실제 서버 API URL로 변경합니다.
2. 필요한 패키지를 설치합니다.
   ```
   flutter pub add provider http
   ```
3. 앱을 실행합니다.

### Result
이렇게 만든 앱은 `ProductListPage`에서 상품 목록을 보여주며, 데이터를 새로고침할 때마다 서버 API로부터 최신 데이터를 가져옵니다. 

`FloatingActionButton`을 눌러 데이터를 새로고침할 수 있고, 로딩 중일 때는 `CircularProgressIndicator`로 로딩 상태를 표시합니다.

### 추후 개선 사항
1. **에러 처리**: 네트워크 오류나 서버 오류에 대한 에러 처리 로직을 추가합니다.
2. **캐싱**: 데이터가 빈번하게 변경되지 않는 경우 로컬 캐싱을 사용하여 네트워크 요청을 줄일 수 있습니다.
3. **검색 및 필터링 기능**: 상품 목록에 검색 및 필터링 기능을 추가하여 사용자에게 더 나은 경험을 제공할 수 있습니다.

### 마무리
이 예제는 Flutter에서 쇼핑몰 앱을 만들 때 DTO, Repository, Provider 패턴을 적용하는 기본적인 방법을 보여줍니다. 이를 기반으로 실제 프로젝트에 맞게 확장하고 개선하여 사용할 수 있습니다.
