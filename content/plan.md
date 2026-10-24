# 압축 공부법

이론을 처음부터 끝까지 다 보지 않고, 자주 나오는 것부터 골라 보는 순서표다. 코딩에서 6문제를 맞힌다고 치면 이론에서 6문제(30점)를 더 맞혀야 합격선 60점에 닿는다. 그래서 목표는 이론 9~11문제 중 **6~7문제**다.

순서는 두 단계다. 먼저 용어를 떠올려 쓰는 단답형(디자인 패턴, 프로토콜, 결합도)을 외워 점수를 챙긴다. 그다음 서브넷·스케줄링·정규형 같은 계산형을 손으로 직접 풀어 본다. 계산형은 읽기만 해서는 시험장에서 안 풀린다.

과목마다 표가 있다. 표의 항목을 누르면 그 개념이 창으로 열리고, ✕나 창 바깥을 누르면(폰은 뒤로 가기도) 닫힌다. 창 아래 「이전·다음」으로 표 순서대로 넘겨 볼 수 있다. 개념마다 끝에 실제 기출 문제가 붙어 있고, 답을 칸에 써서 「채점」을 누르거나 「정답 보기」로 바로 볼 수 있다. 우선순위는 ⭐ 핵심, 🔥 보조, 🤔 후순위이고, 항목 옆 숫자는 2020~2026 기출에 나온 횟수다.

과목별 예상 문제 수와 우선순위는 정처기 감자 「기출문제 기반 압축 공부법 - 이론편」(2026년 3회판)을 따랐다.

| 과목 | 예상 | 맞힐 목표 |
|---|---|---|
| DB | 1~2 | 1 |
| 네트워크/OS | 2~4 | 1~2 |
| SW 개발 | 1~2 | 1 |
| SW 설계 | 1~3 | 2 (범위가 좁아 다 맞히기) |
| 보안 | 1~3 | 1~2 |

## DB

| 주요 항목 | 예상 | 범위 |
|---|---|---|
| 논리 DB 설계 | 0~2 | [⭐ 요개논물](@db/DB설계절차) [⭐ 데이터 모델 3요소](@db/데이터모델) [⭐ 관계 데이터 모델(튜플·카디널리티)](@db/릴레이션용어) [⭐ 관계대수·관계해석](@db/관계대수) |
| 물리 DB 설계 | 0~1 | [🔥 키](@db/키) [🤔 정규화](@db/정규화) [🤔 함수 종속](@db/함수종속) [🤔 무결성](@db/무결성) [🤔 스토리지·RAID](@db/RAID) |
| 트랜잭션 | 0~1 | [🤔 트랜잭션 ACID·로킹·Redo/Undo](@db/트랜잭션) |

## 네트워크/OS

| 주요 항목 | 예상 | 범위 |
|---|---|---|
| 네트워크 7계층 | 0~2 | [⭐ 네트워크 계층 프로토콜](@network/네트워크계층프로토콜) [⭐ 응용 계층 프로토콜](@network/응용계층) [🔥 오류 제어(해밍·CRC)](@network/오류제어) [🔥 전송 방식(패킷·회선 교환)](@network/교환방식) |
| shell script | 0~1 | [🔥 UNIX 기본 명령어](@os/Unix) [🔥 chmod](@os/chmod) |
| IP | 0~1 | [⭐ 라우팅 프로토콜](@network/라우팅프로토콜) [🤔 서브넷 마스크·CIDR·블록 크기](@network/서브넷마스크) [🤔 서브네팅(FLSM)](@calc/FLSM) [🤔 라우터 IP 할당](@network/라우터인터페이스) |
| 메모리, 프로세스 | 0~1 | [⭐ 메모리 관리(페이지 교체)](@os/페이지교체) [⭐ 프로세스 스케줄링](@os/CPU스케줄링) |

## SW 개발

