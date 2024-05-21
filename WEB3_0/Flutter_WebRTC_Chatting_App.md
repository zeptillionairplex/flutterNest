"서버리스(serverless)"라는 용어는 일반적으로 개발자가 직접 서버를 관리하지 않고도 애플리케이션을 개발 및 배포하는 개념을 의미합니다. 이것은 '서버가 전혀 없다'는 뜻은 아니며, 대신 서버 관리의 부담을 줄이고 서버 대신 서비스를 제공하는 클라우드 인프라나 분산 네트워크를 활용하는 것을 의미합니다.

**서버리스의 실제 의미:**
- **개발자 관점:** 직접 서버를 관리하지 않고 애플리케이션 로직에 집중할 수 있음.
- **클라우드 관점:** 실제로 서버가 있지만, 이를 클라우드 서비스가 추상화하여 개발자가 직접 관리할 필요가 없음.
- **예시:** AWS Lambda, Azure Functions, GCP Cloud Functions, 분산 네트워크를 활용한 P2P 애플리케이션.

**Flutter 앱에서의 서버리스 채팅:**
- **완전히 서버 없는 경우:** P2P 네트워크를 활용하여 앱끼리 직접 통신하는 것이 가능하지만, 이를 위해서는 앱 설치 전에 서로의 네트워크 정보를 알고 있어야 하며, NAT 트래버설 및 방화벽 문제를 해결해야 합니다.
- **서버리스 백엔드:** Nest.js와 같은 백엔드 프레임워크를 사용하되, 서버리스 아키텍처에 배포하여 서버 관리 없이 운영하는 방법도 있습니다.

**완전히 서버 없이 P2P 기반 채팅 구현:**
- **라이브러리 활용:** IPFS, Libp2p, WebRTC 등을 사용해 P2P 네트워크 구성.
- **권장 라이브러리:** WebRTC를 사용해 Flutter 앱 간의 P2P 통신을 구현하는 방법이 가장 일반적입니다.

### WebRTC 기반 Flutter 채팅 앱 예제

Flutter에서 WebRTC를 사용하여 P2P 통신을 구축하는 방법을 살펴보겠습니다.

#### 1. `flutter_webrtc` 설정
먼저 WebRTC 라이브러리를 설치합니다.

**pubspec.yaml**
```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_webrtc: ^0.9.23
```

#### 2. Peer-to-Peer 채팅 앱 구현
`lib/main.dart` 파일에 작성합니다.

**main.dart**
```dart
import 'package:flutter/material.dart';
import 'package:flutter_webrtc/flutter_webrtc.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'P2P Chat App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const ChatScreen(),
    );
  }
}

class ChatScreen extends StatefulWidget {
  const ChatScreen({Key? key}) : super(key: key);

  @override
  _ChatScreenState createState() => _ChatScreenState();
}

class _ChatScreenState extends State<ChatScreen> {
  final RTCConfiguration _config = {
    'iceServers': [
      {'urls': 'stun:stun.l.google.com:19302'}
    ]
  };

  late RTCPeerConnection _peerConnection;
  late RTCDataChannel _dataChannel;
  final TextEditingController _controller = TextEditingController();
  final List<String> _messages = [];

  @override
  void initState() {
    super.initState();
    _initializeConnection();
  }

  Future<void> _initializeConnection() async {
    _peerConnection = await createPeerConnection(_config);

    // 데이터 채널 생성
    _dataChannel = await _peerConnection.createDataChannel(
      'chat',
      RTCDataChannelInit(),
    );

    // 데이터 채널 수신 이벤트 핸들러
    _dataChannel.onMessage = (RTCDataChannelMessage message) {
      setState(() {
        _messages.add('Peer: ${message.text}');
      });
    };

    // ICE 후보 이벤트 핸들러
    _peerConnection.onIceCandidate = (RTCIceCandidate candidate) {
      print('ICE Candidate: ${candidate.candidate}');
    };

    // 데이터 채널 이벤트 핸들러
    _peerConnection.onDataChannel = (RTCDataChannel channel) {
      _dataChannel = channel;
      _dataChannel.onMessage = (RTCDataChannelMessage message) {
        setState(() {
          _messages.add('Peer: ${message.text}');
        });
      };
    };

    // SDP 오퍼 생성 및 설정
    RTCSessionDescription offer = await _peerConnection.createOffer();
    await _peerConnection.setLocalDescription(offer);
    print('Local Description: ${offer.sdp}');
  }

  void _sendMessage(String message) {
    _dataChannel.send(RTCDataChannelMessage(message));
    setState(() {
      _messages.add('Me: $message');
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('P2P Chat App'),
      ),
      body: Column(
        children: [
          Expanded(
            child: ListView.builder(
              itemCount: _messages.length,
              itemBuilder: (context, index) {
                return ListTile(
                  title: Text(_messages[index]),
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
                      hintText: 'Enter your message...',
                    ),
                  ),
                ),
                IconButton(
                  icon: const Icon(Icons.send),
                  onPressed: () {
                    if (_controller.text.isNotEmpty) {
                      _sendMessage(_controller.text);
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

### Flutter WebRTC 권한 설정
**Android**
`android/app/src/main/AndroidManifest.xml`에 다음 권한을 추가합니다.

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.CAMERA"/>
<uses-permission android:name="android.permission.RECORD_AUDIO"/>
```

