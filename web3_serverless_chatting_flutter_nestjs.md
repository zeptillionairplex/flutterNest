```bash
Objective: To create a perfect manual for building a chat app without a server using web3.0 technology. 
The app and server should be able to operate immediately with just a simple copy and paste of the code.
1. Answer from the perspective of a developer with 30 years of experience.
2. Use Flutter for the frontend.
3. Use Nest.js for the backend.
4. Ensure the app allows two people to converse without going through a server after installation.
5. The goal is to create a serverless chat app using concepts such as decentralization and blockchain technology.
6. Apply encryption when exchanging data over the internet.
7.Apply free-to-use technology that doesn’t use Ethereum.
8.creating docker-compose, Dockerfile in nest.js backend
9.Tell me how to use internet in build app and show me the what permission code to put in build setting
10. show me whole project file tree structure.
11.Using DTO, Provider, Repository Design Pattern all front-end, back-end
답변은 한글로 해줘
```

### 목차
1. 개요
2. 설치 및 실행 방법
3. 프로젝트 구조
4. 프론트엔드 구현 (Flutter)
5. 백엔드 구현 (Nest.js)
6. 블록체인 연동
7. 데이터 암호화
8. Docker 및 Docker Compose 설정
9. 네트워크 권한 설정
10. 전체 프로젝트 파일 구조
11. 결론

### 1. 개요
이 매뉴얼은 Web3.0 기술을 사용하여 서버 없이 채팅 앱을 빌드하는 방법을 설명합니다. 여기에서는 Flutter를 프론트엔드로, Nest.js를 백엔드로 사용하며, 블록체인 기술을 통해 채팅 데이터를 분산화하고 암호화된 방식으로 서로 전달합니다.

### 2. 설치 및 실행 방법
1. **Flutter** 및 **Nest.js** 설치
   - Flutter: https://flutter.dev/docs/get-started/install
   - Nest.js: `npm install -g @nestjs/cli`

2. **프로젝트 클론**
   ```bash
   git clone <repository_url>
   ```

3. **백엔드 실행 (Nest.js)**
   ```bash
   cd server
   docker-compose up --build
   ```

4. **프론트엔드 실행 (Flutter)**
   ```bash
   cd client
   flutter run
   ```

### 3. 프로젝트 구조
```
project/
 ├── client/               # Flutter 프론트엔드
 └── server/               # Nest.js 백엔드
```

### 4. 프론트엔드 구현 (Flutter)

1. **Flutter 프로젝트 생성**
   ```bash
   flutter create client
   ```

2. **필수 라이브러리 설치**
   `pubspec.yaml`에 다음 의존성을 추가합니다.
   ```yaml
   dependencies:
     flutter:
       sdk: flutter
     http: ^0.13.4
     provider: ^6.0.2
     encrypt: ^5.0.1
     web3dart: ^2.2.0
   ```

