### 완전한 프로젝트 실행을 위한 개발 환경 설정 가이드

**개발 환경 설정을 위해 필요한 기술 스택 및 버전:**

| 기술 스택     | 버전                  | 설명                                                      |
|--------------|-----------------------|----------------------------------------------------------|
| Node.js      | `v18.x.x`             | 백엔드 서버 실행을 위한 JavaScript 런타임                 |
| TypeScript   | `v5.x.x`              | 백엔드 개발 언어                                          |
| NestJS       | `v10.x.x`             | Node.js 백엔드 프레임워크                                 |
| PostgreSQL   | `v15.x.x`             | 데이터베이스                                              |
| Flutter      | `v3.10.x`             | 프론트엔드 프레임워크                                     |
| Dart         | `v3.x.x`              | Flutter 개발 언어                                         |
| Android Studio | `v2022.3.x`        | Flutter 앱 개발을 위한 IDE                               |
| Visual Studio Code | 최신 버전      | Flutter 및 NestJS 개발을 위한 경량 IDE                    |

### 백엔드(NestJS) 개발 환경 설정

#### 1. Node.js 및 NestJS 설치
- [Node.js 설치 페이지](https://nodejs.org/en/)에서 LTS 버전을 다운로드하여 설치합니다.
- NestJS CLI 설치:
```bash
npm install -g @nestjs/cli
```

#### 2. PostgreSQL 설치 및 설정
- [PostgreSQL 공식 페이지](https://www.postgresql.org/download/)에서 운영체제에 맞게 설치 및 설정합니다.
- PostgreSQL 설치 후, `psql` CLI를 사용하여 데이터베이스와 사용자 생성:
```bash
psql -U postgres
CREATE DATABASE shopping_mall;
CREATE USER yourusername WITH ENCRYPTED PASSWORD 'yourpassword';
GRANT ALL PRIVILEGES ON DATABASE shopping_mall TO yourusername;
```

#### 3. NestJS 프로젝트 설정 및 실행
- 프로젝트 생성 및 모듈 설치:
```bash
nest new shopping-mall-backend
cd shopping-mall-backend
npm install @nestjs/passport passport passport-local passport-jwt bcrypt class-validator class-transformer typeorm pg
```

- `src/app.module.ts` 수정:
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { AuthModule } from './auth/auth.module';
import { UsersModule } from './users/users.module';
import { ProductsModule } from './products/products.module';
import { CartModule } from './cart/cart.module';
import { WishlistModule } from './wishlist/wishlist.module';
import { PaymentsModule } from './payments/payments.module';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'yourusername',
      password: 'yourpassword',
      database: 'shopping_mall',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true,
    }),
    AuthModule,
    UsersModule,
    ProductsModule,
    CartModule,
    WishlistModule,
    PaymentsModule,
  ],
})
export class AppModule {}
```

- 각 모듈 및 서비스 파일 생성:
```bash
# 예시: auth 모듈
nest g mo auth
nest g co auth
nest g s auth

# 나머지 모듈도 동일하게 생성
```

- 위에서 언급된 모든 코드들을 각각의 모듈에 맞게 `src/` 디렉토리에 생성합니다.

- 프로젝트 실행:
```bash
npm run start
```

### 프론트엔드(Flutter) 개발 환경 설정

#### 1. Flutter 및 Android Studio 설치
- [Flutter 공식 설치 가이드](https://docs.flutter.dev/get-started/install)에서 운영체제에 맞게 설치합니다.
- Android Studio 설치:
  - [Android Studio 다운로드 페이지](https://developer.android.com/studio)에서 운영체제에 맞게 설치합니다.
  - Android Studio 실행 후, `Flutter` 및 `Dart` 플러그인을 설치합니다.
  - 필요한 Android SDK 및 Emulator 설정을 완료합니다.

#### 2. Flutter 프로젝트 설정 및 실행
- Flutter 프로젝트 생성 및 패키지 설치:
```bash
flutter create shopping_mall_app
cd shopping_mall_app
flutter pub add http provider flutter_secure_storage
```

- `lib/main.dart` 수정:
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'providers/auth_provider.dart';
import 'providers/product_provider.dart';
import 'providers/cart_provider.dart';
import 'providers/payment_provider.dart';
import 'screens/login_screen.dart';
import 'screens/product_list_screen.dart';

void main() {
  runApp(ShoppingMallApp());
}

class ShoppingMallApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => AuthProvider()),
        ChangeNotifierProvider(create: (_) => ProductProvider()),
        ChangeNotifierProvider(create: (_) => CartProvider()),
        ChangeNotifierProvider(create: (_) => PaymentProvider()),
      ],
      child: Consumer<AuthProvider>(
        builder: (context, authProvider, _) {
          return MaterialApp(
            title: 'Shopping Mall App',
            theme: ThemeData(primarySwatch: Colors.blue),
            home: authProvider.isAuthenticated ? ProductListScreen() : LoginScreen(),
          );
        },
      ),
    );
  }
}
```

**Flutter 프로젝트 구조 및 파일 생성**

- 프로젝트 구조에 따라 필요한 디렉토리와 파일을 생성합니다.

```bash
# models, providers, services, screens, widgets 폴더 생성
mkdir -p lib/{models,providers,services,screens,widgets}

# 각 파일 생성
touch lib/models/user.dart
touch lib/models/product.dart
touch lib/models/cart_item.dart
touch lib/models/payment.dart
touch lib/providers/auth_provider.dart
touch lib/providers/product_provider.dart
touch lib/providers/cart_provider.dart
touch lib/providers/payment_provider.dart
touch lib/services/auth_service.dart
touch lib/services/product_service.dart
touch lib/services/cart_service.dart
touch lib/services/payment_service.dart
touch lib/screens/login_screen.dart
touch lib/screens/product_list_screen.dart
touch lib/screens/product_detail_screen.dart
touch lib/screens/cart_screen.dart
touch lib/screens/payment_screen.dart
touch lib/screens/payment_success_screen.dart
touch lib/widgets/product_item.dart
touch lib/widgets/cart_item_widget.dart
```

