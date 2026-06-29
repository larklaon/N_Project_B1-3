# 노코드 자동화 도구 비교 및 자유 주제 자동화 구현 보고서

## 1. 과제 개요

이번 과제의 목적은 코딩 없이 반복 업무를 자동화할 수 있는 노코드 자동화 도구를 직접 사용해 보고, Trigger, Action, 조건 분기(Filter/Router)의 개념을 실제 워크플로우 안에서 이해하는 것이다.

본 보고서에서는 두 가지 프로젝트를 수행하였다.

- **프로젝트 1:** 동일한 자동화 워크플로우를 Make와 Activepieces 두 도구로 구현하고 비교 분석
- **프로젝트 2:** 자유 주제로 개인 할 일 등록 및 우선순위 알림 자동화 파이프라인 구현

두 프로젝트 모두 실제로 동작하는 자동화 워크플로우를 구현하였으며, Trigger 1개 이상, Action 2개 이상, 조건 분기 1개 이상을 포함하도록 구성하였다.

### 제출 자료 구성

- 프로젝트 1 Make: [구현 및 실행 결과 화면](01_project1_make/screenshots/README.md)
- 프로젝트 1 Activepieces: [구현 및 실행 결과 화면](02_project1_activepieces/screenshots/README.md)
- 프로젝트 2 Make: [구현 및 실행 결과 화면](03_project2_make/screenshots/README.md)

각 링크에는 전체 워크플로우, Trigger, 조건 분기, Action 설정, 실행 기록, Google Sheets 결과와 Discord 알림 결과가 순서대로 정리되어 있다.

---

## 2. 핵심 개념 정리

### 2.1 Trigger

Trigger는 자동화가 시작되는 조건이다. 예를 들어 Google Sheets에 새 행이 추가되거나 수정되면, 이를 감지해 워크플로우가 실행된다.

본 과제에서는 Google Sheets의 입력 시트를 Trigger로 사용하였다.

### 2.2 Action

Action은 Trigger 이후 자동으로 실행되는 작업이다. 예를 들어 데이터를 다른 시트에 기록하거나 Discord 채널로 알림을 보내는 작업이 Action에 해당한다.

본 과제에서는 다음과 같은 Action을 사용하였다.

- Google Sheets에 행 추가
- Discord Webhook을 통한 메시지 전송

### 2.3 조건 분기(Filter / Router)

조건 분기는 입력값에 따라 실행 경로를 나누는 기능이다. 예를 들어 `priority` 값이 `긴급`이면 긴급 처리 경로로, `일반`이면 일반 처리 경로로 보내는 방식이다.

본 과제에서는 Make와 Activepieces 모두 Router를 사용해 조건 분기를 구성하였다.

---

# 프로젝트 1. 자동화 도구 비교 구현

## 3. 프로젝트 1 개요

### 3.1 주제

**문의 접수 자동 분류 및 알림 시스템**

Google Sheets의 `접수함` 시트에 문의가 입력되면, `priority` 값에 따라 `긴급` 또는 `일반`으로 분류하고, 각각의 처리 시트에 기록한 뒤 Discord 채널로 알림을 보내는 자동화 워크플로우를 구현하였다.

### 3.2 사용 도구

- Make
- Activepieces
- Google Sheets
- Discord Webhook

### 3.3 워크플로우 구조

```text
Google Sheets 접수함 입력
↓
조건 분기: priority = 긴급 / 일반
↓
각 처리 시트에 행 추가
↓
Discord 알림 전송
```

### 3.4 Google Sheets 구성

스프레드시트 이름: `Automation_Test_Project1`

#### 입력 시트: 접수함

| created_at | name | title | priority | detail |
|---|---|---|---|---|
| 2026-06-17 16:25 | 한유진 | 결제 오류 | 긴급 | 결제 버튼을 눌러도 진행되지 않습니다 |
| 2026-06-17 16:30 | 오민재 | 계정 문의 | 일반 | 프로필 수정 방법이 궁금합니다 |

#### 처리 시트 1: 긴급처리

| created_at | name | title | priority | detail | processed_by |
|---|---|---|---|---|---|

#### 처리 시트 2: 일반처리

| created_at | name | title | priority | detail | processed_by |
|---|---|---|---|---|---|

---

## 4. Make 구현

### 4.1 구현 과정 요약

Make에서는 Scenario를 생성한 뒤 Google Sheets Trigger, Router, Google Sheets Add a Row, HTTP Request 모듈을 연결하였다.

### 4.2 Make 워크플로우 구성

