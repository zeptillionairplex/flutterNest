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
  - [Docker로 MySQL 사용](https://poiemaweb.com/docker-mysql)
<details>
  <summary>docker MySQL에 데이터베이스가 생성 안될때</summary>
`typeorm`을 사용하여 NestJS 애플리케이션과 MySQL 데이터베이스를 Docker Compose로 동시에 실행하려는 경우, Docker Compose 설정 파일을 사용하면 쉽게 구성할 수 있습니다. 다음은 Docker Compose를 사용하여 MySQL과 NestJS 애플리케이션을 함께 실행하는 방법을 단계별로 설명하겠습니다.

### 1. 프로젝트 구조
프로젝트의 기본 구조는 다음과 같이 구성될 수 있습니다.

```
your-project/
│
├── src/
│   └── app.module.ts
├── .env
├── docker-compose.yml
├── Dockerfile
└── package.json
```

### 2. `docker-compose.yml` 파일 작성

먼저 `docker-compose.yml` 파일을 프로젝트 루트에 생성하고 아래 내용을 추가하세요.

```yaml
# docker-compose up --build
# docker-compose down -v
version: '3.8'
services:
  mysql:
    image: mysql:8.0.33
    container_name: mysql_container
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 1234
      MYSQL_DATABASE: shopping_mall
    ports:
      - '3306:3306'
    volumes:
      - mysql_data:/var/lib/mysql

  nestjs:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: nestjs_container
    restart: always
    environment:
      - NODE_ENV=development
      - DB_HOST=mysql
      - DB_PORT=3306
      - DB_USERNAME=root
      - DB_PASSWORD=1234
      - DB_DATABASE=shopping_mall
    depends_on:
      - mysql
    ports:
      - '3000:3000'
    volumes:
      - .:/usr/src/app

  ngrok:
    image: wernight/ngrok
    container_name: ngrok_container
    restart: always
    environment:
      NGROK_AUTH: "token" # 여기에 Ngrok 인증 토큰을 넣으세요
      NGROK_PORT: "nestjs:3000"
    depends_on:
      - nestjs
    ports:
      - '4040:4040'

volumes:
  mysql_data:
```

### 3. `Dockerfile` 파일 작성

프로젝트 루트에 `Dockerfile` 파일을 생성하고 다음 내용을 추가합니다.

```Dockerfile
# 베이스 이미지로 Node.js LTS 버전 사용
FROM node:18

# 작업 디렉토리 설정
WORKDIR /usr/src/app

# 패키지 설치를 위한 package.json과 package-lock.json 복사
COPY package*.json ./

# 패키지 설치
RUN npm install

# Install wait-for-it
RUN apt-get update && apt-get install -y netcat-openbsd jq

COPY wait-for-it.sh /usr/src/app/wait-for-it.sh
RUN chmod +x /usr/src/app/wait-for-it.sh

# Ngrok URL 출력 스크립트 복사
COPY get_ngrok_url.sh /usr/src/app/get_ngrok_url.sh
RUN chmod +x /usr/src/app/get_ngrok_url.sh

# 애플리케이션 소스 복사
COPY . .

# Nest CLI 설치
RUN npm install -g @nestjs/cli

# 빌드
RUN npm run build

# 애플리케이션 실행
# CMD ["sh", "-c", "./wait-for-it.sh mysql:3306 -- ./get_ngrok_url.sh && npm run start:dev"]
CMD ["./wait-for-it.sh", "mysql:3306", "--", "npm", "run", "start:dev", "&", "./get_ngrok_url.sh"]

# 앱 노출 포트
EXPOSE 3000
```
**tsconfig.json**
```typescript
// shopping-mall-backend/tsconfig.json
{
  "compilerOptions": {
    "module": "commonjs",
    "declaration": true,
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "target": "ES2021",
    "sourceMap": true,
    "outDir": "./dist",
    "baseUrl": "./",
    "incremental": true,
    "skipLibCheck": true,
    "strictNullChecks": false,
    "noImplicitAny": false,
    "strictBindCallApply": false,
    "forceConsistentCasingInFileNames": false,
    "noFallthroughCasesInSwitch": false,
    "strictPropertyInitialization": false
  },
  "include": ["src/**/*.ts"],
  "exclude": ["node_modules", "dist"]
}
```

### 4. 환경 변수 설정 (`.env` 파일)

루트 디렉터리에 `.env` 파일을 생성하고 다음 내용을 추가하세요.

```env
# MySQL 환경 변수
DB_HOST=mysql
DB_PORT=3306
DB_USERNAME=root
DB_PASSWORD=1234
DB_DATABASE=shopping_mall
```

### 5. NestJS 설정 (`app.module.ts`)

`src/app.module.ts` 파일에서 `TypeOrmModule` 설정을 업데이트하세요.

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ProductModule } from './product/product.module';
import * as dotenv from 'dotenv';

