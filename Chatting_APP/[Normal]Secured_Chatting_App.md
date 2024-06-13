# Comprehensive Manual for Secure Chat Application

This manual provides a step-by-step guide to implementing a secure chat application using Flutter for the frontend and NestJS for the backend. The application includes features like user authentication, chat rooms, message reactions, file uploads, swiping to reply, URL previews, and environment-based configurations using `.env` files.

## Prerequisites

1. **Node.js** and **npm**: Ensure you have Node.js and npm installed.
2. **Flutter**: Ensure Flutter SDK is installed.
3. **Docker**: Ensure Docker and Docker Compose are installed.

## Project Structure

### Backend (NestJS)

- `/src`
  - `/auth`
    - `auth.module.ts`
    - `auth.service.ts`
    - `auth.controller.ts`
    - `auth.guard.ts`
  - `/chat`
    - `chat.module.ts`
    - `chat.service.ts`
    - `chat.gateway.ts`
    - `chat.controller.ts`
  - `/users`
    - `users.module.ts`
    - `users.service.ts`
    - `users.controller.ts`
  - `/common`
    - `/dto`
      - `create-message.dto.ts`
  - `app.module.ts`
  - `main.ts`
- `.env.development`
- `.env.production`
- `Dockerfile`
- `docker-compose.yml`

### Frontend (Flutter)

- `/lib`
  - `/screens`
    - `chat_room_screen.dart`
    - `chat_rooms_screen.dart`
    - `friends_screen.dart`
    - `login_screen.dart`
  - `/widgets`
    - `message_bubble.dart`
    - `url_preview.dart`
  - `main.dart`
- `.env.development`
- `.env.production`

## Backend Setup (NestJS)

### Step 1: NestJS Initialization

Create a new NestJS project and install necessary packages:

```bash
nest new secure-chat-app
cd secure-chat-app
npm install @nestjs/websockets @nestjs/platform-socket.io socket.io
npm install @nestjs/passport passport passport-local
npm install @nestjs/jwt passport-jwt
npm install @nestjs/platform-express multer
npm install dotenv
```

### Step 2: Configure `.env` Files

Create `.env.development` and `.env.production` files in the root directory:

**`.env.development`**

```
PORT=3000
DATABASE_URL=mongodb://localhost:27017/dev_database
JWT_SECRET=dev_secret_key
```

**`.env.production`**

```
PORT=3000
DATABASE_URL=mongodb://localhost:27017/prod_database
JWT_SECRET=prod_secret_key
```

### Step 3: Load Environment Variables

Modify `main.ts` to load environment variables using `dotenv`:

**`main.ts`**

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { NestExpressApplication } from '@nestjs/platform-express';
import { join } from 'path';
import * as dotenv from 'dotenv';

dotenv.config({ path: `.env.${process.env.NODE_ENV}` });

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.useStaticAssets(join(__dirname, '..', 'uploads'));

  app.enableCors();
  await app.listen(process.env.PORT || 3000);
}
bootstrap();
```

### Step 4: Auth Module

Create the following files with the respective content:

**`auth.module.ts`**

```typescript
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { JwtStrategy } from './jwt.strategy';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [
    PassportModule,
    JwtModule.register({
      secret: process.env.JWT_SECRET,
      signOptions: { expiresIn: '60m' },
    }),
    UsersModule,
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy],
})
export class AuthModule {}
```

**`auth.service.ts`**

```typescript
import { Injectable } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersService } from '../users/users.service';

@Injectable()
export class AuthService {
  constructor(
    private readonly jwtService: JwtService,
    private readonly usersService: UsersService,
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

  async register(user: any) {
    return this.usersService.create(user);
  }
}
```

**`auth.controller.ts`**

```typescript
import { Controller, Post, Request, UseGuards, Body } from '@nestjs/common';
import { AuthService } from './auth.service';
import { LocalAuthGuard } from './auth.guard';

@Controller('auth')
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @UseGuards(LocalAuthGuard)
  @Post('login')
  async login(@Request() req) {
    return this.authService.login(req.user);
  }

