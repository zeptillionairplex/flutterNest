Flutter 앱과 연동되는 Nest.js 백엔드 서버를 구축하려면 아래 단계에 따라 진행하시면 됩니다.

### 1. Nest.js 프로젝트 생성
먼저, Nest CLI를 통해 프로젝트를 생성합니다.

```bash
npm i -g @nestjs/cli
nest new shopping-mall-backend
```

### 2. 필요한 패키지 설치
`shopping-mall-backend` 디렉토리로 이동하여 필요한 패키지를 설치합니다.

```bash
cd shopping-mall-backend
npm install @nestjs/typeorm typeorm mysql
```

### 3. MySQL 데이터베이스 설정
MySQL 데이터베이스를 생성하고 `.env` 파일을 생성하여 설정을 추가합니다.

#### .env
```env
DB_HOST=localhost
DB_PORT=3306
DB_USERNAME=root
DB_PASSWORD=yourpassword
DB_DATABASE=shopping_mall
```

### 4. TypeORM 및 환경 설정
`app.module.ts`에 TypeORM 설정을 추가합니다.

#### src/app.module.ts
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule } from '@nestjs/config';
import { ProductsModule } from './products/products.module';
import { CartModule } from './cart/cart.module';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
    }),
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT, 10),
      username: process.env.DB_USERNAME,
      password: process.env.DB_PASSWORD,
      database: process.env.DB_DATABASE,
      autoLoadEntities: true,
      synchronize: true,
    }),
    ProductsModule,
    CartModule,
  ],
})
export class AppModule {}
```

### 5. 상품 엔티티 및 서비스 생성
먼저 상품 엔티티를 생성합니다.

#### src/products/product.entity.ts
```typescript
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  description: string;

  @Column('decimal')
  price: number;

  @Column()
  imageUrl: string;
}
```

상품 모듈, 서비스, 컨트롤러를 생성합니다.

```bash
nest g module products
nest g service products
nest g controller products
```

#### src/products/products.module.ts
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ProductsService } from './products.service';
import { ProductsController } from './products.controller';
import { Product } from './product.entity';

@Module({
  imports: [TypeOrmModule.forFeature([Product])],
  controllers: [ProductsController],
  providers: [ProductsService],
})
export class ProductsModule {}
```

#### src/products/products.service.ts
```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Product } from './product.entity';

@Injectable()
export class ProductsService {
  constructor(
    @InjectRepository(Product)
    private productsRepository: Repository<Product>,
  ) {}

  findAll(): Promise<Product[]> {
    return this.productsRepository.find();
  }

  findOne(id: number): Promise<Product> {
    return this.productsRepository.findOneBy({ id });
  }

  create(product: Product): Promise<Product> {
    return this.productsRepository.save(product);
  }

  async remove(id: number): Promise<void> {
    await this.productsRepository.delete(id);
  }
}
```

#### src/products/products.controller.ts
```typescript
import { Controller, Get, Post, Body, Param, Delete } from '@nestjs/common';
import { ProductsService } from './products.service';
import { Product } from './product.entity';

@Controller('products')
export class ProductsController {
  constructor(private readonly productsService: ProductsService) {}

  @Get()
  findAll(): Promise<Product[]> {
    return this.productsService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string): Promise<Product> {
    return this.productsService.findOne(+id);
  }

  @Post()
  create(@Body() product: Product): Promise<Product> {
    return this.productsService.create(product);
  }

  @Delete(':id')
  remove(@Param('id') id: string): Promise<void> {
    return this.productsService.remove(+id);
  }
}
```

### 6. 장바구니 엔티티 및 서비스 생성
장바구니 엔티티를 생성합니다.

#### src/cart/cart-item.entity.ts
```typescript
import { Entity, Column, PrimaryGeneratedColumn, ManyToOne } from 'typeorm';
import { Product } from '../products/product.entity';

@Entity()
export class CartItem {
  @PrimaryGeneratedColumn()
  id: number;

  @ManyToOne(() => Product)
  product: Product;

  @Column()
  quantity: number;
}
```

장바구니 모듈, 서비스, 컨트롤러를 생성합니다.

```bash
nest g module cart
nest g service cart
nest g controller cart
```

#### src/cart/cart.module.ts
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { CartService } from './cart.service';
import { CartController } from './cart.controller';
import { CartItem } from './cart-item.entity';
import { Product } from '../products/product.entity';

@Module({
  imports: [TypeOrmModule.forFeature([CartItem, Product])],
  controllers: [CartController],
  providers: [CartService],
})
export class CartModule {}
```

#### src/cart/cart.service.ts
```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { CartItem } from './cart-item.entity';
import { Product } from '../products/product.entity';