### 각 파일 내용

**1. `lib/models/user.dart`**
```dart
class User {
  final String username;
  final String email;
  final String token;

  User({required this.username, required this.email, required this.token});

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      username: json['username'],
      email: json['email'],
      token: json['access_token'],
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'username': username,
      'email': email,
      'access_token': token,
    };
  }
}
```

**2. `lib/models/product.dart`**
```dart
class Product {
  final int id;
  final String name;
  final String description;
  final double price;
  final String imageUrl;

  Product({
    required this.id,
    required this.name,
    required this.description,
    required this.price,
    required this.imageUrl,
  });

  factory Product.fromJson(Map<String, dynamic> json) {
    return Product(
      id: json['id'],
      name: json['name'],
      description: json['description'],
      price: json['price'].toDouble(),
      imageUrl: json['imageUrl'],
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'description': description,
      'price': price,
      'imageUrl': imageUrl,
    };
  }
}
```

**3. `lib/models/cart_item.dart`**
```dart
import 'product.dart';

class CartItem {
  final int id;
  final Product product;
  final int quantity;

  CartItem({
    required this.id,
    required this.product,
    required this.quantity,
  });

  factory CartItem.fromJson(Map<String, dynamic> json) {
    return CartItem(
      id: json['id'],
      product: Product.fromJson(json['product']),
      quantity: json['quantity'],
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'product': product.toJson(),
      'quantity': quantity,
    };
  }
}
```

**4. `lib/models/payment.dart`**
```dart
class Payment {
  final int id;
  final String status;
  final double amount;
  final DateTime paymentDate;

  Payment({
    required this.id,
    required this.status,
    required this.amount,
    required this.paymentDate,
  });

  factory Payment.fromJson(Map<String, dynamic> json) {
    return Payment(
      id: json['id'],
      status: json['status'],
      amount: json['amount'].toDouble(),
      paymentDate: DateTime.parse(json['paymentDate']),
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'status': status,
      'amount': amount,
      'paymentDate': paymentDate.toIso8601String(),
    };
  }
}
```

**5. `lib/providers/auth_provider.dart`**
```dart
import 'package:flutter/material.dart';
import '../models/user.dart';
import '../services/auth_service.dart';

class AuthProvider with ChangeNotifier {
  final AuthService _authService = AuthService();
  User? _user;

  User? get user => _user;
  bool get isAuthenticated => _user != null;

  Future<void> login(String username, String password) async {
    _user = await _authService.login(username, password);
    notifyListeners();
  }

  Future<void> logout() async {
    await _authService.logout();
    _user = null;
    notifyListeners();
  }

  Future<void> tryAutoLogin() async {
    final token = await _authService.getToken();
    if (token != null) {
      _user = User(username: 'Unknown', email: 'unknown@example.com', token: token);
      notifyListeners();
    }
  }
}
```

**6. `lib/providers/product_provider.dart`**
```dart
import 'package:flutter/material.dart';
import '../models/product.dart';
import '../services/product_service.dart';

class ProductProvider with ChangeNotifier {
  final ProductService _productService = ProductService();
  List<Product> _products = [];
  Product? _selectedProduct;

  List<Product> get products => _products;
  Product? get selectedProduct => _selectedProduct;

  Future<void> fetchProducts() async {
    _products = await _productService.fetchProducts();
    notifyListeners();
  }

  Future<void> fetchProductById(int id) async {
    _selectedProduct = await _productService.fetchProductById(id);
    notifyListeners();
  }

  void clearSelectedProduct() {
    _selectedProduct = null;
    notifyListeners();
  }
}
```

**7. `lib/providers/cart_provider.dart`**
```dart
import 'package:flutter/material.dart';
import '../models/cart_item.dart';
import '../services/cart_service.dart';

class CartProvider with ChangeNotifier {
  final CartService _cartService = CartService();
  List<CartItem> _cartItems = [];

  List<CartItem> get cartItems => _cartItems;

  Future<void> fetchCartItems() async {
    _cartItems = await _cartService.fetchCartItems();
    notifyListeners();
  }

  Future<void> addToCart(int productId, int quantity) async {
    await _cartService.addToCart(productId, quantity);
    await fetchCartItems();
  }

  Future<void> removeFromCart(int productId) async {
    await _cartService.removeFromCart(productId);
    await fetchCartItems();
  }
}
```

**8. `lib/providers/payment_provider.dart`**
```dart
import 'package:flutter/material.dart';
import '../models/payment.dart';
import '../services/payment_service.dart';

class PaymentProvider with ChangeNotifier {
  final PaymentService _paymentService = PaymentService();
  List<Payment> _payments = [];

  List<Payment> get payments => _payments;

  Future<void> fetchPayments() async {
    _payments = await _paymentService.fetchPayments();
    notifyListeners();
  }

  Future<void> createPayment(double amount) async {
    await _paymentService.createPayment(amount);
    await fetchPayments();
  }
}
```

**9. `lib/services/auth_service.dart`**
```dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import '../models/user.dart';

class AuthService {
  static const String baseUrl = 'http://localhost:3000/auth';
  final FlutterSecureStorage _storage = FlutterSecureStorage();

  Future<User?> login(String username, String password) async {
    final response = await http.post(
      Uri.parse('$baseUrl/login'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode({'username': username, 'password': password}),
    );

    if (response.statusCode == 201 || response.statusCode == 200) {
      final user = User.fromJson(jsonDecode(response.body));
      await _storage.write(key: 'token', value: user.token);
      return user;
    } else {
      return null;
    }
  }

  Future<void> logout() async {
    await _storage.delete(key: 'token');
  }

  Future<String?> getToken() async {
    return await _storage.read(key: 'token');
  }
}
```