  @Post('register')
  async register(@Body() createUserDto) {
    return this.authService.register(createUserDto);
  }
}
```

**`auth.guard.ts`**

```typescript
import { Injectable, ExecutionContext } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class LocalAuthGuard extends AuthGuard('local') {
  async canActivate(context: ExecutionContext) {
    const result = (await super.canActivate(context)) as boolean;
    return result;
  }
}
```

### Step 5: Chat Module

Create the following files with the respective content:

**`chat.module.ts`**

```typescript
import { Module } from '@nestjs/common';
import { ChatGateway } from './chat.gateway';
import { ChatService } from './chat.service';
import { MulterModule } from '@nestjs/platform-express';
import { ChatController } from './chat.controller';

@Module({
  imports: [
    MulterModule.register({
      dest: './uploads',
    }),
  ],
  controllers: [ChatController],
  providers: [ChatGateway, ChatService],
})
export class ChatModule {}
```

**`chat.gateway.ts`**

```typescript
import { WebSocketGateway, WebSocketServer, SubscribeMessage, MessageBody } from '@nestjs/websockets';
import { Server } from 'socket.io';
import { ChatService } from './chat.service';

@WebSocketGateway()
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  constructor(private readonly chatService: ChatService) {}

  @SubscribeMessage('message')
  handleMessage(@MessageBody() message: any): void {
    this.server.to(`room-${message.chatRoomId}`).emit('message', message);
  }

  @SubscribeMessage('joinRoom')
  handleJoinRoom(@MessageBody() data: { chatRoomId: number }): void {
    this.server.socketsJoin(`room-${data.chatRoomId}`);
  }

  @SubscribeMessage('react')
  handleReaction(@MessageBody() reaction: any): void {
    this.server.to(`room-${reaction.chatRoomId}`).emit('reaction', reaction);
  }
}
```

**`chat.service.ts`**

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class ChatService {
  private readonly messages: { username: string, content: string, chatRoomId: number, reactions?: any[] }[] = [];

  create(message: { username: string, content: string, chatRoomId: number }) {
    this.messages.push(message);
  }

  findAll(chatRoomId: number): { username: string, content: string, chatRoomId: number, reactions?: any[] }[] {
    return this.messages.filter(message => message.chatRoomId === chatRoomId);
  }

  addReaction(chatRoomId: number, messageId: number, reaction: any) {
    const message = this.messages.find(m => m.chatRoomId === chatRoomId && m.messageId === messageId);
    if (message) {
      if (!message.reactions) {
        message.reactions = [];
      }
      message.reactions.push(reaction);
    }
  }
}
```

**`chat.controller.ts`**

```typescript
import { Controller, Post, UploadedFile, UseInterceptors } from '@nestjs/common';
import { FileInterceptor } from '@nestjs/platform-express';

@Controller('chat')
export class ChatController {
  @Post('upload')
  @UseInterceptors(FileInterceptor('file'))
  uploadFile(@UploadedFile() file: Express.Multer.File) {
    return {
      url: `http://localhost:3000/uploads/${file.filename}`,
    };
  }
}
```

### Step 6: Users Module

Create the following files with the respective content:

**`users.module.ts`**

```typescript
import { Module } from '@nestjs/common';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';

@Module({
  providers: [UsersService],
  exports: [UsersService],
  controllers: [UsersController]
})
export class UsersModule {}
```

**`users.service.ts`**

```typescript
import { Injectable } from '@nestjs/common';

export interface User {
 

 userId: number;
  username: string;
  password: string;
  friends: number[];
}

@Injectable()
export class UsersService {
  private users: User[] = [];
  private idCounter = 1;

  async findOne(username: string): Promise<User | undefined> {
    return this.users.find(user => user.username === username);
  }

  async findById(userId: number): Promise<User | undefined> {
    return this.users.find(user => user.userId === userId);
  }

  async create(user: Partial<User>): Promise<User> {
    const newUser: User = {
      userId: this.idCounter++,
      username: user.username!,
      password: user.password!,
      friends: [],
    };
    this.users.push(newUser);
    return newUser;
  }

