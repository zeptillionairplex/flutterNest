### 개발 환경 버전 정보
Flutter와 Nest.js 개발을 위해 필요한 버전 정보를 아래 표로 정리하였습니다.

#### Flutter 개발 환경
| **환경**             | **버전**            | **설명**                                             |
|----------------------|---------------------|-----------------------------------------------------|
| Flutter SDK         | 3.10.0              | Flutter 앱 개발을 위해 필요한 SDK 버전입니다.         |
| Dart SDK            | 3.0.0               | Dart 프로그래밍 언어의 SDK 버전입니다.               |
| Android Studio      | Arctic Fox 2020.3.1 | Flutter 개발에 필요한 IDE 버전입니다.                |
| Gradle Plugin       | 7.2.0               | Android 프로젝트에 사용되는 Gradle 플러그인 버전입니다.|
| Kotlin              | 1.5.31              | Android 프로젝트에서 사용하는 Kotlin 버전입니다.      |

#### Nest.js 개발 환경
| **환경**             | **버전**            | **설명**                                             |
|----------------------|---------------------|-----------------------------------------------------|
| Node.js             | 18.16.0             | Nest.js를 실행하기 위한 Node.js 버전입니다.            |
| npm                 | 9.5.1               | Node 패키지 관리 도구 버전입니다.                     |
| Nest.js CLI         | 10.0.0              | Nest.js 프로젝트 생성 및 관리에 필요한 CLI 버전입니다. |
| Nest.js Framework   | 10.0.0              | Nest.js 프레임워크 버전입니다.                        |
| TypeORM             | 0.3.17              | 데이터베이스 연결을 위해 사용된 ORM 버전입니다.        |
| MySQL               | 8.0.33              | 데이터베이스 버전입니다.                             |

### 각 버전별 설치 방법 및 설정

#### 1. Flutter 개발 환경 설정
1. **Flutter SDK 설치**  
   [Flutter SDK 다운로드 링크](https://docs.flutter.dev/get-started/install)
   [flutter_windows_3.10.0-stable.zip](https://storage.googleapis.com/flutter_infra_release/releases/stable/windows/flutter_windows_3.10.0-stable.zip)
   - 설치 후 환경 변수에 Flutter 경로를 추가합니다.

3. **Android Studio 설치**  
   [Android Studio 다운로드 링크](https://developer.android.com/studio)
   [android-studio-2020.3.1.22-windows.exe](https://redirector.gvt1.com/edgedl/android/studio/install/2020.3.1.22/android-studio-2020.3.1.22-windows.exe)
   - 설치 후 Flutter 및 Dart 플러그인을 설치합니다.

5. **Flutter SDK 및 Android Studio 버전 확인**
   아래 명령어로 Flutter 환경 정보를 확인할 수 있습니다.
   ```bash
   flutter doctor
   ```

6. **Gradle 및 Kotlin 버전 설정**
   `android/build.gradle` 파일에서 해당 버전을 설정합니다.
   ```gradle
   buildscript {
       ext.kotlin_version = '1.5.31'
       repositories {
           google()
           mavenCentral()
       }
       dependencies {
           classpath 'com.android.tools.build:gradle:7.2.0'
           classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
       }
   }
   ```

#### 2. Nest.js 개발 환경 설정
1. **Node.js 설치**  
   [Node.js 다운로드 링크](https://nodejs.org/dist/v18.16.0/)
   [node.exe v18.16.0](https://nodejs.org/dist/v18.16.0/win-x64/node.exe)
   - Node.js 설치 시 npm도 함께 설치됩니다.

3. **Nest.js CLI 설치**  
   아래 명령어로 Nest.js CLI를 설치합니다.
   ```bash
   npm i -g @nestjs/cli@10.0.0
   ```

4. **프로젝트 생성 및 패키지 설치**
   프로젝트를 생성하고 필요한 패키지를 설치합니다.
   ```bash
   nest new shopping-mall-backend
   cd shopping-mall-backend
   npm install @nestjs/typeorm@10.0.0 typeorm@0.3.17 mysql
   npm install @nestjs/config@2.0.0
   ```

5. **프로젝트 구성 파일**
   위에서 제공된 Nest.js 및 Flutter 코드 조각을 각 프로젝트에 복사하여 사용하세요.

### 종합 버전 정보 표
| **환경**             | **버전**            | **설명**                                             |
|----------------------|---------------------|-----------------------------------------------------|
| Flutter SDK         | 3.10.0              | Flutter 앱 개발을 위해 필요한 SDK 버전입니다.         |
| Dart SDK            | 3.0.0               | Dart 프로그래밍 언어의 SDK 버전입니다.               |
| Android Studio      | Arctic Fox 2020.3.1 | Flutter 개발에 필요한 IDE 버전입니다.                |
| Gradle Plugin       | 7.2.0               | Android 프로젝트에 사용되는 Gradle 플러그인 버전입니다.|
| Kotlin              | 1.5.31              | Android 프로젝트에서 사용하는 Kotlin 버전입니다.      |
| Node.js             | 18.16.0             | Nest.js를 실행하기 위한 Node.js 버전입니다.            |
| npm                 | 9.5.1               | Node 패키지 관리 도구 버전입니다.                     |
| Nest.js CLI         | 10.0.0              | Nest.js 프로젝트 생성 및 관리에 필요한 CLI 버전입니다. |
| Nest.js Framework   | 10.0.0              | Nest.js 프레임워크 버전입니다.                        |
| TypeORM             | 0.3.17              | 데이터베이스 연결을 위해 사용된 ORM 버전입니다.        |
| MySQL               | 8.0.33              | 데이터베이스 버전입니다.                             |

이렇게 버전 정보를 정리한 후, 제공된 프로젝트 코드 조각을 그대로 복사하여 실행하면 바로 작동하는 쇼핑몰 앱을 구축할 수 있습니다.
