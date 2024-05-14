쇼핑몰 앱을 Flutter와 NestJS로 구현하는 것은 훌륭한 아이디어입니다. 다음은 Flutter와 NestJS를 사용하여 로그인, 상품 페이지, 장바구니, 위시리스트, 결제 기능을 구현하는 예시 코드입니다. **Flutter**와 **NestJS**의 구조 및 주요 파일을 보여주고, 필요한 곳에 DTO와 Repository 패턴을 사용하였습니다.

### NestJS Backend

**1. 프로젝트 세팅**
```bash
# NestJS 프로젝트 생성 및 모듈 설치
$ nest new shopping-mall-backend
$ cd shopping-mall-backend
$ npm install @nestjs/passport passport passport-local passport-jwt bcrypt class-validator class-transformer typeorm pg
```

**2. 디렉토리 구조**
```
shopping-mall-backend/
├── src/
│   ├── auth/
│   ├── products/
│   ├── cart/
│   ├── wishlist/
│   ├── payments/
│   ├── app.module.ts
│   └── main.ts
└── ...
```

**3. Auth 모듈 - 인증 및 JWT 발급**

`auth/auth.module.ts`
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

`auth/auth.controller.ts`
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

`auth/auth.service.ts`
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

`auth/jwt.strategy.ts`
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

`auth/local.strategy.ts`
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

`auth/local-auth.guard.ts`
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

**4. Users 모듈 - 사용자 관리**

`users/users.module.ts`
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

`users/users.service.ts`
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
}
```

`users/user.entity.ts`
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

**5. Product 모듈 - 상품 관리**

`products/product.entity.ts`
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

`products/product.dto.ts`
```typescript
export class CreateProductDto {
  name: string;
  description: string;
  price: number;
  imageUrl: string;
}
```

`products/products.module.ts`
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

`products/products.service.ts`
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

`products/products.controller.ts`
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

**6. Cart 모듈 - 장바구니 관리**

`cart/cart.entity.ts`
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

`cart/cart.dto.ts`
```typescript
export class AddToCartDto {
  productId: number;
  quantity: number;
}
```

`cart/cart.module.ts`
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

`cart/cart.service.ts`
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

`cart/cart.controller.ts`
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

**7. Wishlist 모듈 - 위시리스트 관리**

`wishlist/wishlist.entity.ts`
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

`wishlist/wishlist.dto.ts`
```typescript
export class AddToWishlistDto {
  productId: number;
}
```

`wishlist/wishlist.module.ts`
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

`wishlist/wishlist.service.ts`
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

`wishlist/wishlist.controller.ts`
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

**8. Payments 모듈 - 결제 관리**

`payments/payment.entity.ts`
```typescript
import { Entity, Column, PrimaryGeneratedColumn, ManyToOne } from 'typeorm';
import { User } from '../users/user.entity';
import { CartItem } from '../cart/cart.entity';

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

`payments/payment.dto.ts`
```typescript
export class CreatePaymentDto {
  amount: number;
}
```

`payments/payments.module.ts`
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

`payments/payments.service.ts`
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

`payments/payments.controller.ts`
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

**9. 기타 파일들**

`app.module.ts`
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

`auth/jwt-auth.guard.ts`
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

**NestJS 백엔드 구조는 여기까지입니다.**