dotenv.config();

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT, 10),
      username: process.env.DB_USERNAME,
      password: process.env.DB_PASSWORD,
      database: process.env.DB_DATABASE,
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true,
    }),
    ProductModule,
  ],
})
export class AppModule {}
```

### 6. `package.json` 스크립트 수정

`package.json` 파일의 스크립트 부분을 다음과 같이 수정하세요.

```json
{
  "scripts": {
    "start": "nest start",
    "start:dev": "nest start --watch",
    "build": "nest build"
  }
}
```
### 7. shopping-mall-backend/get_ngrok_url.sh 파일 생성
```shell
#!/bin/bash

# Ngrok 인증 토큰 설정
export NGROK_AUTHTOKEN="token" # 여기에 ngrok 인증 토큰을 넣으세요

# 설정된 토큰을 사용하여 ngrok 인증
ngrok config add-authtoken $NGROK_AUTHTOKEN

# ngrok을 백그라운드에서 포트 3000에 대해 실행
nohup ngrok http 3000 &

# Wait for ngrok to start
sleep 10

# Fetch the public URL from ngrok API
NGROK_URL=$(curl --silent http://localhost:4040/api/tunnels | jq -r '.tunnels[0].public_url')

# Print the URL
echo "Ngrok URL: $NGROK_URL"
```

### 8. Docker Compose로 실행

이제 모든 설정이 완료되었습니다. Docker Compose를 사용하여 프로젝트를 실행하세요.

```bash
docker-compose up --build
```

이 명령이 실행된 후, MySQL 컨테이너와 NestJS 컨테이너가 동시에 작동하며, NestJS에서 MySQL과 연결하여 데이터베이스와 테이블을 생성할 수 있게 됩니다.

### 문제 해결

1. **포트 충돌 확인**: 다른 서비스가 이미 3306 또는 3000 포트를 사용하고 있는지 확인하세요.
2. **로그 확인**: `docker-compose` 서비스의 로그를 확인하여 어떤 오류가 발생하는지 파악하세요.

```bash
docker-compose logs -f
```

3. **컨테이너 상태 확인**:
컨테이너 상태를 확인하려면 다음과 같이 Docker 명령을 사용하세요.

### 1. 컨테이너 상태 확인

```bash
docker-compose ps
```

이 명령을 사용하면 현재 실행 중인 컨테이너의 상태를 확인할 수 있습니다.

### 2. 개별 컨테이너 상태 확인

특정 컨테이너의 상태 및 로그를 확인하려면 `docker ps`로 컨테이너 ID를 확인한 후 다음 명령을 사용하세요.

```bash
docker logs <컨테이너_ID 또는 컨테이너_이름>
```

예를 들어, `mysql_container`의 로그를 확인하려면 다음과 같이 합니다.

```bash
docker logs mysql_container
```

`nestjs_container`의 로그는 다음과 같이 확인할 수 있습니다.

```bash
docker logs nestjs_container
```

### 3. 컨테이너 안으로 들어가기

각 컨테이너 안으로 들어가서 직접 상태를 확인하려면 `docker exec` 명령을 사용하세요. 예를 들어, `mysql_container`에 들어가서 MySQL에 직접 접근하기 위해서는 다음과 같이 합니다.

```bash
docker exec -it mysql_container bash
```

그 후 MySQL에 로그인하려면 다음 명령을 사용합니다.

```bash
mysql -u root -p
```

그리고 MySQL 비밀번호를 입력한 후, 데이터베이스 및 테이블을 확인할 수 있습니다.

### 4. MySQL 데이터베이스 및 테이블 확인

MySQL에 로그인한 후 데이터베이스와 테이블을 확인하세요.

```sql
SHOW DATABASES;
USE shopping_mall;
SHOW TABLES;
```

### 5. NestJS 및 TypeORM 오류 확인

NestJS 컨테이너의 로그에서 `typeorm` 관련 오류가 발생하는지 확인하세요. 특히, `typeorm` 설정 파일이 `.env` 파일의 환경 변수를 정확히 읽어올 수 있도록 설정되어 있는지 확인해야 합니다.

### 6. 다른 문제 해결 방법

1. **네트워크 문제**: `nestjs` 컨테이너에서 MySQL에 연결할 수 있는지 확인하세요.

   ```bash
   docker exec -it nestjs_container bash
   ```

   안에서 `mysql` 컨테이너로 핑을 보내보세요.

   ```bash
   ping mysql
   ```

   또는 MySQL 연결을 테스트하세요.

   ```bash
   mysql -h mysql -u root -p
   ```

2. **MySQL 초기화 문제**: MySQL 컨테이너가 완전히 초기화되기 전에 `nestjs` 애플리케이션이 연결을 시도하는 경우가 있습니다. 이 경우 `depends_on` 옵션을 사용해도 문제가 해결되지 않을 수 있으므로 `wait-for-it` 또는 `wait-for`와 같은 스크립트를 사용하여 MySQL이 준비될 때까지 NestJS가 기다리도록 할 수 있습니다.

   `nestjs` 컨테이너의 `Dockerfile`에 `wait-for`를 설치하는 예제입니다.

   ```Dockerfile
   # Install wait-for-it
   RUN apt-get update && apt-get install -y netcat-openbsd
   COPY wait-for-it.sh /usr/src/app/wait-for-it.sh
   RUN chmod +x /usr/src/app/wait-for-it.sh
   ```

   그런 다음 `CMD` 또는 `ENTRYPOINT`를 다음과 같이 수정하세요.

   ```Dockerfile
   CMD ["./wait-for-it.sh", "mysql:3306", "--", "npm", "run", "start:dev"]
   ```

`wait-for-it.sh` 스크립트는 [여기](https://github.com/vishnubob/wait-for-it)에서 다운로드할 수 있습니다.

### 결론

위의 단계를 따라가면서 로그를 확인하고, 네트워크 및 환경 변수 관련 문제를 점검하면 MySQL과 NestJS가 정상적으로 작동하도록 설정할 수 있을 것입니다.
</details>  
<details>
  <summary>docker-compose로 데이터베이스는 생성되었지만 테이블은 생성되지 않는 문제 해결방법</summary>
구조를 살펴본 결과, 몇 가지 문제를 파악할 수 있습니다. 테이블이 생성되지 않는 문제는 여러 가지 원인에 의해 발생할 수 있습니다. 가능한 원인을 하나씩 확인하고 해결해 보겠습니다.

### 1. `typeorm` 설정 확인
아래의 `typeorm` 설정을 사용하고 있는 것으로 보입니다.

```typescript
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT, 10),
      username: process.env.DB_USERNAME,
      password: process.env.DB_PASSWORD,
      database: process.env.DB_DATABASE,
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true,
    }),
    ProductModule,
  ],
})
export class AppModule {}
```

#### 문제점 분석
1. `entities` 경로가 잘못되었거나, 엔티티 파일이 `__dirname` 경로에 없을 수 있습니다.
2. `synchronize` 옵션이 `true`로 설정되어 있어야 테이블이 자동으로 생성됩니다.
3. `ProductModule`에 엔티티가 제대로 포함되어 있는지 확인해야 합니다.

### 2. `nestjs` 코드 수정
1. `AppModule`에서 `ProductRepository`를 `ProductModule`에 포함하기 위해, `ProductModule`을 제대로 구성해야 합니다.

```typescript
// src/product/product.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { Product } from './product.entity';
import { ProductRepository } from './product.repository';