| 주요 항목 | 예상 | 범위 |
|---|---|---|
| 테스트 | 1~2 | [⭐ 테스트 커버리지](@sw-dev/커버리지) [⭐ 블랙박스 테스트](@sw-dev/블랙박스) [🔥 테스트 자동화 도구·오라클](@sw-dev/자동화) [🔥 테스트 레벨·V 모델](@sw-dev/테스트레벨) [🔥 테스트 케이스](@sw-dev/테스트케이스) |
| 인터페이스 | 0~1 | [🔥 인터페이스 보안](@sw-dev/인터페이스보안) [🔥 IPSec](@sw-dev/IPSec) [🔥 인터페이스 통신 기술(AJAX 등)](@sw-dev/통신기술) [🤔 데이터 표현 형식(XML·JSON·YAML)](@sw-dev/데이터표현) |
| 통합 구현 | 0~1 | [🤔 형상 관리](@sw-dev/형상관리) |

## SW 설계

| 주요 항목 | 예상 | 범위 |
|---|---|---|
| 앱 설계 | 1~3 | [⭐ 디자인 패턴 총론](@sw-design/디자인패턴총론) [⭐ 생성 패턴](@sw-design/생성패턴) [⭐ 구조 패턴](@sw-design/구조패턴) [⭐ 행위 패턴](@sw-design/행위패턴) [⭐ 응집도·결합도](@sw-design/응집도) [🔥 객체지향 SOLID](@sw-design/SOLID) [🤔 비기능 요구사항](@sw-design/요구사항분류) [🤔 럼바우](@sw-design/럼바우) |
| 요구사항 | 0~1 | [🤔 UML 다이어그램](@sw-design/UML다이어그램) [🤔 4+1 뷰](@sw-design/4+1뷰) [🤔 클래스 다이어그램 관계](@sw-design/클래스다이어그램관계) |
| UI | 0~1 | [🤔 UI 설계 원칙](@sw-design/UI설계원칙) [🤔 UI 종류](@sw-design/UI종류) |
| 방법론 | 0~1 | [🤔 애자일](@sw-design/애자일) [🤔 리팩토링](@sw-design/리팩토링) [🤔 코딩 컨벤션](@sw-design/코딩컨벤션) [🤔 비용 산정(LOC·FP·COCOMO)](@sw-design/비용산정) |

## 보안

| 주요 항목 | 예상 | 범위 |
|---|---|---|
| 암호 알고리즘 | 0~1 | [🤔 대칭키·블록 암호·비대칭키](@security/암호알고리즘) |
| 공격 유형 | 0~2 | [🔥 서비스 거부 공격(DoS·DDoS·DRDoS)](@security/DoS) [🔥 네트워크 공격](@security/네트워크공격) [🔥 취약점 생명주기(제로데이)](@security/취약점생명주기) [🔥 사회공학·BEC·APT](@security/사회공학) [🔥 시스템 공격](@security/시스템공격) [🔥 악성 코드](@security/악성코드) [🤔 웹 공격(SQL Injection·XSS·CSRF)](@security/웹공격) [🤔 SW 보안 취약점(버퍼 오버플로)](@security/SW개발보안) |
| 보안 기능 | 0~2 | [🔥 인증 기술·3A](@security/인증기술) [🔥 서버 접근 통제](@security/접근통제) [🔥 정보보호(CIA·ISMS)](@security/보안관리) [🔥 보안 인증 제도·법제(CC·GDPR)](@security/보안인증제도) [🔥 보안 솔루션(DLP·DRM·방화벽·IDS·SIEM)](@security/보안솔루션) |
| 신기술 | 0~1 | [🤔 네트워크 신기술](@newtech/네트워크신기술) [🤔 SW 신기술](@newtech/SW신기술) [🤔 클라우드](@newtech/클라우드) [🤔 DB 신기술](@newtech/DB) |

## 최근 안 나온 개념

단답으로 한 번씩 묻는 개념을 한곳에 모았다. 설명은 이 사이트 각 개념 페이지의 정리를 옮겼고, 「자세히」를 누르면 그 개념 전체가 열린다.

