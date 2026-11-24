[![FlowPorts — Collect. Transform. Deliver.](docs/images/flowports-cover.svg)](https://flowport.pages.dev/ko/)

[English](README.md) · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

# 데이터를 연결하고 처리하여 필요한 곳으로 전달하세요.

로그, 메트릭, 트레이스와 이벤트를 **Source → Pipeline → Destinations**로 처리하여 관측 및 분석 시스템에 전달합니다.

**[공식 사이트](https://flowport.pages.dev/ko/) · [시작하기](https://flowport.pages.dev/ko/docs/) · [Linux 다운로드](https://flowport.pages.dev/ko/downloads/) · [릴리스 노트](https://flowport.pages.dev/ko/changelog/#v0-5-12)**

## 0.5.12 업데이트

- 필드가 많은 MongoDB serverStatus 응답을 같은 샘플링 시각의 여러 포인트로 나누어 한 배치로 수집합니다. 중첩 BSON은 드라이버의 기본 디코더를 사용합니다.
- PostgreSQL은 대체 호스트에도 TLS를 사용하고 직접 TLS 모드의 ALPN을 유지합니다. MongoDB는 명시적으로 재설정하지 않은 URI 신뢰 루트와 클라이언트 인증서를 유지합니다.
- Azure Blob과 Service Bus는 각 엔드포인트의 인증서를 검증합니다. Pulsar는 새 연결에서 사설 CA를 다시 읽고 TLS 서버 이름을 지정한 경우 클라이언트 인증서와 키도 다시 읽습니다.
- Manager의 Lookup 다운로드는 동시 최대 2개, 합계 최대 128 MiB입니다. 메모리가 부족하면 사용하지 않는 캐시를 회수합니다. 일시적으로 용량이 부족하면 관리 전송과 Worker 다운로드에 HTTP 503 및 Retry-After를 반환합니다.

## 소스에서 전달까지 하나의 경로

| 수집 | 처리 | 전달 |
| --- | --- | --- |
| 애플리케이션, 인프라, 데이터베이스, 메시지 시스템과 클라우드를 위한 72가지 Source. | Pipeline으로 파싱하고 변환하며 로컬 Lookup으로 보강합니다. Tag 라우팅과 제한된 캐시·창 상태를 지원합니다. | 48가지 Destination. 독립 출력 채널과 실행 화면에서 확인, 적체 및 오류를 살펴봅니다. |

[![로컬 Chrome에서 촬영한 FlowPort 0.5.3입니다. 실제 HTTP 레코드를 수신·처리하고 로컬 수신기가 확인했습니다. 격리된 테스트 환경과 합성 데이터를 사용하며 인증 정보는 가렸습니다.](docs/images/first-delivery-0.5.3.jpg)](docs/images/first-delivery-0.5.3.jpg)

*로컬 Chrome에서 촬영한 FlowPort 0.5.3입니다. 실제 HTTP 레코드를 수신·처리하고 로컬 수신기가 확인했습니다. 격리된 테스트 환경과 합성 데이터를 사용하며 인증 정보는 가렸습니다.*

## 첫 번째 레코드를 전달하세요

1. [Linux 패키지 다운로드](https://flowport.pages.dev/ko/downloads/) 후 SHA-256을 확인하세요. amd64 / arm64의 deb, rpm, tar.gz를 제공합니다. 실행에 Node.js, npm 또는 Go 컴파일러가 필요하지 않습니다.
2. Web 초기 설정을 완료하고 데이터 흐름과 소스를 선택하세요. 언제든 초안을 저장할 수 있으며, 변환이 필요 없으면 빈 Pipeline으로 그대로 전달합니다.
3. 편집기에서 목적지를 추가하고 게시한 뒤 Worker 설치를 기다리세요. 전달 검증을 시작한 후 일치하는 샘플을 보내고 목적지의 실제 데이터를 확인하세요.

## 사용 가이드

- **[데이터 흐름과 라우팅](docs/ko/flows.md)** — 생성, 저장, 게시 및 단일 흐름 설정 가져오기/내보내기.
- **[Lookup 데이터 보강](docs/ko/lookup.md)** — 로컬 매칭, 백그라운드 동기화 및 불변 데이터 버전.
- **[Pipeline과 창](docs/ko/pipeline.md)** — 파싱, 단기 중복 제거, 컨텍스트와 집계.

[전체 문서](docs/index.md) · [전달과 복구](https://flowport.pages.dev/ko/docs/delivery/) · [에디션과 라이선스](https://flowport.pages.dev/ko/editions/)

## 전달의 경계 이해하기

- 수신 확인은 현재 Worker의 영구 수락을 뜻하며 목적지 전달을 보장하지 않습니다. Worker 큐는 독립적이며 노드 간 복제되지 않습니다.
- 최초 전송 이후 대상별 빠른 재시도는 기본 5회입니다. 빠른 재시도가 끝나면 미확인 데이터를 큐에 유지하고 최소 60초 뒤 배치 하나를 시험 전송합니다.
- 새 흐름의 기본값은 Worker당 256 MiB, 내부 배치 500개입니다. 입력과 대상별로 분리합니다. 출력 큐가 가득 차면 원본 입력을 유지하고 입력 큐가 가득 차면 새 수집을 거부합니다.
- 프로토콜 확인은 하위 시스템의 업무 처리 완료를 뜻하지 않습니다. 확인 손실로 중복이 발생할 수 있으며, 엄격한 단 한 번 전송이나 목적지 간 트랜잭션을 보장하지 않습니다.

### 업그레이드 전

0.5.12는 제어 프로토콜 v9와 체크포인트 형식 3을 사용합니다. 수집을 중지하고 각 노드의 데이터, master.key, Lookup 리소스와 환경 설정을 백업한 뒤 관리자와 모든 Worker를 함께 업데이트하세요. 롤백하려면 업데이트 전 백업을 복원하세요.

## 질문과 피드백

제품 질문, 재현 가능한 문제 또는 문서 수정은 [Issues](https://github.com/SamuelSupe/flowport/issues/new/choose)에 버전과 민감 정보를 제거한 예시를 첨부하세요.

---

이 저장소는 공개 문서 허브입니다. 공식 사이트 브랜드는 FlowPorts이며 설치하는 소프트웨어는 FlowPort입니다. 제품·사이트 소스, 소스 이력 또는 바이너리를 호스팅하지 않습니다. 커뮤니티 패키지는 로컬 Worker 1개를 지원하며 상용 다중 Worker 배포에는 발행자의 빌드와 라이선스가 필요합니다.