@Module({
  imports: [TypeOrmModule.forFeature([Product])],
  providers: [ProductRepository],
  exports: [TypeOrmModule],
})
export class ProductModule {}
```

2. `AppModule`에서 `ProductModule`을 가져옵니다.

```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ProductModule } from './product/product.module';
import * as dotenv from 'dotenv';

dotenv.config();

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT, 10),
      username: process.env.DB_USERNAME,
      password: process.env.DB_PASSWORD,
      database: process.env.DB_DATABASE,
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true,
    }),
    ProductModule,
  ],
})
export class AppModule {}
```

### 3. `docker-compose.yml` 파일의 문제 확인
1. `Dockerfile`을 확인합니다.
2. `.env` 파일이 제대로 동작하는지 확인합니다.

#### 3.1 `.env` 파일
`.env` 파일이 `nestjs` 서비스의 컨테이너에 올바르게 적용되었는지 확인합니다.

```env
// .env
DB_HOST=mysql
DB_PORT=3306
DB_USERNAME=root
DB_PASSWORD=1234
DB_DATABASE=shopping_mall
```

#### 3.2 `Dockerfile`
`nestjs` 서비스의 `Dockerfile`을 다음과 같이 작성합니다.

```Dockerfile
# Dockerfile
FROM node:16

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

RUN npm run build

