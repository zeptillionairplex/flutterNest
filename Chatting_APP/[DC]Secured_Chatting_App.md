## Comprehensive Manual for Building a Decentralized Chat Application

This manual provides detailed instructions for developing a serverless, secure chat application using Web3.0 technologies. The application uses IPFS, PubSub, WebRTC/Libp2p, and DID authentication. The frontend is built with Flutter, and the backend uses NestJS.

### Table of Contents

1. [Project Structure](#project-structure)
2. [Backend Setup (NestJS)](#backend-setup-nestjs)
   - [Install Dependencies](#install-dependencies)
   - [Configure Swagger](#configure-swagger)
   - [Auth Module](#auth-module)
   - [Chat Module](#chat-module)
   - [Users Module](#users-module)
   - [Common Services](#common-services)
3. [Frontend Setup (Flutter)](#frontend-setup-flutter)
   - [Install Dependencies](#install-dependencies-1)
   - [Project Structure](#project-structure-1)
   - [Models](#models)
   - [Providers](#providers)
   - [Repositories](#repositories)
   - [Services](#services)
   - [Screens](#screens)
   - [Widgets](#widgets)
4. [Docker Integration](#docker-integration)
5. [Running the Application](#running-the-application)
6. [ENV Setting](#env-setting)

---

## Project Structure

### Backend (NestJS)

```
secure-chat-app/
├── src/
│   ├── auth/
│   │   ├── auth.controller.ts
│   │   ├── auth.module.ts
│   │   ├── auth.service.ts
│   │   ├── auth.guard.ts
│   │   ├── dto/
│   │   │   ├── login.dto.ts
│   │   │   ├── register.dto.ts
│   │   └── strategies/
│   │       ├── jwt.strategy.ts
│   │       ├── local.strategy.ts
│   ├── chat/
│   │   ├── chat.controller.ts
│   │   ├── chat.gateway.ts
│   │   ├── chat.module.ts
│   │   ├── chat.service.ts
│   │   ├── schemas/
│   │   │   ├── message.schema.ts
│   │   ├── dto/
│   │   │   ├── create-message.dto.ts
│   │   └── interfaces/
│   │       ├── message.interface.ts
│   ├── common/
│   │   ├── encryption.service.ts
│   │   ├── ipfs.service.ts
│   │   ├── notification.service.ts
│   │   ├── did.service.ts
│   │   ├── libp2p.service.ts
│   ├── users/
│   │   ├── users.controller.ts
│   │   ├── users.module.ts
│   │   ├── users.service.ts
│   │   ├── schemas/
│   │   │   ├── user.schema.ts
│   │   ├── dto/
│   │   │   ├── create-user.dto.ts
│   │   └── interfaces/
│   │       ├── user.interface.ts
│   ├── repositories/
│   │   ├── user.repository.ts
│   │   ├── message.repository.ts
│   ├── app.module.ts
│   ├── main.ts
├── .env.development
├── .env.production
├── Dockerfile
├── docker-compose.yml
├── package.json
├── tsconfig.json
└── README.md
```

### Frontend (Flutter)

```
secure_chat_app/
├── lib/
│   ├── models/
│   │   ├── message_dto.dart
│   │   ├── user_dto.dart
│   ├── providers/
│   │   ├── message_provider.dart
│   │   ├── user_provider.dart
│   ├── repositories/
│   │   ├── message_repository.dart
│   │   ├── user_repository.dart
│   ├── screens/
│   │   ├── chat_room_screen.dart
│   │   ├── login_screen.dart
│   ├── services/
│   │   ├── encryption_service.dart
│   │   ├── ipfs_service.dart
│   │   ├── webrtc_service.dart
│   │   ├── did_service.dart
│   ├── widgets/
│   │   ├── message_bubble.dart
│   ├── main.dart
├── assets/
│   ├── images/
├── .env.development
├── .env.production
├── pubspec.yaml
├── pubspec.lock
└── README.md
```

---

## Backend Setup (NestJS)

### Install Dependencies

네, NestJS CLI를 사용하여 파일 트리에 맞게 모듈, 컨트롤러, 서비스, 그리고 기타 필요한 파일들을 생성하는 명령어들을 정리해드리겠습니다.

### 1. 프로젝트 생성
```sh
nest new secure-chat-app
cd secure-chat-app
```

### 2. 모듈 생성
```sh
nest g module auth
nest g module chat
nest g module users
nest g module common
nest g module repositories
```

### 3. 컨트롤러 생성
```sh
nest g controller auth
nest g controller chat
nest g controller users
```

### 4. 서비스 생성
```sh
nest g service auth
nest g service chat
nest g service users
nest g service common
nest g service repositories
```

### 5. 게이트웨이 생성
```sh
nest g gateway chat
```

### 6. 기타 필요한 파일 생성
이 파일들은 NestJS CLI로 자동 생성되지 않으므로 수동으로 만들어야 합니다. VS Code 터미널에서 다음 명령어를 사용하여 빈 파일을 생성할 수 있습니다.

```sh
New-Item -ItemType Directory -Force -Path .\src\auth\dto
New-Item -ItemType Directory -Force -Path .\src\auth\strategies
New-Item -ItemType Directory -Force -Path .\src\chat\dto
New-Item -ItemType Directory -Force -Path .\src\chat\interfaces
New-Item -ItemType Directory -Force -Path .\src\common
New-Item -ItemType Directory -Force -Path .\src\users\dto
New-Item -ItemType Directory -Force -Path .\src\users\interfaces

# auth 관련 파일들
type nul > src/auth/dto/login.dto.ts
type nul > src/auth/dto/register.dto.ts
type nul > src/auth/strategies/jwt.strategy.ts
type nul > src/auth/strategies/local.strategy.ts

# chat 관련 파일들
type nul > src/chat/dto/create-message.dto.ts
type nul > src/chat/interfaces/message.interface.ts
type nul > src/chat/schemas/message.schema.ts

# users 관련 파일들
type nul > src/users/dto/create-user.dto.ts
type nul > src/users/interfaces/user.interface.ts
type nul > src/users/schemas/user.schema.ts

# 기타 파일들
type nul > src/repositories/user.repository.ts
type nul > src/repositories/message.repository.ts

# 루트 디렉터리의 파일들
type nul > .env.development
type nul > .env.production
type nul > Dockerfile
type nul > docker-compose.yml
type nul > package.json
type nul > tsconfig.json
type nul > README.md
```

```bash
npm install @nestjs/swagger swagger-ui-express
npm install @nestjs/mongoose mongoose
npm install @nestjs/passport passport passport-local passport-jwt @nestjs/jwt
npm install crypto-js
npm install ipfs-http-client
npm install libp2p libp2p-webrtc-star
npm install did-jwt did-resolver key-did-resolver
npm install @nestjs/schedule node-notifier
npm install dotenv
npm install class-validator
```

### Configure Swagger

**`src/main.ts`**

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle('Decentralized Chat App')
    .setDescription('The chat app API description')
    .setVersion('1.0')
    .addBearerAuth()
    .build();
  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document);

  await app.listen(3000);
}
bootstrap();
```

### Integrate Schemas with Mongoose in NestJS Modules

Ensure that your `AppModule` or respective feature modules are set up to use these schemas.

**Path: `src/app.module.ts`**

```typescript
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { ConfigModule } from '@nestjs/config';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { AuthModule } from './auth/auth.module';
import { ChatModule } from './chat/chat.module';
import { UsersModule } from './users/users.module';
import { CommonModule } from './common/common.module';
import { UserSchema } from './users/schemas/user.schema';
import { MessageSchema } from './chat/schemas/message.schema';

@Module({
  imports: [
    ConfigModule.forRoot(),
    MongooseModule.forRoot(process.env.MONGO_URI),
    MongooseModule.forFeature([
      { name: 'User', schema: UserSchema },
      { name: 'Message', schema: MessageSchema },
    ]),
    AuthModule,
    ChatModule,
    UsersModule,
    CommonModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}

```

### Auth Module

**`src/auth/auth.controller.ts`**

```typescript
import { Controller, Post, Body, UnauthorizedException } from '@nestjs/common';
import { AuthService } from './auth.service';
import { CreateUserDto } from '../dto/create-user.dto';

@Controller('auth')
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Post('register')
  async register(@Body() createUserDto: CreateUserDto) {
    return this.authService.register(createUserDto);
  }

  @Post('login')
  async login(@Body() body: { did: string, payload: any, signer: any }) {
    return this.authService.login(body.did, body.payload, body.signer);
  }

  @Post('verify')
  async verify(@Body() body: { token: string }) {
    try {
      return await this.authService.verify(body.token);
    } catch (err) {
      throw new UnauthorizedException();
    }
  }
}
```

**`src/auth/auth.module.ts`**

```typescript
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';
import { UserRepository } from '../repositories/user.repository';
import { DidService } from '../common/did.service';

@Module({
  controllers: [AuthController],
  providers: [AuthService, UserRepository, DidService],
})
export class AuthModule {}
```

**`src/auth/auth.service.ts`**

```typescript
import { Injectable } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersService } from '../users/users.service';
import { DidService } from '../common/did.service';
import { CreateUserDto } from '../dto/create-user.dto';

@Injectable()
export class AuthService {
  constructor(
    private readonly jwtService: JwtService,
    private readonly usersService: UsersService,
    private readonly didService: DidService,
  ) {}

  async validateUser(username: string, pass: string): Promise<any> {
    const user = await this.usersService.findOne(username);
    if (user && user.password === pass) {
      const { password, ...result } = user;
      return result;
    }
    return null;
  }

  async register(user: CreateUserDto) {
    return this.usersService.create(user);
  }

  async login(user: any) {
    const payload = { username: user.username, sub: user.userId };
    return {
      access_token: this.jwtService.sign(payload),
    };
  }

  async verify(token: string) {
    const verified = await this.didService.verifyJWT(token);
    return verified;
  }
}
```

### DTO Files

#### `login.dto.ts`

**Path: `src/auth/dto/login.dto.ts`**

This DTO defines the structure of the data required for the login request.

```typescript
import { IsString } from 'class-validator';

export class LoginDto {
  @IsString()
  readonly username: string;

  @IsString()
  readonly password: string;
}
```

#### `register.dto.ts`

**Path: `src/auth/dto/register.dto.ts`**

This DTO defines the structure of the data required for the registration request.

```typescript
import { IsString } from 'class-validator';

export class RegisterDto {
  @IsString()
  readonly username: string;

  @IsString()
  readonly password: string;

  @IsString()
  readonly did: string;
}
```

### Strategy Files

#### `jwt.strategy.ts`

**Path: `src/auth/strategies/jwt.strategy.ts`**

This strategy validates JWT tokens to authenticate users.

```typescript
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { ConfigService } from '@nestjs/config';
import { UsersService } from '../../users/users.service';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    private readonly configService: ConfigService,
    private readonly usersService: UsersService,
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.get<string>('JWT_SECRET'),
    });
  }

  async validate(payload: any) {
    const user = await this.usersService.findOne(payload.username);
    if (!user) {
      throw new UnauthorizedException();
    }
    return user;
  }
}
```

#### `local.strategy.ts`

**Path: `src/auth/strategies/local.strategy.ts`**

This strategy handles user authentication using a username and password.

```typescript
import { Strategy } from 'passport-local';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { AuthService } from '../auth.service';

@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(private authService: AuthService) {
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

### Chat Module

**`src/chat/chat.controller.ts`**

```typescript
import { Controller, Post, Body, Get, Param } from '@nestjs/common';
import { ChatService } from './chat.service';
import { CreateMessageDto } from '../dto/create-message.dto';

@Controller('messages')
export class ChatController {
  constructor(private readonly chatService: ChatService) {}

  @Post()
  async create(@Body() createMessageDto: CreateMessageDto) {
    return this.chatService.sendMessage(createMessageDto);
  }

  @Get(':chatRoomId')
  async findAll(@Param('chatRoomId') chatRoomId: number) {
    return this.chatService.getMessages(chatRoomId);
  }
}
```

**`src/chat/chat.gateway.ts`**

```typescript
import { WebSocketGateway, WebSocketServer, SubscribeMessage, MessageBody } from '@nestjs/websockets';
import { Server } from 'socket.io';
import { ChatService } from './chat.service';

@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  constructor(private readonly chatService: ChatService) {}

  @SubscribeMessage('joinRoom')
  async handleJoinRoom(@MessageBody() data: { chatRoomId: number }) {
    const topic = `room-${data.chatRoomId}`;
    await this.chat

Service.receiveMessages((msg: string) => {
      this.server.to(topic).emit('message', msg);
    });
    this.server.socketsJoin(topic);
  }

  @SubscribeMessage('sendMessage')
  async handleMessage(@MessageBody() message: { peerId: string, content: string }) {
    await this.chatService.sendMessage(message);
  }
}
```

**`src/chat/chat.module.ts`**

```typescript
import { Module } from '@nestjs/common';
import { ChatService } from './chat.service';
import { ChatController } from './chat.controller';
import { ChatGateway } from './chat.gateway';
import { MessageRepository } from '../repositories/message.repository';
import { IpfsService } from '../common/ipfs.service';
import { NotificationService } from '../common/notification.service';
import { Libp2pService } from '../common/libp2p.service';

@Module({
  controllers: [ChatController],
  providers: [ChatService, ChatGateway, MessageRepository, IpfsService, NotificationService, Libp2pService],
})
export class ChatModule {}
```

**`src/chat/chat.service.ts`**

```typescript
import { Injectable } from '@nestjs/common';
import { CreateMessageDto } from '../dto/create-message.dto';
import { MessageRepository } from '../repositories/message.repository';
import { IpfsService } from '../common/ipfs.service';
import { NotificationService } from '../common/notification.service';
import { Libp2pService } from '../common/libp2p.service';

@Injectable()
export class ChatService {
  constructor(
    private readonly messageRepository: MessageRepository,
    private readonly ipfsService: IpfsService,
    private readonly notificationService: NotificationService,
    private readonly libp2pService: Libp2pService,
  ) {}

  async sendMessage(messageDto: CreateMessageDto): Promise<string> {
    const fileBuffer = Buffer.from(messageDto.content);
    const cid = await this.ipfsService.addFile(fileBuffer);

    // Save the message with the CID
    this.messageRepository.create({
      chatRoomId: messageDto.chatRoomId,
      content: cid,
    });

    // Send a push notification
    this.notificationService.sendNotification(
      'New Message',
      `New message in chat room ${messageDto.chatRoomId}`,
    );

    return cid;
  }

  async getMessages(chatRoomId: number) {
    const messages = this.messageRepository.findAll(chatRoomId);
    const resolvedMessages = await Promise.all(
      messages.map(async (message) => {
        const fileBuffer = await this.ipfsService.getFile(message.content);
        return {
          ...message,
          content: fileBuffer.toString(),
        };
      }),
    );

    return resolvedMessages;
  }

  async receiveMessages(handleMessage: (msg: string) => void) {
    this.libp2pService.handleMessages(handleMessage);
  }
}
```

#### Message Schema, Interface, DTO

**Path: `src/chat/interfaces/message.interface.ts`**

```typescript
import { Document } from 'mongoose';

export interface Message extends Document {
  readonly chatRoomId: number;
  readonly content: string;
  readonly sender: string;
  readonly timestamp: string;
}
```

**Path: `src/chat/schemas/message.schema.ts`**

```typescript
import { Schema } from 'mongoose';

export const MessageSchema = new Schema({
  chatRoomId: { type: Number, required: true },
  content: { type: String, required: true },
  sender: { type: String, required: true },
  timestamp: { type: String, required: true },
});
```

#### `create-message.dto.ts`

**Path: `src/chat/dto/create-message.dto.ts`**

```typescript
import { IsString, IsInt } from 'class-validator';

export class CreateMessageDto {
  @IsInt()
  readonly chatRoomId: number;

  @IsString()
  readonly content: string;

  @IsString()
  readonly sender: string;

  @IsString()
  readonly timestamp: string;
}
```

### Users Module

**`src/users/users.controller.ts`**

```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from '../dto/create-user.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post('register')
  async register(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }
}
```

**`src/users/users.module.ts`**

```typescript
import { Module } from '@nestjs/common';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';
import { UserRepository } from '../repositories/user.repository';

@Module({
  providers: [UsersService, UserRepository],
  controllers: [UsersController],
  exports: [UsersService],
})
export class UsersModule {}
```

**`src/users/users.service.ts`**

```typescript
import { Injectable } from '@nestjs/common';
import { UserRepository } from '../repositories/user.repository';
import { CreateUserDto } from '../dto/create-user.dto';

@Injectable()
export class UsersService {
  constructor(private readonly userRepository: UserRepository) {}

  async create(userDto: CreateUserDto) {
    return this.userRepository.create(userDto);
  }

  async findOne(username: string) {
    return this.userRepository.findOne(username);
  }
}
```

#### User Schema and Interface

**Path: `src/users/interfaces/user.interface.ts`**

```typescript
import { Document } from 'mongoose';

export interface User extends Document {
  readonly username: string;
  readonly password: string;
  readonly did: string;
}
```

**Path: `src/users/schemas/user.schema.ts`**

```typescript
import { Schema } from 'mongoose';

export const UserSchema = new Schema({
  username: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  did: { type: String, required: true },
});
```

### Repositories

#### User Repository

**Path: `src/repositories/user.repository.ts`**

```typescript
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { User } from '../users/interfaces/user.interface';
import { CreateUserDto } from '../users/dto/create-user.dto';

@Injectable()
export class UserRepository {
  constructor(@InjectModel('User') private readonly userModel: Model<User>) {}

  async create(createUserDto: CreateUserDto): Promise<User> {
    const createdUser = new this.userModel(createUserDto);
    return createdUser.save();
  }

  async findOne(username: string): Promise<User | undefined> {
    return this.userModel.findOne({ username }).exec();
  }

  async findById(id: string): Promise<User | undefined> {
    return this.userModel.findById(id).exec();
  }
}
```

#### Message Repository

**Path: `src/repositories/message.repository.ts`**

```typescript
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { Message } from '../chat/interfaces/message.interface';
import { CreateMessageDto } from '../chat/dto/create-message.dto';

@Injectable()
export class MessageRepository {
  constructor(@InjectModel('Message') private readonly messageModel: Model<Message>) {}

  async create(createMessageDto: CreateMessageDto): Promise<Message> {
    const createdMessage = new this.messageModel(createMessageDto);
    return createdMessage.save();
  }

  async findAll(chatRoomId: number): Promise<Message[]> {
    return this.messageModel.find({ chatRoomId }).exec();
  }
}
```

### Common Services

**Encryption Service**: `src/common/encryption.service.ts`

```typescript
import { Injectable } from '@nestjs/common';
import * as CryptoJS from 'crypto-js';

@Injectable()
export class EncryptionService {
  private readonly key: string;

  constructor() {
    this.key = process.env.ENCRYPTION_KEY || 'my_strong_secret_key';
  }

  encrypt(text: string): string {
    return CryptoJS.AES.encrypt(text, this.key).toString();
  }

  decrypt(ciphertext: string): string {
    const bytes = CryptoJS.AES.decrypt(ciphertext, this.key);
    return bytes.toString(CryptoJS.enc.Utf8);
  }
}
```

**IPFS Service**: `src/common/ipfs.service.ts`

```typescript
import { Injectable } from '@nestjs/common';
import * as IPFS from 'ipfs-http-client';

@Injectable()
export class IpfsService {
  private ipfs;

  constructor() {
    this.ipfs = IPFS.create({ url: 'https://ipfs.infura.io:5001' }); // Using Infura's IPFS node
  }

  async addFile(content: Buffer) {
    const { cid } = await this.ipfs.add(content);
    return cid.toString();
  }

  async getFile(cid: string) {
    const file = [];
    for await (const chunk of this.ipfs.cat(cid)) {
      file.push(chunk);
    }
    return Buffer.concat(file);
  }
}
```

**DID Service**: `src/common/did.service.ts`

```typescript
import { Injectable } from '@nestjs/common';
import { Resolver } from 'did-resolver';
import { getResolver } from 'key-did-resolver';
import { createJWT, verifyJWT, JWTVerified } from 'did-jwt';

@Injectable()
export class DidService {
  private resolver: Resolver;

  constructor() {
    this.resolver = new Resolver(getResolver());
  }

  async createJWT(payload: any, issuer: string, signer: any) {
    const jwt = await createJWT(payload, { issuer, signer }, { alg: 'ES256K' });
    return jwt;
  }

  async verifyJWT(jwt: string): Promise<JWTVerified> {
    const verified = await verifyJWT(jwt, { resolver: this.resolver });
    return verified;
  }
}
```

**Libp2p Service**: `src/common/libp2p.service.ts`

```typescript
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import Libp2p from 'libp2p';
import WebRTCStar from 'libp2p-webrtc-star';
import { createFromJSON } from 'peer-id-factory';
import PeerId from 'peer-id';

@Injectable()
export class Libp2pService implements OnModuleInit, OnModuleDestroy {
  private libp2p: Libp2p;

  async onModuleInit() {
    const peerId = await createFromJSON(PeerId.create());

    this.libp2p = await Libp2p.create({
      peerId,
      addresses: {
        listen: ['/dns4/wrtc-star1.par.dwebops.pub/tcp/443/wss/p2p-webrtc-star']
      },
      modules: {
        transport: [WebRTCStar]
      }
    });

    await this.libp2p.start();

    this.libp2p.on('peer:discovery', (peer) => {
      console.log('Discovered:', peer.id.toB58String());
    });

    this.libp2p.on('peer:connect', (peer) => {
      console.log('Connected to:', peer.id.toB58String());
    });
  }

  async onModuleDestroy() {
    await this.libp2p.stop();
  }

  async sendMessage(peerId: string, message: string) {
    const { stream } = await this.libp2p.dialProtocol(peerId, '/chat/1.0.0');
    await stream.sink(Buffer.from(message));
  }

  async handleMessages(handleMessage: (msg: string) => void) {
    this.libp2p.handle('/chat/1.0.0', ({ connection, stream }) => {
      stream.sink((data) => {
        handleMessage(data.toString());
      });
    });
  }
}
```

**Notification Service**: `src/common/notification.service.ts`

```typescript
import { Injectable } from '@nestjs/common';
import { SchedulerRegistry } from '@nestjs/schedule';
import { Notification } from 'node-notifier';

@Injectable()
export class NotificationService {
  constructor(private readonly schedulerRegistry: SchedulerRegistry) {}

  sendNotification(title: string, message: string) {
    const notification = new Notification({
      title: title,
      message: message,
    });

    notification.notify();
  }

  scheduleNotification(title: string, message: string, delay: number) {
    const callback = () => {
      this.sendNotification(title, message);
    };

    const timeout = setTimeout(callback, delay);
    this.schedulerRegistry.addTimeout(`${title}-${Date.now()}`, timeout);
  }
}
```

---

## Frontend Setup (Flutter)

### Install Dependencies

Add the following dependencies to your `pubspec.yaml`:

**`pubspec.yaml`**

```yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^5.0.0
  http: ^0.13.3
  encrypt: ^5.0.1
  socket

_io_client: ^2.0.0
  flutter_dotenv: ^5.0.2
  webrtc: ^0.7.0
  ipfs_client: ^0.1.0
  did_auth: ^0.1.0
```

Run the following command to install the dependencies:

```bash
flutter pub get
```

### Project Structure

Create the following directory structure:

```bash
mkdir -p lib/models lib/providers lib/repositories lib/screens lib/services lib/widgets
touch lib/models/user_dto.dart lib/models/message_dto.dart
touch lib/providers/user_provider.dart lib/providers/message_provider.dart
touch lib/repositories/user_repository.dart lib/repositories/message_repository.dart
touch lib/screens/login_screen.dart lib/screens/chat_room_screen.dart
touch lib/services/encryption_service.dart lib/services/ipfs_service.dart lib/services/webrtc_service.dart lib/services/did_service.dart
touch lib/widgets/message_bubble.dart
```

### Models

**User DTO**: `lib/models/user_dto.dart`

```dart
class UserDTO {
  final String username;
  final String password;
  final String did;

  UserDTO({required this.username, required this.password, required this.did});

  Map<String, dynamic> toJson() {
    return {
      'username': username,
      'password': password,
      'did': did,
    };
  }
}
```

**Message DTO**: `lib/models/message_dto.dart`

```dart
class MessageDTO {
  final int chatRoomId;
  final String content;
  final String sender;
  final String timestamp;
  final String imageUrl;
  final bool isReply;
  final String replyMessage;

  MessageDTO({
    required this.chatRoomId,
    required this.content,
    required this.sender,
    required this.timestamp,
    this.imageUrl = '',
    this.isReply = false,
    this.replyMessage = '',
  });

  Map<String, dynamic> toJson() {
    return {
      'chatRoomId': chatRoomId,
      'content': content,
      'sender': sender,
      'timestamp': timestamp,
      'imageUrl': imageUrl,
      'isReply': isReply,
      'replyMessage': replyMessage,
    };
  }

  factory MessageDTO.fromJson(Map<String, dynamic> json) {
    return MessageDTO(
      chatRoomId: json['chatRoomId'],
      content: json['content'],
      sender: json['sender'],
      timestamp: json['timestamp'],
      imageUrl: json['imageUrl'],
      isReply: json['isReply'],
      replyMessage: json['replyMessage'],
    );
  }
}
```

### Providers

**User Provider**: `lib/providers/user_provider.dart`

```dart
import 'package:flutter/material.dart';
import '../models/user_dto.dart';

class UserProvider with ChangeNotifier {
  UserDTO? _user;

  UserDTO? get user => _user;

  void setUser(UserDTO user) {
    _user = user;
    notifyListeners();
  }
}
```

**Message Provider**: `lib/providers/message_provider.dart`

```dart
import 'package:flutter/material.dart';
import '../models/message_dto.dart';

class MessageProvider with ChangeNotifier {
  List<MessageDTO> _messages = [];
  MessageDTO? _replyMessage;

  List<MessageDTO> get messages => _messages;
  MessageDTO? get replyMessage => _replyMessage;

  void addMessage(MessageDTO message) {
    _messages.add(message);
    notifyListeners();
  }

  void setMessages(List<MessageDTO> messages) {
    _messages = messages;
    notifyListeners();
  }

  void setReplyMessage(MessageDTO message) {
    _replyMessage = message;
    notifyListeners();
  }

  void clearReplyMessage() {
    _replyMessage = null;
    notifyListeners();
  }
}
```

### Repositories

**User Repository**: `lib/repositories/user_repository.dart`

```dart
import 'package:http/http.dart' as http;
import 'dart:convert';
import '../models/user_dto.dart';

class UserRepository {
  Future<void> registerUser(UserDTO userDto) async {
    final response = await http.post(
      Uri.parse('http://localhost:3000/auth/register'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode(userDto.toJson()),
    );

    if (response.statusCode != 200) {
      throw Exception('Failed to register user');
    }
  }

  Future<UserDTO?> getUser(String username) async {
    final response = await http.get(
      Uri.parse('http://localhost:3000/users/$username'),
    );

    if (response.statusCode == 200) {
      return UserDTO.fromJson(jsonDecode(response.body));
    } else {
      return null;
    }
  }
}
```

**Message Repository**: `lib/repositories/message_repository.dart`

```dart
import 'package:http/http.dart' as http;
import 'dart:convert';
import '../models/message_dto.dart';

class MessageRepository {
  Future<void> sendMessage(MessageDTO messageDto) async {
    final response = await http.post(
      Uri.parse('http://localhost:3000/messages'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode(messageDto.toJson()),
    );

    if (response.statusCode != 200) {
      throw Exception('Failed to send message');
    }
  }

  Future<List<MessageDTO>> getMessages(int chatRoomId) async {
    final response = await http.get(
      Uri.parse('http://localhost:3000/messages/$chatRoomId'),
    );

    if (response.statusCode == 200) {
      final List<dynamic> json = jsonDecode(response.body);
      return json.map((e) => MessageDTO.fromJson(e)).toList();
    } else {
      throw Exception('Failed to load messages');
    }
  }
}
```

### Services

**Encryption Service**: `lib/services/encryption_service.dart`

```dart
import 'package:encrypt/encrypt.dart' as encrypt;

class EncryptionService {
  final encrypt.Key key;
  final encrypt.IV iv;

  EncryptionService()
      : key = encrypt.Key.fromUtf8('my32lengthsupersecretnooneknows1'),
        iv = encrypt.IV.fromLength(16);

  String encrypt(String text) {
    final encrypter = encrypt.Encrypter(encrypt.AES(key));
    final encrypted = encrypter.encrypt(text, iv: iv);
    return encrypted.base64;
  }

  String decrypt(String encryptedText) {
    final encrypter = encrypt.Encrypter(encrypt.AES(key));
    final decrypted = encrypter.decrypt64(encryptedText, iv: iv);
    return decrypted;
  }
}
```

**IPFS Service**: `lib/services/ipfs_service.dart`

```dart
import 'package:ipfs_client/ipfs_client.dart';

class IpfsService {
  final IpfsClient _ipfsClient;

  IpfsService() : _ipfsClient = IpfsClient();

  Future<String> addFile(String content) async {
    final result = await _ipfsClient.add(content);
    return result['Hash'];
  }

  Future<String> getFile(String cid) async {
    final result = await _ipfsClient.cat(cid);
    return result;
  }
}
```

**WebRTC Service**: `lib/services/webrtc_service.dart`

```dart
import 'package:flutter_webrtc/flutter_webrtc.dart';

class WebRTCService {
  RTCPeerConnection? _peerConnection;
  final Map<String, dynamic> _iceServers = {
    'iceServers': [
      {'url': 'stun:stun.l.google.com:19302'}
    ]
  };

  Future<void> initialize() async {
    _peerConnection = await createPeerConnection(_iceServers);
  }

  Future<RTCSessionDescription> createOffer() async {
    final offer = await _peerConnection!.createOffer();
    await _peerConnection!.setLocalDescription(offer);
    return offer;
  }

  Future<void> setRemoteDescription(RTCSessionDescription description) async {
    await _peerConnection!.setRemoteDescription(description);
  }

  Future<RTCSessionDescription> createAnswer() async {
    final answer = await _peerConnection!.createAnswer();
    await _peerConnection!.setLocalDescription(answer);
    return answer;
  }

  void addIceCandidate(RTCIceCandidate candidate) {
    _peerConnection!.addCandidate(candidate);
  }
}
```

**DID Service**: `lib/services/did_service.dart`

```dart
import 'package:did_auth/did_auth.dart';

class DidService {
  final DIDAuth _didAuth;

  DidService() : _didAuth = DIDAuth();

  Future<String> createJWT(Map<String, dynamic> payload, String did, String privateKey) async {
    final jwt = await _didAuth.createJWT(payload, issuer: did, signer: privateKey);
    return jwt;
  }

  Future<Map<String, dynamic>> verifyJWT(String jwt) async {
    final verified = await _didAuth.verifyJWT(jwt);
    return verified.payload;
  }
}
```

### Screens

**Login Screen**: `lib/screens/login_screen.dart`

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../models/user_dto.dart';
import '../providers/user_provider.dart';
import '../repositories/user_repository.dart';

class LoginScreen extends StatelessWidget {
  final TextEditingController _usernameController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();

  void _login(BuildContext context) async {
    final username = _usernameController.text;
    final password = _passwordController.text;
    final did = 'some-did'; // Replace with actual DID logic

    final userDto = UserDTO(username: username, password:

 password, did: did);

    try {
      await UserRepository().registerUser(userDto);
      Provider.of<UserProvider>(context, listen: false).setUser(userDto);
      Navigator.of(context).pushReplacementNamed('/chatRooms');
    } catch (error) {
      // Handle error
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Login'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            TextField(
              controller: _usernameController,
              decoration: InputDecoration(labelText: 'Username'),
            ),
            TextField(
              controller: _passwordController,
              decoration: InputDecoration(labelText: 'Password'),
              obscureText: true,
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () => _login(context),
              child: Text('Login'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Chat Room Screen**: `lib/screens/chat_room_screen.dart`

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../models/message_dto.dart';
import '../providers/message_provider.dart';
import '../repositories/message_repository.dart';
import '../widgets/message_bubble.dart';
import '../services/ipfs_service.dart';

class ChatRoomScreen extends StatefulWidget {
  final int chatRoomId;

  ChatRoomScreen({required this.chatRoomId});

  @override
  _ChatRoomScreenState createState() => _ChatRoomScreenState();
}

class _ChatRoomScreenState extends State<ChatRoomScreen> {
  final TextEditingController _controller = TextEditingController();
  late IpfsService _ipfsService;

  @override
  void initState() {
    super.initState();
    _ipfsService = IpfsService();
    _loadMessages();
  }

  void _loadMessages() async {
    try {
      final messages = await MessageRepository().getMessages(widget.chatRoomId);
      Provider.of<MessageProvider>(context, listen: false).setMessages(messages);
    } catch (error) {
      // Handle error
    }
  }

  void _sendMessage() async {
    if (_controller.text.isNotEmpty) {
      final content = _controller.text;
      final cid = await _ipfsService.addFile(content);

      final messageDto = MessageDTO(
        chatRoomId: widget.chatRoomId,
        content: cid,
        sender: 'currentUser', // Replace with actual user
        timestamp: DateTime.now().toIso8601String(),
      );

      final messageProvider = Provider.of<MessageProvider>(context, listen: false);

      try {
        await MessageRepository().sendMessage(messageDto);
        messageProvider.addMessage(messageDto);
        if (messageProvider.replyMessage != null) {
          messageProvider.clearReplyMessage();
        }
        _controller.clear();
      } catch (error) {
        // Handle error
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    final messages = Provider.of<MessageProvider>(context).messages;
    final replyMessage = Provider.of<MessageProvider>(context).replyMessage;

    return Scaffold(
      appBar: AppBar(
        title: Text('Chat Room ${widget.chatRoomId}'),
      ),
      body: Column(
        children: [
          if (replyMessage != null)
            Container(
              padding: EdgeInsets.all(8),
              color: Colors.grey[300],
              child: Row(
                children: [
                  Expanded(
                    child: Text(
                      'Replying to: ${replyMessage.content}',
                      style: TextStyle(fontStyle: FontStyle.italic),
                    ),
                  ),
                  IconButton(
                    icon: Icon(Icons.close),
                    onPressed: () {
                      Provider.of<MessageProvider>(context, listen: false).clearReplyMessage();
                    },
                  ),
                ],
              ),
            ),
          Expanded(
            child: ListView.builder(
              itemCount: messages.length,
              itemBuilder: (context, index) {
                final message = messages[index];
                return GestureDetector(
                  onLongPress: () {
                    Provider.of<MessageProvider>(context, listen: false).setReplyMessage(message);
                  },
                  child: FutureBuilder(
                    future: _ipfsService.getFile(message.content),
                    builder: (context, snapshot) {
                      if (snapshot.connectionState == ConnectionState.done && snapshot.hasData) {
                        return MessageBubble(
                          message: snapshot.data as String,
                          isMe: message.sender == 'currentUser', // Adjust this as per the current user's context
                          username: message.sender,
                          timestamp: message.timestamp,
                          imageUrl: '', // Replace with actual image URL
                          isReply: message == replyMessage,
                          replyMessage: replyMessage?.content ?? '',
                        );
                      }
                      return CircularProgressIndicator();
                    },
                  ),
                );
              },
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _controller,
                    decoration: InputDecoration(hintText: 'Send a message...'),
                  ),
                ),
                IconButton(
                  icon: Icon(Icons.send),
                  onPressed: _sendMessage,
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

### Widgets

**Message Bubble**: `lib/widgets/message_bubble.dart`

```dart
import 'package:flutter/material.dart';

class MessageBubble extends StatelessWidget {
  final String message;
  final bool isMe;
  final String username;
  final String timestamp;
  final String imageUrl;
  final bool isReply;
  final String replyMessage;

  MessageBubble({
    required this.message,
    required this.isMe,
    required this.username,
    required this.timestamp,
    this.imageUrl = '',
    this.isReply = false,
    this.replyMessage = '',
  });

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: isMe ? CrossAxisAlignment.end : CrossAxisAlignment.start,
      children: [
        Padding(
          padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 4),
          child: Text(
            username,
            style: TextStyle(
              fontSize: 12,
              fontWeight: FontWeight.bold,
              color: isMe ? Colors.blueAccent : Colors.redAccent,
            ),
          ),
        ),
        Row(
          mainAxisAlignment: isMe ? MainAxisAlignment.end : MainAxisAlignment.start,
          children: [
            if (!isMe)
              CircleAvatar(
                backgroundImage: NetworkImage(imageUrl),
                radius: 15,
              ),
            SizedBox(width: 10),
            Flexible(
              child: GestureDetector(
                onLongPress: () {
                  // Show options for reply, react, etc.
                },
                child: Container(
                  margin: EdgeInsets.symmetric(vertical: 5, horizontal: 10),
                  padding: EdgeInsets.all(10),
                  decoration: BoxDecoration(
                    color: isMe ? Colors.blue : Colors.grey,
                    borderRadius: BorderRadius.only(
                      topLeft: Radius.circular(10),
                      topRight: Radius.circular(10),
                      bottomLeft: isMe ? Radius.circular(10) : Radius.circular(0),
                      bottomRight: isMe ? Radius.circular(0) : Radius.circular(10),
                    ),
                  ),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      if (isReply)
                        Container(
                          padding: EdgeInsets.all(8),
                          margin: EdgeInsets.only(bottom: 5),
                          decoration: BoxDecoration(
                            color: Colors.black.withOpacity(0.1),
                            borderRadius: BorderRadius.circular(5),
                          ),
                          child: Text(
                            replyMessage,
                            style: TextStyle(
                              fontSize: 12,
                              fontStyle: FontStyle.italic,
                            ),
                          ),
                        ),
                      Text(
                        message,
                        style: TextStyle(color: Colors.white),
                      ),
                    ],
                  ),
                ),
              ),
            ),
          ],
        ),
        Padding(
          padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 4),
          child: Text(
            timestamp,
            style: TextStyle(
              fontSize: 10,
              color: Colors.grey,
            ),
          ),
        ),
      ],
    );
  }
}
```

### Main Application Entry Point

**`lib/main.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'providers/user_provider.dart';
import 'providers/message_provider.dart';
import 'screens/login_screen.dart';
import 'screens/chat_room_screen.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => UserProvider()),
        ChangeNotifierProvider(create: (_) => MessageProvider()),
      ],
      child: MaterialApp(
        title: 'Decentralized Chat App',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        initialRoute: '/',
        routes: {
          '/': (context) => LoginScreen(),
          '/chatRooms': (context) => ChatRoomScreen(chatRoomId: 1), // Example chat room ID
        },
      ),
    );
  }
}
```

---

## Docker Integration

### Dockerfile

**Dockerfile**

```Dockerfile
# Use the official Node.js image as the base image
FROM node:16-alpine

# Set the working directory
WORKDIR /usr/src/app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

#

 Copy the rest of the application code
COPY . .

# Build the NestJS application
RUN npm run build

# Expose the application port
EXPOSE 3000

# Start the NestJS application
CMD ["npm", "run", "start:prod"]
```

### docker-compose.yml

**docker-compose.yml**

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: nest-app
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    volumes:
      - .:/usr/src/app
      - /usr/src/app/node_modules
    command: npm run start:prod

  mongodb:
    image: mongo:latest
    container_name: mongodb
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

---

## Running the Application

### Start the Backend Application

```bash
npm run start:dev
```

### Start the Docker Containers

```bash
docker-compose up
```

### Start the Flutter Application

```bash
flutter run
```

---

This comprehensive manual covers the setup and development of a decentralized chat application using Web3.0 technologies. By following these steps, you can ensure that the application is secure, serverless, and ready for real-world use. The integration of IPFS, PubSub, WebRTC/Libp2p, and DID authentication ensures that the application leverages the best of decentralized technologies.

## ENV Setting
Certainly! Below are examples of `.env.development` and `.env.production` files that you can use to configure your development and production environments.

### .env.development

```plaintext
# Server settings
PORT=3000
NODE_ENV=development

# Database settings
MONGO_URI=mongodb://localhost:27017/secure_chat_app_dev

# Encryption key
ENCRYPTION_KEY=my_strong_secret_key_development

# IPFS settings
IPFS_NODE=https://ipfs.infura.io:5001

# WebRTC settings
STUN_SERVER=stun:stun.l.google.com:19302
TURN_SERVER=turn:your_turn_server_address
TURN_USERNAME=your_turn_username
TURN_PASSWORD=your_turn_password

# DID settings
DID_RESOLVER_ENDPOINT=https://did-resolver.example.com

# JWT settings
JWT_SECRET=your_jwt_secret_development

# Notification settings
NOTIFICATION_SERVICE_ENDPOINT=https://notification.example.com

# Other configurations
OTHER_CONFIG=your_other_config_development
```

### .env.production

```plaintext
# Server settings
PORT=3000
NODE_ENV=production

# Database settings
MONGO_URI=mongodb://localhost:27017/secure_chat_app_prod

# Encryption key
ENCRYPTION_KEY=my_strong_secret_key_production

# IPFS settings
IPFS_NODE=https://ipfs.infura.io:5001

# WebRTC settings
STUN_SERVER=stun:stun.l.google.com:19302
TURN_SERVER=turn:your_turn_server_address
TURN_USERNAME=your_turn_username
TURN_PASSWORD=your_turn_password

# DID settings
DID_RESOLVER_ENDPOINT=https://did-resolver.example.com

# JWT settings
JWT_SECRET=your_jwt_secret_production

# Notification settings
NOTIFICATION_SERVICE_ENDPOINT=https://notification.example.com

# Other configurations
OTHER_CONFIG=your_other_config_production
```

### Using Environment Variables in the Application

Ensure that your application reads these environment variables correctly. For example, in NestJS, you can use the `@nestjs/config` package to load and use environment variables.

**Install the `@nestjs/config` package:**

```bash
npm install @nestjs/config
```

**Update `src/app.module.ts` to include the configuration module:**

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { AuthModule } from './auth/auth.module';
import { ChatModule } from './chat/chat.module';
import { UsersModule } from './users/users.module';
import { EncryptionService } from './common/encryption.service';
import { IpfsService } from './common/ipfs.service';
import { DidService } from './common/did.service';
import { Libp2pService } from './common/libp2p.service';
import { NotificationService } from './common/notification.service';
import { ScheduleModule } from '@nestjs/schedule';

@Module({
  imports: [
    ConfigModule.forRoot({
      envFilePath: `.env.${process.env.NODE_ENV}`,
    }),
    AuthModule,
    ChatModule,
    UsersModule,
    ScheduleModule.forRoot(),
  ],
  providers: [EncryptionService, IpfsService, DidService, Libp2pService, NotificationService],
})
export class AppModule {}
```

**Use environment variables in your services:**

For example, in `src/common/encryption.service.ts`:

```typescript
import { Injectable } from '@nestjs/common';
import * as CryptoJS from 'crypto-js';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class EncryptionService {
  private readonly key: string;

  constructor(private configService: ConfigService) {
    this.key = this.configService.get<string>('ENCRYPTION_KEY');
  }

  encrypt(text: string): string {
    return CryptoJS.AES.encrypt(text, this.key).toString();
  }

  decrypt(ciphertext: string): string {
    const bytes = CryptoJS.AES.decrypt(ciphertext, this.key);
    return bytes.toString(CryptoJS.enc.Utf8);
  }
}
```

This approach ensures that you can easily switch between development and production configurations by simply setting the `NODE_ENV` environment variable appropriately when starting your application.