3. **Flutter 채팅 앱 코드**
   `lib/main.dart`에 다음 코드를 추가합니다.
   ```dart
   import 'package:flutter/material.dart';
   import 'package:provider/provider.dart';
   import 'chat_provider.dart';

   void main() {
     runApp(
       MultiProvider(
         providers: [
           ChangeNotifierProvider(create: (_) => ChatProvider()),
         ],
         child: MyApp(),
       ),
     );
   }

   class MyApp extends StatelessWidget {
     @override
     Widget build(BuildContext context) {
       return MaterialApp(
         title: 'Decentralized Chat',
         theme: ThemeData(
           primarySwatch: Colors.blue,
         ),
         home: ChatPage(),
       );
     }
   }

   class ChatPage extends StatelessWidget {
     final TextEditingController _controller = TextEditingController();

     void _sendMessage(BuildContext context) {
       final provider = Provider.of<ChatProvider>(context, listen: false);
       provider.sendMessage(_controller.text);
       _controller.clear();
     }

     @override
     Widget build(BuildContext context) {
       return Scaffold(
         appBar: AppBar(
           title: Text('Decentralized Chat'),
         ),
         body: Column(
           children: [
             Expanded(
               child: Consumer<ChatProvider>(
                 builder: (context, provider, _) {
                   return ListView.builder(
                     itemCount: provider.messages.length,
                     itemBuilder: (context, index) {
                       final message = provider.messages[index];
                       return ListTile(
                         title: Text(message),
                       );
                     },
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
                       decoration: InputDecoration(
                         hintText: 'Enter your message...',
                       ),
                     ),
                   ),
                   IconButton(
                     icon: Icon(Icons.send),
                     onPressed: () => _sendMessage(context),
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

4. **데이터 관리 및 전송을 위한 Provider 클래스**
   `lib/chat_provider.dart`에 다음 코드를 추가합니다.
   ```dart
   import 'package:flutter/foundation.dart';

   class ChatProvider extends ChangeNotifier {
     List<String> _messages = [];

     List<String> get messages => _messages;

     void sendMessage(String message) {
       // 메시지를 블록체인 네트워크로 전송하는 로직 추가 필요
       _messages.add(message);
       notifyListeners();
     }
   }
   ```
   
<details>
   <summary>"// 메시지를 블록체인 네트워크로 전송하는 로직 추가"</summary>
예, "// 메시지를 블록체인 네트워크로 전송하는 로직 추가 필요" 부분에 당신이 직접 블록체인 네트워크로 메시지를 전송하는 로직을 작성해야 합니다. 
이 예제에서 IPFS를 사용하여 메시지를 탈중앙화된 스토리지에 저장하고, 그 해시를 공유하는 로직을 작성해 드리겠습니다.

### 1. IPFS 연동을 위한 패키지 설치

`pubspec.yaml`에 다음 의존성을 추가하세요:
```yaml
dependencies:
  ipfs_client: ^0.1.0
  http: ^0.13.4
  provider: ^6.0.2
  encrypt: ^5.0.1
```

### 2. IPFS 유틸리티 클래스 구현

`lib/ipfs_util.dart` 파일을 생성하고 다음 코드를 추가하세요:
```dart
import 'dart:convert';
import 'dart:typed_data';
import 'package:http/http.dart' as http;

class IPFSUtil {
  final String _apiUrl = 'http://localhost:5001/api/v0';

  Future<String> addText(String text) async {
    final uri = Uri.parse('$_apiUrl/add');
    final response = await http.post(
      uri,
      headers: {
        'Content-Type': 'multipart/form-data',
      },
      body: {
        'file': text,
      },
    );
    if (response.statusCode == 200) {
      final jsonResponse = jsonDecode(response.body);
      return jsonResponse['Hash'];
    } else {
      throw Exception('Failed to add text to IPFS');
    }
  }

  Future<String> getText(String hash) async {
    final uri = Uri.parse('$_apiUrl/cat?arg=$hash');
    final response = await http.get(uri);
    if (response.statusCode == 200) {
      return utf8.decode(response.bodyBytes);
    } else {
      throw Exception('Failed to retrieve text from IPFS');
    }
  }
}
```

### 3. ChatProvider에 IPFS 로직 추가

`lib/chat_provider.dart` 파일을 다음과 같이 수정하세요:
```dart
import 'package:flutter/foundation.dart';
import 'ipfs_util.dart';
import 'encryption_util.dart';

class ChatProvider extends ChangeNotifier {
  List<String> _messages = [];

  List<String> get messages => _messages;

  final IPFSUtil _ipfsUtil = IPFSUtil();

  void sendMessage(String message) async {
    try {
      // 메시지 암호화
      final encryptedMessage = EncryptionUtil.encrypt(message).base64;

      // IPFS에 메시지 저장
      final hash = await _ipfsUtil.addText(encryptedMessage);

      // 해시 값을 리스트에 추가
      _messages.add('IPFS Hash: $hash');
      notifyListeners();
    } catch (e) {
      _messages.add('Error: $e');
      notifyListeners();
    }
  }