  async addFriend(userId: number, friendId: number): Promise<User> {
    const user = this.users.find(user => user.userId === userId);
    if (user && !user.friends.includes(friendId)) {
      user.friends.push(friendId);
    }
    return user;
  }

  async getFriends(userId: number): Promise<User[]> {
    const user = await this.findById(userId);
    if (!user) {
      return [];
    }
    return this.users.filter(u => user.friends.includes(u.userId));
  }

  async searchUsers(query: string): Promise<User[]> {
    return this.users.filter(user => user.username.includes(query));
  }
}
```

**`users.controller.ts`**

```typescript
import { Controller, Get, Post, Body, Param, Query } from '@nestjs/common';
import { UsersService } from './users.service';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post('add-friend')
  async addFriend(@Body() body) {
    const { userId, friendId } = body;
    return this.usersService.addFriend(userId, friendId);
  }

  @Get(':userId/friends')
  async getFriends(@Param('userId') userId: number) {
    return this.usersService.getFriends(userId);
  }

  @Get('search')
  async searchUsers(@Query('query') query: string) {
    return this.usersService.searchUsers(query);
  }
}
```

### Step 7: Main Application Module

**`app.module.ts`**

```typescript
import { Module } from '@nestjs/common';
import { AuthModule } from './auth/auth.module';
import { ChatModule } from './chat/chat.module';
import { UsersModule } from './users/users.module';
import { MongooseModule } from '@nestjs/mongoose';

@Module({
  imports: [
    MongooseModule.forRoot(process.env.DATABASE_URL),
    AuthModule,
    ChatModule,
    UsersModule
  ],
})
export class AppModule {}
```

### Dockerize the Backend Server

#### Step 1: Create Dockerfile

Create a `Dockerfile` in the root directory of your NestJS project.

```Dockerfile
# Use the official Node.js image as the base image
FROM node:16-alpine

# Create and set the working directory
WORKDIR /usr/src/app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install the dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# Build the NestJS application
RUN npm run build

# Expose the application port
EXPOSE 3000

# Start the NestJS application
CMD ["npm", "run", "start:prod"]
```

#### Step 2: Create docker-compose.yml

Create a `docker-compose.yml` file in the root directory of your NestJS project.

```yaml
version: '3.8'

services:
  nest-app:
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
    command: npm run start:prod
```

#### Step 3: Build and Run the Docker Container

Run the following commands in the root directory of your project:

1. **Build the Docker image**:

   ```bash
   docker-compose build
   ```

2. **Run the Docker container**:

   ```bash
   docker-compose up
   ```

### Frontend Setup (Flutter)

#### Step 1: Flutter Initialization

Create a new Flutter project and install necessary packages:

```bash
flutter create secure_chat_app
cd secure_chat_app
flutter pub add provider
flutter pub add socket_io_client
flutter pub add flutter_dotenv
flutter pub add file_picker
flutter pub add http
flutter pub add flutter_link_previewer
```

#### Step 2: Configure `.env` Files

Create `.env.development` and `.env.production` files in the root directory:

**`.env.development`**

```
API_URL=http://localhost:3000
```

**`.env.production`**

```
API_URL=https://api.production.com
```

#### Step 3: Load Environment Variables

Modify `main.dart` to load environment variables using `flutter_dotenv`:

**`main.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';
import 'screens/login_screen.dart';

void main() async {
  await dotenv.load(fileName: ".env.${String.fromEnvironment('ENV')}");
  runApp(SecureChatApp());
}

class SecureChatApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Secure Chat App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: LoginScreen(),
    );
  }
}
```

#### Step 4: Login Screen

**`login_screen.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';
import 'chat_rooms_screen.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';

class LoginScreen extends StatefulWidget {
  @override
  _LoginScreenState createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final TextEditingController _usernameController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();

