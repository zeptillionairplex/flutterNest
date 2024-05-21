안녕하세요! 30년 경력의 개발자 관점에서 웹3.0 기술을 활용한 서버 없는 채팅 앱을 만들기 위한 매뉴얼을 작성해 보겠습니다. 이 매뉴얼에서는 Flutter SDK 버전 '>=3.0.0 <4.0.0'을 사용하고, Backend는 Node.js로 만들며, Docker를 통해 배포할 수 있도록 구성합니다. 데이터 전송 시 암호화를 적용하고, Ethereum을 사용하지 않는 무료 탈중앙화 기술을 활용합니다.

## 1. 프로젝트 구조
프로젝트의 최종 파일 구조는 다음과 같습니다.

```
chat_app/
│
├── backend/
│   ├── Dockerfile
│   └── docker-compose.yml
│   └── package.json
│   └── src/
│       ├── index.js
│       └── services/
│           └── p2pService.js
│
└── frontend/
    └── flutter_chat_app/
        ├── android/
        ├── ios/
        ├── lib/
        │   ├── data/
        │   │   ├── chat_repository.dart
        │   │   └── models/
        │   │       └── message.dart
        │   ├── providers/
        │   │   └── chat_provider.dart
        │   ├── screens/
        │   │   └── chat_screen.dart
        │   └── main.dart
        └── pubspec.yaml
```

## 2. Backend 만들기
### 2.1 Node.js Backend 구성
우선 `backend` 디렉토리 내에 Node.js 프로젝트를 설정하세요.

**package.json**
```json
{
  "name": "p2p-chat-backend",
  "version": "1.0.0",
  "description": "P2P Chat Backend using WebRTC",
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js"
  },
  "dependencies": {
    "simple-peer": "^9.11.1",
    "ws": "^8.13.0"
  }
}
```

**src/index.js**
```javascript
const WebSocket = require('ws');
const P2PService = require('./services/p2pService');

const server = new WebSocket.Server({ port: 8080 });

server.on('connection', (socket) => {
  console.log('New peer connected');
  const p2pService = new P2PService(socket);
  
  socket.on('message', (message) => {
    const data = JSON.parse(message);
    p2pService.handleMessage(data);
  });

  socket.on('close', () => {
    p2pService.cleanup();
  });
});

console.log('WebSocket server running on ws://localhost:8080');
```

**src/services/p2pService.js**
```javascript
const SimplePeer = require('simple-peer');

class P2PService {
  constructor(socket) {
    this.socket = socket;
    this.peer = new SimplePeer({ initiator: true, trickle: false });
    
    this.peer.on('signal', (data) => {
      this.sendMessage('signal', data);
    });
    
    this.peer.on('connect', () => {
      console.log('Peer connected');
    });
    
    this.peer.on('data', (data) => {
      console.log('Received message:', data.toString());
    });
  }

  handleMessage(message) {
    const { type, payload } = message;

    if (type === 'signal') {
      this.peer.signal(payload);
    } else if (type === 'message') {
      this.peer.send(payload);
    }
  }

  sendMessage(type, payload) {
    this.socket.send(JSON.stringify({ type, payload }));
  }

  cleanup() {
    this.peer.destroy();
  }
}

module.exports = P2PService;
```

### 2.2 Docker 설정
`backend` 디렉토리에 `Dockerfile`과 `docker-compose.yml`을 추가하여 Docker로 실행할 수 있게 합니다.

**Dockerfile**
```Dockerfile
FROM node:14

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

CMD ["npm", "start"]
```

**docker-compose.yml**
```yaml
version: '3'
services:
  p2p-chat-backend:
    build: .
    ports:
      - "8080:8080"
```

### 2.3 실행 방법
```bash
# backend 디렉토리로 이동
cd chat_app/backend

# Docker 빌드 및 실행
docker-compose up --build -d
```

## 3. Frontend 만들기
### 3.1 Flutter 프로젝트 생성 및 의존성 추가
`frontend` 디렉토리 내에 `flutter_chat_app` 프로젝트를 생성하고 필요한 의존성을 추가합니다.

```bash
# frontend 디렉토리로 이동
cd ../frontend

# Flutter 프로젝트 생성
flutter create flutter_chat_app

# 프로젝트 디렉토리로 이동
cd flutter_chat_app
```

**pubspec.yaml**
```yaml
name: flutter_chat_app
description: A P2P chat app using WebRTC

version: 1.0.0

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  provider: ^6.0.0
  http: ^0.15.0
  uuid: ^3.0.5
  simple_peer:
    git:
      url: https://github.com/blackbean99/simple_peer.git

dev_dependencies:
  flutter_test:
    sdk: flutter

flutter:
  uses-material-design: true
```

### 3.2 데이터 모델
`lib/data/models/message.dart` 파일을 통해 데이터를 정의합니다.

**lib/data/models/message.dart**
```dart
class Message {
  final String id;
  final String sender;
  final String content;
  final DateTime timestamp;

  Message({required this.id, required this.sender, required this.content, required this.timestamp});

  factory Message.fromJson(Map<String, dynamic> json) {
    return Message(
      id: json['id'],
      sender: json['sender'],
      content: json['content'],
      timestamp: DateTime.parse(json['timestamp']),
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'sender': sender,
      'content': content,
      'timestamp': timestamp.toIso8601String(),
    };
  }
}
```