  void receiveMessage(String hash) async {
    try {
      // IPFS에서 메시지 가져오기
      final encryptedMessage = await _ipfsUtil.getText(hash);

      // 메시지 복호화
      final decryptedMessage = EncryptionUtil.decrypt(Encrypted.fromBase64(encryptedMessage));
      _messages.add('Decrypted: $decryptedMessage');
      notifyListeners();
    } catch (e) {
      _messages.add('Error: $e');
      notifyListeners();
    }
  }
}
```

### 4. ChatPage에 메시지 수신 기능 추가

`lib/main.dart` 파일에서 `ChatPage` 위젯을 다음과 같이 수정하세요:
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'chat_provider.dart';

void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => ChatProvider()),
      ],
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Decentralized Chat',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: ChatPage(),
    );
  }
}

class ChatPage extends StatelessWidget {
  final TextEditingController _sendController = TextEditingController();
  final TextEditingController _receiveController = TextEditingController();

  void _sendMessage(BuildContext context) {
    final provider = Provider.of<ChatProvider>(context, listen: false);
    provider.sendMessage(_sendController.text);
    _sendController.clear();
  }

void _receiveMessage(BuildContext context) {
    final provider = Provider.of<ChatProvider>(context, listen: false);
    provider.receiveMessage(_receiveController.text);
    _receiveController.clear();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Decentralized Chat'),
      ),
      body: Column(
        children: [
          Expanded(
            child: Consumer<ChatProvider>(
              builder: (context, provider, _) {
                return ListView.builder(
                  itemCount: provider.messages.length,
                  itemBuilder: (context, index) {
                    final message = provider.messages[index];
                    return ListTile(
                      title: Text(message),
                    );
                  },
                );
              },
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Column(
              children: [
                Row(
                  children: [
                    Expanded(
                      child: TextField(
                        controller: _sendController,
                        decoration: InputDecoration(
                          hintText: 'Enter your message...',
                        ),
                      ),
                    ),
                    IconButton(
                      icon: Icon(Icons.send),
                      onPressed: () => _sendMessage(context),
                    ),
                  ],
                ),
                Row(
                  children: [
                    Expanded(
                      child: TextField(
                        controller: _receiveController,
                        decoration: InputDecoration(
                          hintText: 'Enter IPFS hash...',
                        ),
                      ),
                    ),
                    IconButton(
                      icon: Icon(Icons.download),
                      onPressed: () => _receiveMessage(context),
                    ),
                  ],
                )
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

### 최종 프로젝트 구조
```
project/
 ├── client/
 │    ├── android/
 │    ├── ios/
 │    ├── lib/
 │    │    ├── encryption_util.dart
 │    │    ├── ipfs_util.dart
 │    │    ├── chat_provider.dart
 │    │    └── main.dart
 │    ├── pubspec.yaml
 └── server/
      ├── src/
      │    ├── controllers/
      │    │    └── message.controller.ts
      │    ├── dto/
      │    │    └── create-message.dto.ts
      │    ├── entities/
      │    │    └── message.entity.ts
      │    ├── modules/
      │    │    └── message.module.ts
      │    ├── repositories/
      │    │    └── message.repository.ts
      │    ├── services/
      │    │    └── message.service.ts
      │    ├── utils/
      │    │    └── encryption.util.ts
      ├── Dockerfile
      ├── docker-compose.yml
      ├── package.json
      ├── tsconfig.json
```

### 전체적인 코드 요약
**Flutter 프론트엔드:** IPFS와 암호화를 통해 메시지를 안전하게 관리하고 저장합니다.

**Nest.js 백엔드:** Docker를 사용하여 배포 가능하며, 메시지 관리와 암호화 로직을 포함합니다.

### 결론
해당 구조를 통해 Flutter 프론트엔드와 Nest.js 백엔드를 사용하여 블록체인 기술을 활용한 채팅 앱을 구축할 수 있습니다. IPFS와의 연동으로 메시지를 탈중앙화된 스토리지에 안전하게 저장하는 방식으로 서버리스 채팅 앱을 완성합니다.   
</details>  

### 5. 백엔드 구현 (Nest.js)
1. **Nest.js 프로젝트 생성**
   ```bash
   nest new server
   ```

2. **필수 라이브러리 설치**
   `server` 디렉터리에서 다음 명령어를 실행하여 필요한 패키지를 설치합니다.
   ```bash
   cd server
   npm install @nestjs/typeorm typeorm sqlite3
   npm install @nestjs/jwt passport-jwt
   npm install bcrypt
   npm install web3
   ```

3. **백엔드 코드 작성**
   `server/src` 디렉터리 내에 필요한 모듈, 컨트롤러, 서비스 및 엔티티를 작성합니다.

   - **DTO**: `src/dto/create-message.dto.ts`
     ```typescript
     export class CreateMessageDto {
       readonly sender: string;
       readonly receiver: string;
       readonly content: string;
     }
     ```

   - **엔티티**: `src/entities/message.entity.ts`
     ```typescript
     import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

     @Entity()
     export class Message {
       @PrimaryGeneratedColumn()
       id: number;

       @Column()
       sender: string;

       @Column()
       receiver: string;

       @Column()
       content: string;
     }
     ```

   - **리포지토리 패턴**: `src/repositories/message.repository.ts`
     ```typescript
     import { EntityRepository, Repository } from 'typeorm';
     import { Message } from '../entities/message.entity';

     @EntityRepository(Message)
     export class MessageRepository extends Repository<Message> {}
     ```

   - **서비스**: `src/services/message.service.ts`
     ```typescript
     import { Injectable } from '@nestjs/common';
     import { InjectRepository } from '@nestjs/typeorm';
     import { MessageRepository } from '../repositories/message.repository';
     import { CreateMessageDto } from '../dto/create-message.dto';
     import { Message } from '../entities/message.entity';

     @Injectable()
     export class MessageService {
       constructor(
         @InjectRepository(MessageRepository)
         private readonly messageRepository: MessageRepository,
       ) {}

       async create(createMessageDto: CreateMessageDto): Promise<Message> {
         const message = this.messageRepository.create(createMessageDto);
         return this.messageRepository.save(message);
       }

       async findAll(): Promise<Message[]> {
         return this.messageRepository.find();
       }
     }
     ```

   - **컨트롤러**: `src/controllers/message.controller.ts`
     ```typescript
     import { Controller, Get, Post, Body } from '@nestjs/common';
     import { MessageService } from '../services/message.service';
     import { CreateMessageDto } from '../dto/create-message.dto';
     import { Message } from '../entities/message.entity';

     @Controller('messages')
     export class MessageController {
       constructor(private readonly messageService: MessageService) {}

       @Post()
       async create(@Body() createMessageDto: CreateMessageDto): Promise<Message> {
         return this.messageService.create(createMessageDto);
       }

       @Get()
       async findAll(): Promise<Message[]> {
         return this.messageService.findAll();
       }
     }
     ```

   - **모듈**: `src/modules/message.module.ts`
     ```typescript
     import { Module } from '@nestjs/common';
     import { TypeOrmModule } from '@nestjs/typeorm';
     import { MessageRepository } from '../repositories/message.repository';
     import { MessageService } from '../services/message.service';
     import { MessageController } from '../controllers/message.controller';

     @Module({
       imports: [TypeOrmModule.forFeature([MessageRepository])],
       providers: [MessageService],
       controllers: [MessageController],
     })
     export class MessageModule {}
     ```

   - **메인 모듈에 추가**: `src/app.module.ts`
     ```typescript
     import { Module } from '@nestjs/common';
     import { TypeOrmModule } from '@nestjs/typeorm';
     import { MessageModule } from './modules/message.module';

     @Module({
       imports: [
         TypeOrmModule.forRoot({
           type: 'sqlite',
           database: 'database.sqlite',
           entities: [__dirname + '/**/*.entity{.ts,.js}'],
           synchronize: true,
         }),
         MessageModule,
       ],
     })
     export class AppModule {}
     ```

4. **Nest.js 서버 실행**
   ```bash
   npm run start
   ```

### 6. 블록체인 연동
블록체인 연동을 위해 IPFS와 같은 탈중앙화 스토리지 시스템을 사용하여 메시지를 저장할 수 있습니다. 여기서는 Flutter와 Nest.js에서 IPFS를 연동하여 메시지를 저장하는 방법을 설명합니다.

**IPFS 서버 실행 방법**

1. **IPFS 설치**
   - IPFS 공식 사이트에서 설치: https://docs.ipfs.tech/install/
   - 또는 다음 명령어로 설치:
     ```bash
     wget https://dist.ipfs.tech/go-ipfs/v0.10.0/go-ipfs_v0.10.0_linux-amd64.tar.gz
     tar -xvzf go-ipfs_v0.10.0_linux-amd64.tar.gz
     cd go-ipfs
     sudo bash install.sh
     ```

2. **IPFS 서버 시작**
   ```bash
   ipfs init
   ipfs daemon
   ```

### 7. 데이터 암호화
채팅 데이터 암호화를 위해 Flutter와 Nest.js에서 각각 암호화 기능을 구현합니다.

**Flutter에서 데이터 암호화:**

1. **Encryption 패키지 추가**
   `pubspec.yaml`에 다음 의존성을 추가합니다.
   ```yaml
   dependencies:
     encrypt: ^5.0.1
   ```

2. **암호화 유틸리티 클래스 구현**
   `lib/encryption_util.dart` 파일을 생성하고 다음 코드를 추가합니다.
   ```dart
   import 'package:encrypt/encrypt.dart';

   class EncryptionUtil {
     static final _key = Key.fromUtf8('my32lengthsupersecretnooneknows1');
     static final _iv = IV.fromLength(16);

     static Encrypted encrypt(String text) {
       final encrypter = Encrypter(AES(_key));
       return encrypter.encrypt(text, iv: _iv);
     }

     static String decrypt(Encrypted encrypted) {
       final encrypter = Encrypter(AES(_key));
       return encrypter.decrypt(encrypted, iv: _iv);
     }
   }
   ```

3. **암호화 적용**
   `lib/chat_provider.dart` 파일을 수정하여 `EncryptionUtil`을 사용합니다.
   ```dart
   import 'package:encrypt/encrypt.dart';
   import 'package:flutter/foundation.dart';
   import 'encryption_util.dart';

   class ChatProvider extends ChangeNotifier {
     List<String> _messages = [];

     List<String> get messages => _messages;

     void sendMessage(String message) {
       final encrypted = EncryptionUtil.encrypt(message);
       _messages.add('Encrypted: ${encrypted.base64}');
       notifyListeners();
     }

     void receiveMessage(String encryptedMessageBase64) {
       final encrypted = Encrypted.fromBase64(encryptedMessageBase64);
       final decrypted = EncryptionUtil.decrypt(encrypted);
       _messages.add('Decrypted: $decrypted');
       notifyListeners();
     }
   }
   ```

**Nest.js에서 데이터 암호화:**

1. **bcrypt 패키지 설치**
   ```bash
   npm install bcrypt
   ```

2. **암호화 유틸리티 클래스 구현**
   `src/utils/encryption.util.ts` 파일을 생성하고 다음 코드를 추가합니다.
   ```typescript
   import * as bcrypt from 'bcrypt';

   export class EncryptionUtil {
     static async hash(text: string): Promise<string> {
       const salt = await bcrypt.genSalt();
       return bcrypt.hash(text, salt);
     }

     static async compare(text: string, hash: string): Promise<boolean> {
       return bcrypt.compare(text, hash);
     }
   }
   ```

3. **암호화 적용**
   `src/services/message.service.ts` 파일을 수정하여 `EncryptionUtil`을 사용합니다.
   ```typescript
   import { Injectable } from '@nestjs/common';
   import { InjectRepository } from '@nestjs/typeorm';
   import { MessageRepository } from '../repositories/message.repository';
   import { CreateMessageDto } from '../dto/create-message.dto';
   import { Message } from '../entities/message.entity';
   import { EncryptionUtil } from '../utils/encryption.util';

   @Injectable()
   export class MessageService {
     constructor(
       @InjectRepository(MessageRepository)
       private readonly messageRepository: MessageRepository,
     ) {}

     async create(createMessageDto: CreateMessageDto): Promise<Message> {
       const hashedContent = await EncryptionUtil.hash(createMessageDto.content);
       const message = this.messageRepository.create({ ...createMessageDto, content: hashedContent });
       return this.messageRepository.save(message);
     }

     async findAll(): Promise<Message[]> {
       return this.messageRepository.find();
     }
   }
   ```

### 8. Docker 및 Docker Compose 설정

**Dockerfile** (`server/Dockerfile`)
```Dockerfile
# 베이스 이미지로 Node.js 사용
FROM node:16

# 작업 디렉터리 설정
WORKDIR /app

# 패키지 설치
COPY package*.json ./
RUN npm install

# 소스 코드 복사
COPY . .

# Nest.js 빌드
RUN npm run build

# 포트 노출
EXPOSE 3000

# 앱 실행
CMD ["npm", "run", "start:prod"]
```

**Docker Compose 파일** (`server/docker-compose.yml`)
```yaml
version: '3'
services:
  server:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    volumes:
      - ./database.sqlite:/app/database.sqlite
```

### 9. 네트워크 권한 설정

**Flutter 네트워크 권한 설정**

1. `client/android/app/src/main/AndroidManifest.xml` 파일을 수정하여 인터넷 권한을 추가합니다.
   ```xml
   <manifest xmlns:android="http://schemas.android.com/apk/res/android"
       package="com.example.decentralized_chat">
       <uses-permission android:name="android.permission.INTERNET"/>
       <application
           android:label="decentralized_chat"
           android:icon="@mipmap/ic_launcher">
           <activity
               android:name=".MainActivity"
               android:exported="true">
               <intent-filter>
                   <action android:name="android.intent.action.MAIN" />
                   <category android:name="android.intent.category.LAUNCHER" />
               </intent-filter>
           </activity>
       </application>
   </manifest>
   ```

2. `client/ios/Runner/Info.plist` 파일에 인터넷 권한을 추가합니다.
   ```xml
   <key>NSAppTransportSecurity</key>
   <dict>
       <key>NSAllowsArbitraryLoads</key>
       <true/>
   </dict>
   ```

### 10. 전체 프로젝트 파일 구조

```
project/
 ├── client/
 │    ├── android/
 │    ├── ios/
 │    ├── lib/
 │    │    ├── encryption_util.dart
 │    │    ├── chat_provider.dart
 │    │    └── main.dart
 │    ├── pubspec.yaml
 │    └── ...
 └── server/
      ├── src/
      │    ├── controllers/
      │    │    └── message.controller.ts
      │    ├── dto/
      │    │    └── create-message.dto.ts
      │    ├── entities/
      │    │    └── message.entity.ts
      │    ├── modules/
      │    │    └── message.module.ts
      │    ├── repositories/
      │    │    └── message.repository.ts
      │    ├── services/
      │    │    └── message.service.ts
      │    └── utils/
      │         └── encryption.util.ts
      ├── Dockerfile
      ├── docker-compose.yml
      ├── package.json
      ├── tsconfig.json
      └── ...
```

### 11. 결론
이 매뉴얼을 통해 Flutter와 Nest.js, IPFS를 사용하여 완전히 탈중앙화된 서버리스 채팅 앱을 구축하는 방법을 배웠습니다. 이 앱은 블록체인 기술을 활용하여 메시지를 안전하게 교환하며, Docker를 사용하여 간단하게 배포할 수 있습니다.

### 추가 제안 사항
1. **메시지 서명 및 검증**: 메시지에 디지털 서명을 추가하여 메시지의 진위를 검증할 수 있습니다.
2. **IPFS 본격화**: 메시지를 IPFS에 저장하고 해시를 공유하는 방식을 고려해 보세요.
3. **분산 아이덴티티**: 탈중앙화된 시스템에서 각 사용자의 아이덴티티를 관리하는 방식을 추가할 수 있습니다.

### 참고 자료
- [Flutter 공식 문서](https://flutter.dev/docs)
- [Nest.js 공식 문서](https://docs.nestjs.com/)
- [IPFS 기술 문서](https://docs.ipfs.tech/)