**10. `lib/services/product_service.dart`**
```dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import '../models/product.dart';
import 'auth_service.dart';

class ProductService {
  static const String baseUrl = 'http://localhost:3000/products';
  final AuthService _authService = AuthService();

  Future<List<Product>> fetchProducts() async {
    final token = await _authService.getToken();
    final response = await http.get(
      Uri.parse(baseUrl),
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer $token',
      },
    );

    if (response.statusCode == 200) {
      List<dynamic> data = jsonDecode(response.body);
      return data.map((item) => Product.fromJson(item)).toList();
    } else {
      throw Exception('Failed to load products');
    }
  }

  Future<Product?> fetchProductById(int id) async {
    final token = await _authService.getToken();
    final response = await http.get(
      Uri.parse('$baseUrl/$id'),
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer $token',
      },
    );

    if (response.statusCode == 200) {
      return Product.fromJson(jsonDecode(response.body));
    } else {
      return null;
    }
  }
}
```

**11. `lib/services/cart_service.dart`**
```dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import '../models/cart_item.dart';
import 'auth_service.dart';

class CartService {
  static const String baseUrl = 'http://localhost:3000/cart';
  final AuthService _authService = AuthService();

  Future<List<CartItem>> fetchCartItems() async {
    final token = await _authService.getToken();
    final response = await http.get(
      Uri.parse(baseUrl),
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer $token',
      },
    );

    if (response.statusCode == 200) {
      List<dynamic> data = jsonDecode(response.body);
      return data.map((item) => CartItem.fromJson(item)).toList();
    } else {
      throw Exception('Failed to load cart items');
    }
  }

  Future<void> addToCart(int productId, int quantity) async {
    final token = await _authService.getToken();
    final response = await http.post(
      Uri.parse(baseUrl),
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer $token',
      },
      body: jsonEncode({'productId': productId, 'quantity': quantity}),
    );

    if (response.statusCode != 200 && response.statusCode != 201) {
      throw Exception('Failed to add item to cart');
    }
  }

  Future<void> removeFromCart(int productId) async {
    final token = await _authService.getToken();
    final response = await http.delete(
      Uri.parse('$baseUrl/$productId'),
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer $token',
      },
    );

    if (response.statusCode != 200) {
      throw Exception('Failed to remove item from cart');
    }
  }
}
```

**12. `lib/services/payment_service.dart`**
```dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import '../models/payment.dart';
import 'auth_service.dart';

class PaymentService {
  static const String baseUrl = 'http://localhost:3000/payments';
  final AuthService _authService = AuthService();

  Future<List<Payment>> fetchPayments() async {
    final token = await _authService.getToken();
    final response = await http.get(
      Uri.parse(baseUrl),
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer $token',
      },
    );

    if (response.statusCode == 200) {
      List<dynamic> data = jsonDecode(response.body);
      return data.map((item) => Payment.fromJson(item)).toList();
    } else {
      throw Exception('Failed to load payments');
    }
  }

  Future<void> createPayment(double amount) async {
    final token = await _authService.getToken();
    final response = await http.post(
      Uri.parse(baseUrl),
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer $token',
      },
      body: jsonEncode({'amount': amount}),
    );

    if (response.statusCode != 200 && response.statusCode != 201) {
      throw Exception('Failed to create payment');
    }
  }
}
```

**13. `lib/screens/login_screen.dart`**
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/auth_provider.dart';
import 'product_list_screen.dart';

class LoginScreen extends StatefulWidget {
  @override
  _LoginScreenState createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _usernameController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _isLoading = false;
  String? _errorMessage;

  void _login() async {
    setState(() {
      _isLoading = true;
      _errorMessage = null;
    });

    final authProvider = Provider.of<AuthProvider>(context, listen: false);
    try {
      await authProvider.login(_usernameController.text, _passwordController.text);
      if (authProvider.isAuthenticated) {
        Navigator.of(context).pushReplacement(
          MaterialPageRoute(builder: (context) => ProductListScreen()),
        );
      } else {
        setState(() {
          _errorMessage = 'Invalid credentials';
        });
      }
    } catch (e) {
      setState(() {
        _errorMessage = 'An error occurred';
      });
    }

    setState(() {
      _isLoading = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            TextField(
              controller: _usernameController,
              decoration: const InputDecoration(labelText: 'Username'),
            ),
            TextField(
              controller: _passwordController,
              decoration: const InputDecoration(labelText: 'Password'),
              obscureText: true,
            ),
            if (_errorMessage != null) ...[
              const SizedBox(height: 8),
              Text(
                _errorMessage!,
                style: const TextStyle(color: Colors.red),
              ),
            ],
            const SizedBox(height: 16),
            _isLoading
                ? const CircularProgressIndicator()
                : ElevatedButton(
                    onPressed: _login,
                    child: const Text('Login'),
                  ),
          ],
        ),
      ),
    );
  }
}
```

**14. `lib/screens/product_list_screen.dart`**
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/product_provider.dart';
import '../widgets/product_item.dart';
import 'product_detail_screen.dart';

class ProductListScreen extends StatefulWidget {
  @override
  _ProductListScreenState createState() => _ProductListScreenState();
}

class _ProductListScreenState extends State<ProductListScreen> {
  bool _isLoading = true;

  @override
  void initState() {
    super.initState();
    _fetchProducts();
  }

  Future<void> _fetchProducts() async {
    await Provider.of<ProductProvider>(context, listen: false).fetchProducts();
    setState(() {
      _isLoading = false;
    });
  }

  void _openProductDetail(int id) {
    Navigator.of(context).push(
      MaterialPageRoute(builder: (context) => ProductDetailScreen(productId: id)),
    );
  }

  @override
  Widget build(BuildContext context) {
    final products = Provider.of<ProductProvider>(context).products;

    return Scaffold(
      appBar: AppBar(title: const Text('Products')),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : ListView.builder(
              itemCount: products.length,
              itemBuilder: (context, index) {
                return ProductItem(
                  product: products[index],
                  onTap: () => _openProductDetail(products[index].id),
                );
              },
            ),
    );
  }
}
```

**15. `lib/screens/product_detail_screen.dart`**
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../models/product.dart';
import '../providers/product_provider.dart';
import '../providers/cart_provider.dart';