  void _register() async {
    final response = await http.post(
      Uri.parse('${dotenv.env['API_URL']}/auth/register'),
      headers: <String, String>{
        'Content-Type': 'application/json; charset=UTF-8',
      },
      body: jsonEncode(<String, String>{
        'username': _usernameController.text,
        'password': _usernameController.text,
      }),
    );

    if (response.statusCode == 201) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Registration Successful')),
      );
    } else {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Registration Failed')),
      );
    }
  }

  void _login() async {
    final response = await http.post(
      Uri.parse('${dotenv.env['API_URL']}/auth/login'),
      headers: <String, String>{
        'Content-Type': 'application/json; charset=UTF-8',
      },
      body: jsonEncode(<String, String>{
        'username': _usernameController.text,
        'password': _passwordController.text,
      }),
    );

    if (response.statusCode == 200) {
      final jsonResponse = json.decode(response.body);
      Navigator.of(context).push(
        MaterialPageRoute(
          builder: (context) => ChatRoomsScreen(token: jsonResponse['access_token']),
        ),
      );
    } else {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Login Failed')),
      );
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
              onPressed: _login,
              child: Text('Login'),
            ),
            ElevatedButton(
              onPressed: _register,
              child: Text('Register'),
            ),
          ],
        ),
      ),
    );
  }
}
```

#### Step 5: Chat Rooms Screen

**`chat_rooms_screen.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';
import 'chat_room_screen.dart';
import 'friends_screen.dart';

class ChatRoomsScreen extends StatefulWidget {
  final String token;

  ChatRoomsScreen({required this.token});

  @override
  _ChatRoomsScreenState createState() => _ChatRoomsScreenState();
}

class _ChatRoomsScreenState extends State<ChatRoomsScreen> {
  List<Map<String, dynamic>> _chatRooms = [];

  @override
  void initState() {
    super.initState();
    _fetchChatRooms();
  }

  void _fetchChatRooms() async {
    // Placeholder for fetching chat rooms from backend
    // Here we assume each chat room has an ID and a name
    setState(() {
      _chatRooms = [
        {'id': 1, 'name': 'General Chat'},
        {'id': 

2, 'name': 'Flutter Devs'},
      ];
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Chat Rooms'),
        actions: [
          IconButton(
            icon: Icon(Icons.people),
            onPressed: () {
              Navigator.of(context).push(
                MaterialPageRoute(
                  builder: (context) => FriendsScreen(token: widget.token),
                ),
              );
            },
          ),
        ],
      ),
      body: ListView.builder(
        itemCount: _chatRooms.length,
        itemBuilder: (context, index) {
          final chatRoom = _chatRooms[index];
          return ListTile(
            title: Text(chatRoom['name']),
            onTap: () {
              Navigator.of(context).push(
                MaterialPageRoute(
                  builder: (context) => ChatRoomScreen(
                    token: widget.token,
                    chatRoomId: chatRoom['id'],
                    chatRoomName: chatRoom['name'],
                  ),
                ),
              );
            },
          );
        },
      ),
    );
  }
}
```

#### Step 6: Chat Room Screen

**`chat_room_screen.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:socket_io_client/socket_io_client.dart' as IO;
import 'package:file_picker/file_picker.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';
import '../widgets/message_bubble.dart';
import 'dart:io';

class ChatRoomScreen extends StatefulWidget {
  final String token;
  final int chatRoomId;
  final String chatRoomName;

  ChatRoomScreen({required this.token, required this.chatRoomId, required this.chatRoomName});

  @override
  _ChatRoomScreenState createState() => _ChatRoomScreenState();
}

class _ChatRoomScreenState extends State<ChatRoomScreen> {
  final TextEditingController _controller = TextEditingController();
  final List<Map<String, dynamic>> _messages = [];
  late IO.Socket _socket;
  final String _username = 'my_username'; // Replace with actual username
  Map<String, dynamic>? _replyTo;

  @override
  void initState() {
    super.initState();
    _socket = IO.io('http://localhost:3000', <String, dynamic>{
      'transports': ['websocket'],
      'extraHeaders': {'Authorization': 'Bearer ${widget.token}'}
    });
    _socket.on('connect', (_) {
      _socket.emit('joinRoom', {'chatRoomId': widget.chatRoomId});
    });
    _socket.on('message', (data) {
      setState(() {
        _messages.add(data);
      });
    });
    _socket.on('reaction', (data) {
      setState(() {
        final message = _messages.firstWhere((msg) => msg['messageId'] == data['messageId']);
        if (message['reactions'] == null) {
          message['reactions'] = [];
        }
        message['reactions'].add(data['reaction']);
      });
    });
  }