CMD ["npm", "run", "start:dev"]
```

#### 3.3 `docker-compose.yml` 파일 확인
`docker-compose.yml` 파일을 확인하여 환경 변수가 제대로 전달되고 있는지 확인합니다.

```yaml
version: '3.8'
services:
  mysql:
    image: mysql:8.0.33
    container_name: mysql_container
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 1234
      MYSQL_DATABASE: shopping_mall
    ports:
      - '3306:3306'
    volumes:
      - mysql_data:/var/lib/mysql

  nestjs:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: nestjs_container
    restart: always
    environment:
      - NODE_ENV=development
      - DB_HOST=mysql
      - DB_PORT=3306
      - DB_USERNAME=root
      - DB_PASSWORD=1234
      - DB_DATABASE=shopping_mall
    depends_on:
      - mysql
    ports:
      - '3000:3000'
    volumes:
      - .:/app

volumes:
  mysql_data:
```

### 4. 전체 재빌드 및 실행

`docker-compose`를 다시 빌드하고 실행하여 동작을 확인합니다.

1. 기존 컨테이너와 볼륨을 정리합니다.

```bash
docker-compose down -v
```

2. `docker-compose`를 다시 빌드하고 실행합니다.

```bash
docker-compose up --build
```

### 5. 컨테이너 상태 및 로그 확인

`nestjs` 컨테이너의 로그를 확인하여 오류 메시지가 없는지 확인합니다.

```bash
docker logs nestjs_container
```

컨테이너가 제대로 실행되고 있는지 확인합니다.

```bash
docker ps
```

### 6. 데이터베이스 확인

`mysql` 컨테이너 내부에 접속하여 `shopping_mall` 데이터베이스에 테이블이 생성되었는지 확인합니다.

1. `mysql_container`에 접속합니다.

```bash
docker exec -it mysql_container mysql -u root -p
```

2. 비밀번호를 입력하고 `shopping_mall` 데이터베이스로 이동합니다.

```sql
mysql> USE shopping_mall;
```

3. 테이블 목록을 확인합니다.

```sql
mysql> SHOW TABLES;
```

`Product` 엔티티에 대응하는 테이블이 생성되어 있어야 합니다.

### 7. 문제 해결을 위한 추가 확인 사항

1. **엔티티 파일 경로**: `typeorm` 설정에서 `entities` 경로가 올바른지 확인합니다.
   - 예: `entities: [__dirname + '/**/*.entity{.ts,.js}']`

2. **환경 변수 확인**: `nestjs` 컨테이너 내에서 환경 변수가 제대로 로드되었는지 확인합니다.
   - `docker exec -it nestjs_container printenv DB_HOST`

3. **TypeORM 디버그 로그 활성화**: `TypeOrmModule.forRoot` 설정에서 `logging: true`를 추가하여 디버그 로그를 확인합니다.

```typescript
TypeOrmModule.forRoot({
  type: 'mysql',
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT, 10),
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_DATABASE,
  entities: [__dirname + '/**/*.entity{.ts,.js}'],
  synchronize: true,
  logging: true, // 추가
}),
```

4. **네스트 애플리케이션 디버그 로그 활성화**: `main.ts`에서 `Logging`을 활성화합니다.

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, { logger: ['error', 'warn', 'log'] });
  await app.listen(3000);
}
bootstrap();
```

5. 로그를 확인하여 문제의 원인을 식별하고, 오류 메시지를 기반으로 문제를 해결합니다.

### 8. 추가 문제 해결 방법

- **`synchronize` 대신 마이그레이션 사용**: `synchronize: true` 대신 마이그레이션을 사용하여 테이블을 관리합니다.

- **Docker Compose 의존성 관리**: `depends_on`으로 서비스 간 의존성을 지정하지만, 서비스가 완전히 준비될 때까지 기다리지 않으므로, `wait-for-it` 또는 `wait-for-x` 같은 도구를 사용하여 데이터베이스에 연결할 수 있을 때까지 기다리도록 설정합니다.

```yaml
    command: ["wait-for-it", "mysql:3306", "--", "npm", "run", "start:dev"]
```

`wait-for-it` 스크립트를 컨테이너에 복사하고 실행합니다.

### 결론

위 단계를 통해 문제를 확인하고 해결할 수 있을 것입니다. 만약 그래도 문제가 해결되지 않는다면, `docker-compose` 로그와 `nestjs` 컨테이너의 로그를 확인하여 문제의 원인을 파악하는 것이 중요합니다.
</details>

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
import { Module, OnModuleInit } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { ProductModule } from './product/product.module';
import { NgrokService } from './ngrok/ngrok.service';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true, // 모든 모듈에서 전역적으로 사용 가능
    }),
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        type: 'mysql',
        host: configService.get<string>('DB_HOST'),
        port: configService.get<number>('DB_PORT'),
        username: configService.get<string>('DB_USERNAME'),
        password: configService.get<string>('DB_PASSWORD'),
        database: configService.get<string>('DB_DATABASE'),
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: true, // 개발 환경에서만 사용하세요. 프로덕션에서는 false로 설정하세요.
        logging: true,
      }),
    }),
    ProductModule,
  ],
  providers: [NgrokService],
})
export class AppModule implements OnModuleInit {
  constructor(private readonly ngrokService: NgrokService) {}

