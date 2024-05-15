Nest.js를 사용하여 Flutter의 DTO, Repository, Provider 패턴 예제 코드와 통신하는 간단한 서버를 만드는 방법을 살펴보겠습니다. Nest.js에서의 패턴 적용을 보여주기 위해 간단한 상품 목록 API를 작성하겠습니다.

### 프로젝트 구조
```
nestjs-shopping-mall/
├── src/
│   ├── common/
│   │   └── dto/
│   │       └── product.dto.ts
│   ├── products/
│   │   ├── products.controller.ts
│   │   ├── products.module.ts
│   │   ├── products.repository.ts
│   │   ├── products.service.ts
│   │   └── product.entity.ts
│   └── app.module.ts
└── main.ts
```

### 1. Nest.js 설치 및 시작
1. Nest CLI를 사용하여 프로젝트 생성:
   ```bash
   npm i -g @nestjs/cli
   nest new nestjs-shopping-mall
   ```

2. 필요한 패키지 설치:
   ```bash
   npm install @nestjs/typeorm typeorm sqlite3
   ```

### 2. 프로젝트 코드 작성
#### **`product.dto.ts`** (DTO)
상품 정보를 주고받기 위한 데이터 전송 객체를 정의합니다.

```typescript
// src/common/dto/product.dto.ts
export class ProductDTO {
  id: number;
  name: string;
  price: number;
}
```

#### **`product.entity.ts`** (Entity)
TypeORM 엔터티를 사용하여 데이터베이스 테이블과 매핑되는 클래스를 정의합니다.

```typescript
// src/products/product.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity('products')
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('float')
  price: number;
}
```

#### **`products.repository.ts`** (Repository)
TypeORM 레포지토리를 사용하여 데이터베이스에서 상품 데이터를 관리합니다.

```typescript
// src/products/products.repository.ts
import { Repository, EntityRepository } from 'typeorm';
import { Product } from './product.entity';

@EntityRepository(Product)
export class ProductRepository extends Repository<Product> {}
```
<details>
   <summary>class "ProductRepository" {}부분에 아무런 코드가 없는데도, 데이터베이스에 접속이 가능한건가?</summary>

```typescript
// src/products/products.repository.ts
import { Repository, EntityRepository } from 'typeorm';
import { Product } from './product.entity';

@EntityRepository(Product)
export class ProductRepository extends Repository<Product> {}
```
|에서 class "ProductRepository" {}부분에 아무런 코드가 없는데도, 데이터베이스에 접속이 가능한건가?

네, `ProductRepository` 클래스에 별다른 코드가 없더라도, NestJS와 TypeORM의 구조 덕분에 데이터베이스에 접근하여 데이터를 관리할 수 있습니다. 이는 TypeORM의 **자동 생성된 리포지토리 기능** 덕분입니다.

### 자동 생성된 레포지토리
`Repository`, `EntityRepository`는 TypeORM에서 제공하는 기능으로, `Repository` 클래스는 이미 CRUD와 관련된 메서드를 내장하고 있습니다.

> `class ProductRepository extends Repository<Product> {}`

위 코드에서 `ProductRepository`는 TypeORM의 `Repository` 클래스를 상속받습니다. `Repository` 클래스는 `find`, `findOne`, `save`, `delete` 등 기본적인 CRUD 기능을 제공합니다.

### 예시: `find` 메서드 사용

**Service 코드**
```typescript
// src/products/products.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { ProductRepository } from './products.repository';
import { ProductDTO } from '../common/dto/product.dto';
import { Product } from './product.entity';

@Injectable()
export class ProductsService {
  constructor(
    @InjectRepository(ProductRepository)
    private readonly productRepository: ProductRepository,
  ) {}

  async findAll(): Promise<ProductDTO[]> {
    const products = await this.productRepository.find();
    return products.map((product) => ({
      id: product.id,
      name: product.name,
      price: product.price,
    }));
  }

  async findOne(id: number): Promise<ProductDTO> {
    const product = await this.productRepository.findOne(id);
    if (!product) {
      throw new NotFoundException(`Product with ID ${id} not found`);
    }
    return {
      id: product.id,
      name: product.name,
      price: product.price,
    };
  }

  async create(productDto: ProductDTO): Promise<ProductDTO> {
    const product = this.productRepository.create(productDto);
    await this.productRepository.save(product);
    return {
      id: product.id,
      name: product.name,
      price: product.price,
    };
  }
}
```