class ProductDetailScreen extends StatefulWidget {
  final int productId;

  ProductDetailScreen({required this.productId});

  @override
  _ProductDetailScreenState createState() => _ProductDetailScreenState();
}

class _ProductDetailScreenState extends State<ProductDetailScreen> {
  bool _isLoading = true;
  Product? _product;

  @override
  void initState() {
    super.initState();
    _fetchProduct();
  }

  Future<void> _fetchProduct() async {
    await Provider.of<ProductProvider>(context, listen: false).fetchProductById(widget.productId);
    setState(() {
      _product = Provider.of<ProductProvider>(context, listen: false).selectedProduct;
      _isLoading = false;
    });
  }

  void _addToCart() {
    final cartProvider = Provider.of<CartProvider>(context, listen: false);
    cartProvider.addToCart(_product!.id, 1);
  
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(content: Text('Added to cart')),
    );
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Product Details')),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : _product == null
              ? const Center(child: Text('Product not found'))
              : Padding(
                  padding: const EdgeInsets.all(16.0),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Image.network(_product!.imageUrl),
                      const SizedBox(height: 16),
                      Text(
                        _product!.name,
                        style: const TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
                      ),
                      const SizedBox(height: 8),
                      Text(
                        '\$${_product!.price}',
                        style: const TextStyle(fontSize: 20, color: Colors.green),
                      ),
                      const SizedBox(height: 16),
                      Text(_product!.description),
                      const Spacer(),
                      ElevatedButton(
                        onPressed: _addToCart,
                        child: const Text('Add to Cart'),
                      ),
                    ],
                  ),
                ),
  );
}
```

**16. `lib/screens/cart_screen.dart`**
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/cart_provider.dart';
import '../widgets/cart_item_widget.dart';
import 'payment_screen.dart';

class CartScreen extends StatefulWidget {
  @override
  _CartScreenState createState() => _CartScreenState();
}

class _CartScreenState extends State<CartScreen> {
  bool _isLoading = true;

  @override
  void initState() {
    super.initState();
    _fetchCartItems();
  }

  Future<void> _fetchCartItems() async {
    await Provider.of<CartProvider>(context, listen: false).fetchCartItems();
    setState(() {
      _isLoading = false;
    });
  }

  void _proceedToPayment() {
    Navigator.of(context).push(
      MaterialPageRoute(builder: (context) => PaymentScreen()),
    );
  }

  @override
  Widget build(BuildContext context) {
    final cartItems = Provider.of<CartProvider>(context).cartItems;

    return Scaffold(
      appBar: AppBar(title: const Text('Cart')),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : cartItems.isEmpty
              ? const Center(child: Text('Your cart is empty'))
              : Column(
                  children: [
                    Expanded(
                      child: ListView.builder(
                        itemCount: cartItems.length,
                        itemBuilder: (context, index) {
                          return CartItemWidget(cartItem: cartItems[index]);
                        },
                      ),
                    ),
                    Padding(
                      padding: const EdgeInsets.all(16.0),
                      child: ElevatedButton(
                        onPressed: _proceedToPayment,
                        child: const Text('Proceed to Payment'),
                      ),
                    ),
                  ],
                ),
    );
  }
}
```

**17. `lib/screens/payment_screen.dart`**
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/cart_provider.dart';
import '../providers/payment_provider.dart';
import 'payment_success_screen.dart';

class PaymentScreen extends StatefulWidget {
  @override
  _PaymentScreenState createState() => _PaymentScreenState();
}

class _PaymentScreenState extends State<PaymentScreen> {
  bool _isLoading = true;

  @override
  void initState() {
    super.initState();
    _fetchCartItems();
  }

  Future<void> _fetchCartItems() async {
    await Provider.of<CartProvider>(context, listen: false).fetchCartItems();
    setState(() {
      _isLoading = false;
    });
  }

  double _calculateTotalAmount() {
    final cartItems = Provider.of<CartProvider>(context, listen: false).cartItems;
    return cartItems.fold(0.0, (total, cartItem) => total + cartItem.product.price * cartItem.quantity);
  }