  async onModuleInit() {
    await this.ngrokService.getNgrokUrl();
  }
}
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
import { CreateProductDto } from './create-product.dto';
import { UpdateProductDto } from './update-product.dto';

@Injectable()
export class ProductService {
    constructor(
    @InjectRepository(ProductRepository)
        private readonly productRepository: ProductRepository,
    ) {}

    async findAll(): Promise<Product[]> {
        return await this.productRepository.find();
    }

    async findOne(id: number): Promise<Product> {
        return await this.productRepository.findOne({ where: { id } });
    }

    async create(productDto: CreateProductDto): Promise<Product> {
        const newProduct = this.productRepository.create(productDto);
            return await this.productRepository.save(newProduct);
    }

    async update(id: number, updateProductDto: UpdateProductDto): Promise<Product> {
        await this.productRepository.update(id, updateProductDto);
            return await this.findOne(id);
    }

    async remove(id: number): Promise<void> {
        await this.productRepository.delete(id);
    }
}
```

<details>
<summary>Type 'number' has no properties in common with type 'FindOneOptions'.ts(2559) 해결방법</summary>  
에러 메시지 Type `number` has no properties in common with type `FindOneOptions<Product>.ts(2559)`는 findOne 메서드에 넘겨주는 매개변수가 number 타입인 것에 문제가 있다는 것을 의미합니다. findOne 메서드는 TypeORM의 0.3.x 버전에서 `FindOneOptions`나 `FindOptionsWhere`와 같은 객체를 필요로 합니다.

이전 TypeORM 버전 0.2.x에서는 findOne 메서드가 단일 ID를 받아서 사용했지만, 0.3.x부터는 옵션 객체를 필요로 하게 되었습니다.

해결 방법
findOne 메서드에 옵션 객체를 전달하도록 코드를 수정해야 합니다.
</details>

**create-product.dto.ts**
```typescript
// src/product/create-product.dto.ts
export class CreateProductDto {
    readonly name: string;
    readonly price: number;
    readonly description: string;
    readonly imageUrl: string;
}
```

**update-product.dto.ts**
```typescript
// src/product/update-product.dto.ts
export class UpdateProductDto {
    readonly name?: string;
    readonly price?: number;
    readonly description?: string;
    readonly imageUrl?: string;
}
```
**ngrok.service.ts**
```typescript
// src/ngrok/ngrok.service.ts
import { Injectable, Logger } from '@nestjs/common';
import axios from 'axios'; //npm install axios

@Injectable()
export class NgrokService {
    private readonly logger = new Logger(NgrokService.name);

    async getNgrokUrl(): Promise<void> {
    try {
        // const response = await axios.get('http://localhost:4040/api/tunnels');
        // ngrok_container의 경우 docker-compose로 [mysql/shopping-mall/ngrok] docker image가 한대 묶여서
        // 하나의 container안에 있고, ngrok docker의 이름이 "ngrok_container"이며, "localhost"가 아닌
        // "ngrok_container"라고해야 에러가 발생하지 않는다.
        const response = await axios.get('http://ngrok_container:4040/api/tunnels');
        const publicUrl = response.data.tunnels[0].public_url;
        this.logger.log(`Ngrok URL: ${publicUrl}`);
    } catch (error) {
            this.logger.error('Error fetching ngrok URL', error.toString());
        }
    }
}
```

**6. 상품 컨트롤러(Controller) 구현**
```typescript
// src/product/product.controller.ts
import { Controller, Get, Post, Put, Delete, Param, Body } from '@nestjs/common';
import { ProductService } from './product.service';
import { CreateProductDto } from './create-product.dto';
import { UpdateProductDto } from './update-product.dto';
import { Product } from './product.entity';

@Controller('products')
export class ProductController {
    constructor(private readonly productService: ProductService) {}

    @Get()
    async findAll(): Promise<Product[]> {
        return await this.productService.findAll();
    }

    @Get(':id')
    async findOne(@Param('id') id: number): Promise<Product> {
        return await this.productService.findOne(id);
    }

    @Post()
    async create(@Body() createProductDto: CreateProductDto): Promise<Product> {
        return await this.productService.create(createProductDto);
    }

    @Put(':id')
    async update(
    @Param('id') id: number,
    @Body() updateProductDto: UpdateProductDto,
    ): Promise<Product> {
        return await this.productService.update(id, updateProductDto);
    }