```text
Google Sheets - Watch New Rows
↓
Router
├─ Filter: priority = 긴급
│  ├─ Google Sheets - Add a Row → 긴급처리
│  └─ HTTP - Make a request → Discord 긴급 알림
└─ Filter: priority = 일반
   ├─ Google Sheets - Add a Row → 일반처리
   └─ HTTP - Make a request → Discord 일반 알림
```

### 4.3 Make Trigger

- 모듈: Google Sheets - Watch New Rows
- 대상 스프레드시트: `Automation_Test_Project1`
- 대상 시트: `접수함`
- 역할: 접수함 시트에 새 행이 추가되면 자동화 시작

### 4.4 Make 조건 분기

Make의 Router와 Filter를 사용하여 다음 조건을 설정하였다.

| 분기 | 조건 |
|---|---|
| 긴급 분기 | `priority` = `긴급` |
| 일반 분기 | `priority` = `일반` |

### 4.5 Make Action

각 분기에는 두 개의 Action을 구성하였다.

| 분기 | Action 1 | Action 2 |
|---|---|---|
| 긴급 | Google Sheets `긴급처리` 시트에 행 추가 | Discord 긴급 알림 전송 |
| 일반 | Google Sheets `일반처리` 시트에 행 추가 | Discord 일반 알림 전송 |

### 4.6 Make 실행 결과

테스트 결과, `priority` 값이 `긴급`인 데이터는 `긴급처리` 시트에 기록되고 Discord에 긴급 알림이 전송되었다. `priority` 값이 `일반`인 데이터는 `일반처리` 시트에 기록되고 Discord에 일반 알림이 전송되었다.

전체 Scenario 구성과 실행 결과는 [프로젝트 1 Make 스크린샷](01_project1_make/screenshots/README.md)에서 확인할 수 있다.

---

## 5. Activepieces 구현

### 5.1 구현 과정 요약

Activepieces에서는 Flow를 생성한 뒤 Google Sheets Trigger, Router, Google Sheets Add Row, HTTP Request 단계를 연결하였다.

초기에는 `New Row Added` Trigger를 사용했으나 실제 자동 실행에서 새 행 감지가 안정적으로 발생하지 않아, 최종적으로 `New or Updated Row` Trigger를 사용하였다. 이후 행 추가 및 수정 감지가 정상적으로 작동하였다.

### 5.2 Activepieces 워크플로우 구성

```text
Google Sheets - New or Updated Row
↓
Router
├─ Branch: priority = 긴급
│  ├─ Google Sheets - Add Row → 긴급처리
│  └─ HTTP Request → Discord 긴급 알림
├─ Branch: priority = 일반
│  ├─ Google Sheets - Add Row → 일반처리
│  └─ HTTP Request → Discord 일반 알림
└─ Otherwise
   └─ 별도 Action 없음
```

### 5.3 Activepieces Trigger

- 단계: Google Sheets - New or Updated Row
- 대상 스프레드시트: `Automation_Test_Project1`
- 대상 시트: `접수함`
- 역할: 접수함 시트에 행이 추가되거나 수정되면 자동화 시작

### 5.4 Activepieces 조건 분기

Activepieces의 Router를 사용하여 다음 조건을 설정하였다.

| 분기 | 조건 |
|---|---|
| 긴급 분기 | `priority` exactly matches `긴급` |
| 일반 분기 | `priority` exactly matches `일반` |
| Otherwise | 위 조건에 해당하지 않는 경우 |

`Otherwise` 분기는 예외 처리를 위한 기본 분기이며, 본 구현에서는 `긴급`과 `일반`만 처리 대상으로 하였기 때문에 별도 Action을 넣지 않았다.

### 5.5 Activepieces Action

각 분기에는 두 개의 Action을 구성하였다.

| 분기 | Action 1 | Action 2 |
|---|---|---|
| 긴급 | Google Sheets `긴급처리` 시트에 행 추가 | Discord 긴급 알림 전송 |
| 일반 | Google Sheets `일반처리` 시트에 행 추가 | Discord 일반 알림 전송 |

### 5.6 Activepieces 실행 결과

테스트 결과, `priority` 값에 따라 긴급 분기와 일반 분기가 정상적으로 실행되었다. 각 분기는 Google Sheets 처리 시트에 데이터를 기록하고 Discord Webhook을 통해 알림 메시지를 전송하였다.

전체 Flow 구성과 실행 결과는 [프로젝트 1 Activepieces 스크린샷](02_project1_activepieces/screenshots/README.md)에서 확인할 수 있다.

---

## 6. Make와 Activepieces 비교 분석