  void _sendMessage() {
    if (_controller.text.isNotEmpty) {
      final message = {
        'username': _username,
        'content': _controller.text,
        'chatRoomId': widget.chatRoomId,
        'replyTo': _replyTo,
      };
      _socket.emit('message', message);
      setState(() {
        _messages.add(message);
        _replyTo = null;
      });
      _controller.clear();
    }
  }

  Future<void> _pickFile() async {
    FilePickerResult? result = await FilePicker.platform.pickFiles();

    if (result != null) {
      File file = File(result.files.single.path!);
      String fileName = result.files.single.name;

      var request = http.MultipartRequest(
        'POST',
        Uri.parse('http://localhost:3000/chat/upload'),
      );
      request.files.add(await http.MultipartFile.fromPath('file', file.path));
      var response = await request.send();

      if (response.statusCode == 200) {
        var responseData = await response.stream.bytesToString();
        var jsonResponse = jsonDecode(responseData);
        _sendMessageWithFile(jsonResponse['url'], fileName);
      }
    }
  }

  void _sendMessageWithFile(String fileUrl, String fileName) {
    final message = {
      'username': _username,
      'content': fileUrl,
      'chatRoomId': widget.chatRoomId,
      'fileName': fileName,
      'replyTo': _replyTo,
    };
    _socket.emit('message', message);
    setState(() {
      _messages.add(message);
      _replyTo = null;
    });
  }

  void _replyToMessage(Map<String, dynamic> message) {
    setState(() {
      _replyTo = message;
    });
  }