  void _proceedToPayment() async {
    setState(() {
      _isLoading = true;
    });

    final totalAmount = _calculateTotalAmount();
    final paymentProvider = Provider.of<PaymentProvider>(context, listen: false);
    await paymentProvider.createPayment(totalAmount);

    setState(() {
      _isLoading = false;
    });

    Navigator.of(context).pushReplacement(
      MaterialPageRoute(builder: (context) => PaymentSuccessScreen(totalAmount: PaymentSuccessScreen(totalAmount: totalAmount)
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    final cartItems = Provider.of<CartProvider>(context).cartItems;

    return Scaffold(
      appBar: AppBar(title: const Text('Payment')),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : Column(
              children: [
                Expanded(
                  child: ListView.builder(
                    itemCount: cartItems.length,
                    itemBuilder: (context, index) {
                      final cartItem = cartItems[index];
                      return ListTile(
                        leading: Image.network(cartItem.product.imageUrl, width: 50, height: 50),
                        title: Text(cartItem.product.name),
                        subtitle: Text('Quantity: ${cartItem.quantity}'),
                        trailing: Text('\$${(cartItem.product.price * cartItem.quantity).toStringAsFixed(2)}'),
                      );
                    },
                  ),
                ),
                Padding(
                  padding: const EdgeInsets.all(16.0),
                  child: Column(
                    children: [
                      Text(
                        'Total: \$${_calculateTotalAmount().toStringAsFixed(2)}',
                        style: const TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
                      ),
                      const SizedBox(height: 16),
                      ElevatedButton(
                        onPressed: _proceedToPayment,
                        child: const Text('Pay Now'),
                      ),
                    ],
                  ),
                ),
              ],
            ),
    );
  }
}
```

**18. `lib/screens/payment_success_screen.dart`**
```dart
import 'package:flutter/material.dart';

class PaymentSuccessScreen extends StatelessWidget {
  final double totalAmount;

  PaymentSuccessScreen({required this.totalAmount});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Payment Success')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          crossAxisAlignment: CrossAxisAlignment.center,
          children: [
            const Icon(Icons.check_circle, color: Colors.green, size: 100),
            const SizedBox(height: 16),
            Text(
              'Payment of \$${totalAmount.toStringAsFixed(2)} was successful!',
              style: const TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
              textAlign: TextAlign.center,
            ),
            const SizedBox(height: 32),
            ElevatedButton(
              onPressed: () {
                Navigator.of(context).popUntil((route) => route.isFirst);
              },
              child: const Text('Back to Products'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**19. `lib/widgets/product_item.dart`**
```dart
import 'package:flutter/material.dart';
import '../models/product.dart';

class ProductItem extends StatelessWidget {
  final Product product;
  final VoidCallback onTap;

  ProductItem({required this.product, required this.onTap});

  @override
  Widget build(BuildContext context) {
    return ListTile(
      leading: Image.network(product.imageUrl, width: 50, height: 50),
      title: Text(product.name),
      subtitle: Text('\$${product.price.toStringAsFixed(2)}'),
      onTap: onTap,
    );
  }
}
```

**20. `lib/widgets/cart_item_widget.dart`**
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../models/cart_item.dart';
import '../providers/cart_provider.dart';

class CartItemWidget extends StatelessWidget {
  final CartItem cartItem;

  CartItemWidget({required this.cartItem});

  void _removeFromCart(BuildContext context) {
    final cartProvider = Provider.of<CartProvider>(context, listen: false);
    cartProvider.removeFromCart(cartItem.product.id);

    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(content: Text('Item removed from cart')),
    );
  }

  @override
  Widget build(BuildContext context) {
    return ListTile(
      leading: Image.network(cartItem.product.imageUrl, width: 50, height: 50),
      title: Text(cartItem.product.name),
      subtitle: Text('Quantity: ${cartItem.quantity}'),
      trailing: IconButton(
        icon: const Icon(Icons.delete),
        onPressed: () => _removeFromCart(context),
      ),
    );
  }
}
```

**개발 환경 설정 요약**

| 기술 스택      | 버전                | 설치 및 설정 방법                                                                                             |
|---------------|---------------------|--------------------------------------------------------------------------------------------------------------|
| Node.js       | `v18.x.x`           | [Node.js 설치 페이지](https://nodejs.org/en/)에서 LTS 버전 설치                                               |
| TypeScript    | `v5.x.x`            | `npm install -g typescript`로 설치                                                                          |
| NestJS        | `v10.x.x`           | `npm install -g @nestjs/cli`로 설치                                                                          |
| PostgreSQL    | `v15.x.x`           | [PostgreSQL 설치 페이지](https://www.postgresql.org/download/)에서 운영체제에 맞게 다운로드 및 설치         |
| Flutter       | `v3.10.x`           | [Flutter 설치 가이드](https://docs.flutter.dev/get-started/install)에서 운영체제에 맞게 다운로드 및 설치   |
| Dart          | `v3.x.x`            | Flutter 설치 시 함께 설치됨                                                                                 |
| Android Studio| `v2022.3.x`         | [Android Studio 다운로드 페이지](https://developer.android.com/studio)에서 다운로드 및 설치                |
| Visual Studio Code | 최신 버전     | [VSCode 다운로드 페이지](https://code.visualstudio.com/Download)에서 운영체제에 맞게 다운로드 및 설치      |

### 개발 환경 설정 상세 요약

| 단계          | 명령어 및 설명                                                                                                   |
|---------------|-----------------------------------------------------------------------------------------------------------------|
| **Node.js 설치** | [Node.js 설치 페이지](https://nodejs.org/en/)에서 LTS 버전 설치                                               |
| **NestJS 설치** | `npm install -g @nestjs/cli`로 NestJS CLI 설치                                                                 |
| **프로젝트 생성**| `nest new shopping-mall-backend`로 NestJS 프로젝트 생성                                                       |
| **필수 패키지 설치** | `npm install @nestjs/passport passport passport-local passport-jwt bcrypt class-validator class-transformer typeorm pg` 설치 |
| **PostgreSQL 설치** | [PostgreSQL 설치 페이지](https://www.postgresql.org/download/)에서 운영체제에 맞게 다운로드 및 설치        |
| **PostgreSQL 설정** | `psql -U postgres`로 데이터베이스에 접속 후, `CREATE DATABASE shopping_mall;` 및 사용자 생성                |
| **Flutter 설치** | [Flutter 설치 가이드](https://docs.flutter.dev/get-started/install)에서 운영체제에 맞게 다운로드 및 설치     |
| **Android Studio 설치** | [Android Studio 다운로드 페이지](https://developer.android.com/studio)에서 다운로드 및 설치           |
| **필수 패키지 설치** | `flutter pub add http provider flutter_secure_storage`로 Flutter 프로젝트에서 필요한 패키지 설치         |
| **Flutter 프로젝트 생성** | `flutter create shopping_mall_app`로 Flutter 프로젝트 생성                                            |
| **VSCode 및 플러그인 설치** | [VSCode 다운로드 페이지](https://code.visualstudio.com/Download)에서 다운로드 및 설치                |

**NestJS 프로젝트 구성**

1. **NestJS 프로젝트 생성 및 설정**

```bash
nest new shopping-mall-backend
cd shopping-mall-backend
npm install @nestjs/passport passport passport-local passport-jwt bcrypt class-validator class-transformer typeorm pg
```

2. **PostgreSQL 설정**

- PostgreSQL 설치 후, `psql` CLI를 사용하여 데이터베이스와 사용자 생성:
```sql
psql -U postgres
CREATE DATABASE shopping_mall;
CREATE USER yourusername WITH ENCRYPTED PASSWORD 'yourpassword';
GRANT ALL PRIVILEGES ON DATABASE shopping_mall TO yourusername;
```

3. **`src/app.module.ts` 수정**

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { AuthModule } from './auth/auth.module';
import { UsersModule } from './users/users.module';
import { ProductsModule } from './products/products.module';
import { CartModule } from './cart/cart.module';
import { WishlistModule } from './wishlist/wishlist.module';
import { PaymentsModule } from './payments/payments.module';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'yourusername',
      password: 'yourpassword',
      database: 'shopping_mall',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true,
    }),
    AuthModule,
    UsersModule,
    ProductsModule,
    CartModule,
    WishlistModule,
    PaymentsModule,
  ],
})
export class AppModule {}
```

4. **각 모듈 및 서비스 파일 생성**

```bash
# auth 모듈
nest g mo auth
nest g co auth
nest g s auth

# users 모듈
nest g mo users
nest g co users
nest g s users

# products 모듈
nest g mo products
nest g co products
nest g s products

# cart 모듈
nest g mo cart
nest g co cart
nest g s cart

# wishlist 모듈
nest g mo wishlist
nest g co wishlist
nest g s wishlist

# payments 모듈
nest g mo payments
nest g co payments
nest g s payments
```

5. **각 모듈의 구현 코드 작성**

**auth 모듈**

`src/auth/auth.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { PassportModule } from '@nestjs/passport';
import { JwtModule } from '@nestjs/jwt';
import { AuthService } from './auth.service';
import { LocalStrategy } from './local.strategy';
import { JwtStrategy } from './jwt.strategy';
import { UsersModule } from '../users/users.module';
import { AuthController } from './auth.controller';

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.register({
      secret: 'SECRET_KEY', // 실제로는 환경변수로 관리하세요.
      signOptions: { expiresIn: '1h' },
    }),
  ],
  providers: [AuthService, LocalStrategy, JwtStrategy],
  controllers: [AuthController],
})
export class AuthModule {}
```

`src/auth/auth.controller.ts`
```typescript
import { Controller, Post, Request, UseGuards } from '@nestjs/common';
import { AuthService } from './auth.service';
import { LocalAuthGuard } from './local-auth.guard';

@Controller('auth')
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @UseGuards(LocalAuthGuard)
  @Post('login')
  async login(@Request() req) {
    return this.authService.login(req.user);
  }
}
```

`src/auth/auth.service.ts`
```typescript
import { Injectable } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersService } from '../users/users.service';

@Injectable()
export class AuthService {
  constructor(
    private readonly usersService: UsersService,
    private readonly jwtService: JwtService,
  ) {}

  async validateUser(username: string, pass: string): Promise<any> {
    const user = await this.usersService.findOne(username);
    if (user && user.password === pass) {
      const { password, ...result } = user;
      return result;
    }
    return null;
  }

  async login(user: any) {
    const payload = { username: user.username, sub: user.userId };
    return {
      access_token: this.jwtService.sign(payload),
    };
  }
}
```

`src/auth/jwt.strategy.ts`
```typescript
import { Injectable } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: 'SECRET_KEY',
    });
  }

  async validate(payload: any) {
    return { userId: payload.sub, username: payload.username };
  }
}
```

`src/auth/local.strategy.ts`
```typescript
import { Strategy } from 'passport-local';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { AuthService } from './auth.service';

@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(private readonly authService: AuthService) {
    super();
  }

  async validate(username: string, password: string): Promise<any> {
    const user = await this.authService.validateUser(username, password);
    if (!user) {
      throw new UnauthorizedException();
    }
    return user;
  }
}
```

`src/auth/local-auth.guard.ts`
```typescript
import { Injectable, ExecutionContext } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class LocalAuthGuard extends AuthGuard('local') {
  canActivate(context: ExecutionContext) {
    return super.canActivate(context);
  }
}
```

**users 모듈**

`src/users/users.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersService } from './users.service';
import { User } from './user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

`src/users/users.service.ts`
```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}

  async findOne(username: string): Promise<User | undefined> {
    return this.userRepository.findOneBy({ username });
  }

  async findOneById(id: number): Promise<User | undefined> {
    return this.userRepository.findOneBy({ id });
  }
}
```

`src/users/user.entity.ts`
```typescript
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @Column()
  password: string;

  @Column()
  email: string;
}
```

**products 모듈**

`src/products/product.entity.ts`
```typescript
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  description: string;

  @Column('decimal')
  price: number;

  @Column()
  imageUrl: string;
}
```

`src/products/product.dto.ts`
```typescript
export class CreateProductDto {
  name: string;
  description: string;
  price: number;
  imageUrl: string;
}
```

`src/products/products.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ProductsService } from './products.service';
import { ProductsController } from './products.controller';
import { Product } from './product.entity';

@Module({
  imports: [TypeOrmModule.forFeature([Product])],
  providers: [ProductsService],
  controllers: [ProductsController],
})
export class ProductsModule {}
```

`src/products/products.service.ts`
```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Product } from './product.entity';
import { CreateProductDto } from './product.dto';