| 비교 항목 | Make | Activepieces |
|---|---|---|
| UI/UX | 시각적 Scenario Builder 형태로 전체 흐름을 한눈에 보기 쉽다. Router와 모듈 연결이 직관적이다. | Flow 기반 UI를 제공하며 Router와 Branch 구조가 명확하다. 다만 일부 설정 위치를 찾는 데 시간이 걸렸다. |
| 설정 난이도 | Google Sheets Trigger, Router, HTTP Request 설정이 비교적 안정적으로 진행되었다. | 단계별 Test me 기능은 편리했지만, Google Sheets 자동 Trigger 감지에서 시행착오가 있었다. |
| 조건 분기 방식 | Router 뒤에 각 경로별 Filter를 붙이는 방식이다. 조건이 시각적으로 잘 드러난다. | Router 안에 Branch를 만들고 조건을 설정한다. 기본적으로 Otherwise 분기가 제공된다. |
| 실행 로그 확인 | 실행 후 각 모듈의 출력값을 확인할 수 있어 데이터 흐름을 검증하기 쉽다. | Runs 화면과 각 Step의 Test me 기능을 통해 실행 결과를 확인할 수 있다. |
| Webhook 설정 | HTTP 모듈에서 JSON string 방식으로 Discord Webhook 요청을 구성하였다. | HTTP Request 단계에서 JSON Body를 입력하여 Discord Webhook 요청을 구성하였다. |
| 무료 플랜 활용성 | 과제 규모에서는 무료 플랜 범위 안에서 구현 가능하였다. | 과제 규모에서는 무료 플랜 범위 안에서 구현 가능하였다. |
| 예외 처리 구조 | 별도 Error Handler를 추가해 오류 대응 경로를 만들 수 있다. | 각 단계에 Error Handler와 Retry 옵션이 제공되며, Router의 Otherwise 분기로 예외값을 분리할 수 있다. |

---

## 7. 각 도구의 장단점

### 7.1 Make 장점

- 시각적 노드 기반 UI가 직관적이다.
- Router와 Filter 구조를 캡처했을 때 자동화 흐름이 잘 드러난다.
- 실행 후 각 모듈의 입력값과 출력값을 확인하기 쉽다.
- Google Sheets와 HTTP Webhook 연동이 안정적으로 작동하였다.

### 7.2 Make 단점

- 처음 사용하는 경우 Trigger 시작 지점, Filter 설정, HTTP Body 설정 방식이 다소 낯설 수 있다.
- JSON Body 설정에서 UI 버전에 따라 입력 방식이 달라 헷갈릴 수 있다.

### 7.3 Activepieces 장점

- Flow 구조가 단순하고 Step별 테스트가 쉽다.
- Router에서 Branch와 Otherwise 분기를 제공하여 예외 흐름을 표현하기 좋다.
- 각 단계에 Error Handler, Retry 옵션이 표시되어 오류 대응 구조를 확장하기 쉽다.

### 7.4 Activepieces 단점

- `New Row Added` Trigger가 실제 자동 실행에서 바로 동작하지 않아 `New or Updated Row`로 변경해야 했다.
- Make에 비해 일부 설정 화면의 위치와 용어가 익숙하지 않아 초기 설정에 시간이 걸렸다.
- 자동 실행과 Step 테스트의 차이를 구분해야 했다.

---

## 8. 도구별 적합한 상황

Make는 시각적으로 워크플로우를 구성하고, 실행 로그를 확인하며, 안정적으로 자동화를 구현해야 하는 상황에 적합하다. 특히 과제처럼 전체 흐름을 캡처하고 설명해야 하는 경우에는 Make의 Scenario Builder가 유리하다.

Activepieces는 Branch, Otherwise, Step별 Test me 기능을 활용하여 흐름을 세부적으로 테스트하고 싶은 경우에 적합하다. 다만 Google Sheets Trigger처럼 실제 자동 실행 감지가 필요한 경우에는 Trigger 종류를 신중히 선택해야 한다.

본 과제에서는 두 도구 모두 같은 목적의 자동화를 구현할 수 있었지만, 설정 안정성과 흐름 파악 측면에서는 Make가 더 편리했고, 단계별 테스트와 예외 분기 표현 측면에서는 Activepieces가 장점이 있었다.

---

# 프로젝트 2. 자유 주제 자동화 설계 및 구현

## 9. 프로젝트 2 개요

### 9.1 자유 주제

**개인 할 일 등록 및 우선순위 알림 자동화**

개인 할 일이나 과제를 Google Sheets에 입력하면, 우선순위에 따라 중요 할 일과 일반 할 일로 분기하고, 각각 로그 시트에 기록한 뒤 Discord로 알림을 보내는 자동화 파이프라인을 구현하였다.