[📌 최근 안 나온 개념](#m/recent)

:::m recent 최근 안 나온 개념
같이 묻거나 서로 헷갈리는 개념끼리는 한 줄로 묶었다. 설명은 이 사이트 개념 페이지의 정리를 옮겼고, 「자세히」를 누르면 그 개념 전체가 열린다.

### 오류 제어

| 주제 | 정리 | 구분 포인트 | |
|---|---|---|---|
| FEC · BEC | **FEC**(순방향 오류 수정): 재전송 요구 없이 수신 측이 스스로 수정, 해밍 코드<br>**BEC**(역방향 오류 수정): 오류 감지 후 송신 측에 재전송 요청 | **누가 고치나**로 가른다<br>FEC: 수신 측이 되묻지 않고 혼자 수정<br>BEC: 수신 측은 감지만, **송신 측에 재전송 요청**(ARQ) | [자세히](@network/오류제어) |
| 해밍 · 패리티 · CRC | **해밍 코드**: FEC 방식, 2의 거듭제곱 위치에 패리티 비트, 오류 위치를 찾아 단일 비트 오류 수정<br>**패리티 검사**: 1비트 추가, 짝수/홀수 개수 검증<br>**CRC**(순환 중복 검사): 다항식 코드, 집단 오류 검출 | **고칠 수 있나**로 가른다<br>해밍: 여러 패리티 비트로 검출 + **수정**(FEC)<br>패리티: 단일 비트로 **검출만**, 수정 불가<br>CRC: 다항식, **집단 오류** 검출 | [자세히](@network/오류제어) |

### 네트워크

| 주제 | 정리 | 구분 포인트 | |
|---|---|---|---|
| ARP · RARP | **ARP**: IP → MAC 변환<br>**RARP**: MAC → IP 변환, ARP의 역기능 | **화살표 방향**<br>ARP: IP를 알고 MAC을 찾음<br>RARP: MAC은 알지만 IP를 모를 때(부팅 시 IP 할당) | [자세히](@network/네트워크계층) |
| ICMP · NAT | **ICMP**: IP 오류 보고·네트워크 상태 진단, Echo Request/Reply → ping 명령어<br>**NAT**: 사설 IP를 공인 IP로 변환(응답은 다시 사설 IP로), IP 주소 절약, 내부 네트워크 보안 강화 | ICMP: IP에 없는 **오류 보고** 기능을 메움, ping<br>NAT: **주소를 바꿈**(사설 ↔ 공인) | [자세히](@network/네트워크계층) |
| 가상 회선 · 데이터그램 | **가상 회선**: 논리적인 통신 경로를 미리 설정, 연결형 교환 방식<br>**데이터그램**: 비연결형 교환 방식, 순서에 무관하게 독립적으로 전송, 헤더를 붙여 개별 전달 | 가상 회선: 연결 설정 필요, **고정 경로**, 순서 보장<br>데이터그램: 연결 설정 불필요, **패킷마다 경로 다름**, 순서 보장 안 됨 | [자세히](@network/교환방식) |
| HTTP · Hypertext · HTML | **HTTP**: 전송 프로토콜, 요청/응답 처리(GET·POST·PUT), 80번 포트<br>**하이퍼텍스트**: 링크로 다른 문서와 연결되는 텍스트<br>**HTML**: 웹 문서를 만드는 마크업 언어 | HTTP: 보내는 **규약**(프로토콜)<br>Hypertext: 링크로 이어진 **텍스트**<br>HTML: 문서를 만드는 **언어** | [자세히](@network/응용계층) |
| 프로토콜 3요소 | **구문(Syntax)**: 데이터의 형식·부호화·신호 수준<br>**의미(Semantics)**: 각 비트·필드의 뜻, 제어 정보와 오류 처리<br>**타이밍(Timing)**: 속도 조절과 순서 제어 |  | [자세히](@network/프로토콜의정의) |
| ATM | 53바이트 고정 길이 셀(헤더 5 + 페이로드 48), 비동기 전송 모드, 고속 스위칭 |  | [자세히](@network/데이터링크) |
| RIP | 거리 벡터(벨만-포드 알고리즘), 홉 수로 거리 측정, 최대 15홉(16홉부터 도달 불가) |  | [자세히](@network/라우팅) |
| 애드혹 네트워크 | 인프라 없는 통신, 자율적 네트워크 구성, 중앙 제어 장치 없음 |  | [자세히](@newtech/네트워크신기술) |

### 인터페이스·웹 서비스

| 주제 | 정리 | 구분 포인트 | |
|---|---|---|---|
| IPSec · L2TP | **IPSec**: VPN 기술, 3계층(네트워크), 인증 헤더(AH) + 캡슐화(ESP)로 IP 패킷 암호화<br>**L2TP**: VPN 기술, L2F + PPTP 결합, 2계층(데이터 링크), 터널링만 → 암호화는 IPSec과 결합 | IPSec: **3계층**, 직접 암호화<br>L2TP: **2계층**, 터널링만(암호화는 IPSec과 결합)<br>지문에 "L2F + PPTP"가 보이면 L2TP | [자세히](@sw-dev/인터페이스보안) |
| SOAP · WSDL | **SOAP**: XML 기반 메시지, 원격 프로시저 호출, 엄격한 규칙의 프로토콜<br>**WSDL**: 웹 서비스 명세, XML 형식, 기능·파라미터·데이터 타입 기술 | 둘 다 XML 기반이라 **동사**로 가른다<br>메시지 교환 → SOAP<br>기술(describe) → WSDL<br>(등록·검색 → UDDI) | [자세히](@sw-dev/웹서비스) |
| JSON · XML | **JSON**: 가벼운 데이터 형식, 속성-값 쌍, 빠른 교환, REST 주요 형식<br>**XML**: 다목적 마크업 언어(W3C), 태그 직접 정의, 데이터와 표시 방식 분리 | JSON: **key-value 쌍**, 괄호 `{}` `[]`, 주석 없음<br>XML: **태그** `<>` 마크업, 주석 있음, SOAP 웹 서비스 | [자세히](@sw-dev/데이터표현) |
| AJAX | 비동기 요청, 페이지 새로고침 없이 화면 일부만 갱신 |  | [자세히](@sw-dev/인터페이스통신) |
| EAI 구축 유형 | **Point-to-Point**: 1:1 직접 연결<br>**Hub & Spoke**: 중앙 허브 경유<br>**Message Bus**: 공통 버스(미들웨어)<br>**Hybrid**: 그룹 내부는 허브, 그룹 간은 버스 | 1:1이면 Point-to-Point<br>허브 하나를 거치면 Hub & Spoke(**허브 장애 시 전체 마비**)<br>공통 버스면 Message Bus<br>섞으면 Hybrid | [자세히](@sw-dev/EAI) |

### 보안

| 주제 | 정리 | 구분 포인트 | |
|---|---|---|---|
| MAC · RBAC · DAC | **MAC**(강제적): 시스템이 정한 보안 등급·규칙으로 접근 강제<br>**RBAC**(역할 기반): 사용자에게 할당된 역할에 따라 권한 부여<br>**DAC**(임의적): 데이터 소유자가 직접 권한 관리 (예: chmod) | 한글 단서로 가른다<br>"임의적"·개인 신분 → DAC<br>"강제적"·시스템 규칙 → MAC(MAC 주소 아님)<br>"비임의적"·역할 단위 → RBAC | [자세히](@security/접근통제) |
| 대칭키 · 비대칭키 | **대칭키**: DES(최초 표준, 56비트), AES(NIST, 128비트), ARIA(국내 국가 표준), SEED(KISA, 128비트)<br>**비대칭키**: RSA(소인수분해), ECC(타원곡선, 짧은 키), 디피-헬만(키 교환) | 대칭키: 같은 키, 빠름<br>비대칭키: 공개키·개인키 분리, 느림, 키 교환 안전<br>AES·ARIA는 숫자가 같으니 NIST → AES, 국내 국가 표준 → ARIA | [자세히](@security/암호알고리즘) |
| IDEA · Skipjack | **IDEA**: 스위스 개발, 128비트 키, 64비트 블록, 8라운드<br>**Skipjack**: NSA 개발, 음성 암호화, 80비트 키, 64비트 블록 | 스위스·128비트 키·8라운드 → IDEA<br>NSA·음성·80비트 키 → Skipjack (22-2에 한 문제로) | [자세히](@security/암호알고리즘) |
| MD5 · TKIP | **MD5**: RFC 1321, 128비트 해시, MD4 대체, 현재 충돌 취약<br>**TKIP**: WEP 보완, 패킷별 키 믹싱, MIC 적용, 무선랜 보안 | MD5: **해시** 함수(무결성)<br>TKIP: **무선랜** 보안 프로토콜(WEP 보완) | [자세히](@security/해시) |
| 3A · SSO | **3A**: 인증(Authentication) → 권한 부여(Authorization) → 계정 관리(Accounting)<br>**SSO**: 한 번 인증으로 여러 서비스에 자동 접속 | 인증(Authen, "어쎈") = 문을 열고 들어감<br>권한 부여(Author, "어떠한") = 어느 방·금고까지 되나<br>순서는 인증 → 권한 부여 → 계정 관리 고정<br>SSO의 목적은 인증(권한 위임은 OAuth) | [자세히](@security/인증기술) |
| TrustZone · 타이포스쿼팅 | **TrustZone**: ARM 프로세서가 하드웨어로 일반 영역과 보안 영역 분리<br>**타이포스쿼팅**: 주소 오타를 노려 비슷한 도메인을 미리 등록, URL 하이재킹 | TrustZone은 **하드웨어** 보안 기술<br>타이포스쿼팅은 **사용자 오타**를 노림 → DNS를 조작하면 파밍 | [자세히](@security/보안신기술) |
| 템퍼프루핑 · SIEM | **템퍼프루핑**: 위·변조 감지 시 코드 변형·실행 중단 → 무결성 보호<br>**SIEM**: 보안 로그·이벤트를 한곳에 모아 상관 분석·경고 | 템퍼프루핑: 변조 감지 → **실행 중단**(자기 방어)<br>SIEM: 로그를 **모아서 본다**(탐지), 자동 대응은 SOAR | [자세히](@security/보안솔루션) |
| 웜 · 트로이 목마 · 바이러스 | **웜**: 네트워크 자율 전파, 자가 복제<br>**트로이 목마**: 정상 프로그램 위장, 자가 복제·감염 안 함<br>**바이러스**: 프로그램을 변형해 감염, 자가 복제, 자율 전파 안 함 | **스스로 퍼지나·복제하나**로 가른다<br>웜: 독립 프로그램, 네트워크로 스스로 전파<br>트로이 목마: 위장, 복제·감염 안 함<br>바이러스: 다른 프로그램에 감염, 스스로 전파는 안 함 | [자세히](@security/악성코드) |
| 랜드 · 스머프 어택 | **랜드 어택**: 출발지와 목적지 주소를 같게 만든 패킷, 서비스 거부<br>**스머프 어택**: 출발지 주소 변조 + ICMP Echo 브로드캐스트, 타깃 마비 | 출발지 = 목적지 → 랜드<br>ICMP + **브로드캐스트** → 스머프 (ICMP + "크기"면 Ping of Death) | [자세히](@security/DoS) |
| 루트킷(Rootkit) | root 권한 유지 + 자기 존재 은폐 도구 묶음, ps·ls·netstat 결과 변조 |  | [자세히](@security/시스템공격) |
| APT | 지능형 지속 위협, 특정 표적을 오래 은밀히 침투, 제로데이·악성코드·사회공학 조합 |  | [자세히](@security/사회공학) |
| RTO | Recovery Time Objective, 장애 발생 후 정상 가동까지 허용 시간 | RTO: 정상 가동까지 허용 **시간**<br>RPO: 손실을 허용할 **데이터** 양(시점) | [자세히](@security/보안관리) |

### SW 설계·개발

| 주제 | 정리 | 구분 포인트 | |
|---|---|---|---|
| 싱글톤 · 방문자 · 반복자 | **싱글톤(Singleton)**: 생성 패턴, 인스턴스가 오직 하나임을 보장<br>**방문자(Visitor)**: 행위 패턴, 새 기능(연산)만 따로 추가하며 각 클래스를 돌아다님<br>**반복자(Iterator)**: 행위 패턴, 내부 구조를 노출하지 않고 순서대로 접근 | 객체를 **하나만** → 싱글톤(생성)<br>클래스는 그대로 두고 **기능만 추가** → 방문자(행위)<br>내부를 숨기고 **next로 차례로** → 반복자(행위) | [자세히](@sw-design/디자인패턴총론) |
| 응집도 종류 | 높은 것부터 기능적 → 순차적 → 통신적(교환적) → 절차적 → 시간적 → 논리적 → 우연적 | 응집도는 **높을수록**, 결합도는 **낮을수록** 좋다 | [자세히](@sw-design/모듈응집도) |
| SOLID | SRP(단일 책임) / OCP(개방-폐쇄) / LSP(리스코프 치환) / ISP(인터페이스 분리) / DIP(의존 역전) | "인터페이스를 쪼갠다" → ISP<br>"클래스의 책임 하나" → SRP | [자세히](@sw-design/객체지향SOLID) |
| 헤더(릴리스 노트) | 릴리스 노트 자체를 식별: 문서 이름, 제품 이름, 버전 번호, 릴리스 날짜, 참고 날짜, 노트 버전 |  | [자세히](@sw-design/릴리스노트) |
| NUI | Natural User Interface, 신체 동작·터치·제스처 같은 자연스러운 동작으로 조작 (예: 멀티 터치, Kinect) |  | [자세히](@sw-design/UI종류) |
| 형상 관리 도구 | CVS, SVN, Git |  | [자세히](@sw-dev/형상관리) |
| 살충제 패러독스 | 같은 테스트를 반복하면 새 결함을 찾기 어려움 → 주기적으로 케이스 검토·갱신 |  | [자세히](@sw-dev/테스트7원칙) |

### DB·OS·신기술

| 주제 | 정리 | 구분 포인트 | |
|---|---|---|---|
| DB 명령어 분류 | **DDL**: CREATE·ALTER·DROP·TRUNCATE (스키마 정의)<br>**DML**: SELECT·INSERT·UPDATE·DELETE (데이터 조작)<br>**DCL**: GRANT·REVOKE, 국내 시험은 COMMIT·ROLLBACK도 DCL<br>문장마다 붙는 키워드는 바로 아래 표 |  | [자세히](@sql/SQL분류) |
| 로킹(Locking) | 병행 제어 기법, 트랜잭션이 접근하는 데이터 항목을 잠가 동시 접근 제어(상호 배제로 직렬화) |  | [자세히](@db/트랜잭션) |
| RAID 0 | 스트라이핑, 패리티·중복 없음, 성능 향상, 장애 시 데이터 손실 |  | [자세히](@db/스토리지) |
| IPC | 프로세스 간 통신: 공유 메모리, 소켓, 세마포어, 메시지 큐, 파이프·시그널 |  | [자세히](@os/프로세스상태) |
| LOD | Linked Data + Open Data, 공개 데이터를 표준 형식으로 서로 연결한 데이터망 |  | [자세히](@newtech/DB) |

### DB 명령어: 문장마다 붙는 키워드

INTO·SET·CHECK처럼 빈칸으로 나오는 예약어가 어느 문장에 붙는지 모았다. 굵은 글씨가 그 문장에만 붙는 키워드다.

| 문장 | 모양 | 헷갈리는 점 | |
|---|---|---|---|
| 조회 | `SELECT 컬럼 FROM 테이블 WHERE 조건 GROUP BY 컬럼 HAVING 집계조건 ORDER BY 컬럼 DESC` | **WHERE**는 행 조건, **HAVING**은 GROUP BY로 묶은 뒤 집계 조건(COUNT·SUM 등)<br>오름차순 **ASC**(생략 가능), 내림차순 **DESC** | [자세히](@sql/키워드빈칸:SELECT) |
| 조건 | `LIKE '이%'` · `IN ('컴공','전자')` · `BETWEEN 70 AND 85` · `IS NULL` · `DISTINCT` | **LIKE**: %는 여러 글자, _는 한 글자<br>**BETWEEN**은 양 끝 포함<br>NULL은 `= NULL`이 아니라 **IS NULL** | [자세히](@sql/키워드빈칸:SELECT) |
| 별칭·조인 | `SUM(급여) AS 합계` · `사원 JOIN 부서 ON 사원.부서번호 = 부서.부서번호` · `JOIN 부서 USING (부서번호)` · `NATURAL JOIN 부서` | **AS**는 생략 가능<br>**ON**: 조인 조건을 직접 씀<br>**USING**: 이름이 같은 컬럼 중 조인할 컬럼만 지정<br>**NATURAL JOIN**: 같은 이름 컬럼으로 자동 동등 조인, 그 컬럼은 한 번만 나옴 | [자세히](@sql/조인·서브쿼리·집합) |
| 서브쿼리 비교 | `급여 > ALL (SELECT …)` · `급여 > ANY (SELECT …)` · `WHERE EXISTS (SELECT …)` | **ALL**: 모두보다 → 최댓값과 비교<br>**ANY**(=SOME): 하나보다만 → 최솟값과 비교<br>`IN`은 `= ANY`, `NOT IN`은 `<> ALL`<br>**EXISTS**: 값 비교 없이 행이 하나라도 있는지만 | [자세히](@sql/키워드빈칸:SELECT) |
| 집합 연산 | `SELECT … UNION SELECT …` · `UNION ALL` · `INTERSECT` · `EXCEPT` | **UNION**: 합치고 중복 제거, **UNION ALL**: 중복 남김<br>**INTERSECT**: 양쪽에 다 있는 행<br>**EXCEPT**: 앞에만 있는 행(Oracle은 MINUS)<br>ORDER BY는 맨 뒤에 한 번, 전체에 적용 | [자세히](@sql/조인·서브쿼리·집합) |
| 삽입 | `INSERT INTO 교수(교수번호, 이름) VALUES (1, '김교수')` | 삽입은 **INSERT INTO** 테이블 **VALUES** (값)<br>컬럼 목록을 생략하면 모든 컬럼을 순서대로<br>VALUES 대신 SELECT를 쓰면 조회 결과를 삽입(괄호 없음) | [자세히](@sql/키워드빈칸:INSERT) |
| 수정 | `UPDATE 교수 SET 나이 = 46 WHERE 교수번호 = 1` | 수정은 **UPDATE** 테이블 **SET** 컬럼 = 값<br>WHERE가 없으면 전체 행이 바뀐다 | [자세히](@sql/키워드빈칸:INSERT) |
| 삭제(행) | `DELETE FROM 교수 WHERE 교수번호 = 3` | **DELETE FROM**: WHERE가 없으면 전체 행 삭제, 테이블은 남는다(DML)<br>테이블째 없애는 건 **DROP**(DDL) | [자세히](@sql/키워드빈칸:INSERT) |
| 테이블 생성·제약조건 | `CREATE TABLE 교수( 교수번호 INT PRIMARY KEY, 이름 TEXT NOT NULL, 이메일 TEXT UNIQUE, 나이 INT CHECK (나이 >= 20), 학과코드 TEXT DEFAULT 'D1', CONSTRAINT fk FOREIGN KEY (학과코드) REFERENCES 학과(학과코드) ON DELETE SET NULL )` | **PRIMARY KEY**: NULL·중복 불가<br>**NOT NULL**: 비울 수 없음<br>**UNIQUE**: 중복만 막고 NULL 허용<br>**CHECK**: 값이 조건식을 만족해야 함<br>**DEFAULT**: 값을 안 주면 넣을 기본값<br>**FOREIGN KEY (컬럼) REFERENCES 부모(컬럼)** 순서 | [자세히](@sql/키워드빈칸:DDL) |
| CONSTRAINT (제약조건 이름 붙이기) | `CONSTRAINT fk_학과 FOREIGN KEY (학과코드) REFERENCES 학과(학과코드)` · `CONSTRAINT 성별제약 CHECK (VALUE IN ('남','여'))` | **CONSTRAINT 이름** 뒤에 제약조건(FOREIGN KEY·CHECK 등)을 쓴다<br>REFERENCES 뒤에는 부모 **테이블명**, 괄호 안에 부모 **컬럼**<br>26-1에 외래키 문장이 5칸 빈칸으로 나왔다 | [자세히](@sql/키워드빈칸:DDL) |
| 도메인 | `CREATE DOMAIN 성별 CHAR(1) DEFAULT '남' CONSTRAINT 성별제약 CHECK (VALUE IN ('남','여'))` | **CREATE DOMAIN**: 값의 범위에 이름을 붙임<br>허용 값은 **CHECK (VALUE IN …)**, 26-2에 CHECK 빈칸으로 나왔다 | [자세히](@sql/키워드빈칸:DDL) |
| 부모 삭제 시 | `ON DELETE CASCADE` · `ON DELETE SET NULL` · `ON DELETE SET DEFAULT` · `ON DELETE RESTRICT` | **CASCADE**: 자식 행도 같이 삭제<br>**SET NULL**: 자식 외래키를 NULL로<br>**SET DEFAULT**: 자식 외래키를 기본값으로<br>**RESTRICT**: 자식이 있으면 삭제 거부 | [자세히](@sql/결과예측:DISTINCT) |
| 테이블 변경 | `ALTER TABLE 교수 ADD 전화 TEXT` · `ALTER TABLE 교수 DROP COLUMN 전화` · `ALTER TABLE 교수 MODIFY 나이 INT` | 컬럼 추가 **ADD**, 컬럼 삭제 **DROP COLUMN**<br>자료형 변경 **MODIFY**(Oracle·MySQL), 표준은 `ALTER 나이 SET DATA TYPE INT` | [자세히](@sql/키워드빈칸:DDL) |
| 삭제(객체) | `DROP TABLE 교수 CASCADE` · `DROP VIEW 고령교수` · `DROP INDEX idx_교수_이름` | **CASCADE**: 참조하는 객체까지 함께 삭제, **RESTRICT**: 참조가 있으면 거부<br>**TRUNCATE**는 행을 다 지우지만 분류는 DDL | [자세히](@sql/키워드빈칸:DDL) |
| 뷰·인덱스 | `CREATE VIEW 고령교수 AS SELECT 이름 FROM 교수 WHERE 나이 >= 45` · `CREATE INDEX idx_교수_이름 ON 교수(이름)` | 뷰는 **AS** + SELECT, 인덱스는 **ON** 테이블(컬럼)<br>뷰는 ALTER로 못 바꾼다 → DROP 뒤 다시 CREATE | [자세히](@sql/뷰·인덱스) |
| 권한 부여 | `GRANT SELECT ON 학생 TO USER1` · `GRANT UPDATE ON 학생 TO USER1 WITH GRANT OPTION` · `GRANT SELECT ON 학생 TO PUBLIC` | 뼈대 **GRANT 권한 ON 테이블 TO 사용자** (무엇을 → 어디에 → 누구에게)<br>권한 자리: SELECT·INSERT·UPDATE·DELETE·ALL, 그 밖에 ALTER·INDEX·REFERENCES·CREATE<br>**TO PUBLIC**: 모든 사용자에게<br>**WITH GRANT OPTION**(문장 끝): 받은 사람이 남에게 다시 줄 수 있음<br>여러 권한을 묶어 그룹에 주는 단위는 **ROLE** | [자세히](@sql/키워드빈칸:DCL) |
| 권한 회수 | `REVOKE SELECT ON 학생 FROM USER1` · `REVOKE UPDATE ON 학생 FROM USER1 CASCADE` · `REVOKE GRANT OPTION FOR UPDATE ON 학생 FROM USER1` | 뼈대 **REVOKE 권한 ON 테이블 FROM 사용자** (GRANT는 TO, REVOKE는 FROM: 누구"에게" 주고 누구"로부터" 거둔다)<br>**CASCADE**(문장 끝, REVOKE에만): 그 사용자가 남에게 준 권한까지 연쇄 회수<br>**GRANT OPTION FOR**(REVOKE 바로 뒤): 권한은 두고 재부여 자격만 회수<br>21-3에 `GRANT SELECT ON 학생 TO USER1` 빈칸 | [자세히](@sql/키워드빈칸:DCL) |
| 트랜잭션 | `COMMIT` · `ROLLBACK` · `SAVEPOINT sp1` · `ROLLBACK TO sp1` | **COMMIT**: 확정, **ROLLBACK**: 마지막 COMMIT 이후 전부 취소<br>**SAVEPOINT**로 중간 지점을 찍고 **ROLLBACK TO**로 그 지점까지만 취소<br>국내 시험은 COMMIT·ROLLBACK도 DCL로 분류 | [자세히](@sql/뷰·인덱스) |

:::m-end