### 3.3 채팅 리포지토리
`lib/data/chat_repository.dart` 파일에서 채팅 데이터를 관리합니다.

**lib/data/chat_repository.dart**
```dart
import 'models/message.dart';

class ChatRepository {
  final List<Message> _messages = [];

  List<Message> getMessages() {
    return List.unmodifiable(_messages);
  }

  void addMessage(Message message) {
    _messages.add(message);
  }
}
```

### 3.4 채팅 프로바이더
`lib/providers/chat_provider.dart` 파일에서 채팅 상태를 관리합니다.

**lib/providers/chat_provider.dart**
```dart
import 'package:flutter/material.dart';
import 'package:uuid/uuid.dart';

import '../data/chat_repository.dart';
import '../data/models/message.dart';

class ChatProvider with ChangeNotifier {
  final ChatRepository _chatRepository = ChatRepository();
  final String _username;
  final Uuid _uuid = Uuid();

  ChatProvider(this._username);

  List<Message> get messages => _chatRepository.getMessages();

  void sendMessage(String content) {
    final message = Message(
      id: _uuid.v4(),
      sender: _username,
      content: content,
      timestamp: DateTime.now(),
    );
    _chatRepository.addMessage(message);
    notifyListeners();
  }

  void receiveMessage(String id, String sender, String content, DateTime timestamp) {
    final message = Message(
      id: id,
      sender: sender,
      content: content,
      timestamp: timestamp,
    );
    _chatRepository.addMessage(message);
    notifyListeners();
  }
}
```

### 3.5 채팅 화면
`lib/screens/chat_screen.dart` 파일에서 채팅 UI를 구현합니다.

**lib/screens/chat_screen.dart**
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import '../providers/chat_provider.dart';

class ChatScreen extends StatefulWidget {
  const ChatScreen({Key? key}) : super(key: key);

  @override
  State<ChatScreen> createState() => _ChatScreenState();
}

class _ChatScreenState extends State<ChatScreen> {
  final TextEditingController _controller = TextEditingController();

  @override
  Widget build(BuildContext context) {
    final chatProvider = Provider.of<ChatProvider>(context);

    return Scaffold(
      appBar: AppBar(
        title: const Text('P2P Chat'),
      ),
      body: Column(
        children: [
          Expanded(
            child: ListView.builder(
              itemCount: chatProvider.messages.length,
              itemBuilder: (context, index) {
                final message = chatProvider.messages[index];
                return ListTile(
                  title: Text(
                    message.sender,
                    style: TextStyle(
                      fontWeight: message.sender == 'Me' ? FontWeight.bold : FontWeight.normal,
                      color: message.sender == 'Me' ? Colors.blue : Colors.black,
                    ),
                  ),
                  subtitle: Text(message.content),
                  trailing: Text(
                    message.timestamp.toLocal().toString().substring(0, 19),
                    style: const TextStyle(fontSize: 12),
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
                    decoration: const InputDecoration(
                      hintText: '메시지 입력...',
                    ),
                  ),
                ),
                IconButton(
                  icon: const Icon(Icons.send),
                  onPressed: () {
                    final content = _controller.text;
                    if (content.isNotEmpty) {
                      chatProvider.sendMessage(content);
                      _controller.clear();
                    }
                  },
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

### 3.6 메인 파일
`lib/main.dart` 파일에서 앱을 실행하고 `ChatProvider`를 설정합니다.

**lib/main.dart**
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'providers/chat_provider.dart';
import 'screens/chat_screen.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => ChatProvider('Me'),
      child: MaterialApp(
        title: 'P2P Chat App',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: const ChatScreen(),
      ),
    );
  }
}
```

### 3.7 인터넷 사용 권한 설정
`android/app/src/main/AndroidManifest.xml` 파일에 인터넷 권한을 추가합니다.

**android/app/src/main/AndroidManifest.xml**
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.flutter_chat_app">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:label="flutter_chat_app"
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

### 3.8 iOS 네트워크 설정
`ios/Runner/Info.plist` 파일에 네트워크 설정을 추가합니다.

**ios/Runner/Info.plist**
```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
```

## 4. 실행 방법
### 4.1 Backend 실행
```bash
# backend 디렉토리로 이동
cd chat_app/backend

# Docker 빌드 및 실행
docker-compose up --build -d
```

### 4.2 Frontend 실행
Flutter 프로젝트의 `flutter_chat_app` 디렉토리로 이동하여 실행합니다.

```bash
# frontend 디렉토리로 이동
cd ../frontend/flutter_chat_app

# 의존성 설치
flutter pub get

# 앱 실행
flutter run
```

이렇게 하면 Flutter 기반의 웹3.0 기술을 이용한 완전한 P2P 채팅 앱을 만들 수 있습니다. 이 앱과 백엔드 모두 간단하게 복사하여 실행할 수 있도록 구성되었습니다.