@Injectable()
export class ProductsService {
  constructor(
    @InjectRepository(Product)
    private readonly productRepository: Repository<Product>,
  ) {}

  async findAll(): Promise<Product[]> {
    return this.productRepository.find();
  }

  async findOne(id: number): Promise<Product | undefined> {
    return this.productRepository.findOneBy({ id });
  }

  async create(createProductDto: CreateProductDto): Promise<Product> {
    const newProduct = this.productRepository.create(createProductDto);
    return this.productRepository.save(newProduct);
  }
}
```

`src/products/products.controller.ts`
```typescript
import { Controller, Get, Post, Body, Param } from '@nestjs/common';
import { ProductsService } from './products.service';
import { CreateProductDto } from './product.dto';

@Controller('products')
export class ProductsController {
  constructor(private readonly productsService: ProductsService) {}

  @Get()
  async findAll() {
    return this.productsService.findAll();
  }

  @Get(':id')
  async findOne(@Param('id') id: number) {
    return this.productsService.findOne(id);
  }

  @Post()
  async create(@Body() createProductDto: CreateProductDto) {
    return this.productsService.create(createProductDto);
  }
}
```

**cart 모듈**

`src/cart/cart.entity.ts`
```typescript
import { Entity, Column, PrimaryGeneratedColumn, ManyToOne } from 'typeorm';
import { User } from '../users/user.entity';
import { Product } from '../products/product.entity';