**iOS**
`ios/Runner/Info.plist`에 다음 권한을 추가합니다.

```xml
<key>NSCameraUsageDescription</key>
<string>We need access to your camera for video calls</string>
<key>NSMicrophoneUsageDescription</key>
<string>We need access to your microphone for voice calls</string>
```

### 상대방과 연결하기
위의 코드는 P2P 연결을 시작하는 기본 코드이며 상대방과의 연결을 위해서는 상대방의 SDP 오퍼 또는 응답을 교환해야 합니다. 이를 위해 QR 코드, 이메일, 채팅 앱 등의 다른 방법을 활용하여 서로의 SDP 정보를 공유할 수 있습니다.

### RTCSessionDescription 교환 로직
SDP 오퍼와 응답을 교환하는 방법을 추가합니다.

```dart
void _setRemoteDescription(String sdp, String type) async {
  RTCSessionDescription description = RTCSessionDescription(sdp, type);
  await _peerConnection.setRemoteDescription(description);
}

void _createAnswer() async {
  RTCSessionDescription answer = await _peerConnection.createAnswer();
  await _peerConnection.setLocalDescription(answer);
  print('Local Answer: ${answer.sdp}');
}
```

### 사용 방법
1. 앱을 실행하여 오퍼를 생성합니다.
2. 상대방에게 해당 오퍼를 공유합니다.
3. 상대방이 오퍼를 수락하고 응답(answer)을 생성하여 다시 공유합니다.
4. 서로의 SDP 정보로 원격 설명을 설정합니다.

각자 오퍼나 응답을 공유하는 것은 앱 내부에서 할 수 없기 때문에 이메일, 메시지 앱 등 외부 수단을 사용해야 합니다.

오퍼나 응답을 공유하는 것은 앱 내부에서 할 수 없기 때문에 이메일, 메시지 앱 등 외부 수단을 사용해야 합니다.

### 추가 설명
위에서 구현한 Flutter WebRTC 채팅 앱은 상대방과 직접 연결해 채팅할 수 있도록 기본적인 P2P 통신 구조를 제공합니다. 그러나 실제로는 다음과 같은 추가 기능이 필요할 수 있습니다.

1. **SDP 교환 자동화**: QR 코드나 메시지로 SDP 오퍼를 자동으로 교환하는 기능.
2. **NAT 트래버설**: NAT 환경에서 P2P 연결을 성공적으로 이뤄내기 위한 TURN 서버 사용.
3. **ICE 후보 교환**: 안정적인 연결을 위해 ICE 후보 정보를 교환하는 로직.

Flutter 앱에서 서로 교환할 수 있는 QR 코드를 생성하고 읽을 수 있도록 QR 코드 생성 버튼과 스캔 기능을 추가해보겠습니다. 이를 위해 다음 라이브러리를 사용합니다:

- **qr_flutter**: QR 코드를 생성하기 위한 라이브러리
- **qr_code_scanner**: QR 코드를 스캔하기 위한 라이브러리

### `pubspec.yaml` 업데이트
**pubspec.yaml** 파일에 필요한 라이브러리를 추가합니다.

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_webrtc: ^0.9.23
  qr_flutter: ^4.0.0
  qr_code_scanner: ^1.0.0
