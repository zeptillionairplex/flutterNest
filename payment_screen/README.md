### 개발 환경 설정
**1. Flutter 환경 설정**
- Flutter 설치: [공식 설치 가이드](https://docs.flutter.dev/get-started/install)
- Flutter 버전: `>=3.10.0`
- Android Studio 설치 및 Flutter 플러그인 추가
- Flutter SDK 경로 설정
```bash
$ flutter doctor
```
- 필요한 추가 패키지 설치
```bash
$ flutter pub get
```

**2. Node.js 및 Nest.js 환경 설정**
- Node.js 설치: [공식 사이트](https://nodejs.org/)
- Node.js 버전: `>=18.0.0`
- Nest.js CLI 설치
```bash
$ npm install -g @nestjs/cli
```
- MySQL 설치
  - MacOS: `brew install mysql`
  - Windows: [MySQL Installer](https://dev.mysql.com/downloads/installer/)

**3. Nest.js 프로젝트 생성 및 MySQL 연결**
- 프로젝트 생성
```bash
$ nest new shopping-mall-backend
```
- 프로젝트 디렉토리로 이동
```bash
$ cd shopping-mall-backend
```
- MySQL 데이터베이스 연결을 위한 TypeORM 설치
```bash
$ npm install --save @nestjs/typeorm typeorm mysql2
```

### 백엔드 코드 작성
**1. `src/app.module.ts`에서 MySQL 연결 설정**
```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ProductModule } from './product/product.module';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'your_password',
      database: 'shopping_mall',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true,
    }),
    ProductModule,
  ],
})
export class AppModule {}
```

**2. 상품(Product) 모듈 생성**
```bash
$ nest generate module product
$ nest generate service product
$ nest generate controller product
```

**3. 상품 엔티티(Entity) 생성**
```typescript
// src/product/product.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('decimal')
  price: number;

  @Column()
  description: string;

  @Column()
  imageUrl: string;
}
```

**4. 상품 레포지토리(Repository) 생성**
```typescript
// src/product/product.repository.ts
import { EntityRepository, Repository } from 'typeorm';
import { Product } from './product.entity';

@EntityRepository(Product)
export class ProductRepository extends Repository<Product> {}
```

**5. 상품 서비스(Service) 구현**
```typescript
// src/product/product.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { ProductRepository } from './product.repository';
import { Product } from './product.entity';

@Injectable()
export class ProductService {
  constructor(
    @InjectRepository(ProductRepository)
    private productRepository: ProductRepository,
  ) {}

  async findAll(): Promise<Product[]> {
    return this.productRepository.find();
  }

  async findOne(id: number): Promise<Product> {
    return this.productRepository.findOne(id);
  }

  async create(product: Product): Promise<Product> {
    return this.productRepository.save(product);
  }

  async update(id: number, product: Product): Promise<Product> {
    await this.productRepository.update(id, product);
    return this.productRepository.findOne(id);
  }

  async delete(id: number): Promise<void> {
    await this.productRepository.delete(id);
  }
}
```

**6. 상품 컨트롤러(Controller) 구현**
```typescript
// src/product/product.controller.ts
import { Controller, Get, Post, Put, Delete, Param, Body } from '@nestjs/common';
import { ProductService } from './product.service';
import { Product } from './product.entity';

@Controller('products')
export class ProductController {
  constructor(private readonly productService: ProductService) {}

  @Get()
  async findAll(): Promise<Product[]> {
    return this.productService.findAll();
  }

  @Get(':id')
  async findOne(@Param('id') id: number): Promise<Product> {
    return this.productService.findOne(id);
  }

  @Post()
  async create(@Body() product: Product): Promise<Product> {
    return this.productService.create(product);
  }

  @Put(':id')
  async update(@Param('id') id: number, @Body() product: Product): Promise<Product> {
    return this.productService.update(id, product);
  }

  @Delete(':id')
  async delete(@Param('id') id: number): Promise<void> {
    return this.productService.delete(id);
  }
}
```

**7. 상품 모듈에서 서비스 및 레포지토리 등록**
```typescript
// src/product/product.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ProductService } from './product.service';
import { ProductController } from './product.controller';
import { ProductRepository } from './product.repository';

@Module({
  imports: [TypeOrmModule.forFeature([ProductRepository])],
  providers: [ProductService],
  controllers: [ProductController],
})
export class ProductModule {}
```

### 프런트엔드 코드 작성
**1. Flutter 프로젝트 생성**
```bash
$ flutter create shopping_mall_frontend
```

**2. 프로젝트 디렉토리로 이동**
```bash
$ cd shopping_mall_frontend
```

**3. 필요한 패키지 설치**
`pubspec.yaml` 파일에 필요한 패키지 추가:
```yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.0.5
  http: ^0.14.0
  json_annotation: ^4.8.0
  flutter_dotenv: ^5.0.2
  fluttertoast: ^8.2.1
```

**4. 환경 변수 설정**
- 프로젝트 루트에 `.env` 파일 생성:
```env
API_BASE_URL=http://localhost:3000
```

- `pubspec.yaml`에 `flutter_dotenv` 관련 설정 추가:
```yaml
flutter:
  assets:
    - .env
```

**5. 모델 클래스 작성**
- 디렉토리 `lib/models` 생성 후 `product.dart` 파일 작성:
```dart
// lib/models/product.dart
import 'package:json_annotation/json_annotation.dart';

part 'product.g.dart';

@JsonSerializable()
class Product {
  int? id;
  String name;
  double price;
  String description;
  String imageUrl;

  Product({this.id, required this.name, required this.price, required this.description, required this.imageUrl});

  factory Product.fromJson(Map<String, dynamic> json) => _$ProductFromJson(json);

  Map<String, dynamic> toJson() => _$ProductToJson(this);
}
```

- `product.g.dart` 파일 생성:
```bash
$ flutter pub run build_runner build
```

**6. API 서비스 작성**
- `lib/services` 디렉토리 생성 후 `product_service.dart` 파일 작성:
```dart
// lib/services/product_service.dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import 'package:flutter_dotenv/flutter_dotenv.dart';
import '../models/product.dart';

class ProductService {
  final String baseUrl = dotenv.env['API_BASE_URL']!;

  Future<List<Product>> fetchProducts() async {
    final response = await http.get(Uri.parse('$baseUrl/products'));

    if (response.statusCode == 200) {
      List<dynamic> data = json.decode(response.body);
      return data.map((item) => Product.fromJson(item)).toList();
    } else {
      throw Exception('Failed to load products');
    }
  }

  Future<Product> fetchProduct(int id) async {
    final response = await http.get(Uri.parse('$baseUrl/products/$id'));

    if (response.statusCode == 200) {
      return Product.fromJson(json.decode(response.body));
    } else {
      throw Exception('Failed to load product');
    }
  }

  Future<Product> createProduct(Product product) async {
    final response = await http.post(
      Uri.parse('$baseUrl/products'),
      headers: {'Content-Type': 'application/json'},
      body: json.encode(product.toJson()),
    );

    if (response.statusCode == 201) {
      return Product.fromJson(json.decode(response.body));
    } else {
      throw Exception('Failed to create product');
    }
  }

  Future<Product> updateProduct(int id, Product product) async {
    final response = await http.put(
      Uri.parse('$baseUrl/products/$id'),
      headers: {'Content-Type': 'application/json'},
      body: json.encode(product.toJson()),
    );

    if (response.statusCode == 200) {
      return Product.fromJson(json.decode(response.body));
    } else {
      throw Exception('Failed to update product');
    }
  }

  Future<void> deleteProduct(int id) async {
    final response = await http.delete(Uri.parse('$baseUrl/products/$id'));

    if (response.statusCode != 200) {
      throw Exception('Failed to delete product');
    }
  }
}
```

**7. 상태 관리 Provider 작성**

- `lib/providers` 디렉토리 생성 후 `product_provider.dart` 파일 작성:

```dart
// lib/providers/product_provider.dart
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
    try {
      _products = await _productService.fetchProducts();
      notifyListeners();
    } catch (e) {
      print('Error fetching products: $e');
    }
  }

  Future<void> fetchProduct(int id) async {
    try {
      _selectedProduct = await _productService.fetchProduct(id);
      notifyListeners();
    } catch (e) {
      print('Error fetching product: $e');
    }
  }

  Future<void> createProduct(Product product) async {
    try {
      final newProduct = await _productService.createProduct(product);
      _products.add(newProduct);
      notifyListeners();
    } catch (e) {
      print('Error creating product: $e');
    }
  }

  Future<void> updateProduct(int id, Product product) async {
    try {
      final updatedProduct = await _productService.updateProduct(id, product);
      final index = _products.indexWhere((p) => p.id == id);
      if (index != -1) {
        _products[index] = updatedProduct;
        notifyListeners();
      }
    } catch (e) {
      print('Error updating product: $e');
    }
  }

  Future<void> deleteProduct(int id) async {
    try {
      await _productService.deleteProduct(id);
      _products.removeWhere((p) => p.id == id);
      notifyListeners();
    } catch (e) {
      print('Error deleting product: $e');
    }
  }
}
```

**8. 메인 페이지 작성**

- `lib/pages` 디렉토리 생성 후 `main_page.dart` 파일 작성:

```dart
// lib/pages/main_page.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/product_provider.dart';
import 'product_detail_page.dart';

class MainPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Shopping Mall')),
      body: Consumer<ProductProvider>(
        builder: (context, productProvider, child) {
          return FutureBuilder(
            future: productProvider.fetchProducts(),
            builder: (context, snapshot) {
              if (snapshot.connectionState == ConnectionState.waiting) {
                return Center(child: CircularProgressIndicator());
              } else {
                return ListView.builder(
                  itemCount: productProvider.products.length,
                  itemBuilder: (context, index) {
                    final product = productProvider.products[index];
                    return ListTile(
                      title: Text(product.name),
                      subtitle: Text('\$${product.price}'),
                      onTap: () {
                        Navigator.push(
                          context,
                          MaterialPageRoute(
                            builder: (context) => ProductDetailPage(productId: product.id!),
                          ),
                        );
                      },
                    );
                  },
                );
              }
            },
          );
        },
      ),
    );
  }
}
```

**9. 상품 상세 페이지 작성**

- `lib/pages/product_detail_page.dart` 파일 작성:

```dart
// lib/pages/product_detail_page.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/product_provider.dart';
import '../models/product.dart';

class ProductDetailPage extends StatelessWidget {
  final int productId;

  const ProductDetailPage({Key? key, required this.productId}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Product Detail')),
      body: Consumer<ProductProvider>(
        builder: (context, productProvider, child) {
          return FutureBuilder(
            future: productProvider.fetchProduct(productId),
            builder: (context, snapshot) {
              if (snapshot.connectionState == ConnectionState.waiting) {
                return Center(child: CircularProgressIndicator());
              } else if (snapshot.hasError) {
                return Center(child: Text('Error loading product'));
              } else {
                final product = productProvider.selectedProduct;
                if (product == null) {
                  return Center(child: Text('Product not found'));
                }

                return Padding(
                  padding: const EdgeInsets.all(16.0),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Image.network(product.imageUrl),
                      SizedBox(height: 16),
                      Text(
                        product.name,
                        style: Theme.of(context).textTheme.headline5,
                      ),
                      SizedBox(height: 8),
                      Text(
                        '\$${product.price}',
                        style: Theme.of(context).textTheme.headline6,
                      ),
                      SizedBox(height: 16),
                      Text(
                        product.description,
                        style: Theme.of(context).textTheme.bodyText1,
                      ),
                      SizedBox(height: 32),
                      ElevatedButton(
                        onPressed: () {
                          // TODO: Add to cart functionality
                          ScaffoldMessenger.of(context).showSnackBar(
                            SnackBar(content: Text('${product.name} added to cart!')),
                          );
                        },
                        child: Text('Add to Cart'),
                      ),
                    ],
                  ),
                );
              }
            },
          );
        },
      ),
    );
  }
}
```

**10. 쇼핑 카트 페이지 작성**

- `lib/pages/cart_page.dart` 파일 작성:

```dart
// lib/pages/cart_page.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/cart_provider.dart';

class CartPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Shopping Cart')),
      body: Consumer<CartProvider>(
        builder: (context, cartProvider, child) {
          if (cartProvider.items.isEmpty) {
            return Center(child: Text('Your cart is empty!'));
          }

          return ListView.builder(
            itemCount: cartProvider.items.length,
            itemBuilder: (context, index) {
              final item = cartProvider.items[index];
              return ListTile(
                title: Text(item.product.name),
                subtitle: Text('\$${item.product.price} x ${item.quantity}'),
                trailing: IconButton(
                  icon: Icon(Icons.remove_shopping_cart),
                  onPressed: () => cartProvider.removeFromCart(item.product.id!),
                ),
              );
            },
          );
        },
      ),
      bottomNavigationBar: Consumer<CartProvider>(
        builder: (context, cartProvider, child) {
          return Padding(
            padding: const EdgeInsets.all(8.0),
            child: ElevatedButton(
              onPressed: () {
                // TODO: Navigate to payment page
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(content: Text('Proceed to payment!')),
                );
              },
              child: Text('Proceed to Payment (\$${cartProvider.totalPrice})'),
            ),
          );
        },
      ),
    );
  }
}
```

**11. 쇼핑 카트 모델 및 Provider 작성**

- `lib/models/cart_item.dart` 파일 작성:

```dart
// lib/models/cart_item.dart
import 'product.dart';

class CartItem {
  final Product product;
  int quantity;

  CartItem({required this.product, this.quantity = 1});

  void increment() {
    quantity++;
  }

  void decrement() {
    if (quantity > 1) {
      quantity--;
    }
  }
}
```

- `lib/providers/cart_provider.dart` 파일 작성:

```dart
// lib/providers/cart_provider.dart
import 'package:flutter/material.dart';
import '../models/cart_item.dart';
import '../models/product.dart';

class CartProvider with ChangeNotifier {
  final List<CartItem> _items = [];

  List<CartItem> get items => _items;

  double get totalPrice => _items.fold(0, (total, item) => total + item.product.price * item.quantity);

  void addToCart(Product product) {
    final index = _items.indexWhere((item) => item.product.id == product.id);
    if (index != -1) {
      _items[index].increment();
    } else {
      _items.add(CartItem(product: product));
    }
    notifyListeners();
  }

  void removeFromCart(int productId) {
    _items.removeWhere((item) => item.product.id == productId);
    notifyListeners();
  }

  void clearCart() {
    _items.clear();
    notifyListeners();
  }
}
```

**12. 메인 앱 작성**

- `lib/main.dart` 파일 작성:

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'pages/main_page.dart';
import 'providers/product_provider.dart';
import 'providers/cart_provider.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';

Future<void> main() async {
  await dotenv.load(fileName: ".env");
  runApp(ShoppingMallApp());
}

class ShoppingMallApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => ProductProvider()),
        ChangeNotifierProvider(create: (_) => CartProvider()),
      ],
      child: MaterialApp(
        title: 'Shopping Mall',
        theme: ThemeData(
          primarySwatch: Colors.blue,
          visualDensity: VisualDensity.adaptivePlatformDensity,
        ),
        home: MainPage(),
      ),
    );
  }
}
```

**13. 결제 페이지 작성**

- `lib/pages/payment_page.dart` 파일 작성:

```dart
// lib/pages/payment_page.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/cart_provider.dart';
import 'payment_completion_page.dart';

class PaymentPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Payment Page')),
      body: Consumer<CartProvider>(
        builder: (context, cartProvider, child) {
          return Column(
            children: [
              Expanded(
                child: ListView.builder(
                  itemCount: cartProvider.items.length,
                  itemBuilder: (context, index) {
                    final item = cartProvider.items[index];
                    return ListTile(
                      title: Text(item.product.name),
                      subtitle: Text('\$${item.product.price} x ${item.quantity}'),
                    );
                  },
                ),
              ),
              Padding(
                padding: const EdgeInsets.all(8.0),
                child: Column(
                  children: [
                    Text('Total: \$${cartProvider.totalPrice}', style: Theme.of(context).textTheme.headline6),
                    SizedBox(height: 16),
                    ElevatedButton(
                      onPressed: () {
                        Navigator.pushReplacement(
                          context,
                          MaterialPageRoute(builder: (context) => PaymentCompletionPage()),
                        );
                      },
                      child: Text('Complete Payment'),
                    ),
                  ],
                ),
              ),
            ],
          );
        },
      ),
    );
  }
}
```

**14. 결제 완료 페이지 작성**

- `lib/pages/payment_completion_page.dart` 파일 작성:

```dart
// lib/pages/payment_completion_page.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/cart_provider.dart';
import 'main_page.dart';

class PaymentCompletionPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Payment Complete')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Thank you for your purchase!', style: Theme.of(context).textTheme.headline5),
            SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Provider.of<CartProvider>(context, listen: false).clearCart();
                Navigator.pushAndRemoveUntil(
                  context,
                  MaterialPageRoute(builder: (context) => MainPage()),
                  (route) => false,
                );
              },
              child: Text('Back to Shopping'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**15. 주문 및 배송 확인 페이지 작성**

- `lib/pages/order_confirmation_page.dart` 파일 작성:

```dart
// lib/pages/order_confirmation_page.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/cart_provider.dart';

class OrderConfirmationPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Order and Delivery Confirmation')),
      body: Consumer<CartProvider>(
        builder: (context, cartProvider, child) {
          if (cartProvider.items.isEmpty) {
            return Center(
              child: Text('You have no completed orders yet.'),
            );
          }

          return ListView.builder(
            itemCount: cartProvider.items.length,
            itemBuilder: (context, index) {
              final item = cartProvider.items[index];
              return ListTile(
                title: Text(item.product.name),
                subtitle: Text('Quantity: ${item.quantity}'),
              );
            },
          );
        },
      ),
    );
  }
}
```

**16. 반품 및 환불 페이지 작성**

- `lib/pages/return_and_refund_page.dart` 파일 작성:

```dart
// lib/pages/return_and_refund_page.dart
import 'package:flutter/material.dart';

class ReturnAndRefundPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Return and Refund')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Return and Refund Policy',
              style: Theme.of(context).textTheme.headline6,
            ),
            SizedBox(height: 16),
            Text(
              '1. You can request a return or refund within 30 days of purchase.',
              style: Theme.of(context).textTheme.bodyText1,
            ),
            SizedBox(height: 8),
            Text(
              '2. To process a return or refund, please contact our customer service team.',
              style: Theme.of(context).textTheme.bodyText1,
            ),
            SizedBox(height: 8),
            Text(
              '3. All returned items must be in their original condition.',
              style: Theme.of(context).textTheme.bodyText1,
            ),
            SizedBox(height: 8),
            Text(
              '4. Refunds will be processed within 7-10 business days.',
              style: Theme.of(context).textTheme.bodyText1,
            ),
            SizedBox(height: 32),
            ElevatedButton(
              onPressed: () {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(content: Text('Return or Refund request submitted!')),
                );
              },
              child: Text('Submit Request'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**17. 계정 페이지 작성**

- `lib/pages/account_page.dart` 파일 작성:

```dart
// lib/pages/account_page.dart
import 'package:flutter/material.dart';
import 'order_confirmation_page.dart';
import 'return_and_refund_page.dart';

class AccountPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Account')),
      body: ListView(
        children: [
          ListTile(
            title: Text('Order and Delivery Confirmation'),
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(builder: (context) => OrderConfirmationPage()),
              );
            },
          ),
          ListTile(
            title: Text('Return and Refund'),
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(builder: (context) => ReturnAndRefundPage()),
              );
            },
          ),
        ],
      ),
    );
  }
}
```

**18. 메인 앱에 네비게이션 적용**

- `lib/pages/main_page.dart` 파일 수정:

```dart
// lib/pages/main_page.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/product_provider.dart';
import 'product_detail_page.dart';
import 'cart_page.dart';
import 'account_page.dart';

class MainPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Shopping Mall')),
      body: Consumer<ProductProvider>(
        builder: (context, productProvider, child) {
          return FutureBuilder(
            future: productProvider.fetchProducts(),
            builder: (context, snapshot) {
              if (snapshot.connectionState == ConnectionState.waiting) {
                return Center(child: CircularProgressIndicator());
              } else {
                return ListView.builder(
                  itemCount: productProvider.products.length,
                  itemBuilder: (context, index) {
                    final product = productProvider.products[index];
                    return ListTile(
                      title: Text(product.name),
                      subtitle: Text('\$${product.price}'),
                      onTap: () {
                        Navigator.push(
                          context,
                          MaterialPageRoute(
                            builder: (context) => ProductDetailPage(productId: product.id!),
                          ),
                        );
                      },
                    );
                  },
                );
              }
            },
          );
        },
      ),
      bottomNavigationBar: BottomNavigationBar(
        onTap: (index) {
          switch (index) {
            case 0:
              Navigator.push(context, MaterialPageRoute(builder: (context) => CartPage()));
              break;
            case 1:
              Navigator.push(context, MaterialPageRoute(builder: (context) => AccountPage()));
              break;
          }
        },
        items: [
          BottomNavigationBarItem(
            icon: Icon(Icons.shopping_cart),
            label: 'Cart',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.account_circle),
            label: 'Account',
          ),
        ],
      ),
    );
  }
}
```

이제 모든 페이지와 핵심 기능을 구현했으므로, Flutter 프론트엔드와 Nest.js 백엔드의 간단한 쇼핑몰이 완성되었습니다. 

**19. 백엔드 서버 실행**

백엔드 서버를 실행합니다.

```bash
$ npm run start
```

**20. 프론트엔드 앱 실행**

프론트엔드 앱을 실행합니다.

```bash
$ flutter run
```

**21. 최종 구조**

프로젝트의 최종 폴더 구조는 아래와 같습니다.

```
shopping_mall_frontend/
├── lib/
│   ├── models/
│   │   ├── cart_item.dart
│   │   └── product.dart
│   ├── pages/
│   │   ├── account_page.dart
│   │   ├── cart_page.dart
│   │   ├── main_page.dart
│   │   ├── order_confirmation_page.dart
│   │   ├── payment_completion_page.dart
│   │   ├── payment_page.dart
│   │   ├── product_detail_page.dart
│   │   └── return_and_refund_page.dart
│   ├── providers/
│   │   ├── cart_provider.dart
│   │   └── product_provider.dart
│   ├── services/
│   │   └── product_service.dart
│   └── main.dart
├── pubspec.yaml
└── .env

