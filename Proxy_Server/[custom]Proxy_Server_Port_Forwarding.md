
### 1. Nest.js CLI 설치

먼저 Nest.js CLI를 글로벌로 설치합니다.

```bash
npm install -g @nestjs/cli
```

### 2. 새로운 Nest.js 프로젝트 생성

Nest.js CLI를 사용하여 새로운 프로젝트를 생성합니다.

```bash
nest new MyNestProxyProject
```

이후에 프로젝트 생성이 완료되면 해당 디렉토리로 이동합니다.

```bash
cd MyNestProxyProject
```

다음은 앞서 설명한 대로 Localtunnel을 설치하고, 스크립트를 작성하며, 프로젝트를 설정하는 방법을 정리한 매뉴얼입니다.

## Nest.js와 Localtunnel을 이용한 프록시 서버 설정 매뉴얼

이 매뉴얼은 Nest.js 프로젝트에서 Localtunnel을 사용하여 프록시 서버를 설정하고, Nest.js 서버 시작 시 Localtunnel URL을 자동으로 터미널에 출력하는 방법을 설명합니다.

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

### 5. 프로젝트 실행

터미널에서 다음 명령어를 실행하여 프로젝트를 시작합니다.

```bash
npm run start
```

서버가 시작되면 터미널에 Localtunnel URL이 출력됩니다. 이 URL을 통해 외부에서 로컬 서버에 접근할 수 있습니다.

### 요약

- **Localtunnel 설치**: 프로젝트에 Localtunnel을 설치합니다.
- **Localtunnel 스크립트 작성**: Localtunnel을 실행하고 URL을 터미널에 출력하는 스크립트를 작성합니다.
- **Nest.js 프록시 서버 설정**: 프록시 서비스를 작성하고, Nest.js 서버와 통합합니다.
- **서버 실행**: Nest.js 서버를 실행하면 Localtunnel URL이 터미널에 자동으로 출력됩니다.

이 매뉴얼을 따라하면, Nest.js 서버를 외부에서 접근 가능하게 하고, 프로그램적으로 Localtunnel URL을 터미널에 출력하는 프록시 서버를 설정할 수 있습니다.