@Entity()
export class CartItem {
  @PrimaryGeneratedColumn()
  id: number;

  @ManyToOne(() => User)
  user: User;

  @ManyToOne(() => Product)
  product: Product;

  @Column()
  quantity: number;
}
```

`src/cart/cart.dto.ts`
```typescript
export class AddToCartDto {
  productId: number;
  quantity: number;
}
```
`src/cart/cart.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { CartService } from './cart.service';
import { CartController } from './cart.controller';
import { CartItem } from './cart.entity';
import { ProductsModule } from '../products/products.module';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [
    TypeOrmModule.forFeature([CartItem]),
    ProductsModule,
    UsersModule,
  ],
  providers: [CartService],
  controllers: [CartController],
})
export class CartModule {}
```

`src/cart/cart.service.ts`
```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { CartItem } from './cart.entity';
import { AddToCartDto } from './cart.dto';
import { ProductsService } from '../products/products.service';
import { UsersService } from '../users/users.service';

@Injectable()
export class CartService {
  constructor(
    @InjectRepository(CartItem)
    private readonly cartRepository: Repository<CartItem>,
    private readonly productsService: ProductsService,
    private readonly usersService: UsersService,
  ) {}

  async findAllByUser(userId: number): Promise<CartItem[]> {
    return this.cartRepository.find({
      where: { user: { id: userId } },
      relations: ['product'],
    });
  }

  async addToCart(userId: number, addToCartDto: AddToCartDto): Promise<CartItem> {
    const product = await this.productsService.findOne(addToCartDto.productId);
    const user = await this.usersService.findOneById(userId);
    const existingItem = await this.cartRepository.findOne({
      where: { user, product },
    });

    if (existingItem) {
      existingItem.quantity += addToCartDto.quantity;
      return this.cartRepository.save(existingItem);
    }

    const newItem = this.cartRepository.create({
      user,
      product,
      quantity: addToCartDto.quantity,
    });
    return this.cartRepository.save(newItem);
  }

  async removeFromCart(userId: number, productId: number): Promise<void> {
    const user = await this.usersService.findOneById(userId);
    const product = await this.productsService.findOne(productId);
    await this.cartRepository.delete({ user, product });
  }
}
```

`src/cart/cart.controller.ts`
```typescript
import {
  Controller,
  Get,
  Post,
  Delete,
  Body,
  Param,
  UseGuards,
  Request,
} from '@nestjs/common';
import { CartService } from './cart.service';
import { JwtAuthGuard } from '../auth/jwt-auth.guard';
import { AddToCartDto } from './cart.dto';

@Controller('cart')
@UseGuards(JwtAuthGuard)
export class CartController {
  constructor(private readonly cartService: CartService) {}

  @Get()
  async findAll(@Request() req) {
    return this.cartService.findAllByUser(req.user.userId);
  }

  @Post()
  async addToCart(@Request() req, @Body() addToCartDto: AddToCartDto) {
    return this.cartService.addToCart(req.user.userId, addToCartDto);
  }

  @Delete(':productId')
  async removeFromCart(@Request() req, @Param('productId') productId: number) {
    return this.cartService.removeFromCart(req.user.userId, productId);
  }
}
```

**wishlist 모듈**

`src/wishlist/wishlist.entity.ts`
```typescript
import { Entity, Column, PrimaryGeneratedColumn, ManyToOne } from 'typeorm';
import { User } from '../users/user.entity';
import { Product } from '../products/product.entity';

@Entity()
export class WishlistItem {
  @PrimaryGeneratedColumn()
  id: number;

  @ManyToOne(() => User)
  user: User;

  @ManyToOne(() => Product)
  product: Product;
}
```

`src/wishlist/wishlist.dto.ts`
```typescript
export class AddToWishlistDto {
  productId: number;
}
```

`src/wishlist/wishlist.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { WishlistService } from './wishlist.service';
import { WishlistController } from './wishlist.controller';
import { WishlistItem } from './wishlist.entity';
import { ProductsModule } from '../products/products.module';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [
    TypeOrmModule.forFeature([WishlistItem]),
    ProductsModule,
    UsersModule,
  ],
  providers: [WishlistService],
  controllers: [WishlistController],
})
export class WishlistModule {}
```

`src/wishlist/wishlist.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { WishlistService } from './wishlist.service';
import { WishlistController } from './wishlist.controller';
import { WishlistItem } from './wishlist.entity';
import { ProductsModule } from '../products/products.module';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [
    TypeOrmModule.forFeature([WishlistItem]),
    ProductsModule,
    UsersModule,
  ],
  providers: [WishlistService],
  controllers: [WishlistController],
})
export class WishlistModule {}
```

`src/wishlist/wishlist.service.ts`
```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { WishlistItem } from './wishlist.entity';
import { AddToWishlistDto } from './wishlist.dto';
import { ProductsService } from '../products/products.service';
import { UsersService } from '../users/users.service';

@Injectable()
export class WishlistService {
  constructor(
    @InjectRepository(WishlistItem)
    private readonly wishlistRepository: Repository<WishlistItem>,
    private readonly productsService: ProductsService,
    private readonly usersService: UsersService,
  ) {}

  async findAllByUser(userId: number): Promise<WishlistItem[]> {
    return this.wishlistRepository.find({
      where: { user: { id: userId } },
      relations: ['product'],
    });
  }

  async addToWishlist(userId: number, addToWishlistDto: AddToWishlistDto): Promise<WishlistItem> {
    const product = await this.productsService.findOne(addToWishlistDto.productId);
    const user = await this.usersService.findOneById(userId);
    const existingItem = await this.wishlistRepository.findOne({
      where: { user, product },
    });

    if (!existingItem) {
      const newItem = this.wishlistRepository.create({
        user,
        product,
      });
      return this.wishlistRepository.save(newItem);
    }

    return existingItem;
  }