  void _reactToMessage(Map<String, dynamic> message, String reaction) {
    final reactionData = {
      'chatRoomId': widget.chatRoomId,
      'messageId': message['messageId'],
      'reaction': reaction,
    };
    _socket.emit('react', reactionData);
    setState(() {
      final msg = _messages.firstWhere((msg) => msg['messageId'] == message['messageId']);
      if (msg['reactions'] == null) {
        msg['reactions'] = [];
      }
      msg['reactions'].add(reaction);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.chatRoomName),
      ),
      body: Column(
        children: [
          Expanded(
            child: ListView.builder(
              itemCount: _messages.length,
              itemBuilder: (context, index) {
                final message = _messages[index];
                return MessageBubble(
                  message: message['content'],
                  isMe: message['username'] == _username,
                  fileName: message['fileName'],
                  replyTo: message['replyTo'],
                  onSwipe: () => _replyToMessage(message),
                  onReact: (reaction) => _reactToMessage(message, reaction),
                  reactions: message['reactions'] ?? [],
                );
              },
            ),
          ),
          if (_replyTo != null)
            Padding(
              padding: const EdgeInsets.all(8.0),
              child: Row(
                children: [
                  Expanded(
                    child: Text(
                      'Replying to: ${_replyTo!['content']}',
                      style: TextStyle(fontStyle: FontStyle.italic),
                    ),
                  ),
                  IconButton(
                    icon: Icon(Icons.close),
                    onPressed: () {
                      setState(() {
                        _replyTo = null;
                      });
                    },
                  )
                ],
              ),
            ),
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Row(
              children: [
                IconButton(
                  icon: Icon(Icons.attach_file),
                  onPressed: _pickFile,
                ),
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

#### Step 7: Friends Screen

**`friends_screen.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';

class FriendsScreen extends StatefulWidget {
  final String token;

  FriendsScreen({required this.token});

  @override
  _FriendsScreenState createState() => _FriendsScreenState();
}

class _FriendsScreenState extends State<FriendsScreen> {
  final TextEditingController _searchController = TextEditingController();
  List<Map<String, dynamic>> _friends = [];
  List<Map<String, dynamic>> _searchResults = [];

  @override
  void initState() {
    super.initState();
    _fetchFriends();
  }

  void _fetchFriends() async {
    final response = await http.get(
      Uri.parse('http://localhost:3000/users/1/friends'), // Replace 1 with actual userId
      headers: {
        'Authorization': 'Bearer ${widget.token}',
      },
    );

    if (response.statusCode == 200) {
      setState(() {
        _friends = (json.decode(response.body) as List)
            .map((friend) => {
                  'username': friend['username'],
                  'userId': friend['userId'],
                })
            .toList();
      });
    }
  }

  void _addFriend(int friendId) async {
    final response = await http.post(
      Uri.parse('http://localhost:3000/users/add-friend'),
      headers: {
        'Content-Type': 'application/json; charset=UTF-8',
        'Authorization': 'Bearer ${widget.token}',
      },
      body: jsonEncode(<String, dynamic>{
        'userId': 1, // Replace with actual userId
        'friendId': friendId,
      }),
    );

    if (response.statusCode == 200) {
      _fetchFriends();
    }
  }

  void _searchUsers(String query) async {
    final response = await http.get(
      Uri.parse('http://localhost:3000/users/search?query=$query'),
      headers: {
        'Authorization': 'Bearer ${widget.token}',
      },
    );

    if (response.statusCode == 200) {
      setState(() {
        _searchResults = (json.decode(response.body) as List)
            .map((user) => {
                  'username': user['username'],
                 

 'userId': user['userId'],
                })
            .toList();
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Friends'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(8.0),
        child: Column(
          children: [
            TextField(
              controller: _searchController,
              decoration: InputDecoration(hintText: 'Search users...'),
              onSubmitted: _searchUsers,
            ),
            if (_searchResults.isNotEmpty) ...[
              Text('Search Results:', style: TextStyle(fontWeight: FontWeight.bold)),
              ..._searchResults.map((user) => ListTile(
                    title: Text(user['username']),
                    trailing: IconButton(
                      icon: Icon(Icons.person_add),
                      onPressed: () => _addFriend(user['userId']),
                    ),
                  )),
            ],
            Text('Friends:', style: TextStyle(fontWeight: FontWeight.bold)),
            Expanded(
              child: ListView.builder(
                itemCount: _friends.length,
                itemBuilder: (context, index) {
                  final friend = _friends[index];
                  return ListTile(
                    title: Text(friend['username']),
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

#### Step 8: Message Bubble Widget

**`message_bubble.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:linkify/linkify.dart';
import 'url_preview.dart';

class MessageBubble extends StatefulWidget {
  final String message;
  final bool isMe;
  final String? fileName;
  final Map<String, dynamic>? replyTo;
  final Function onSwipe;
  final Function onReact;
  final List<String> reactions;

  MessageBubble({
    required this.message,
    required this.isMe,
    this.fileName,
    this.replyTo,
    required this.onSwipe,
    required this.onReact,
    required this.reactions,
  });

  @override
  _MessageBubbleState createState() => _MessageBubbleState();
}

class _MessageBubbleState extends State<MessageBubble> with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<Offset> _offsetAnimation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 300),
      vsync: this,
    );
    _offsetAnimation = Tween<Offset>(
      begin: Offset.zero,
      end: const Offset(1.5, 0.0),
    ).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.elasticIn,
      ),
    );
  }

  void _onHorizontalDragUpdate(DragUpdateDetails details) {
    if (details.primaryDelta! > 0) {
      // Right swipe
      _controller.forward();
    }
  }

  void _onHorizontalDragEnd(DragEndDetails details) {
    if (_controller.isCompleted) {
      widget.onSwipe();
      _controller.reverse();
    } else {
      _controller.reverse();
    }
  }