### 주요 메서드
`ProductRepository`가 상속하는 `Repository` 클래스는 다음과 같은 메서드를 제공합니다.

- `find`: 모든 엔티티 검색
- `findOne`: 하나의 엔티티 검색 (조건으로)
- `save`: 엔티티 저장 또는 업데이트
- `delete`: 엔티티 삭제

### `EntityRepository`와 `TypeOrmModule` 연동
`EntityRepository`는 특정 엔티티에 대한 레포지토리를 정의하는 데 사용됩니다. 이를 사용하기 위해서는 `TypeOrmModule`의 `forFeature` 메서드를 통해 모듈에 등록해야 합니다.

**`products.module.ts`**
```typescript
// src/products/products.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ProductsController } from './products.controller';
import { ProductsService } from './products.service';
import { ProductRepository } from './products.repository';

@Module({
  imports: [TypeOrmModule.forFeature([ProductRepository])],
  controllers: [ProductsController],
  providers: [ProductsService],
})
export class ProductsModule {}
```

### 결론
`ProductRepository` 클래스에 코드가 없더라도, TypeORM의 `Repository` 클래스를 상속하기 때문에 이미 내장된 CRUD 메서드를 사용할 수 있습니다. 이를 통해 개발자는 추가적인 구현 없이도 데이터베이스에 접근하고, 데이터를 관리할 수 있습니다.
</details>

#### **`products.service.ts`** (Service)
서비스 계층에서 레포지토리를 사용해 비즈니스 로직을 구현합니다.

```typescript
// src/products/products.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { ProductRepository } from './products.repository';
import { ProductDTO } from '../common/dto/product.dto';
import { Product } from './product.entity';

@Injectable()
export class ProductsService {
  constructor(
    @InjectRepository(ProductRepository)
    private readonly productRepository: ProductRepository,
  ) {}

  async findAll(): Promise<ProductDTO[]> {
    const products = await this.productRepository.find();
    return products.map((product) => ({
      id: product.id,
      name: product.name,
      price: product.price,
    }));
  }

  async findOne(id: number): Promise<ProductDTO> {
    const product = await this.productRepository.findOne(id);
    if (!product) {
      throw new NotFoundException(`Product with ID ${id} not found`);
    }
    return {
      id: product.id,
      name: product.name,
      price: product.price,
    };
  }

  async create(productDto: ProductDTO): Promise<ProductDTO> {
    const product = this.productRepository.create(productDto);
    await this.productRepository.save(product);
    return {
      id: product.id,
      name: product.name,
      price: product.price,
    };
  }
}
```

#### **`products.controller.ts`** (Controller)
컨트롤러 계층에서 HTTP 요청을 처리하고 서비스 계층에 위임합니다.

```typescript
// src/products/products.controller.ts
import { Controller, Get, Post, Body, Param } from '@nestjs/common';
import { ProductsService } from './products.service';
import { ProductDTO } from '../common/dto/product.dto';

@Controller('products')
export class ProductsController {
  constructor(private readonly productsService: ProductsService) {}

  @Get()
  async findAll(): Promise<ProductDTO[]> {
    return this.productsService.findAll();
  }

  @Get(':id')
  async findOne(@Param('id') id: number): Promise<ProductDTO> {
    return this.productsService.findOne(id);
  }

  @Post()
  async create(@Body() productDto: ProductDTO): Promise<ProductDTO> {
    return this.productsService.create(productDto);
  }
}
```

#### **`products.module.ts`** (Module)
모듈 계층에서 컨트롤러와 서비스, 레포지토리를 연결합니다.

```typescript
// src/products/products.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ProductsController } from './products.controller';
import { ProductsService } from './products.service';
import { ProductRepository } from './products.repository';

@Module({
  imports: [TypeOrmModule.forFeature([ProductRepository])],
  controllers: [ProductsController],
  providers: [ProductsService],
})
export class ProductsModule {}
```

#### **`app.module.ts`**
`AppModule`에서 전체 애플리케이션의 모듈을 통합합니다.

