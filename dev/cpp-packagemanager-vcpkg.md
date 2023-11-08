---
title: C++ 패키지 매니저 vcpkg에 대한 설명
---
# C++ Package Manager - vcpkg
문서에는 cmake 도 지원하고 mac과 linux 도 지원한다고 나와 있습니다.

- 참고 영상 : [https://youtu.be/0h1lC3QHLHU](https://youtu.be/0h1lC3QHLHU)
- 깃 저장소 : [https://github.com/Microsoft/vcpkg/tree/master](https://github.com/Microsoft/vcpkg/tree/master)

### vcpkg 설치
git 이 설치되어 있어야 하며, 글을 쓴 2022년 8월 22일 기준으로 최신 tag는 2022.08.15 입니다.

```shell
git clone https://github.com/microsoft/vcpkg.git
cd vcpkg
git checkout 2022.08.15
bootstrap-vcpkg.bat
```

Quick Start 글에는 tag 로 전환하지 않고 바로 배치파일을 실행 했는데, 저는 tag 된 버전으로 바꾼 다음 실행했습니다.

맥(Apple silicon/M1)에서는 tag를 전환하지 않고 bootstrap-vcpkg.sh 를 실행했는데 잘 동작했습니다.

환경 변수 VCPKG_DEFAULT_TRIPLET 에 : 뒤에 붙는 x64-windows 같은 것의 기본 값을 정해줄 수 있습니다.

### visual Studio 2022 와 통합 순서

nuget package manager 와 연결되어 사용되는 것으로 보이며, 저의 vcpkg 설치 폴더는 C:/SDK/vcpkg 로 아래 명령을 실행할 때는 설치 경로를 감안해서 입력하시면 됩니다.

```shell
vcpkg integrate install
```

이 명령을 실행하면 관리자 권한을 요구하며 cmake 프로젝트를 사용할 때 도움말이 나옵니다. 그 다음

```shell
vcpkg integrate project
```

이 명령을 실행하면 nuget package manager 의 console 에 아래 명령을 입력하라는 내용이 나옵니다.

```shell
Install-Package "vcpkg.C.SDK.vcpkg" -Source "C:\SDK\vcpkg\scripts\buildsystems"
```

Visual Studio 에서 프로젝트를 열고 Package Manager Console 에 위의 Install-Package 관련 내용을 넣으면 됩니다.

### CMake 와 함께 사용할 때

```shell
cmake -B [build directory] -S . -DCMAKE_TOOLCHAIN_FILE=C:/SDK/vcpkg/scripts/buildsystems/vcpkg.cmake
```

를 넣으라고 문서에 나와 있습니다.

CLion에서 사용할 때 설정의 Build,Execution,Deployment 의 CMake 항목 중 CMake option 에 -DCMAKE_TOOLCHAIN_FILE=C:/SDK/vcpkg/scripts/buildsystems/vcpkg.cmake 를 넣으면 됩니다.