```

### `lib/screens/chat_screen.dart` 수정
**chat_screen.dart** 파일을 수정하여 QR 코드 생성 및 스캔 기능을 추가합니다.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_webrtc/flutter_webrtc.dart';
import 'package:qr_flutter/qr_flutter.dart';
import 'package:qr_code_scanner/qr_code_scanner.dart';

class ChatScreen extends StatefulWidget {
  const ChatScreen({Key? key}) : super(key: key);

  @override
  _ChatScreenState createState() => _ChatScreenState();
}

class _ChatScreenState extends State<ChatScreen> {
  final RTCConfiguration _config = {
    'iceServers': [
      {'urls': 'stun:stun.l.google.com:19302'}
    ]
  };

  late RTCPeerConnection _peerConnection;
  late RTCDataChannel _dataChannel;
  final TextEditingController _controller = TextEditingController();
  final List<String> _messages = [];
  String _localSDP = '';

  @override
  void initState() {
    super.initState();
    _initializeConnection();
  }

  Future<void> _initializeConnection() async {
    _peerConnection = await createPeerConnection(_config);

    // 데이터 채널 생성
    _dataChannel = await _peerConnection.createDataChannel(
      'chat',
      RTCDataChannelInit(),
    );

    // 데이터 채널 수신 이벤트 핸들러
    _dataChannel.onMessage = (RTCDataChannelMessage message) {
      setState(() {
        _messages.add('Peer: ${message.text}');
      });
    };

    // ICE 후보 이벤트 핸들러
    _peerConnection.onIceCandidate = (RTCIceCandidate candidate) {
      print('ICE Candidate: ${candidate.candidate}');
    };

    // 데이터 채널 이벤트 핸들러
    _peerConnection.onDataChannel = (RTCDataChannel channel) {
      _dataChannel = channel;
      _dataChannel.onMessage = (RTCDataChannelMessage message) {
        setState(() {
          _messages.add('Peer: ${message.text}');
        });
      };
    };

    // SDP 오퍼 생성 및 설정
    RTCSessionDescription offer = await _peerConnection.createOffer();
    await _peerConnection.setLocalDescription(offer);
    print('Local Description: ${offer.sdp}');
    _localSDP = offer.sdp;
  }

  void _sendMessage(String message) {
    _dataChannel.send(RTCDataChannelMessage(message));
    setState(() {
      _messages.add('Me: $message');
    });
  }

  void _setRemoteDescription(String sdp, String type) async {
    RTCSessionDescription description = RTCSessionDescription(sdp, type);
    await _peerConnection.setRemoteDescription(description);
  }

  void _createAnswer() async {
    RTCSessionDescription answer = await _peerConnection.createAnswer();
    await _peerConnection.setLocalDescription(answer);
    print('Local Answer: ${answer.sdp}');
    _localSDP = answer.sdp;
  }

  void _scanQRCode(BuildContext context) async {
    final result = await Navigator.of(context).push(MaterialPageRoute(
      builder: (context) => const QRScanScreen(),
    ));

    if (result != null) {
      _setRemoteDescription(result, 'offer');
      _createAnswer();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('P2P Chat App'),
      ),
      body: Column(
        children: [
          Expanded(
            child: ListView.builder(
              itemCount: _messages.length,
              itemBuilder: (context, index) {
                return ListTile(
                  title: Text(_messages[index]),
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
                      hintText: 'Enter your message...',
                    ),
                  ),
                ),
                IconButton(
                  icon: const Icon(Icons.send),
                  onPressed: () {
                    if (_controller.text.isNotEmpty) {
                      _sendMessage(_controller.text);
                      _controller.clear();
                    }
                  },
                ),
              ],
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                ElevatedButton(
                  onPressed: () {
                    showDialog(
                      context: context,
                      builder: (context) {
                        return AlertDialog(
                          title: const Text('Share SDP via QR Code'),
                          content: QrImage(
                            data: _localSDP,
                            version: QrVersions.auto,
                            size: 200.0,
                          ),
                          actions: [
                            TextButton(
                              onPressed: () => Navigator.pop(context),
                              child: const Text('Close'),
                            ),
                          ],
                        );
                      },
                    );
                  },
                  child: const Text('Generate QR Code'),
                ),
                ElevatedButton(
                  onPressed: () => _scanQRCode(context),
                  child: const Text('Scan QR Code'),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

class QRScanScreen extends StatefulWidget {
  const QRScanScreen({Key? key}) : super(key: key);

  @override
  _QRScanScreenState createState() => _QRScanScreenState();
}

class _QRScanScreenState extends State<QRScanScreen> {
  final GlobalKey _qrKey = GlobalKey(debugLabel: 'QR');
  late QRViewController _controller;
  String _scannedData = '';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Scan QR Code'),
      ),
      body: Column(
        children: [
          Expanded(
            child: QRView(
              key: _qrKey,
              onQRViewCreated: (QRViewController controller) {
                this._controller = controller;
                controller.scannedDataStream.listen((scanData) {
                  setState(() {
                    _scannedData = scanData.code!;
                    Navigator.pop(context, _scannedData);
                  });
                });
              },
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                ElevatedButton(
                  onPressed: () => Navigator.pop(context),
                  child: const Text('Cancel'),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

### 전체 코드
`lib/screens/chat_screen.dart` 파일 전체 코드:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_webrtc/flutter_webrtc.dart';
import 'package:qr_flutter/qr_flutter.dart';
import 'package:qr_code_scanner/qr_code_scanner.dart';

class ChatScreen extends StatefulWidget {
  const ChatScreen({Key? key}) : super(key: key);

  @override
  _ChatScreenState createState() => _ChatScreenState();
}

class _ChatScreenState extends State<ChatScreen> {
  final RTCConfiguration _config = {
    'iceServers': [
      {'urls': 'stun:stun.l.google.com:19302'}
    ]
  };

  late RTCPeerConnection _peerConnection;
  late RTCDataChannel _dataChannel;
  final TextEditingController _controller = TextEditingController();
  final List<String> _messages = [];
  String _localSDP = '';

  @override
  void initState() {
    super.initState();
    _initializeConnection();
  }

  Future<void> _initializeConnection() async {
    _peerConnection = await createPeerConnection(_config);

    // 데이터 채널 생성
    _dataChannel = await _peerConnection.createDataChannel(
      'chat',
      RTCDataChannelInit(),
    );

    // 데이터 채널 수신 이벤트 핸들러
    _dataChannel.onMessage = (RTCDataChannelMessage message) {
      setState(() {
        _messages.add('Peer: ${message.text}');
      });
    };

    // ICE 후보 이벤트 핸들러
    _peerConnection.onIceCandidate = (RTCIceCandidate candidate) {
      print('ICE Candidate: ${candidate.candidate}');
    };

    // 데이터 채널 이벤트 핸들러
    _peerConnection.onDataChannel = (RTCDataChannel channel) {
      _dataChannel = channel;
      _dataChannel.onMessage = (RTCDataChannelMessage message) {
        setState(() {
          _messages.add('Peer: ${message.text}');
        });
      };
    };

    // SDP 오퍼 생성 및 설정
    RTCSessionDescription offer = await _peerConnection.createOffer();
    await _peerConnection.setLocalDescription(offer);
    print('Local Description: ${offer.sdp}');
    _localSDP = offer.sdp;
  }

  void _sendMessage(String message) {
    _dataChannel.send(RTCDataChannelMessage(message));
    setState(() {
      _messages.add('Me: $message');
    });
  }

  void _setRemoteDescription(String sdp, String type) async {
    RTCSessionDescription description = RTCSessionDescription(sdp, type);
    await _peerConnection.setRemoteDescription(description);
  }

  void _createAnswer() async {
    RTCSessionDescription answer = await _peerConnection.createAnswer();
    await _peerConnection.setLocalDescription(answer);
    print('Local Answer: ${answer.sdp}');
    _localSDP = answer.sdp;
  }

  void _scanQRCode(BuildContext context) async {
    final result = await Navigator.of(context).push(MaterialPageRoute(
      builder: (context) => const QRScanScreen(),
    ));

    if (result != null) {
      _setRemoteDescription(result, 'offer');
      _createAnswer();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('P2P Chat App'),
      ),
      body: Column(
        children: [
          Expanded(
            child: ListView.builder(
              itemCount: _messages.length,
              itemBuilder: (context, index) {
                return ListTile(
                  title: Text(_messages[index]),
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
                      hintText: 'Enter your message...',
                    ),
                  ),
                ),
                IconButton(
                  icon: const Icon(Icons.send),
                  onPressed: () {
                    if (_controller.text.isNotEmpty) {
                      _sendMessage(_controller.text);
                      _controller.clear();
                    }
                  },
                ),
              ],
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                ElevatedButton(
                  onPressed: () {
                    showDialog(
                      context: context,
                      builder: (context) {
                        return AlertDialog(
                          title: const Text('Share SDP via QR Code'),
                          content: QrImage(
                            data: _localSDP,
                            version: QrVersions.auto,
                            size: 200.0,
                          ),
                          actions: [
                            TextButton(
                              onPressed: () => Navigator.pop(context),
                              child: const Text('Close'),
                            ),
                          ],
                        );
                      },
                    );
                  },
                  child: const Text('Generate QR Code'),
                ),
                ElevatedButton(
                  onPressed: () => _scanQRCode(context),
                  child: const Text('Scan QR Code'),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

class QRScanScreen extends StatefulWidget {
  const QRScanScreen({Key? key}) : super(key: key);

  @override
  _QRScanScreenState createState() => _QRScanScreenState();
}

class _QRScanScreenState extends State<QRScanScreen> {
  final GlobalKey _qrKey = GlobalKey(debugLabel: 'QR');
  late QRViewController _controller;
  String _scannedData = '';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Scan QR Code'),
      ),
      body: Column(
        children: [
          Expanded(
            child: QRView(
              key: _qrKey,
              onQRViewCreated: (QRViewController controller) {
                this._controller = controller;
                controller.scannedDataStream.listen((scanData) {
                  setState(() {
                    _scannedData = scanData.code!;
                    Navigator.pop(context, _scannedData);
                  });
                });
              },
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                ElevatedButton(
                  onPressed: () => Navigator.pop(context),
                  child: const Text('Cancel'),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```