```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ProductsModule } from './products/products.module';
import { Product } from './products/product.entity';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'sqlite',
      database: 'shopping-mall.db',
      synchronize: true,
      entities: [Product],
    }),
    ProductsModule,
  ],
})
export class AppModule {}
```

#### **`main.ts`**
Nest.js 서버의 메인 엔트리 포인트입니다.

```typescript
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.setGlobalPrefix('api'); // 모든 API 엔드포인트에 /api 접두어 추가
  await app.listen(3000);
}
bootstrap();
```

### 실행 방법
1. 필요한 패키지 설치:
   ```bash
   npm install
   ```

2. Nest.js 서버 실행:
   ```bash
   npm run start
   ```

3. API 테스트
   - **모든 상품 조회:** `GET http://localhost:3000/api/products`
   - **단일 상품 조회:** `GET http://localhost:3000/api/products/1`
   - **상품 생성:** `POST http://localhost:3000/api/products`
     ```json
     {
       "name": "Product A",
       "price": 99.99
     }
     ```

### 설명
- **DTO 패턴:** `ProductDTO`를 통해 서버와 클라이언트 간 데이터 전송을 위한 표준화된 형식을 정의합니다.
- **Repository 패턴:** `ProductRepository`를 통해 데이터베이스와의 통신을 캡슐화하고, 고수준의 데이터 접근 API를 제공합니다.
- **Provider 패턴:** `ProductsService`는 비즈니스 로직을 캡슐화하며, 컨트롤러에서 의존성 주입을 통해 사용됩니다.

### Flutter와의 연동
Flutter 프로젝트의 `ProductRepository`에서 Nest.js 서버의 API를 호출하도록 `baseUrl`을 변경합니다.

`main.dart`에 정확하게 반영된 코드를 제공하겠습니다.

**`main.dart`**
`main.dart`에서 Nest.js 서버와 통신하도록 `ProductRepository`의 `baseUrl`을 변경합니다.

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
            repository: ProductRepository(baseUrl: 'http://localhost:3000/api'),
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
다음은 프로젝트에서 모든 파일을 종합한 구조입니다.

**Flutter 프로젝트**
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
            repository: ProductRepository(baseUrl: 'http://localhost:3000/api'),
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

### Nest.js 서버와 Flutter 클라이언트 통신 방법
1. **Nest.js 서버 실행**
   - Nest.js 프로젝트 루트에서 서버를 실행:
     ```bash
     npm run start
     ```

2. **Flutter 앱 실행**
   - Flutter 프로젝트 루트에서 앱을 실행:
     ```bash
     flutter run
     ```

### Nest.js 서버 전체 코드 구조
Nest.js 서버의 전체 파일 구조와 코드를 종합하면 다음과 같습니다.

**Nest.js 프로젝트 구조**
```
nestjs-shopping-mall/
├── src/
│   ├── common/
│   │   └── dto/
│   │       └── product.dto.ts
│   ├── products/
│   │   ├── products.controller.ts
│   │   ├── products.module.ts
│   │   ├── products.repository.ts
│   │   ├── products.service.ts
│   │   └── product.entity.ts
│   └── app.module.ts
└── main.ts
```

**`product.dto.ts`**
```typescript
// src/common/dto/product.dto.ts
export class ProductDTO {
  id: number;
  name: string;
  price: number;
}
```

**`product.entity.ts`**
```typescript
// src/products/product.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity('products')
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('float')
  price: number;
}
```

**`products.repository.ts`**
```typescript
// src/products/products.repository.ts
import { Repository, EntityRepository } from 'typeorm';
import { Product } from './product.entity';

@EntityRepository(Product)
export class ProductRepository extends Repository<Product> {}
```

