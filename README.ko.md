# FlowPort

[English](README.md) · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

[공식 웹사이트](https://flowport.pages.dev/ko/) · [다운로드](https://flowport.pages.dev/ko/downloads/) · [문서](https://flowport.pages.dev/ko/docs/) · [변경 이력](https://flowport.pages.dev/ko/changelog/)

FlowPort는 **Source → Pipeline → Destinations**를 통해 로그, 메트릭, 트레이스, 이벤트를 수집하고 처리한 뒤 대상별로 라우팅하는 플랫폼입니다.

**이 저장소에는 제품 문서와 공식 웹사이트 링크만 게시합니다. 제품 및 웹사이트의 소스 코드, 소스 이력, 바이너리는 공개하지 않습니다.** Linux 패키지, 체크섬, 릴리스 매니페스트는 Cloudflare의 공식 웹사이트에서 제공합니다. 문서 공개가 제품 소스 코드의 오픈 소스화를 의미하지는 않습니다. 에디션과 라이선스는 공식 웹사이트를 확인하세요.

## 시작하기

1. [Linux 패키지를 다운로드하고 설치](https://flowport.pages.dev/ko/downloads/)하세요. amd64와 arm64용 deb, rpm, tar.gz 형식을 제공합니다. FlowPort 실행에는 Node.js, npm, Go 컴파일러가 필요하지 않습니다.
2. [빠른 시작](https://flowport.pages.dev/ko/docs/)에 따라 Destination을 설정하고 데이터 흐름을 만든 뒤 게시하세요.
3. 실제로 하위 전송 대상에 데이터가 도착했는지 확인하세요. 수신 시 반환되는 HTTP 204는 로컬에 영구 저장하여 수신했음을 확인할 뿐, 하위 대상에 전달되었음을 뜻하지 않습니다.

## 동작 방식과 제약

- 카탈로그에는 Source 72종과 Destination 48종이 있으며, 처리 후 Tag를 기준으로 라우팅할 수 있습니다. 해당 연동 가이드를 참고하고 실제 서비스, 버전, 권한 조합을 검증하세요.
- 다운로드하는 커뮤니티 패키지는 단일 Worker로 동작합니다. 상용 다중 Worker 배포에는 공급자가 제공하는 상용 빌드와 일치하는 라이선스가 필요합니다.
- 각 Worker는 독립적인 로컬 영구 큐를 사용합니다. 업무 데이터 큐는 노드 간에 복제되지 않습니다.
- 대상 전송이 실패하면 처리된 출력을 로컬 큐에 보관하고 백오프 방식으로 재시도합니다. 새 데이터 수신은 계속됩니다. 용량 제한을 넘으면 일반 미완료 배치를 원래 수신 순서에 따라 FIFO로 제거합니다.
- 정상 대상 분기는 독립적으로 전송을 계속합니다. 파싱 또는 Pipeline 처리에 실패하면 수신한 입력 페이로드를 해제합니다. 이전 형식의 격리된 실패 페이로드는 명시적으로 내보내거나 정리해야 합니다.
- FIFO 제거로 데이터가 손실될 수 있습니다. 원격 대상이 데이터를 수신한 뒤 로컬 확인 전에 프로세스가 비정상 종료되면 중복 전송이 발생할 수 있습니다. 정확히 한 번의 전송이나 전역 전송 순서는 보장하지 않습니다.

문서 기준 버전은 **0.1.25**입니다. 업그레이드하기 전에 [릴리스 매니페스트](https://flowport.pages.dev/ko/downloads/manifest/)와 [업그레이드 안내](https://flowport.pages.dev/ko/changelog/)를 확인하세요.

0.1.25 새 설치는 [웹 초기화](https://flowport.pages.dev/ko/docs/#web-setup)에서 관리자 비밀번호와 관리/HTTP 수집용 도메인 또는 IP, 포트를 설정합니다. 시작 후 로컬 `manager/setup-token`을 읽으세요. 기존 비밀번호와 큐는 유지됩니다. 오프라인 복구와 백업 검증은 [운영 안내](https://flowport.pages.dev/ko/docs/delivery/#offline-maintenance)를 참고하세요.

## 문서와 피드백

[문서 목록](docs/index.md)에서 5개 언어의 가이드로 이동할 수 있습니다. 이 저장소의 [빠른 시작](docs/getting-started.md), [전달과 복구](docs/delivery.md), [운영과 업그레이드](docs/operations.md) 가이드는 중국어 간체로 작성되어 있습니다. 공식 웹사이트에서는 전체 한국어 가이드를 제공합니다.

공개 가능한 제품 질문이나 문서 피드백은 [Issues](https://github.com/SamuelSupe/flowport/issues)에 남겨 주세요. 버전, 운영체제, 민감 정보를 제거한 재현 절차를 포함해 주세요. 인증 정보, 개인 키, 업무 페이로드 또는 민감 정보가 담긴 데이터 디렉터리는 업로드하지 마세요.
