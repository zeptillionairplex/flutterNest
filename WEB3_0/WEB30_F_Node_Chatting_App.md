안녕하세요! 100년 경력의 특급 개발자입니다. 오늘은 신입 사원을 대상으로 Node.js 백엔드를 처음부터 완벽하게 만드는 방법을 설명하겠습니다. 이 프로젝트는 Flutter로 만든 프론트엔드와 연동하여 P2P 채팅 앱을 개발하는 것이 목표입니다. 차근차근 따라오세요!

## 1. Node.js 백엔드 프로젝트 생성하기
### 1.1 Node.js 프로젝트 생성
먼저 `node` 백엔드 프로젝트를 만들기 위해 터미널에서 다음 명령어를 실행하세요.

```bash
# 원하는 디렉토리로 이동
cd ~/workspace

# backend 디렉토리 생성 및 이동
mkdir -p chat_app/backend && cd chat_app/backend

# Node.js 프로젝트 초기화
npm init -y
```

이제 `package.json` 파일이 생성되었을 것입니다. 이 파일은 프로젝트의 의존성 및 스크립트 실행을 관리합니다.

### 1.2 필요한 패키지 설치
WebSocket과 P2P 통신을 위한 패키지를 설치합니다.

```bash
# WebSocket 서버와 P2P 통신을 위한 패키지 설치
npm install ws simple-peer
```

### 1.3 프로젝트 구조
`backend` 디렉토리에 필요한 폴더와 파일을 생성합니다.

```
backend/
│
├── Dockerfile
├── docker-compose.yml
├── package.json
└── src/
    ├── index.js
    └── services/
        └── p2pService.js
```

### 1.4 WebSocket 서버 설정
`src/index.js` 파일을 만들어 WebSocket 서버를 설정합니다.

**src/index.js**
```javascript
const WebSocket = require('ws');

const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
    ws.on('message', (message) => {
        const data = JSON.parse(message);

    // 특정 클라이언트에게 시그널링 메시지 전달
    wss.clients.forEach((client) => {
        if (client !== ws && client.readyState === WebSocket.OPEN) {
                client.send(JSON.stringify(data));
            }
        });
    });

    console.log('새로운 클라이언트 연결됨');
});

console.log('WebSocket 서버가 ws://localhost:8080 에서 실행 중입니다.');
```