**`products.service.ts`**
```typescript
// src/products/products.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { ProductRepository } from './products.repository';
import { ProductDTO } from '../common/dto/product.dto';
import { Product } from './product.entity';

@Injectable()
export class ProductsService {
  constructor(
    @InjectRepository(ProductRepository)
    private readonly productRepository: ProductRepository,
  ) {}

  async findAll(): Promise<ProductDTO[]> {
    const products = await this.productRepository.find();
    return products.map((product) => ({
      id: product.id,
      name: product.name,
      price: product.price,
    }));
  }

  async findOne(id: number): Promise<ProductDTO> {
    const product = await this.productRepository.findOne(id);
    if (!product) {
      throw new NotFoundException(`Product with ID ${id} not found`);
    }
    return {
      id: product.id,
      name: product.name,
      price: product.price,
    };
  }

  async create(productDto: ProductDTO): Promise<ProductDTO> {
    const product = this.productRepository.create(productDto);
    await this.productRepository.save(product);
    return {
      id: product.id,
      name: product.name,
      price: product.price,
    };
  }
}
```

**`products.controller.ts`**
```typescript
// src/products/products.controller.ts
import { Controller, Get, Post, Body, Param } from '@nestjs/common';
import { ProductsService } from './products.service';
import { ProductDTO } from '../common/dto/product.dto';

@Controller('products')
export class ProductsController {
  constructor(private readonly productsService: ProductsService) {}

  @Get()
  async findAll(): Promise<ProductDTO[]> {
    return this.productsService.findAll();
  }

  @Get(':id')
  async findOne(@Param('id') id: number): Promise<ProductDTO> {
    return this.productsService.findOne(id);
  }

  @Post()
  async create(@Body() productDto: ProductDTO): Promise<ProductDTO> {
    return this.productsService.create(productDto);
  }
}
```

**`products.module.ts`**
```typescript
// src/products/products.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ProductsController } from './products.controller';
import { ProductsService } from './products.service';
import { ProductRepository } from './products.repository';

@Module({
  imports: [TypeOrmModule.forFeature([ProductRepository])],
  controllers: [ProductsController],
  providers: [ProductsService],
})
export class ProductsModule {}
```

**`app.module.ts`**
```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ProductsModule } from './products/products.module';
import { Product } from './products/product.entity';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'sqlite',
      database: 'shopping-mall.db',
      synchronize: true,
      entities: [Product],
    }),
    ProductsModule,
  ],
})
export class AppModule {}
```

**`main.ts`**
```typescript
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.setGlobalPrefix('api'); // 모든 API 엔드포인트에 /api 접두어 추가
  await app.listen(3000);
}
bootstrap();
```

### Nest.js 서버 실행
1. 필요한 패키지 설치:
   ```bash
   npm install
   ```

2. Nest.js 서버 실행:
   ```bash
   npm run start
   ```

### Flutter 앱과의 연동
Nest.js 서버가 실행되면 Flutter 앱에서 상품 목록을 가져오거나 추가할 수 있습니다. `main.dart` 파일에서 `ProductRepository`의 `baseUrl`을 다음과 같이 설정해야 합니다.

```dart
// lib/main.dart
ProductRepository(baseUrl: 'http://localhost:3000/api'),
```

### API 테스트
1. **모든 상품 조회:** `GET http://localhost:3000/api/products`
2. **단일 상품 조회:** `GET http://localhost:3000/api/products/1`
3. **상품 생성:** `POST http://localhost:3000/api/products`
   ```json
   {
     "name": "Product A",
     "price": 99.99
   }
   ```

### 정리
1. **Nest.js 패턴**
   - **DTO 패턴:** `ProductDTO`를 사용하여 서버와 클라이언트 간 데이터 전송을 위한 표준화된 형식을 정의합니다.
   - **Repository 패턴:** `ProductRepository`를 통해 데이터베이스와의 통신을 캡슐화하고, 고수준의 데이터 접근 API를 제공합니다.
   - **Provider 패턴:** `ProductsService`는 비즈니스 로직을 캡슐화하며, 컨트롤러에서 의존성 주입을 통해 사용합니다.

2. **Flutter 패턴**
   - **DTO 패턴:** `ProductDTO`를 사용하여 Flutter에서 서버의 데이터를 다루기 쉽게 합니다.
   - **Repository 패턴:** `ProductRepository`를 통해 API와 통신하는 로직을 중앙화합니다.
   - **Provider 패턴:** `ProductProvider`를 사용하여 상태를 관리하고 UI와 데이터를 연결합니다.

이처럼 Nest.js 서버와 Flutter 클라이언트를 연동하여 간단한 쇼핑몰 API를 구현할 수 있습니다.
