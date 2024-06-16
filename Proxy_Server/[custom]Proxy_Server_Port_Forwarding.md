## Nest.js와 Localtunnel을 이용한 프록시 서버 설정 매뉴얼 (Docker 포함)

이 매뉴얼은 Nest.js 프로젝트를 Docker로 컨테이너화하고, Localtunnel을 사용하여 프록시 서버를 설정하며, 서버 시작 시 Localtunnel URL을 터미널에 출력하는 방법을 설명합니다. Docker Compose를 사용하여 여러 컨테이너를 관리하는 방법도 포함합니다.

### 파일 구조

```
/MyNestProxyProject
    |-- src
        |-- main.ts
        |-- app.module.ts
        |-- proxy
            |-- proxy.module.ts
            |-- proxy.controller.ts
            |-- proxy.service.ts
    |-- package.json
    |-- tsconfig.json
    |-- start-localtunnel.js
    |-- Dockerfile
    |-- docker-compose.yml
```

### 1. 준비 작업

#### 1.1 Nest.js 프로젝트 생성

터미널에서 다음 명령어를 실행하여 Nest.js 프로젝트를 생성합니다.

```bash
npm install -g @nestjs/cli
nest new MyNestProxyProject
cd MyNestProxyProject
```

#### 1.2 Localtunnel 설치

프로젝트 디렉토리에서 다음 명령어를 실행하여 Localtunnel을 설치합니다.

```bash
npm install localtunnel
```

### 2. Localtunnel 스크립트 작성

프로젝트 루트에 `start-localtunnel.js` 파일을 생성하고 다음 코드를 작성합니다.

**start-localtunnel.js**

```javascript
const localtunnel = require('localtunnel');

(async () => {
  const port = 3000; // Nest.js 서버가 실행 중인 포트
  const tunnel = await localtunnel({ port });

  console.log(`LocalTunnel is running at: ${tunnel.url}`);

  // 터널이 닫히는 경우 핸들링
  tunnel.on('close', () => {
    console.log('LocalTunnel is closed');
  });
})();
```

### 3. Nest.js 프록시 서버 설정

#### 3.1 프록시 서비스 작성

**src/proxy/proxy.service.ts**

```typescript
import { Injectable } from '@nestjs/common';
import { createProxyMiddleware } from 'http-proxy-middleware';
import { Request, Response } from 'express';

@Injectable()
export class ProxyService {
  getProxyMiddleware(target: string) {
    return createProxyMiddleware({
      target,
      changeOrigin: true,
      pathRewrite: { '^/proxy': '' },
      onProxyReq: (proxyReq, req: Request, res: Response) => {
        console.log(`Proxying request to: ${target}${req.url}`);
      },
    });
  }
}
```

#### 3.2 프록시 컨트롤러 작성

**src/proxy/proxy.controller.ts**

```typescript
import { Controller, All, Req, Res, Next } from '@nestjs/common';
import { ProxyService } from './proxy.service';
import { Request, Response, NextFunction } from 'express';

@Controller('proxy')
export class ProxyController {
  constructor(private readonly proxyService: ProxyService) {}

  @All('*')
  handleRequest(@Req() req: Request, @Res() res: Response, @Next() next: NextFunction) {
    const target = 'http://localhost:3000'; // 여기서 로컬 서버의 주소를 지정합니다.
    const proxy = this.proxyService.getProxyMiddleware(target);
    proxy(req, res, next);
  }
}
```

#### 3.3 프록시 모듈 작성

**src/proxy/proxy.module.ts**

```typescript
import { Module } from '@nestjs/common';
import { ProxyController } from './proxy.controller';
import { ProxyService } from './proxy.service';

@Module({
  controllers: [ProxyController],
  providers: [ProxyService],
})
export class ProxyModule {}
```

#### 3.4 앱 모듈 업데이트

**src/app.module.ts**

```typescript
import { Module } from '@nestjs/common';
import { ProxyModule } from './proxy/proxy.module';

@Module({
  imports: [ProxyModule],
})
export class AppModule {}
```

### 4. Localtunnel과 서버 통합

**src/main.ts**

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { exec } from 'child_process';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);

  // LocalTunnel 스크립트 실행
  const localtunnelProcess = exec('node start-localtunnel.js');

  // LocalTunnel 출력 로그를 동일한 터미널에 출력
  localtunnelProcess.stdout.on('data', (data) => {
    console.log(data.toString());
  });

  localtunnelProcess.stderr.on('data', (data) => {
    console.error(data.toString());
  });
}

bootstrap();
```

### 5. Docker 설정

#### 5.1 Dockerfile 작성

프로젝트 루트에 `Dockerfile` 파일을 생성하고 다음 코드를 작성합니다.

**Dockerfile**

```Dockerfile
# Use the official Nest.js base image
FROM node:14

# Set the working directory
WORKDIR /usr/src/app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# Expose the port the app runs on
EXPOSE 3000

# Run the application
CMD ["npm", "run", "start"]
```

#### 5.2 Docker Compose 파일 작성

프로젝트 루트에 `docker-compose.yml` 파일을 생성하고 다음 코드를 작성합니다.

**docker-compose.yml**

```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - '3000:3000'
    command: sh -c "npm run start"
```

### 6. 프로젝트 실행

#### 6.1 Docker Compose를 사용하여 프로젝트 실행

터미널에서 다음 명령어를 실행하여 Docker Compose로 프로젝트를 시작합니다.

```bash
docker-compose up --build
```

서버가 시작되면 터미널에 Localtunnel URL이 출력됩니다. 이 URL을 통해 외부에서 로컬 서버에 접근할 수 있습니다.

### 요약

- **Localtunnel 설치**: 프로젝트에 Localtunnel을 설치합니다.
- **Localtunnel 스크립트 작성**: Localtunnel을 실행하고 URL을 터미널에 출력하는 스크립트를 작성합니다.
- **Nest.js 프록시 서버 설정**: 프록시 서비스를 작성하고, Nest.js 서버와 통합합니다.
- **Docker 설정**: Dockerfile과 Docker Compose 파일을 작성하여 프로젝트를 컨테이너화합니다.
- **프로젝트 실행**: Docker Compose를 사용하여 프로젝트를 실행하면 Localtunnel URL이 터미널에 자동으로 출력됩니다.

이 매뉴얼을 따라하면, Nest.js 서버를 Docker로 컨테이너화하고, 외부에서 접근 가능하게 하며, 프로그램적으로 Localtunnel URL을 터미널에 출력하는 프록시 서버를 설정할 수 있습니다.
