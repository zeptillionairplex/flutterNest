문제는 `flutter doctor`가 Android Studio에서 번들된 Java 버전을 찾을 수 없다고 보고하는 것입니다. Android Studio 내부의 `jbr` 폴더에 Java가 설치되어 있지만, `flutter doctor`는 이를 인식하지 못하는 것 같습니다. 이 문제를 해결하려면 다음 단계를 시도해 보세요.

### 단계 1: Android Studio 내부 JBR Java 버전 확인
1. Android Studio 내부에 Java Runtime이 있는지 확인합니다.
   - `C:\Program Files\Android\Android Studio\jbr`

2. 해당 디렉터리 내 `bin\java` 실행 파일이 존재하는지 확인하세요.

### 단계 2: 환경 변수 설정 확인
1. **시스템 환경 변수 설정**:
   - 시스템 환경 변수 `JAVA_HOME`을 Android Studio `jbr` Java 경로로 설정합니다.
     - 예: `C:\Program Files\Android\Android Studio\jbr`

   - 그리고 `PATH` 환경 변수에 `JAVA_HOME\bin`을 추가합니다.

2. **환경 변수 설정 방법**:
   - **Windows**:
     1. 시작 메뉴에서 "환경 변수"를 검색하고 "시스템 환경 변수 편집"을 선택합니다.
     2. "환경 변수" 버튼을 클릭합니다.
     3. `JAVA_HOME`이라는 새로운 시스템 변수를 추가하고 값으로 `C:\Program Files\Android\Android Studio\jbr` 입력합니다.
     4. `PATH` 변수에 `;%JAVA_HOME%\bin`을 추가합니다.
<details>
<summary>`;%JAVA_HOME%\bin` 앞에 붙는 `;`의 의미와 `%JAVA_HOME%`를 붙이는 의미</summary>
환경 변수 설정 시 `;%JAVA_HOME%\bin` 앞에 붙는 `;`의 의미와 `%JAVA_HOME%` 등 변수 사용에 대해 설명해 드리겠습니다.

### 1. `;`의 역할
`PATH` 환경 변수에 여러 경로를 추가할 때 경로는 세미콜론 (`;`)으로 구분됩니다. 예를 들어, 기존 `PATH`에 새로운 경로를 추가할 때 이렇게 할 수 있습니다.

```cmd
set PATH=%PATH%;C:\new\path
```

따라서, 기존 `PATH`에 `JAVA_HOME`의 `bin` 디렉토리를 추가할 때도 앞에 `;`을 붙여 줍니다.

```cmd
set PATH=%PATH%;%JAVA_HOME%\bin
```

### 2. `%JAVA_HOME%`의 사용
`%JAVA_HOME%`은 Java 설치 경로를 가리키는 환경 변수입니다. 

`JAVA_HOME`이 `C:\Program Files\Java\jdk-11.0.10`을 가리킨다면 `%JAVA_HOME%\bin`은 `C:\Program Files\Java\jdk-11.0.10\bin`을 의미합니다.

### 요약
`;%JAVA_HOME%\bin`을 `PATH`에 추가하려고 할 때, 앞에 `;`를 붙이는 이유는 기존 `PATH`와 구분하기 위해서입니다. 예를 들어, 기존 `PATH`에 추가하는 경우 완전한 명령어는 다음과 같습니다.

```cmd
set PATH=%PATH%;%JAVA_HOME%\bin
```

이렇게 하면 기존 `PATH`에 `JAVA_HOME`의 `bin` 경로가 추가됩니다.

### 시스템 환경 변수에 추가하기
1. **Windows 설정을 통해 추가**
   - 제어판 -> 시스템 -> 고급 시스템 설정을 엽니다.
   - 환경 변수 버튼을 클릭합니다.
   - 시스템 변수에서 `Path`를 선택하고 수정 버튼을 클릭합니다.
   - 새 항목을 추가하거나 기존 항목을 수정하여 `%JAVA_HOME%\bin`을 추가합니다.

2. **명령 프롬프트에서 일시적으로 추가**
   ```cmd
   set PATH=%PATH%;%JAVA_HOME%\bin
   ```

3. **명령 프롬프트에서 영구적으로 추가**
   ```cmd
   setx PATH "%PATH%;%JAVA_HOME%\bin"
   ```

이렇게 하면 다음부터는 `%JAVA_HOME%\bin` 경로가 `PATH`에 포함되어 있어, Java 실행 파일을 어디서든 실행할 수 있습니다.
</details>

### 단계 3: `flutter config` 업데이트
1. `flutter`가 올바른 Java 경로를 인식하도록 `flutter config` 명령어를 사용합니다.

   ```bash
   flutter config --android-studio-dir "C:\Program Files\Android\Android Studio"
   ```

### 단계 4: Android Studio 플러그인 및 업데이트 확인
1. **Flutter 및 Dart 플러그인 설치**:
   - Android Studio에서 `File` > `Settings` > `Plugins`로 이동하여 Flutter 및 Dart 플러그인이 설치되어 있는지 확인합니다.

2. **Android Studio 업데이트**:
   - Android Studio가 최신 버전인지 확인하고 필요한 경우 업데이트합니다.

### 단계 5: `flutter doctor` 다시 실행
설정을 완료한 후 `flutter doctor`를 다시 실행하여 문제가 해결되었는지 확인합니다.

```bash
flutter doctor -v
```

### 추가 조치: Flutter 재설치
위의 모든 단계를 수행한 후에도 문제가 지속된다면 Flutter SDK를 재설치하는 것도 하나의 해결책이 될 수 있습니다.

### 요약
1. Android Studio 내부 Java 경로 확인
2. 시스템 환경 변수 설정
3. `flutter config` 업데이트
4. 플러그인 및 업데이트 확인
5. `flutter doctor` 다시 실행

위 단계를 따라 해당 문제를 해결할 수 있을 것입니다. 그래도 문제가 해결되지 않는다면, 추가적인 로그를 제공하거나 Flutter 공식 GitHub 이슈 페이지에 문제를 제출하는 것도 추천드립니다.