@Injectable()
export class CartService {
  constructor(
    @InjectRepository(CartItem)
    private cartRepository: Repository<CartItem>,
    @InjectRepository(Product)
    private productsRepository: Repository<Product>,
  ) {}

  async findAll(): Promise<CartItem[]> {
    return this.cartRepository.find({ relations: ['product'] });
  }

  async addToCart(productId: number, quantity: number): Promise<CartItem> {
    const product = await this.productsRepository.findOneBy({ id: productId });

    if (!product) {
      throw new Error('Product not found');
    }

    let cartItem = await this.cartRepository.findOneBy({ product });

    if (cartItem) {
      cartItem.quantity += quantity;
    } else {
      cartItem = this.cartRepository.create({ product, quantity });
    }

    return this.cartRepository.save(cartItem);
  }

  async removeFromCart(id: number): Promise<void> {
    await this.cartRepository.delete(id);
  }

  async clearCart(): Promise<void> {
    await this.cartRepository.clear();
  }
}
```

#### src/cart/cart.controller.ts
```typescript
import { Controller, Get, Post, Delete, Body, Param } from '@nestjs/common';
import { CartService } from './cart.service';

@Controller('cart')
export class CartController {
  constructor(private readonly cartService: CartService) {}

  @Get()
  findAll() {
    return this.cartService.findAll();
  }

  @Post()
  addToCart(
    @Body('productId') productId: number,
    @Body('quantity') quantity: number,
  ) {
    return this.cartService.addToCart(productId, quantity);
  }

  @Delete(':id')
  removeFromCart(@Param('id') id: number) {
    return this.cartService.removeFromCart(id);
  }

  @Delete()
  clearCart() {
    return this.cartService.clearCart();
  }
}
```

### 7. Flutter 앱과의 통합
Flutter 앱에서 Nest.js로부터 상품 및 장바구니 데이터를 가져오고, 장바구니에 상품을 추가하는 기능을 구현하세요.

#### Flutter: 상품 목록 가져오기
`lib/providers/products_provider.dart` 파일을 수정하여 상품 목록을 API에서 가져오도록 변경하세요.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import '../models/product.dart';

class ProductsProvider with ChangeNotifier {
  List<Product> _items = [];

  List<Product> get items => [..._items];

  Future<void> fetchAndSetProducts() async {
    final url = Uri.parse('http://localhost:3000/products');
    final response = await http.get(url);
    final List<dynamic> responseData = json.decode(response.body);
    _items = responseData
        .map((item) => Product(
              id: item['id'].toString(),
              title: item['title'],
              description: item['description'],
              price: double.parse(item['price'].toString()),
              imageUrl: item['imageUrl'],
            ))
        .toList();
    notifyListeners();
  }

  Product findById(String id) {
    return _items.firstWhere((product) => product.id == id);
  }
}
```

#### Flutter: 상품 목록 화면에서 데이터 가져오기
`lib/screens/products_overview_screen.dart` 파일에서 상품 데이터를 가져오도록 변경하세요.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/products_provider.dart';
import '../widgets/products_grid.dart';
import '../widgets/badge.dart';
import '../providers/cart_provider.dart';

물론이죠! 계속해서 "Flutter: 상품 목록 화면에서 데이터 가져오기" 부분부터 답변을 이어가겠습니다.

### Flutter: 상품 목록 화면에서 데이터 가져오기
`lib/screens/products_overview_screen.dart` 파일을 업데이트하여 앱이 실행될 때 상품 목록을 가져오도록 변경하세요.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/products_provider.dart';
import '../widgets/products_grid.dart';
import '../widgets/badge.dart';
import '../providers/cart_provider.dart';
import '../screens/cart_screen.dart';

class ProductsOverviewScreen extends StatefulWidget {
  @override
  _ProductsOverviewScreenState createState() => _ProductsOverviewScreenState();
}

class _ProductsOverviewScreenState extends State<ProductsOverviewScreen> {
  var _isLoading = false;

  @override
  void initState() {
    super.initState();
    _fetchProducts();
  }