shopping_mall_backend/
├── src/
│   ├── app.module.ts
│   ├── main.ts
│   └── product/
│       ├── product.controller.ts
│       ├── product.entity.ts
│       ├── product.module.ts
│       ├── product.repository.ts
│       └── product.service.ts
├── tsconfig.json
└── .env
```

**22. 보완 및 확장 아이디어**

- **사용자 인증 및 계정 관리**: Firebase Authentication 또는 자체 인증 시스템을 사용하여 사용자 로그인 및 회원가입 기능 추가.
- **주문 관리 및 이력**: 주문 내역을 저장하고, 주문 상태를 추적하는 기능 추가.
- **다양한 결제 수단**: 결제 수단을 확장하여 신용카드, 페이팔, 가상계좌 등 다양한 결제 방법 지원.
- **상품 리뷰 및 평가**: 사용자가 구매한 상품에 대한 리뷰와 평점을 남길 수 있는 기능 추가.
- **관리자 페이지**: 상품 관리, 주문 관리, 사용자 관리 등을 위한 관리자 페이지 추가.

**23. 참고자료**

- [Flutter 공식 문서](https://flutter.dev/docs)
- [NestJS 공식 문서](https://docs.nestjs.com/)
- [TypeORM 공식 문서](https://typeorm.io/)

이 매뉴얼을 통해 간단한 쇼핑몰 앱을 완성할 수 있습니다. 각 기능을 확장하고 다른 페이지를 추가하여 프로젝트를 발전시켜 나가시길 바랍니다.