    @Delete(':id')
    async remove(@Param('id') id: number): Promise<void> {
        return await this.productService.remove(id);
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
// import { Product } from './product.entity';
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
<details>
  <summary>`.g.dart` 파일이 생성되지 않는 문제 해결 방법</summary>
  `.g.dart` 파일이 생성되지 않는 문제는 주로 `build_runner`와 `json_serializable` 패키지 설정 오류 또는 의존성 누락에서 발생합니다. 해결 방법은 다음과 같습니다:

### 1. `pubspec.yaml` 파일 설정
`build_runner`와 `json_serializable` 패키지를 `dev_dependencies`에 추가합니다.

```yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.0.5
  http:
  json_annotation: ^4.8.0
  flutter_dotenv: ^5.0.2
  fluttertoast: ^8.2.1

dev_dependencies:
  build_runner: ^2.3.3
  json_serializable: ^6.6.1
```

### 2. `build_runner` 실행
여기서는 `flutter pub run`을 사용하여 `build_runner`를 실행합니다.

#### 터미널 명령어
터미널에서 아래 명령어를 실행하세요.

```bash
flutter pub get
flutter pub run build_runner build
```

### 3. `.g.dart` 파일 재생성
위 명령어로도 문제가 해결되지 않는다면, `.dart_tool` 폴더와 `build` 폴더를 삭제한 후 다시 `build_runner`를 실행해보세요.

#### 캐시 및 빌드 폴더 삭제 후 재생성
1. `.dart_tool`과 `build` 폴더를 삭제하세요.
2. 다음 명령어를 실행하세요.

```bash
flutter pub get
flutter pub run build_runner clean
flutter pub run build_runner build
```

### 4. `product.dart` 파일 예제
아래 예시처럼 `json_serializable` 관련 어노테이션을 사용합니다.

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

  Product({
    this.id,
    required this.name,
    required this.price,
    required this.description,
    required this.imageUrl,
  });

  factory Product.fromJson(Map<String, dynamic> json) => _$ProductFromJson(json);

  Map<String, dynamic> toJson() => _$ProductToJson(this);
}
```

### 문제 해결 팁
- **버전 호환성 문제**: `build_runner`와 `json_serializable` 버전 간의 호환성 문제가 발생할 수 있으므로, 사용 중인 Flutter 버전에 맞는 최신 버전을 사용하세요.
- **어노테이션 오류**: 클래스에 `@JsonSerializable()` 어노테이션을 정확히 사용했는지 확인하세요.
- **터미널 경로 확인**: 터미널에서 프로젝트의 루트 디렉토리로 이동한 후 명령어를 실행하세요.

이 과정을 따라도 `.g.dart` 파일이 생성되지 않는다면, 오류 메시지를 공유해주시면 추가적인 문제 해결을 도와드리겠습니다.
</details>

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

**24. 관리자 페이지 작성**

관리자 페이지에서는 상품 관리, 주문 관리, 사용자 관리 등을 수행할 수 있도록 구현합니다.

**1. 관리자 페이지 메인**

- `lib/pages/admin/admin_main_page.dart` 파일 작성:

```dart
// lib/pages/admin/admin_main_page.dart
import 'package:flutter/material.dart';
import 'admin_product_page.dart';
import 'admin_order_page.dart';
import 'admin_user_page.dart';

class AdminMainPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Admin Dashboard')),
      body: ListView(
        children: [
          ListTile(
            title: Text('Product Management'),
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(builder: (context) => AdminProductPage()),
              );
            },
          ),
          ListTile(
            title: Text('Order Management'),
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(builder: (context) => AdminOrderPage()),
              );
            },
          ),
          ListTile(
            title: Text('User Management'),
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(builder: (context) => AdminUserPage()),
              );
            },
          ),
        ],
      ),
    );
  }
}
```

**2. 상품 관리 페이지**

- `lib/pages/admin/admin_product_page.dart` 파일 작성:

```dart
// lib/pages/admin/admin_product_page.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../../providers/product_provider.dart';
import '../../models/product.dart';

class AdminProductPage extends StatelessWidget {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _priceController = TextEditingController();
  final _descriptionController = TextEditingController();
  final _imageUrlController = TextEditingController();