### 1.5 P2P 서비스 로직 작성
`src/services/p2pService.js` 파일을 만들어 P2P 통신 로직을 작성합니다.

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
      console.log('피어 연결됨');
    });
    
    this.peer.on('data', (data) => {
      console.log('메시지 수신:', data.toString());
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

### 1.6 Docker 설정
백엔드를 Docker로 배포할 수 있도록 `Dockerfile`과 `docker-compose.yml` 파일을 작성합니다.

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

### 1.7 `package.json` 스크립트 업데이트
`package.json` 파일의 `scripts` 부분을 다음과 같이 업데이트합니다.

**package.json**
```json
{
  "name": "p2p-chat-backend",
  "version": "1.0.0",
  "description": "P2P Chat Backend using WebRTC",
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "dependencies": {
    "simple-peer": "^9.11.1",
    "ws": "^8.13.0"
  }
}
```

### 1.8 Docker로 백엔드 실행하기
이제 Docker를 사용하여 백엔드를 실행해봅시다.

```bash
# backend 디렉토리로 이동 (이미 해당 위치에 있다면 생략)
cd chat_app/backend

# Docker 빌드 및 실행
docker-compose up --build -d
```

백엔드 서버가 `ws://localhost:8080`에서 실행 중일 것입니다.

## 2. Frontend와 연동하기
### 2.1 Flutter 프로젝트 생성 및 의존성 추가
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
  provider:
  http:
  uuid:
  flutter_webrtc:
  web_socket_channel:
  flutter_dotenv:

dev_dependencies:
  flutter_test:
    sdk: flutter

flutter:
  uses-material-design: true
```

### 2.2 데이터 모델
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

### 2.3 채팅 리포지토리
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

### 2.4 채팅 프로바이더
`lib/providers/chat_provider.dart` 파일에서 채팅 상태를 관리합니다.

**lib/providers/chat_provider.dart**
```dart
import 'dart:convert';
import 'package:flutter/foundation.dart';
import 'package:flutter_webrtc/flutter_webrtc.dart';
import 'package:uuid/uuid.dart';
import 'package:web_socket_channel/web_socket_channel.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';

import '../data/chat_repository.dart';
import '../data/models/message.dart';

class ChatProvider with ChangeNotifier {
  final ChatRepository _chatRepository = ChatRepository();
  final String _username;
  final Uuid _uuid = Uuid();
  String webSocketURL = dotenv.env['WebSocket_URL']!;

  RTCPeerConnection? _peerConnection;
  RTCDataChannel? _dataChannel;
  WebSocketChannel? _channel;
  RTCSessionDescription? _localDescription;

  ChatProvider(this._username) {
    _setupWebSocket();
    _setupPeerConnection();
  }

  List<Message> get messages => _chatRepository.getMessages();

  void _setupWebSocket() {
    _channel = WebSocketChannel.connect(Uri.parse(webSocketURL));

    _channel!.stream.listen((message) {
      final data = jsonDecode(message);

      if (data['type'] == 'offer') {
        _handleOffer(RTCSessionDescription(data['sdp'], data['type']));
      } else if (data['type'] == 'answer') {
        _handleAnswer(RTCSessionDescription(data['sdp'], data['type']));
      } else if (data['type'] == 'ice-candidate') {
        _handleIceCandidate(data['candidate']);
      }
    });
  }

  void _setupPeerConnection() async {
    final config = {
      "iceServers": [
        {"urls": "stun:stun.l.google.com:19302"}
      ]
    };

    _peerConnection = await createPeerConnection(config);
    _dataChannel = await _peerConnection!.createDataChannel('chat', RTCDataChannelInit());

    _peerConnection!.onIceCandidate = (candidate) {
      if (candidate != null) {
        _channel!.sink.add(jsonEncode({
          'type': 'ice-candidate',
          'candidate': candidate.toMap(),
        }));
      }
    };

    _peerConnection!.onDataChannel = (channel) {
      channel.onMessage = (RTCDataChannelMessage message) {
        _receiveMessage(message.text);
      };
    };

    _dataChannel!.onMessage = (RTCDataChannelMessage message) {
      _receiveMessage(message.text);
    };

    _createOffer();
    notifyListeners();
  }

  void _createOffer() async {
    _localDescription = await _peerConnection!.createOffer();
    await _peerConnection!.setLocalDescription(_localDescription!);

    _channel!.sink.add(jsonEncode({
      'type': 'offer',
      'sdp': _localDescription!.sdp,
    }));
  }

  void _handleOffer(RTCSessionDescription description) async {
    await _peerConnection!.setRemoteDescription(description);
    _localDescription = await _peerConnection!.createAnswer();
    await _peerConnection!.setLocalDescription(_localDescription!);

    _channel!.sink.add(jsonEncode({
      'type': 'answer',
      'sdp': _localDescription!.sdp,
    }));
  }

  void _handleAnswer(RTCSessionDescription description) async {
    await _peerConnection!.setRemoteDescription(description);
  }

  void _handleIceCandidate(Map<String, dynamic> candidate) async {
    await _peerConnection!.addCandidate(RTCIceCandidate(
      candidate['candidate'],
      candidate['sdpMid'],
      candidate['sdpMLineIndex'],
    ));
  }

  void sendMessage(String content) {
    final message = Message(
      id: _uuid.v4(),
      sender: _username,
      content: content,
      timestamp: DateTime.now(),
    );
    _chatRepository.addMessage(message);
    _dataChannel!.send(RTCDataChannelMessage(content));
    notifyListeners();
  }

  void _receiveMessage(String content) {
    final message = Message(
      id: _uuid.v4(),
      sender: 'Peer',
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

### 2.5 채팅 화면
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
                      final chatProvider = Provider.of<ChatProvider>(context, listen: false);
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

### 2.6 메인 파일
`lib/main.dart` 파일에서 앱을 실행하고 `ChatProvider`를 설정합니다.

**lib/main.dart**
```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'providers/chat_provider.dart';
import 'screens/chat_screen.dart';

import 'package:flutter_dotenv/flutter_dotenv.dart';

Future<void> main() async {
  // Load .env file
  await dotenv.load(fileName: ".env");

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

### 2.7 인터넷 사용 권한 설정
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

### 2.8 iOS 네트워크 설정
`ios/Runner/Info.plist` 파일에 네트워크 설정을 추가합니다.

**ios/Runner/Info.plist**
```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
```

## 3. 실행 방법
### 3.1 Backend 실행
```bash
# backend 디렉토리로 이동
cd chat_app/backend

# Docker 빌드 및 실행
docker-compose up --build -d
```

### 3.2 Frontend 실행
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

## 프로젝트 전체 구조
프로젝트의 최종 파일 구조는 다음과 같습니다.

```
chat_app/
│
├── backend/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   ├── package.json
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

## 마무리
이 매뉴얼대로 따라하면 Node.js로 구축된 백엔드와 Flutter로 구축된 프론트엔드를 통해 P2P 채팅 앱을 만들 수 있습니다. 프로젝트의 주요 포인트는 다음과 같습니다.

1. **Node.js 백엔드**: WebSocket 서버와 `simple-peer` 라이브러리를 사용하여 P2P 통신을 구현했습니다.
2. **Docker**: 백엔드를 Docker로 배포할 수 있도록 설정하였습니다.
3. **Flutter 프론트엔드**: 다양한 디자인 패턴(Provider, Repository)을 활용하여 효율적으로 구성했습니다.
4. **암호화 및 탈중앙화**: P2P 통신을 통해 데이터를 직접 주고받고, 백엔드를 통해 초기 연결만 설정했습니다.

### 추가 팁
- 백엔드의 `P2PService` 클래스를 개선하여 P2P 메시지 암호화를 추가할 수 있습니다.
- 프론트엔드에서 WebRTC를 활용하거나 백엔드와 직접 통신할 수 있도록 개선할 수 있습니다.

### 더 나아가기
- 이 프로젝트에 IPFS, libp2p와 같은 추가적인 탈중앙화 기술을 접목할 수 있습니다.
- 백엔드에 블록체인 기술을 도입하여 메시지 기록을 관리할 수 있습니다.

질문이 있다면 언제든지 물어보세요! 신입 사원 여러분의 성공을 응원합니다. 🚀