  Future<void> _fetchProducts() async {
    setState(() {
      _isLoading = true;
    });
    await Provider.of<ProductsProvider>(context, listen: false)
        .fetchAndSetProducts();
    setState(() {
      _isLoading = false;
    });
  }

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
      body: _isLoading ? Center(child: CircularProgressIndicator()) : ProductsGrid(),
    );
  }
}
```

### Flutter: 장바구니에 상품 추가하기
`lib/providers/cart_provider.dart` 파일을 수정하여 장바구니에 상품을 추가할 때 API를 호출하도록 변경하세요.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import '../models/cart_item.dart';

class CartProvider with ChangeNotifier {
  Map<String, CartItem> _items = {};

  String baseUrl = 'http://localhost:3000';

  Map<String, CartItem> get items => {..._items};

  int get itemCount => _items.length;

  double get totalAmount {
    double total = 0.0;
    _items.forEach((key, cartItem) {
      total += cartItem.price * cartItem.quantity;
    });
    return total;
  }

  Future<void> addItem(String productId, String title, double price) async {
    final url = Uri.parse('$baseUrl/cart');
    final response = await http.post(
      url,
      headers: {'Content-Type': 'application/json'},
      body: json.encode({
        'productId': int.parse(productId),
        'quantity': 1,
      }),
    );

    if (response.statusCode == 201) {
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
    } else {
      throw Exception('Failed to add item to cart');
    }
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

### Flutter: 상품 상세 페이지에서 장바구니에 추가하기
`lib/screens/product_detail_screen.dart` 파일을 수정하여 상품 상세 페이지에서 장바구니에 상품을 추가할 수 있도록 API를 호출하세요.

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

### Flutter: 장바구니 화면 구현
`lib/screens/cart_screen.dart` 파일을 생성하여 장바구니 화면을 구현하세요.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/cart_provider.dart';
import '../widgets/cart_item.dart';

class CartScreen extends StatelessWidget {
  static const routeName = '/cart';

  @override
  Widget build(BuildContext context) {
    final cart = Provider.of<CartProvider>(context);
    return Scaffold(
      appBar: AppBar(
        title: Text('Your Cart'),
      ),
      body: Column(
        children: [
          Card(
            margin: EdgeInsets.all(15),
            child: Padding(
              padding: EdgeInsets.all(8),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Text(
                    'Total',
                    style: TextStyle(fontSize: 20),
                  ),
                  Spacer(),
                  Chip(
                    label: Text(
                      '\$${cart.totalAmount.toStringAsFixed(2)}',
                      style: TextStyle(
                        color: Theme.of(context).primaryTextTheme.headline6?.color,
                      ),
                    ),
                    backgroundColor: Theme.of(context).primaryColor,
                  ),
                  TextButton(
                    onPressed: () {},
                    child: Text('ORDER NOW'),
                  )
                ],
              ),
            ),
          ),
          SizedBox(height: 10),
          Expanded(
            child: ListView.builder(
              itemCount: cart.items.length,
              itemBuilder: (ctx, i) => CartItemWidget(
                id: cart.items.values.toList()[i].id,
                productId: cart.items.keys.toList()[i],
                title: cart.items.values.toList()[i].title,
                quantity: cart.items.values.toList()[i].quantity,
                price: cart.items.values.toList()[i].price,
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

### Flutter: 장바구니 항목 위젯 구현
`lib/widgets/cart_item.dart` 파일을 생성하여 장바구니 항목 위젯을 구현하세요.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/cart_provider.dart';

class CartItemWidget extends StatelessWidget {
  final String id;
  final String productId;
  final String title;
  final int quantity;
  final double price;

  CartItemWidget({
    required this.id,
    required this.productId,
    required this.title,
    required this.quantity,
    required this.price,
  });

  @override
  Widget build(BuildContext context) {
    return Dismissible(
      key: ValueKey(id),
      direction: DismissDirection.endToStart,
      onDismissed: (direction) {
        Provider.of<CartProvider>(context, listen: false).removeItem(productId);
      },
      background: Container(
        color: Theme.of(context).errorColor,
        child: Icon(
          Icons.delete,
          color: Colors.white,
          size: 40,
        ),
        alignment: Alignment.centerRight,
        padding: EdgeInsets.only(right: 20),
        margin: EdgeInsets.symmetric(
          horizontal: 15,
          vertical: 4,
        ),
      ),
      child: Card(
        margin: EdgeInsets.symmetric(
          horizontal: 15,
          vertical: 4,
        ),
        child: Padding(
          padding: EdgeInsets.all(8),
          child: ListTile(
            leading: CircleAvatar(
              child: Padding(
                padding: EdgeInsets.all(5),
                child: FittedBox(
                  child: Text('\$${price

알겠습니다! `lib/widgets/cart_item.dart` 파일을 계속해서 완성하겠습니다.

```dart
                child: Text('\$${price.toStringAsFixed(2)}'),
              ),
            ),
          ),
          title: Text(title),
          subtitle: Text('Total: \$${(price * quantity).toStringAsFixed(2)}'),
          trailing: Text('$quantity x'),
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

****