  void _clearForm() {
    _nameController.clear();
    _priceController.clear();
    _descriptionController.clear();
    _imageUrlController.clear();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Product Management')),
      body: Consumer<ProductProvider>(
        builder: (context, productProvider, child) {
          return Column(
            children: [
              Expanded(
                child: FutureBuilder(
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
                            trailing: IconButton(
                              icon: Icon(Icons.delete),
                              onPressed: () {
                                productProvider.deleteProduct(product.id!);
                              },
                            ),
                          );
                        },
                      );
                    }
                  },
                ),
              ),
              Padding(
                padding: const EdgeInsets.all(8.0),
                child: Form(
                  key: _formKey,
                  child: Column(
                    children: [
                      TextFormField(
                        controller: _nameController,
                        decoration: InputDecoration(labelText: 'Product Name'),
                        validator: (value) {
                          if (value == null || value.isEmpty) {
                            return 'Please enter a product name';
                          }
                          return null;
                        },
                      ),
                      TextFormField(
                        controller: _priceController,
                        decoration: InputDecoration(labelText: 'Price'),
                        keyboardType: TextInputType.number,
                        validator: (value) {
                          if (value == null || value.isEmpty) {
                            return 'Please enter a price';
                          }
                          if (double.tryParse(value) == null) {
                            return 'Please enter a valid number';
                          }
                          return null;
                        },
                      ),
                      TextFormField(
                        controller: _descriptionController,
                        decoration: InputDecoration(labelText: 'Description'),
                        validator: (value) {
                          if (value == null || value.isEmpty) {
                            return 'Please enter a description';
                          }
                          return null;
                        },
                      ),
                      TextFormField(
                        controller: _imageUrlController,
                        decoration: InputDecoration(labelText: 'Image URL'),
                        validator: (value) {
                          if (value == null || value.isEmpty) {
                            return 'Please enter an image URL';
                          }
                          return null;
                        },
                      ),
                      SizedBox(height: 16),
                      ElevatedButton(
                        onPressed: () {
                          if (_formKey.currentState!.validate()) {
                            final product = Product(
                              name: _nameController.text,
                              price: double.parse(_priceController.text),
                              description: _descriptionController.text,
                              imageUrl: _imageUrlController.text,
                            );

                            productProvider.createProduct(product);
                            _clearForm();
                            ScaffoldMessenger.of(context).showSnackBar(
                              SnackBar(content: Text('Product added successfully!')),
                            );
                          }
                        },
                        child: Text('Add Product'),
                      ),
                    ],
                  ),
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

**3. 주문 관리 페이지**

- `lib/pages/admin/admin_order_page.dart` 파일 작성:

```dart
// lib/pages/admin/admin_order_page.dart
import 'package:flutter/material.dart';

class AdminOrderPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Order Management')),
      body: ListView(
        children: [
          ListTile(
            title: Text('Order #1'),
            subtitle: Text('Status: Delivered'),
            trailing: Icon(Icons.check_circle, color: Colors.green),
          ),
          ListTile(
            title: Text('Order #2'),
            subtitle: Text('Status: Pending'),
            trailing: Icon(Icons.hourglass_empty, color: Colors.orange),
          ),
          ListTile(
            title: Text('Order #3'),
            subtitle: Text('Status: Cancelled'),
            trailing: Icon(Icons.cancel, color: Colors.red),
          ),
        ],
      ),
    );
  }
}
```

**4. 사용자 관리 페이지**

- `lib/pages/admin/admin_user_page.dart` 파일 작성:

```dart
// lib/pages/admin/admin_user_page.dart
import 'package:flutter/material.dart';

class AdminUserPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('User Management')),
      body: ListView(
        children: [
          ListTile(
            title: Text('User #1'),
            subtitle: Text('Role: Admin'),
            trailing: Icon(Icons.admin_panel_settings, color: Colors.blue),
          ),
          ListTile(
            title: Text('User #2'),
            subtitle: Text('Role: Customer'),
            trailing: Icon(Icons.person, color: Colors.green),
          ),
        ],
      ),
    );
  }
}
```

**5. 메인 앱에 관리자 페이지 추가**

- `lib/pages/main_page.dart` 파일 수정:

```dart
// lib/pages/main_page.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/product_provider.dart';
import 'product_detail_page.dart';
import 'cart_page.dart';
import 'account_page.dart';
import 'admin/admin_main_page.dart';

class MainPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Shopping Mall'),
        actions: [
          IconButton(
            icon: Icon(Icons.admin_panel_settings),
            onPressed: () {
              Navigator.push(
                context,
                MaterialPageRoute(builder: (context) => AdminMainPage()),
              );
            },
          ),
        ],
      ),
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

**6. 최종 구조**

최종 프로젝트 구조는 아래와 같습니다:

```
shopping_mall_frontend/
├── lib/
│   ├── models/
│   │   ├── cart_item.dart
│   │   └── product.dart
│   ├── pages/
│   │
│   │   ├── admin/
│   │   │   ├── admin_main_page.dart
│   │   │   ├── admin_order_page.dart
│   │   │   ├── admin_product_page.dart
│   │   │   └── admin_user_page.dart
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

**7. 관리자 페이지 확장 아이디어**

- **주문 관리 확장**: 주문의 상태를 변경하고, 주문 세부 정보를 확인할 수 있도록 기능 추가.
- **사용자 관리 확장**: 사용자 등급을 변경하거나, 사용자 정보를 수정할 수 있는 기능 추가.
- **상품 관리 확장**: 상품 이미지를 업로드할 수 있는 기능 추가.
- **대시보드**: 관리자 페이지에 통계 및 각종 지표를 확인할 수 있는 대시보드 추가.

### 예시 코드 확장

**1. 주문 관리 확장**

- `lib/pages/admin/admin_order_page.dart` 파일 수정:

```dart
// lib/pages/admin/admin_order_page.dart
import 'package:flutter/material.dart';

class AdminOrderPage extends StatelessWidget {
  final List<Map<String, dynamic>> _orders = [
    {'id': 1, 'status': 'Delivered', 'items': 3, 'total': 120.00},
    {'id': 2, 'status': 'Pending', 'items': 1, 'total': 50.00},
    {'id': 3, 'status': 'Cancelled', 'items': 2, 'total': 80.00},
  ];

  void _changeOrderStatus(BuildContext context, int orderId, String newStatus) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Order #$orderId status changed to $newStatus')),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Order Management')),
      body: ListView.builder(
        itemCount: _orders.length,
        itemBuilder: (context, index) {
          final order = _orders[index];
          return ListTile(
            title: Text('Order #${order['id']}'),
            subtitle: Text('Status: ${order['status']} | Items: ${order['items']} | Total: \$${order['total']}'),
            trailing: PopupMenuButton<String>(
              onSelected: (value) => _changeOrderStatus(context, order['id'], value),
              itemBuilder: (context) => [
                PopupMenuItem(value: 'Delivered', child: Text('Delivered')),
                PopupMenuItem(value: 'Pending', child: Text('Pending')),
                PopupMenuItem(value: 'Cancelled', child: Text('Cancelled')),
              ],
            ),
          );
        },
      ),
    );
  }
}
```

**2. 사용자 관리 확장**

- `lib/pages/admin/admin_user_page.dart` 파일 수정:

```dart
// lib/pages/admin/admin_user_page.dart
import 'package:flutter/material.dart';

class AdminUserPage extends StatelessWidget {
  final List<Map<String, dynamic>> _users = [
    {'id': 1, 'name': 'Admin User', 'role': 'Admin'},
    {'id': 2, 'name': 'Regular User', 'role': 'Customer'},
  ];

  void _changeUserRole(BuildContext context, int userId, String newRole) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('User #$userId role changed to $newRole')),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('User Management')),
      body: ListView.builder(
        itemCount: _users.length,
        itemBuilder: (context, index) {
          final user = _users[index];
          return ListTile(
            title: Text(user['name']),
            subtitle: Text('Role: ${user['role']}'),
            trailing: PopupMenuButton<String>(
              onSelected: (value) => _changeUserRole(context, user['id'], value),
              itemBuilder: (context) => [
                PopupMenuItem(value: 'Admin', child: Text('Admin')),
                PopupMenuItem(value: 'Customer', child: Text('Customer')),
                PopupMenuItem(value: 'Moderator', child: Text('Moderator')),
              ],
            ),
          );
        },
      ),
    );
  }
}
```

### 결론

이렇게 관리자 페이지를 추가함으로써 쇼핑몰 앱의 기능이 크게 확장되었습니다. 이제 상품 관리, 주문 관리, 사용자 관리 등 다양한 기능을 제공하는 완전한 쇼핑몰 앱이 되었습니다. 

**최종 프로젝트 구조**

```
shopping_mall_frontend/
├── lib/
│   ├── models/
│   │   ├── cart_item.dart
│   │   └── product.dart
│   ├── pages/
│   │   ├── admin/
│   │   │   ├── admin_main_page.dart
│   │   │   ├── admin_order_page.dart
│   │   │   ├── admin_product_page.dart
│   │   │   └── admin_user_page.dart
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

**확장 아이디어**

- **통계 및 분석**: 관리자 페이지에 매출, 사용자 활동, 주문 상태 등의 통계를 시각화하여 표시.
- **상품 검색 및 필터링**: 상품 목록을 다양한 기준으로 필터링할 수 있는 검색 기능 추가.
- **알림 시스템**: 신규 주문, 환불 요청 등 중요한 이벤트에 대한 알림 시스템 구축.

**마무리**

이 매뉴얼을 따라 Flutter 프론트엔드 및 Nest.js 백엔드로 쇼핑몰 프로젝트를 완성할 수 있습니다. 앱의 모든 기능을 확장하고 다양한 추가 기능을 더하여 완전한 쇼핑몰 플랫폼을 구축해보세요.