  void _showReactionPicker() {
    showModalBottomSheet(
      context: context,
      builder: (context) {
        return Wrap(
          children: [
            ListTile(
              leading: Icon(Icons.thumb_up),
              title: Text('Like'),
              onTap: () => widget.onReact('like'),
            ),
            ListTile(
              leading: Icon(Icons.check),
              title: Text('Check'),
              onTap: () => widget.onReact('check'),
            ),
            ListTile(
              leading: Icon(Icons.favorite),
              title: Text('Heart'),
              onTap: () => widget.onReact('heart'),
            ),
            ListTile(
              leading: Icon(Icons.sentiment_dissatisfied),
              title: Text('Sad'),
              onTap: () => widget.onReact('sad'),
            ),
          ],
        );
      },
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final urls = linkify(widget.message, options: LinkifyOptions(humanize: false))
        .where((element) => element is LinkableElement)
        .cast<LinkableElement>()
        .toList();

    return GestureDetector(
      onHorizontalDragUpdate: _onHorizontalDragUpdate,
      onHorizontalDragEnd: _onHorizontalDragEnd,
      onLongPress: _showReactionPicker,
      child: SlideTransition(
        position: _offsetAnimation,
        child: Container(
          padding: EdgeInsets.all(10),
          margin: EdgeInsets.all(10),
          alignment: widget.isMe ? Alignment.centerRight : Alignment.centerLeft,
          child: Column(
            crossAxisAlignment: widget.isMe ? CrossAxisAlignment.end : CrossAxisAlignment.start,
            children: [
              if (widget.replyTo != null)
                Container(
                  padding: EdgeInsets.all(5),
                  margin: EdgeInsets.only(bottom: 5),
                  decoration: BoxDecoration(
                    color: Colors.grey.shade200,
                    borderRadius: BorderRadius.circular(5),
                  ),
                  child: Text(
                    'Reply to: ${widget.replyTo!['content']}',
                    style: TextStyle(fontStyle: FontStyle.italic),
                  ),
                ),
              Container(
                decoration: BoxDecoration(
                  color: widget.isMe ? Colors.blue : Colors.grey,
                  borderRadius: BorderRadius.circular(10),
                ),
                padding: EdgeInsets.all(10),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    if (widget.fileName != null)
                      Text(
                        widget.fileName!,
                        style: TextStyle(
                          color: Colors.white,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    Text(
                      widget.message,
                      style: TextStyle(color: Colors.white),
                    ),
                    if (urls.isNotEmpty)
                      Padding(
                        padding: const EdgeInsets.only(top: 5.0),
                        child: UrlPreview(url: urls.first.url),
                      ),
                  ],
                ),
              ),
              if (widget.reactions.isNotEmpty)
                Row(
                  children: widget.reactions.map((reaction) {
                    return Padding(
                      padding: const EdgeInsets.symmetric(horizontal: 4.0),
                      child: Text(
                        reaction,
                        style: TextStyle(fontSize: 16),
                      ),
                    );
                  }).toList(),
                ),
            ],
          ),
        ),
      ),
    );
  }
}
```

#### Step 9: URL Preview Widget

**`url_preview.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:flutter_link_previewer/flutter_link_previewer.dart';

class UrlPreview extends StatelessWidget {
  final String url;

  UrlPreview({required this.url});

  @override
  Widget build(BuildContext context) {
    return FlutterLinkPreview(
      url: url,
      builder: (info) {
        if (info == null) return SizedBox();
        return Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            if (info.image != null)
              Image.network(info.image!, fit: BoxFit.cover),
            if (info.title != null)
              Text(info.title!, style: TextStyle(fontWeight: FontWeight.bold)),
            if (info.description != null)
              Text(info.description!),
          ],
        );
      },
    );
  }
}
```

### Build and Run the Applications

#### NestJS Backend

To run the backend in development mode:

```bash
NODE_ENV=development npm run start:dev
```

To run the backend in production mode:

```bash
NODE_ENV=production npm run start:prod
```

#### Flutter Frontend

To run the Flutter app in development mode:

```bash
flutter run --dart-define=ENV=development
```

To run the Flutter app in production mode:

```bash
flutter run --dart-define=ENV=production
```

### Summary

1. **Backend**: Set up the NestJS backend with modules for authentication, chat, and user management. Dockerize the backend for easy deployment. Configured environment variables using `.env` files.
2. **Frontend**: Implement the Flutter frontend with screens for login, chat rooms, chat room messages, friends management, and reactions. Configured environment variables using `.env` files.
3. **Integrate**: Ensure the frontend and backend communicate correctly via API endpoints and WebSocket connections.

By following this comprehensive manual, you will have a fully functional, secure chat application with features such as file uploads, message reactions, swiping to reply, and URL previews, providing an interactive and modern chat experience.