  async removeFromWishlist(userId: number, productId: number): Promise<void> {
    const user = await this.usersService.findOneById(userId);
    const product = await this.productsService.findOne(productId);
    await this.wishlistRepository.delete({ user, product });
  }
}
```

`src/wishlist/wishlist.controller.ts`
```typescript
import {
  Controller,
  Get,
  Post,
  Delete,
  Body,
  Param,
  UseGuards,
  Request,
} from '@nestjs/common';
import { WishlistService } from './wishlist.service';
import { JwtAuthGuard } from '../auth/jwt-auth.guard';
import { AddToWishlistDto } from './wishlist.dto';

@Controller('wishlist')
@UseGuards(JwtAuthGuard)
export class WishlistController {
  constructor(private readonly wishlistService: WishlistService) {}

  @Get()
  async findAll(@Request() req) {
    return this.wishlistService.findAllByUser(req.user.userId);
  }

  @Post()
  async addToWishlist(@Request() req, @Body() addToWishlistDto: AddToWishlistDto) {
    return this.wishlistService.addToWishlist(req.user.userId, addToWishlistDto);
  }

  @Delete(':productId')
  async removeFromWishlist(@Request() req, @Param('productId') productId: number) {
    return this.wishlistService.removeFromWishlist(req.user.userId, productId);
  }
}
```

**payments 모듈**

`src/payments/payment.entity.ts`
```typescript
import { Entity, Column, PrimaryGeneratedColumn, ManyToOne } from 'typeorm';
import { User } from '../users/user.entity';

@Entity()
export class Payment {
  @PrimaryGeneratedColumn()
  id: number;

  @ManyToOne(() => User)
  user: User;

  @Column()
  status: string;

  @Column('decimal')
  amount: number;

  @Column()
  paymentDate: Date;
}
```

`src/payments/payment.dto.ts`
```typescript
export class CreatePaymentDto {
  amount: number;
}
```

`src/payments/payments.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { PaymentsService } from './payments.service';
import { PaymentsController } from './payments.controller';
import { Payment } from './payment.entity';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [TypeOrmModule.forFeature([Payment]), UsersModule],
  providers: [PaymentsService],
  controllers: [PaymentsController],
})
export class PaymentsModule {}
```

`src/payments/payments.service.ts`
```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Payment } from './payment.entity';
import { CreatePaymentDto } from './payment.dto';
import { UsersService } from '../users/users.service';

@Injectable()
export class PaymentsService {
  constructor(
    @InjectRepository(Payment)
    private readonly paymentRepository: Repository<Payment>,
    private readonly usersService: UsersService,
  ) {}

  async createPayment(userId: number, createPaymentDto: CreatePaymentDto): Promise<Payment> {
    const user = await this.usersService.findOneById(userId);
    const payment = this.paymentRepository.create({
      user,
      amount: createPaymentDto.amount,
      status: 'Pending',
      paymentDate: new Date(),
    });
    return this.paymentRepository.save(payment);
  }

  async findPaymentsByUser(userId: number): Promise<Payment[]> {
    return this.paymentRepository.find({
      where: { user: { id: userId } },
    });
  }

  async updatePaymentStatus(paymentId: number, status: string): Promise<Payment> {
    const payment = await this.paymentRepository.findOneBy({ id: paymentId });
    if (payment) {
      payment.status = status;
      return this.paymentRepository.save(payment);
    }
    return null;
  }
}
```

`src/payments/payments.controller.ts`
```typescript
import {
  Controller,
  Get,
  Post,
  Param,
  Body,
  UseGuards,
  Request,
  Patch,
} from '@nestjs/common';
import { PaymentsService } from './payments.service';
import { JwtAuthGuard } from '../auth/jwt-auth.guard';
import { CreatePaymentDto } from './payment.dto';

@Controller('payments')
@UseGuards(JwtAuthGuard)
export class PaymentsController {
  constructor(private readonly paymentsService: PaymentsService) {}

  @Get()
  async findUserPayments(@Request() req) {
    return this.paymentsService.findPaymentsByUser(req.user.userId);
  }

  @Post()
  async createPayment(@Request() req, @Body() createPaymentDto: CreatePaymentDto) {
    return this.paymentsService.createPayment(req.user.userId, createPaymentDto);
  }

  @Patch(':paymentId/status')
  async updatePaymentStatus(
    @Param('paymentId') paymentId: number,
    @Body('status') status: string,
  ) {
    return this.paymentsService.updatePaymentStatus(paymentId, status);
  }
}
```

**app.module.ts 및 기타 파일**

`src/app.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { AuthModule } from './auth/auth.module';
import { UsersModule } from './users/users.module';
import { ProductsModule } from './products/products.module';
import { CartModule } from './cart/cart.module';
import { WishlistModule } from './wishlist/wishlist.module';
import { PaymentsModule } from './payments/payments.module';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'yourusername',
      password: 'yourpassword',
      database: 'shopping_mall',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true,
    }),
    AuthModule,
    UsersModule,
    ProductsModule,
    CartModule,
    WishlistModule,
    PaymentsModule,
  ],
})
export class AppModule {}
```

`src/auth/jwt-auth.guard.ts`
```typescript
import { Injectable, ExecutionContext } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  canActivate(context: ExecutionContext) {
    return super.canActivate(context);
  }
}
```

`src/main.ts`
```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors(); // CORS 설정
  await app.listen(3000);
}
bootstrap();
```

**백엔드 실행**

```bash
npm run start
```

**프론트엔드(Flutter) 프로젝트 구성**

1. **Flutter 프로젝트 생성 및 패키지 설치**

```bash
flutter create shopping_mall_app
cd shopping_mall_app
flutter pub add http provider flutter_secure_storage
```

2. **Flutter 프로젝트 구조 및 파일 생성**

```bash
# models, providers, services, screens, widgets 폴더 생성
mkdir -p lib/{models,providers,services,screens,widgets}

# 각 파일 생성
touch lib/models/user.dart
touch lib/models/product.dart

