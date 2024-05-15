NestJS에서 Repository 패턴을 적용하기 위해서는 `TypeORM`과 같은 ORM 라이브러리를 활용하는 것이 일반적입니다. `Repository`는 데이터 접근을 추상화하여 비즈니스 로직을 깔끔하게 유지하도록 해줍니다.

### Repository 패턴을 적용한 간단한 예시

다음은 `TypeORM`을 사용하여 `User` 엔티티 및 관련 리포지토리를 만드는 예시입니다.

#### 1. 프로젝트 설정
먼저 `TypeORM` 패키지를 설치합니다.

```bash
npm install @nestjs/typeorm typeorm sqlite3
```

#### 2. User 엔티티 생성
```typescript
// src/user/user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;
}
```

#### 3. User 리포지토리 생성
NestJS에서 리포지토리를 직접 정의하지 않고 `TypeORM`의 `Repository` 클래스를 이용하여 자동으로 생성할 수 있습니다. 하지만 필요한 경우 커스텀 메서드를 추가하기 위해 리포지토리 클래스를 따로 정의할 수 있습니다.

```typescript
// src/user/user.repository.ts
import { Repository, EntityRepository } from 'typeorm';
import { User } from './user.entity';

@EntityRepository(User)
export class UserRepository extends Repository<User> {
  // 커스텀 메서드 추가
  async findByName(name: string): Promise<User | undefined> {
    return this.findOne({ where: { name } });
  }
}
```

#### 4. UserService 생성
서비스 계층에서 리포지토리를 사용하여 비즈니스 로직을 구현합니다.

```typescript
// src/user/user.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { UserRepository } from './user.repository';
import { User } from './user.entity';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(UserRepository)
    private readonly userRepository: UserRepository
  ) {}

  findAll(): Promise<User[]> {
    return this.userRepository.find();
  }

  findOne(id: number): Promise<User | undefined> {
    return this.userRepository.findOne(id);
  }

  findByName(name: string): Promise<User | undefined> {
    return this.userRepository.findByName(name);
  }

  async create(user: User): Promise<User> {
    return this.userRepository.save(user);
  }

  async remove(id: number): Promise<void> {
    await this.userRepository.delete(id);
  }
}
```

#### 5. UserModule 생성
모듈은 `UserService`와 `UserRepository`를 다른 모듈에서 사용할 수 있도록 제공합니다.

```typescript
// src/user/user.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UserService } from './user.service';
import { User } from './user.entity';
import { UserRepository } from './user.repository';

@Module({
  imports: [TypeOrmModule.forFeature([User, UserRepository])],
  providers: [UserService],
  exports: [UserService],
})
export class UserModule {}
```

#### 6. AppModule 설정
애플리케이션의 루트 모듈에 TypeORM을 설정하고 `UserModule`을 가져옵니다.

```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UserModule } from './user/user.module';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'sqlite',
      database: 'database.sqlite',
      entities: [__dirname + '/**/*.entity.{js,ts}'],
      synchronize: true,
    }),
    UserModule,
  ],
})
export class AppModule {}
```

#### 7. UserController 생성
컨트롤러에서 서비스를 사용하여 API를 노출합니다.

```typescript
// src/user/user.controller.ts
import { Controller, Get, Post, Delete, Param, Body } from '@nestjs/common';
import { UserService } from './user.service';
import { User } from './user.entity';

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get()
  findAll(): Promise<User[]> {
    return this.userService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: number): Promise<User | undefined> {
    return this.userService.findOne(id);
  }

  @Post()
  create(@Body() user: User): Promise<User> {
    return this.userService.create(user);
  }

  @Delete(':id')
  async remove(@Param('id') id: number): Promise<void> {
    await this.userService.remove(id);
  }
}
```

### 전체 코드 정리
이제 전체 코드를 정리해 보겠습니다.

1. **User 엔티티** (`user.entity.ts`)

```typescript
// src/user/user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;
}
```

2. **User 리포지토리** (`user.repository.ts`)

```typescript
// src/user/user.repository.ts
import { Repository, EntityRepository } from 'typeorm';
import { User } from './user.entity';

@EntityRepository(User)
export class UserRepository extends Repository<User> {
  async findByName(name: string): Promise<User | undefined> {
    return this.findOne({ where: { name } });
  }
}
```

3. **User 서비스** (`user.service.ts`)

```typescript
// src/user/user.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { UserRepository } from './user.repository';
import { User } from './user.entity';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(UserRepository)
    private readonly userRepository: UserRepository
  ) {}

  findAll(): Promise<User[]> {
    return this.userRepository.find();
  }

  findOne(id: number): Promise<User | undefined> {
    return this.userRepository.findOne(id);
  }

  findByName(name: string): Promise<User | undefined> {
    return this.userRepository.findByName(name);
  }

  async create(user: User): Promise<User> {
    return this.userRepository.save(user);
  }

  async remove(id: number): Promise<void> {
    await this.userRepository.delete(id);
  }
}
```

4. **User 컨트롤러** (`user.controller.ts`)

```typescript
// src/user/user.controller.ts
import { Controller, Get, Post, Delete, Param, Body } from '@nestjs/common';
import { UserService } from './user.service';
import { User } from './user.entity';

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get()
  findAll(): Promise<User[]> {
    return this.userService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: number): Promise<User | undefined> {
    return this.userService.findOne(id);
  }

  @Post()
  create(@Body() user: User): Promise<User> {
    return this.userService.create(user);
  }

  @Delete(':id')
  async remove(@Param('id') id: number): Promise<void> {
    await this.userService.remove(id);
  }
}
```

5. **User 모듈** (`user.module.ts`)

```typescript
// src/user/user.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UserService } from './user.service';
import { UserController } from './user.controller';
import { User } from './user.entity';
import { UserRepository } from './user.repository';

@Module({
  imports: [TypeOrmModule.forFeature([User, UserRepository])],
  providers: [UserService],
  controllers: [UserController],
  exports: [UserService],
})
export class UserModule {}
```

6. **App 모듈** (`app.module.ts`)

```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UserModule } from './user/user.module';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'sqlite',
      database: 'database.sqlite',
      entities: [__dirname + '/**/*.entity.{js,ts}'],
      synchronize: true,
    }),
    UserModule,
  ],
})
export class AppModule {}
```

### 실행 및 테스트
위의 코드를 작성한 후에 애플리케이션을 실행하고, REST 클라이언트(Postman, cURL 등)를 사용하여 다음과 같은 요청을 보낼 수 있습니다.

- **모든 사용자 조회**
  ```bash
  curl -X GET http://localhost:3000/users
  ```

- **특정 사용자 조회**
  ```bash
  curl -X GET http://localhost:3000/users/1
  ```

- **사용자 생성**
  ```bash
  curl -X POST http://localhost:
