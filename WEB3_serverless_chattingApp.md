웹3.0 기술을 사용하여 이더리움 없이 무료로 채팅 앱을 구축하기 위해서는 완전히 탈중앙화된 블록체인 기반 솔루션을 사용하지 않고, P2P 네트워크를 활용하는 방법이 있습니다. 이를 통해 사용자들은 서버를 거치지 않고 직접 통신할 수 있습니다.

## P2P 기반 서버리스 채팅 앱 매뉴얼

### 기술 개요
- **프론트엔드:** Flutter
- **백엔드:** Nest.js (데이터 중계/처리 없이 P2P 네트워크 연동)
- **P2P 네트워크:** Libp2p (IPFS)
- **지갑:** MetaMask (사용자 인증)

### 프로젝트 구조
```
chat-app/
├── backend/
│   └── nestjs/
└── frontend/
    └── flutter/
```

### 1. 준비 사항
#### 1.1 Flutter 설치 및 구성
1. [Flutter 설치 가이드](https://flutter.dev/docs/get-started/install)를 따라 Flutter를 설치합니다.

#### 1.2 Nest.js 설치 및 구성
1. Node.js를 설치합니다.
2. 다음 명령어로 Nest.js CLI를 설치합니다.
```bash
npm install -g @nestjs/cli
```

#### 1.3 IPFS 및 Libp2p 설치
1. IPFS를 설치합니다.
```bash
npm install -g ipfs
```

2. Libp2p를 설치합니다.
```bash
npm install libp2p
```

### 2. 프로젝트 설정
#### 2.1 백엔드 구성 (Nest.js)
##### 2.1.1 Nest.js 프로젝트 생성
```bash
nestjs new p2p-chat-backend
```

##### 2.1.2 의존성 설치
```bash
cd p2p-chat-backend
npm install ipfs libp2p
```

##### 2.1.3 P2P 서비스 구현 `src/p2p/p2p.service.ts`
```typescript
import { Injectable } from '@nestjs/common';
import { create } from 'ipfs-core';
import Libp2p from 'libp2p';

@Injectable()
export class P2PService {
  private ipfs: any;
  private libp2p: any;

  async init() {
    // IPFS 노드 생성
    this.ipfs = await create();
    console.log('IPFS node is ready');

    // Libp2p 노드 생성
    this.libp2p = new Libp2p({
      modules: {
        transport: [],
        streamMuxer: [],
        connEncryption: [],
      },
    });
    console.log('Libp2p node is ready');
  }

  async sendMessage(message: string) {
    const { cid } = await this.ipfs.add(message);
    return cid.toString();
  }

  async receiveMessage(cid: string) {
    const data = await this.ipfs.cat(cid);
    let content = '';

    for await (const chunk of data) {
      content += chunk.toString();
    }

    return content;
  }
}
```

##### 2.1.4 P2P 모듈 구성 `src/p2p/p2p.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { P2PService } from './p2p.service';

@Module({
  providers: [P2PService],
  exports: [P2PService],
})
export class P2PModule {}
```

##### 2.1.5 채팅 컨트롤러 구현 `src/p2p/p2p.controller.ts`
```typescript
import { Controller, Post, Body, Get, Query } from '@nestjs/common';
import { P2PService } from './p2p.service';

@Controller('p2p')
export class P2PController {
  constructor(private readonly p2pService: P2PService) {}

  @Post('send')
  async sendMessage(@Body('message') message: string) {
    return { cid: await this.p2pService.sendMessage(message) };
  }

  @Get('receive')
  async receiveMessage(@Query('cid') cid: string) {
    return { message: await this.p2pService.receiveMessage(cid) };
  }
}
```

##### 2.1.6 메인 모듈에 P2P 모듈 추가 `src/app.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { P2PModule } from './p2p/p2p.module';

@Module({
  imports: [P2PModule],
})
export class AppModule {}
```

##### 2.1.7 서버 실행
```bash
npm run start
```

### 2.2 프론트엔드 구성 (Flutter)
#### 2.2.1 Flutter 프로젝트 생성
```bash
flutter create p2p_chat
```

#### 2.2.2 의존성 추가
`pubspec.yaml`에 다음 의존성 추가:
```yaml
dependencies:
  flutter:
    sdk: flutter
  http: ^0.13.3
  provider: ^5.0.0
```

#### 2.2.3 모델 클래스 생성 `lib/models/message.dart`
```dart
class Message {
  final String content;
  final String cid;

  Message({required this.content, required this.cid});

  factory Message.fromJson(Map<String, dynamic> json) {
    return Message(
      content: json['message'],
      cid: json['cid'] ?? '',
    );
  }

  Map<String, dynamic> toJson() {
    return {'message': content, 'cid': cid};
  }
}
```

#### 2.2.4 채팅 서비스 구현 `lib/services/chat_service.dart`
```dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import '../models/message.dart';

class ChatService {
  static const String baseUrl = 'http://localhost:3000/p2p';

  Future<Message> sendMessage(String content) async {
    final response = await http.post(
      Uri.parse('$baseUrl/send'),
      headers: {'Content-Type': 'application/json'},
      body: json.encode({'message': content}),
    );

    if (response.statusCode == 200) {
      return Message.fromJson(json.decode(response.body));
    } else {
      throw Exception('Failed to send message');
    }
  }

  Future<Message> receiveMessage(String cid) async {
    final response = await http.get(Uri.parse('$baseUrl/receive?cid=$cid'));

    if (response.statusCode == 200) {
      return Message.fromJson(json.decode(response.body));
    } else {
      throw Exception('Failed to receive message');
    }
  }
}
```

#### 2.2.5 채팅 화면 구현 `lib/screens/chat_screen.dart`
```dart
import 'package:flutter/material.dart';
import '../models/message.dart';
import '../services/chat_service.dart';

class ChatScreen extends StatefulWidget {
  @override
  _ChatScreenState createState() => _ChatScreenState();
}

class _ChatScreenState extends State<ChatScreen> {
  final _chatService = ChatService();
  final _messageController = TextEditingController();
  final _cidController = TextEditingController();
  List<Message> _messages = [];

  Future<void> _sendMessage() async {
    final content = _messageController.text;

    if (content.isEmpty) {
      return;
    }

    try {
      final message = await _chatService.sendMessage(content);
      setState(() {
        _messages.add(message);
      });
      _messageController.clear();
    } catch (e) {
      print('Error sending message: $e');
    }
  }

  Future<void> _receiveMessage() async {
    final cid = _cidController.text;

    if (cid.isEmpty) {
      return;
    }

    try {
      final message = await _chatService.receiveMessage(cid);
      setState(() {
        _messages.add(message);
      });
      _cidController.clear();
    } catch (e) {
      print('Error receiving message: $e');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('P2P Chat'),
      ),
      body: Column(
        children: [
          Expanded(
            child: ListView.builder(
              itemCount: _messages.length,
              itemBuilder: (context, index) {
                final message = _messages[index];
                return ListTile(
                  title: Text(message.content),
                  subtitle: Text('CID: ${message.cid}'),
                );
              },
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Column(
              children: [
                TextField(
                  controller: _messageController,
                  decoration: InputDecoration(
                    labelText: 'Enter your message',
                    border: OutlineInputBorder(),
                  ),
                ),
                SizedBox(height: 8.0),
                ElevatedButton(
                  onPressed: _sendMessage,
                  child: Text('Send'),
                ),
                SizedBox(height: 8.0),
                TextField(
                  controller: _cidController,
                  decoration: InputDecoration(
                    labelText: 'CID to receive',
                    border: OutlineInputBorder(),
                  ),
                ),
                SizedBox(height: 8.0),
                ElevatedButton(
                  onPressed: _receiveMessage,
                  child: Text('Receive'),
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
```dart
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

#### 2.2.6 메인 앱 구성 `lib/main.dart`
```dart
import 'package:flutter/material.dart';
import 'screens/chat_screen.dart';

void main() {
  runApp(P2PChatApp());
}

class P2PChatApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'P2P Chat',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: ChatScreen(),
    );
  }
}
```

### 2.2.7 Flutter 앱 실행
```bash
flutter run
```

### 3. 결론
이 매뉴얼은 P2P 네트워크 기반의 서버리스 채팅 애플리케이션을 구축하는 방법을 안내합니다. 이 솔루션은 Libp2p와 IPFS를 활용하여 메시지가 서버를 거치지 않고 직접 사용자 간에 전달되도록 설계되었습니다.

### 추가 개선 사항
- **메시지 암호화:** 메시지를 IPFS에 저장하기 전에 암호화하여 프라이버시를 강화.
- **사용자 인증 및 프로필:** 사용자의 지갑 주소 또는 P2P 식별자를 기반으로 프로필 및 닉네임을 생성.
- **다중 채널 지원:** 여러 채팅 채널을 지원하여 다양한 주제별 대화 가능.

이 매뉴얼을 통해 개발자들은 이더리움 없이도 완전히 무료로 서버리스 채팅 애플리케이션을 구축하는 방법을 배울 수 있습니다.