### 9.2 자동화할 반복 업무 정의

평소 과제나 개인 업무를 정리할 때, 할 일을 입력하고 우선순위에 따라 따로 기록하거나 알림을 보내는 작업은 반복적으로 발생한다. 이를 수작업으로 관리하면 중요한 일을 놓치거나 같은 내용을 여러 곳에 다시 입력해야 하는 문제가 생길 수 있다.

따라서 Google Sheets에 할 일을 한 번 입력하면, 자동으로 우선순위별 로그에 기록되고 Discord 알림까지 전송되도록 자동화하였다.

### 9.3 선정 도구

프로젝트 2에서는 **Make**를 사용하였다.

### 9.4 Make 선정 이유

Make를 선정한 이유는 다음과 같다.

- 프로젝트 1에서 사용해 본 결과, Google Sheets와 Discord Webhook 연동이 안정적으로 동작하였다.
- Router와 Filter를 통해 우선순위별 조건 분기를 시각적으로 구성하기 쉽다.
- 실행 로그에서 각 모듈의 입력값과 출력값을 확인하기 쉬워 검증과 캡처에 적합하다.
- 과제 규모에서는 무료 플랜 범위 안에서 구현 가능하였다.

---

## 10. 프로젝트 2 워크플로우 설계

### 10.1 Google Sheets 구성

스프레드시트 이름: `Project2_Todo_Automation`

#### 입력 시트: 할일입력

| created_at | task | category | priority | due_date | memo |
|---|---|---|---|---|---|
| 2026-06-17 17:10 | 자동화 보고서 작성 | 과제 | 높음 | 2026-06-18 | 프로젝트 1 비교표 포함 |
| 2026-06-17 17:15 | 참고자료 정리 | 개인 | 보통 | 2026-06-20 | 제출 전 확인 |

#### 로그 시트 1: 중요할일로그

| created_at | task | category | priority | due_date | memo | processed_by |
|---|---|---|---|---|---|---|

#### 로그 시트 2: 일반할일로그

| created_at | task | category | priority | due_date | memo | processed_by |
|---|---|---|---|---|---|---|

### 10.2 워크플로우 다이어그램

```text
Google Sheets - 할일입력 시트에 새 행 추가
↓
Make Router
├─ Filter: priority = 높음
│  ├─ Google Sheets - Add a Row → 중요할일로그
│  └─ HTTP - Make a request → Discord 중요 할 일 알림
└─ Filter: priority = 보통
   ├─ Google Sheets - Add a Row → 일반할일로그
   └─ HTTP - Make a request → Discord 일반 할 일 알림
```

---

## 11. 프로젝트 2 구현 내용

### 11.1 Trigger

- 도구: Make
- 모듈: Google Sheets - Watch New Rows
- 대상 스프레드시트: `Project2_Todo_Automation`
- 대상 시트: `할일입력`
- 역할: 할 일 입력 시트에 새 행이 추가되면 자동화 실행

### 11.2 조건 분기

Make의 Router와 Filter를 사용해 다음 두 경로로 분기하였다.

| 분기 | 조건 |
|---|---|
| 중요 할 일 분기 | `priority` = `높음` |
| 일반 할 일 분기 | `priority` = `보통` |

### 11.3 Action

각 분기에는 두 개의 Action을 구성하였다.

| 분기 | Action 1 | Action 2 |
|---|---|---|
| 중요 할 일 | Google Sheets `중요할일로그` 시트에 행 추가 | Discord 중요 할 일 알림 전송 |
| 일반 할 일 | Google Sheets `일반할일로그` 시트에 행 추가 | Discord 일반 할 일 알림 전송 |

### 11.4 Discord 알림 예시

중요 할 일 알림 예시:

```text
🔥 [중요 할 일 등록]
작업: 자동화 보고서 작성
분류: 과제
마감일: 2026-06-18
메모: 프로젝트 1 비교표 포함
처리도구: Make
```

일반 할 일 알림 예시:

```text
📌 [일반 할 일 등록]
작업: 참고자료 정리
분류: 개인
마감일: 2026-06-20
메모: 제출 전 확인
처리도구: Make
```

---

## 12. 프로젝트 2 실행 결과

테스트 결과, `priority` 값이 `높음`인 데이터는 `중요할일로그` 시트에 기록되고 Discord에 중요 할 일 알림이 전송되었다.

또한 `priority` 값이 `보통`인 데이터는 `일반할일로그` 시트에 기록되고 Discord에 일반 할 일 알림이 전송되었다.

이를 통해 프로젝트 2 역시 Trigger, 조건 분기, Action 2개 이상을 포함한 실제 자동 실행 구조로 구현되었음을 확인하였다.

전체 Scenario 구성과 실행 결과는 [프로젝트 2 Make 스크린샷](03_project2_make/screenshots/README.md)에서 확인할 수 있다.

---

# 13. 전체 요구사항 충족 여부

## 13.1 공통 요구사항

| 요구사항 | 충족 여부 | 설명 |
|---|---|---|
| 실제로 동작하는 자동화 워크플로우 구현 | 충족 | 프로젝트 1, 2 모두 실제 실행 확인 |
| Trigger 1개 이상 포함 | 충족 | Google Sheets Trigger 사용 |
| Action 2개 이상 포함 | 충족 | Google Sheets 기록 + Discord 알림 |
| 조건 분기 1개 이상 포함 | 충족 | Router/Filter 사용 |
| 각 분기 경로 1회 이상 실행 | 충족 | 긴급/일반, 높음/보통 분기 각각 테스트 |

## 13.2 프로젝트 1 요구사항

| 요구사항 | 충족 여부 | 설명 |
|---|---|---|
| 서로 다른 2개 이상의 자동화 도구 사용 | 충족 | Make, Activepieces 사용 |
| 동일한 워크플로우 구조로 구현 | 충족 | Google Sheets → 조건 분기 → Sheets 기록 → Discord 알림 |
| 각 도구별 구성 화면 캡처 | 충족 | [Make 화면](01_project1_make/screenshots/README.md), [Activepieces 화면](02_project1_activepieces/screenshots/README.md) |
| 실행 결과 화면 캡처 | 충족 | 실행 로그, Google Sheets 결과와 Discord 알림 결과 포함 |
| 비교 분석 보고서 작성 | 충족 | 본 보고서에 비교 분석 포함 |
| 최소 5개 이상 비교 항목 | 충족 | UI/UX, 설정 난이도, 조건 분기, 실행 로그, Webhook, 무료 플랜 등 비교 |
| 장단점 및 적합한 상황 작성 | 충족 | 7~8장에 작성 |

## 13.3 프로젝트 2 요구사항

| 요구사항 | 충족 여부 | 설명 |
|---|---|---|
| 자동화할 반복 업무 1개 정의 | 충족 | 개인 할 일 등록 및 우선순위 알림 |
| 도구 1개 선정 및 이유 작성 | 충족 | Make 선정 및 이유 작성 |
| 자동 실행 구조 구현 | 충족 | Google Sheets 새 행 Trigger 기반 |
| 워크플로우 흐름 설명 포함 | 충족 | 다이어그램 및 단계별 설명 포함 |
| 구현 화면 캡처 | 충족 | [Make Scenario 구성 화면](03_project2_make/screenshots/README.md) 포함 |
| 실행 결과 화면 캡처 | 충족 | 실행 로그, Google Sheets 결과와 Discord 알림 결과 포함 |

---

## 14. 보안 및 제출물 관리

제출물에는 API Key, 토큰, 비밀번호, Webhook URL 등 민감정보가 노출되지 않도록 주의하였다.

특히 다음 항목은 캡처 또는 공유 전 마스킹해야 한다.

- Discord Webhook URL
- Google 계정 이메일 전체
- Spreadsheet ID
- 인증 토큰 또는 API Key처럼 보이는 문자열
- 개인 식별 정보가 포함된 계정명

민감정보는 `***` 또는 일부 가림 처리 방식으로 마스킹한다.

---

## 15. 결론

이번 과제를 통해 Make와 Activepieces를 사용하여 동일한 자동화 워크플로우를 구현하고 비교하였다. 두 도구 모두 Google Sheets 입력값을 기반으로 조건 분기를 수행하고, 결과를 다시 Google Sheets에 기록하며 Discord로 알림을 보내는 자동화 흐름을 구성할 수 있었다.

Make는 시각적 구성과 실행 로그 확인이 편리하여 안정적으로 과제를 수행하기에 적합했다. Activepieces는 Step별 테스트와 Otherwise 분기 구조가 장점이었으나, Google Sheets Trigger 설정에서는 시행착오가 있었다.

프로젝트 2에서는 Make를 사용해 개인 할 일 등록 및 우선순위 알림 자동화를 구현하였다. 이를 통해 단순한 도구 비교를 넘어, 실제 반복 업무를 정의하고 자동화 가능성을 판단한 뒤 적합한 도구를 선택하여 구현하는 과정을 경험할 수 있었다.

결과적으로 본 과제를 통해 Trigger, Action, Router/Filter의 역할을 이해하고, 반복 업무를 자동화 워크플로우로 설계·구현하는 기초 역량을 확보하였다.
