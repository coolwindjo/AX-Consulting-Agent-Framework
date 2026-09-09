---
aliases:
  - CoolFam CoolBot 개인 PC 연계 Runbook
tags: [coolfam, openclaw, google-drive, field-case]
type: field-runbook
status: operational-go
created: 2026-08-20
last_verified: 2026-09-10
revision: operational services preserved; recurring backup deferred by user; legacy attachment cleanup inventory pending
---

# CoolFam 운영 Runbook

CoolBot이 자동화를 실행하고 MacBook은 결과물을 사용한다. 개인 원천자료는 필요한 때만 임시 공유하고, 이메일 KB 출력 폴더는 예약 게시를 위해 지정 범위로 공유한다. **AgwA와 CnwC 전체를 동기화하지 않는다.**

실행용 문장은 [LLM 안내서](./coolfam-runbook-llm-usage-guide.md)에만 둔다. 이 문서는 구조·권한·완료 조건·현재 상태의 정본이다. 이번 개정은 문서 변경이며 폴더 생성, 공유 변경, 이메일 접근, cron 이관을 실행한 것이 아니다.

## 1. 목표와 범위

1. MacBook에서 매일 실행 중인 두 Naver 계정의 이메일 RAG KB build를 CoolBot으로 이관한다.
2. MacBook에서 KB 문서를 직접 열고 검색하며, 다른 CoolBot 결과물도 쉽게 찾는다.
3. 개인 원천자료 접근은 현재 요청에 필요한 범위·시간으로 한정한다. 별도로 지정한 이메일 KB 출력 폴더만 예약 게시에 필요한 동안 Editor 공유한다.

범용 양방향 Context·Skill 동기화, 전체 Vault 이전, 개인 Skill 공동 카탈로그는 현재 구축 범위에서 제외한다. 기존 파일이나 설정을 자동 삭제하지 않는다. 이메일 이관은 별도 실행 rollout으로 관리하되, 관련 없는 PS-014·범용 Bridge Stage 3 완료를 선행조건으로 요구하지 않는다.

## 2. 현재 확인한 상태

`observed`는 직접 점검, `user-reported`는 사용자 확인, `proposed`는 미적용 설계다. 검증일이 문서 전체의 구현 완료를 뜻하지 않는다.

| 항목 | 상태 | 근거 |
|---|---|---|
| CoolBot AgwA | `/Users/coolbot_macmini/.openclaw/workspace`, 로컬 정본 | observed · 2026-09-06 |
| CoolBot 실행환경 | OpenClaw 2026.9.2, Gateway·Node 실행, loopback | observed · local collector |
| CoolBot 디스크 보호 | FileVault 켜짐, Time Machine 목적지 미설정 | observed · local collector |
| MacBook CnwC | 개인 Drive의 기존 Context·Vault·Version 결합 root 유지 | user-reported |
| CnwC 공유 | 개인 계정 소유, CoolBot 계정에 임시 Viewer | user-reported; 현재 링크의 폴더·목록 접근만 observed, 전체 ACL 미검증 |
| MacBook 이메일 build | DIRECT_MAILBOX·IMAPS, 매일 06:00 Europe/Berlin; 최종 snapshot 후 scheduler 재개 예정, wrapper 경로 drift는 cutover 시 교정 필요 | user-supplied E0 및 후속 결과; 이 task에서 원격 상태를 추정하지 않음 |
| CoolBot 이메일 cron | 실제 scheduler·프로세스 점검에서 해당 build 관찰 0; 실제 활성 16건, manifest 14건 중 활성 13건 | user-supplied A; 불일치는 아래 별도 분류 |
| 일반 결과물 공유영역 | 기존 `CoolFamDrive/OpenClaw_Output` 사용 중; CoolBot 소유·단일 writer, 개인 역할 계정 Editor, 표시된 추가·공개 접근자 없음 | observed · 2026-09-10; 양쪽 materialization·동일 Cloud item·MacBook 직접 열람/검색 PASS |
| 이메일 읽기·보관 권한 | 두 Naver 계정 읽기와 AgwA 밖 보호된 로컬 KB 상시 보관 허용 | user-confirmed · 2026-09-06; 구성 적용은 별도 |
| MacBook 이용 방식 | KB 문서를 직접 열고 검색 | user-confirmed · 2026-09-06 |

계정 역할: 개인 소유자 `coolwind@hotmail.co.kr`, CoolBot 소유자 `coolfam830@gmail.com`. 이메일 계정은 문서에서 `NAVER_PERSONAL`, `NAVER_FAMILY`로 식별하며 실제 매핑·인증정보는 보호된 로컬 설정/OS credential store에 둔다.

## 3. 저장 위치와 데이터 흐름

| 위치 | 역할 | 보관 대상 |
|---|---|---|
| CoolBot AgwA — Agent Workspace for Automation | 자동화 제어 | 비밀 없는 실행 정의·운영 규칙·비식별 상태 |
| MacBook CnwC — Context Workspace for Continuity | 개인 작업공간 | 기존 개인 문맥·Vault 유지; CoolBot 결과물로 가는 진입점 |
| MacBook ClwV — Cloud workspace for Version control / ObvO — Obsidian Vault for Organization | CnwC와 같은 기존 root의 논리 역할 | 새 root를 만들지 않음 |
| CoolBot CllC — Cloud library for Community Sharing | 결과물 정본 | 일반 산출물; 이메일 KB는 개인 소유 별도 출력 폴더 |
| 개인 계정 소유 이메일 KB 출력 폴더 | Sensitive 열람 문서 정본 | 개인 소유자, CoolBot Editor; 일반 Artifacts와 별도 위치 |
| CoolBot 보호된 로컬 저장소 | 이메일 실행 데이터 | credential 참조, DB·index·cache·runtime; AgwA·Drive 밖 |

일반 결과물의 확정 논리 경로:

```text
CoolFamDrive/OpenClaw_Output/
  <project-or-deliverable>/   # 일반 완성 결과물
```

기존 구조가 owner·writer·Editor 경계를 충족하므로 별도 `CoolBot-Shared`는 만들지 않는다. CoolBot은 위 정본에 쓰고, MacBook은 같은 공유 폴더의 materialized 진입점으로 읽는다. 개인 소유 이메일 KB 폴더는 이 경로의 대상·하위 구조가 아니며 일반 Artifacts에서 제외한다.

이메일 KB 출력은 개인 계정 소유 Drive의 지정 전용 폴더를 사용한다. CoolBot 계정은 Editor로 참여한다. 폴더·개별 파일 소유권과 두 계정 ACL 및 MacBook 합성 게시·열람은 검증됐으며, 기존 CoolBot 소유 일반 출력 폴더의 ACL을 이 대상 증거로 대신하지 않는다. 실제 ID·링크·로컬 경로는 공용 문서에 기록하지 않는다.

보호된 로컬 저장소는 AgwA·Drive 밖에 사용자 전용으로 구성했다. 검증된 transfer bundle과 snapshot repository, 복원 shadow, FTS·vector index를 별도 generation으로 보존한다. 실제 경로·credential 참조는 공용 문서에 기록하지 않는다.

```text
개인 Drive ──현재 요청 + 임시 Viewer──▶ CoolBot의 제한된 일회 처리
두 Naver 계정 ──별도 상시 읽기 권한──▶ CoolBot 로컬 build·KB
CoolBot ──일반 결과만──▶ CoolFamDrive/OpenClaw_Output ──▶ MacBook materialized 공유 폴더
CoolBot 로컬 이메일 KB ──완성된 열람 문서──▶ 개인 소유 Email-RAG-KB (CoolBot Editor) ──▶ MacBook
```

- 동기화 대상은 공유 결과물의 일반 파일뿐이다. AgwA, CnwC 본체, 실행 중 DB를 서로 복제하지 않는다.
- MacBook은 개인 Drive에 공유영역 Shortcut을 두고 CnwC 시작 노트에서 연결한다. Shortcut이 Obsidian 검색에 자동 포함된다고 가정하지 않는다.
- 두 PC의 로컬 materialization을 검증한다. MacBook의 폴더 열기·문서 검색이 안 되면 실제 로컬 경로를 확인해 해당 문서 폴더를 별도 Vault/검색 대상으로 연다. CnwC 전체 이전이나 이중 사본은 만들지 않는다.
- 결과물 생성 writer는 CoolBot 하나다. MacBook은 생성된 일반 결과물을 읽고, 수정 의견은 별도 노트로 남긴다. Editor 권한이 양쪽 자동 writer 운영을 뜻하지 않는다.

## 4. 공유와 개인정보 규칙

### 상시 결과물 공유

일반 결과물 정본은 기존 `CoolFamDrive/OpenClaw_Output`를 재사용한다. CoolBot 소유·단일 writer와 개인 역할 계정 Editor가 확인됐고, 표시된 추가·공개 접근자는 없다. 별도 `CoolBot-Shared` 생성·rename·파일 이동은 하지 않는다. MacBook에서는 동일 Cloud item의 materialized 공유 폴더를 단일 진입점으로 사용하며, endpoint 확인 전 Shortcut을 새로 만들지 않는다.

일반 결과물과 이메일 KB 문서를 구분한다. 기존 후보의 writer·내용 범주·상속 권한을 점검한 뒤 필요한 폴더만 공유하며, 부모 전체를 편의상 공유하지 않는다. 이메일 영역은 개인 소유자와 CoolBot Editor 두 계정만 접근해야 한다. 폴더 소유권과 CoolBot이 생성한 개별 파일의 실제 소유권은 별도로 확인하며, 필요하면 개인 계정 소유 파일을 먼저 만들거나 소유권 이전이 지원되는지 확인한다. 폴더 소유만으로 모든 하위 파일 소유권을 단정하지 않는다. 부모의 상속 권한으로 다른 사용자가 접근할 수 있다면 하위 폴더 이름만으로 격리됐다고 판단하지 말고, 별도 제한 위치를 확정한다. 일반 결과물 공유 확대가 이메일 권한 확대로 이어져서는 안 된다.

### 이메일 출력의 지정 공유

- 사용자가 제공한 전용 출력 폴더를 확인한 뒤 해당 폴더만 일일 게시 대상으로 등록한다. 실제 ID·경로는 AgwA 밖의 제한된 로컬 설정에 두고 문서에는 역할 별칭만 기록한다.
- 지속 권한은 문서 게시·완전성 확인·운영에 필요한 해당 출력 읽기에 한정한다. CnwC·상위·형제 폴더나 다른 개인자료 탐색을 허용하지 않는다.
- 매 실행 시 공유 유효성을 확인한다. 공유 해제나 대상 불일치 시 게시를 중단하고 재공유를 요청한다. 다른 폴더로 자동 우회하거나 권한을 복원하지 않는다.
- 별도 파일·백업 암호화는 추가하지 않는 사용자 결정을 따른다. 기존 FileVault·Keychain·서비스 기본 보호는 해제하지 않는다. Editor 공유는 CoolBot의 내용 접근을 허용하며, 공유 해제는 이미 받은 로컬 사본을 삭제하지 않는다.

### 임시 개인 원천 Drive 접근

1. CoolBot은 필요한 목적·자료 범위를 설명하고 해당 작업에서 사용할 링크와 Viewer 공유를 요청한다.
2. 현재 요청과 현재 제공된 링크가 함께 있을 때만 CoolBot 계정으로 지정 범위를 one-shot 읽는다. 과거 링크·registry·공유 상태만으로 재탐색하지 않는다.
3. 공유가 끝나면 사용자가 해제하고, CoolBot은 동일 진입점의 접근 차단을 확인한다. 자동 해제는 별도로 구현·승인된 경우만 사용한다.
4. AgwA·memory·task·output·Git·일반 로그에 원문/추출문, 개인 파일명, 공유 URL을 남기지 않는다. 도구가 내용을 transcript에 노출하면 해당 도구로 읽지 않는다. 보호 폴더 보안 질문은 별도로 유지한다.
5. 기억할 것은 비민감 목적 별칭, 소유자 역할, 다음에 다시 공유 요청이 필요하다는 사실뿐이다. 실제 폴더 대응은 개인 계정 안에서 관리한다.

공유 해제는 이미 다운로드한 파일·cache를 지우지 않는다. 제한 처리의 잔존물·세션·로그를 별도 확인하고 승인된 보존/폐기 정책으로 처리한다. 무단 대량 삭제는 하지 않는다.

### 이메일 전용 예외의 한계

두 Naver 계정 읽기와 보호된 로컬 KB의 지속 보관은 사용자가 별도 허용했다. 이메일 출력 폴더의 지정 공유는 위 별도 계약을 따르며, 다른 개인 Google Drive 원천·계정·첨부 전체 수집·외부 모델 전송으로 권한을 확대하지 않는다.

최신 사용자 결정: 앞선 스킵은 오해에 따른 것으로 철회했다. MacBook 직접 열람·검색을 위해 이메일 KB 문서를 개인 계정 소유·CoolBot Editor인 두 계정 전용 Drive 영역에 별도 파일 암호화 없이 저장하도록 허용한다(`AUTHORIZED_FOR_KB_DOCUMENTS`). 저장 허용 여부를 다시 묻거나 별도 원격 검색 서비스를 선행조건으로 요구하지 않는다. 지정 폴더의 합성 게시·ACL·양쪽 열람 검증은 완료됐고 실제 이메일 KB 게시·운영은 아직 미완료다.

Drive 허용 범위는 최소 필드의 완성된 KB 열람 문서다. 원문 전체 아카이브·첨부·credential·실행 DB·vector index·cache·로그는 제외한다. 메일 파생 문서도 민감자료이므로 일반 Artifacts·AgwA·memory·task에 섞지 않는다. 구체적 문서 필드·기간·보존정책은 이관 계획에서 정한다. 상태 보고에는 비식별 건수·세대·성공시각만 사용한다.

## 5. 구축 순서와 완료 조건

System Master와 rollback 담당자는 조승현이다. 각 변경 전에 정확한 대상·변경분·되돌리기 방법을 확인한다. 문서 저장에는 반복 승인을 만들지 않으며, 공유권한·credential·원천 접근·스케줄 전환은 구체적인 변경계획에 승인받는다.

| 단계 | 실행 | 통과 조건 |
|---|---|---|
| 1. 현황 확인 | MacBook 실행 정의·실제 scheduler·KB 형식, CoolBot runtime·공유 목적지·ACL을 읽기 전용 확인 | 원천·출력·의존성·시각/시간대·증분 cursor·중복 방지·복구 범위가 식별됨 |
| 2. 결과물 연결 | 승인된 공유 폴더·Editor 권한·MacBook 진입점 구성; 비민감 문서 1개 시험 | 같은 Cloud item, 양쪽 열기·검색·freshness, 권한 경계 확인 |
| 3. 이메일 수동 검증 | 승인된 구현·credential을 CoolBot 로컬에 구성; 합성 데이터 후 승인 표본으로 shadow build | CoolBot build·검색, 중복/누락·재실행·privacy·복구 PASS; 두 계정 ACL과 MacBook의 게시 문서 열람·검색 PASS |
| 4. 예약 이관 | MacBook 기존 예약을 가역 중지하고 실행 중 작업 종료 확인 후 CoolBot cron 활성화 | production writer 1개, 예약 실행과 manifest 일치, 첫 예약 결과를 MacBook에서 확인 |
| 5. 운영 확인 | 첫 7회 예약 실행의 상태·신선도·실패 복구 관찰 | 완료 기록 및 담당자 인수; 범위 자동 확대 없음 |

이메일 단계는 `COOLFAM-EMAIL-RAG-2026-001`로 추적한다. 현황 확인은 결과물 폴더를 만들기 전에도 가능하다. 이전 범용 rollout의 GO를 새 이메일 범위의 승인·검증으로 재사용하지 않는다.

MacBook scheduler를 중지하기 전에는 CoolBot shadow가 별도 출력·cursor를 사용한다. 실제 cron 변경 시 `config/cron-jobs.json`도 함께 갱신한다. 이번 문서에 실행용 cron 시각을 추측해 넣지 않는다.

## 6. 이메일 구현에서 확인할 것

- E0 보고: DIRECT_MAILBOX이며 MacBook mail DB 의존 없음. msgvault 0.18.0, AFM 0.9.14 `apple-nl-contextual-multi` 512차원 loopback embedding, 아카이브 SQLite/FTS5와 별도 sqlite-vec DB. CoolBot에 동일 runtime·전처리 계약으로 설치·합성 응답·전체 shadow vector 검증을 완료했다.
- 확정된 Option 2 이관 묶음은 MacBook 예약 중지·drain 뒤 같은 시점에 확보한 전체 기간 archive/cursor와 attachment store·pack index다. 기존 vector index는 복사하지 않고 CoolBot에서 재생성한다. credential은 별도 검증된 Device-Local 전달본을 사용하며 로그·cache·Skill 전체는 복사하지 않는다. 기존 자동화 경로 drift는 승인된 수정 후 정확한 runner 호출 및 rollback을 검증한다.
- scope는 두 사용자 지정 계정의 읽기뿐이다. 메일 발송·삭제·이동·읽음 표시 변경을 포함하지 않는다.
- 기존 실행시각/시간대, 대상 mailbox·기간·제외범위, 저장 필드, 보존기간, 외부 embedding/LLM 이용 여부를 확인한다. 새 외부 전송은 별도 승인한다.
- AgwA 밖에서 내용 처리를 수행하고 원문·검색 결과가 agent transcript·memory·trace에 자동 저장되지 않는지 합성 데이터로 검증한다.
- 두 계정 sync 성공 후에만 index한다. shadow의 DB·cursor·watermark·generation은 production과 분리한다. DB handle/잠금이 전체 daily-build mutex를 증명하지 않으므로 sync부터 index·문서 생성·게시 완료까지 재진입·동시 실행 시험을 한다.
- 마지막 성공시각/대상 기간을 표시하고 실패 시 마지막 정상 generation을 유지한다. 두 계정 중 하나 실패, 중단 후 재시도, 오류 메시지의 제목·본문 누출을 합성 데이터로 시험한다.
- 완성된 문서 세대를 검증 후 게시한다. 비식별 manifest의 generation·해시·완전성을 양쪽에서 확인하고 부분 동기화 중인 세대를 정상본으로 노출하지 않는다. DB를 직접 sync하지 않는다.
- MacBook에서 이메일 문서의 실제 로컬 접근·합성 문구 검색·freshness를 시험한다. Shortcut 생성이나 일반 결과물 열람 PASS를 이메일 문서 검색 PASS로 대신하지 않는다.

### 선택적 첨부 보존 계약

두 source 모두 사용자가 직접 만든 `중요` 메일함의 **계정별 exact IMAP mailbox identifier** 소속만 중요 판정에 사용한다. 표시 이름, `\Flagged`, 별표, 서버의 special-use 추정은 사용하지 않으며 하위 메일함도 자동 포함하지 않는다. 실제 identifier는 각 계정에서 읽기 전용으로 확인해 보호된 로컬 설정에 저장하고 공용 문서에는 source alias와 검증 상태만 기록한다.

- 동일 메시지가 여러 mailbox에 나타나면 모든 확인된 membership을 합친 뒤 중요 메일함 소속 여부를 먼저 평가한다. 어느 한 membership이라도 해당 source의 exact 중요 mailbox identifier와 일치하면 전체 MIME과 첨부를 보관한다.
- 중요하지 않은 메일은 본문과 필요한 비첨부 메타데이터를 보관한다. attachment object·payload·파일명은 저장하지 않고, `message_raw`에는 첨부 payload와 파일명을 제거해 재직렬화한 수정 MIME만 저장한다.
- 수정 MIME은 `sanitized_nonimportant`와 정책 버전을 기록하고 원본 MIME·원본 무결성 보존본으로 표시하지 않는다. 원문 보기·내보내기 UI도 첨부 제외 사본임을 알려야 한다.
- 나중에 exact 중요 mailbox로 이동한 기존 메시지는 멱등 backfill 대상으로 만든다. 서버 원문 재조회에 성공한 경우에만 전체 MIME·첨부를 보충하며, 원문이 없으면 `UNRECOVERABLE_FROM_SOURCE`로 기록한다.
- 중요 mailbox에서 나가도 이미 보관한 MIME·첨부는 자동 삭제하지 않는다. 동일 content-addressed attachment object의 다른 메시지 참조가 하나라도 있으면 physical object와 pack을 보존한다.
- Drive KB에는 첨부 payload·파일명·attachment object를 게시하지 않는다.

msgvault 0.18.0 기본 흐름은 IMAP 전체 MIME을 읽고 `message_raw`와 attachment store에 보관하므로 위 계약을 기본 설정만으로 충족하지 못한다. 최소 변경안은 전체 MIME을 메모리에서 한 번 파싱한 뒤 중요 메일은 기존 저장 경로로, 일반 메일은 본문·메타데이터와 수정 MIME 경로로 분기하는 것이다. `BODYSTRUCTURE` 기반 본문 part 선택 다운로드는 영구 저장량보다 네트워크·메모리 절감이 추가로 필요할 때 별도 단계로 검토한다.

### 합성 데이터 개발·검증 계획

운영 이메일·credential·scheduler를 사용하지 않는 격리 runtime에서 다음 변경과 시험을 먼저 수행한다.

1. **설정·식별자:** source별 exact 중요 mailbox identifier 필드와 검증 상태를 추가한다. 빈 값, 표시 이름만 일치, 부모/하위 mailbox, source 간 identifier 차이를 합성 fixture로 시험한다.
2. **membership 우선 판정:** 한 메시지가 일반·중요 mailbox에 동시에 나타나는 fixture에서 중요 판정이 저장 분기보다 먼저 완료되는지 확인한다. mailbox 열거가 일부 실패하면 일반으로 처리하지 말고 generation을 실패시킨다.
3. **저장 분기:** 일반 MIME의 text/plain·text/html·필수 헤더는 유지하고 attachment payload·filename·attachment row/object가 0인지 DB와 pack inventory로 검사한다. 중요 MIME은 원본 hash와 모든 첨부 hash가 일치해야 한다.
4. **수정본 표지:** 일반 메일의 `message_raw`가 수정본·정책 버전으로 식별되고 원본 hash/무결성 PASS를 주장하지 않는지 확인한다. 원문 보기·export에는 첨부 제외 안내가 나타나야 한다.
5. **재색인:** 수정 MIME 재파싱, FTS5 재생성, sqlite-vec 재생성 후 본문 합성 질의 결과가 기준과 일치하는지 확인한다. 첨부 문구는 일반 메일 검색 결과에 없어야 한다.
6. **승격 backfill:** 일반→중요 이동, 재시도, 중복 이벤트를 시험한다. 원문 존재 시 한 번만 첨부를 보충하고, 원문 부재 fixture는 `UNRECOVERABLE_FROM_SOURCE`이며 기존 본문을 유지해야 한다.
7. **강등·공유 object:** 중요→일반 이동으로 기존 자료가 삭제되지 않는지, 여러 메시지가 같은 object를 참조할 때 한 참조 변경이 physical object·pack을 제거하지 않는지 확인한다.
8. **중단·복구·누출:** 파싱·DB commit·pack write 각 지점의 강제 중단, 전체 mutex, 재실행, backup/restore를 시험한다. 합성 filename·본문·첨부 canary가 로그·transcript에 나타나지 않는지 검사한다.

기존 MacBook archive·cursor·vector index·attachment store는 이 합성 개발에서 입력이나 정리 대상으로 사용하지 않고 그대로 보존한다. 기존 자료의 정리는 별도 read-only inventory에서 중요 참조, 공유 object 참조, archive에만 남은 유일 사본, pack 단위 실제 회수 가능 용량을 확인한 뒤 별도 승인으로 다룬다.

### Option 2 실행 계약 — 전체 archive 보존·index 재생성

- **범위:** 기존 archive의 전체 기간, DB 내부 cursor/checkpoint, attachment store와 pack index를 동일 generation으로 보존·이관한다. 90일·12개월 제한, 기존 MIME·첨부 정리, server 재수집으로의 대체를 적용하지 않는다.
- **신규 정책 분리:** 선택적 첨부 보존은 cutover 이후 새로 수집하거나 중요 mailbox로 승격되는 메일에만 적용한다. 이관된 기존 첨부와 전체 MIME은 자동 변환·삭제하지 않는다.
- **원본:** MacBook의 실제 scheduler가 사용하는 production archive DB와 상대 attachment root. exact local path는 실행 직전 E0가 scheduler·runner·DB open 상태로 확인하고 공용 결과에는 노출하지 않는다.
- **MacBook 복구 사본:** `$HOME/Library/Application Support/CoolFam/Email-RAG-Migration/rollback/<generation>/`의 owner-only 0700 디렉터리. SQLite online backup 또는 msgvault snapshot 뒤 WAL 일관성, archive·cursor, attachment packs/index, 크기·SHA-256 manifest를 함께 검증한다. 원본은 그대로 유지한다.
- **직접 전달:** 복구 generation 하나를 owner-only archive bundle로 만든 뒤 사용자가 Finder AirDrop으로 CoolBot에 1회 전달한다. Drive KB 출력 폴더, Cloud, 채팅, 명령행 인자를 사용하지 않는다. 별도 파일 암호화는 추가하지 않고 양쪽 FileVault와 AirDrop 전송 보호를 유지한다.
- **CoolBot 대상:** `$HOME/.local/share/coolbot-email-rag/migration-inbox/<generation>/`에서 수신 hash를 검증한 뒤 `$HOME/.local/share/coolbot-email-rag/production-shadow/<generation>/`로 복원한다. 현재 두 root와 rollback manifest root는 0700·EMPTY로 준비됐다.
- **index:** archive/cursor/attachment 무결성 PASS 후 FTS5를 archive 본문에서 재생성하고, 별도 sqlite-vec DB를 호환 embedding runtime으로 새 generation에 구축한다. MacBook vector DB는 복사·수정하지 않는다.
- **기준점 검증:** `PREP-20260907T144624+0200` 기준 전체 12,628건, source별 5,572/7,056건, 중복 source key 0, attachment reference 3,065·unique object 2,039, cursor 2개·folder high-water 33개를 CoolBot shadow에 복원했다. repository·page·blob·SQLite·attachment 무결성 PASS, FTS 12,628/12,628·누락 0, vector 12,614건·blank 14건·missing/failed 0, FTS/vector/hybrid 검색 경로 PASS다. server-only/archive-only는 별도 source 비교 전까지 미확인으로 유지한다.
- **실측 소요:** 복원 6.4초, FTS 전체 재생성 3초, vector 500건 benchmark 29초(약 17.2건/초), vector 전체 11분 5초. 최종 증분은 기준점 이후 변경량에 따라 다르며 전체 FTS/vector 재생성을 기본값으로 요구하지 않는다.
- **rollback:** 실패 시 CoolBot writer를 중지하고 수신/복원 generation을 사용 중지한다. MacBook 원본과 scheduler 정의는 변경하지 않은 상태로 보존하며 같은 정의를 ACTIVE로 복원한다. 목표 30분은 실제 복원 rehearsal 전까지 미검증이다.

## 7. 복구와 중단

목표 복귀시간은 기존 합의인 30분을 유지하되, 새 이관 범위의 복구시험 전에는 달성했다고 기록하지 않는다. 외장하드는 사용하지 않는다.

- 변경 대상이 확정된 뒤 runner·설정·scheduler 상태·cursor·필요 KB의 복구 manifest를 만든다. 변경 목록이 비어 있으면 rollback backup 범위를 추측하지 않는다.
- 같은 내부 디스크의 로컬 snapshot은 설정 복구용이며 endpoint 장애용 독립 backup이 아니다. Git·provider history·Drive sync도 검증된 backup을 대체하지 않는다.
- 별도 파일·backup 암호화는 적용하지 않는다. 독립 backup의 실제 저장 위치·보존 세대는 아직 미확정이다. 이메일 KB 문서 공유가 DB·cursor·runner의 복구 사본을 대신하지 않는다. 운영 전 다른 장치/저장소의 복구를 검증하거나 독립 backup 부재 시 구체적 손실 범위를 사용자가 수용한다. credential은 backup 묶음에서 제외하고 OS credential store에 유지한다.
- 전환용 최종 backup은 MacBook 예약 중지와 진행 작업 종료 후 확보한다. 기존 암호화 자료·디스크 보호를 해제하거나 삭제하지 않는다.
- 실패 시 CoolBot cron을 먼저 중지하고 진행 중 writer가 없음을 확인한다. 마지막 정상 KB·cursor를 확인한 뒤 MacBook 기존 예약을 복원한다. 두 scheduler를 동시에 활성화하지 않는다.
- 잘못된 게시 세대는 사용 중지하고 이전 정상본을 복원한다. 개인정보 노출 시 접근 차단을 우선하되 해제가 기존 사본 삭제를 뜻하지 않음을 알린다.
- 원천 메일은 변경하지 않는다. credential 폐기, KB 영구 삭제, Drive 권한 변경은 확인된 범위와 권한으로만 처리한다.

<a id="11-적용-기록"></a>

## 8. 현재 적용 기록

원시 창 출력을 별도 CoolFam 파일로 모으지 않는다. 이 절에 비식별 결과만 한 번 기록한다. 원문·개인 경로·링크를 증거 명목으로 저장하지 않는다.

| 날짜 | 범위 | 결과·근거 |
|---|---|---|
| 2026-08-20 | 기존 COOLFAM-ROLLOUT-2026-001 Stage 1 | HOLD. backup·privacy·Skill·Registry 미완료로 보고 |
| 2026-08-21 | 기존 Stage 1 재판정 | 사용자 통합 GO 보고. PS-014 텍스트 2개 selected-scope PASS; 나머지 privacy 후보 10개·non-text 9개까지 해소됐다는 의미는 아님 |
| 2026-08-21 | 기존 Bridge·Email 후속 설계 | 문서상 Stage 2/3 미시작, Email 별도 보류. 이후 실제 적용 증거는 이번 작업에서 확보하지 않음 |
| 2026-09-06 | 이메일 출력 소유자·암호화 변경 | 사용자 결정: 개인 Drive 소유자, CoolBot Editor; 별도 파일/backup 암호화 미적용. 새 대상은 미확정이며 기존 일반 결과물 ACL 증거와 구분. 문서만 수정 |
| 2026-09-06 | A·E0 통합 및 게시 범위 변경 | 사용자 제공 두 endpoint baseline 수집 완료. READY_FOR_PLAN, 구현 GO 아님. 이메일 Drive 게시 스킵은 이후 사용자 정정으로 철회됨. 최신 결정은 KB 문서 게시 허용; 문서만 수정 |
| 2026-09-06 | COOLFAM-EMAIL-RAG-2026-001 합성 게시 | `SYNTHETIC_PUBLISH_PASS`. 지정 이메일 출력 폴더는 개인 계정 소유·CoolBot Editor이며 두 계정 외 ACL 0건. CoolBot이 만든 합성 문서·manifest·완료 marker는 각각 CoolBot 소유이고 개인 계정 Editor. MacBook에서 3/3 materialization, hash·manifest·게시 순서·실제 열람·합성 검색·freshness PASS. 실제 이메일·credential·scheduler 변경 0건 |
| 2026-09-06 | 선택적 첨부 보존 설계 | 사용자 결정: source별 exact 사용자 생성 중요 mailbox membership만 판정에 사용. 일반 메일은 본문·필수 메타데이터와 첨부 제거 수정 MIME만 보관하고 filename·payload 미보관. 승격 backfill·원문 부재 상태·강등 시 보존·공유 object 보호를 포함한 합성 시험계획 확정. 문서만 수정 |
| 2026-09-07 | 선택적 첨부 보존 격리 개발 | msgvault v0.18.0 격리 브랜치에서 exact mailbox membership 저장 분기, 첨부·filename 제거 수정 MIME, 다중 membership 우선, 중요 승격 멱등 보충, sync-full·daemon 설정 전달을 구현. 합성 sync·중복 참조·잠금·pack rollback 및 관련 회귀시험 PASS. 실제 계정·운영 KB·scheduler 변경 0건; 원문 부재 상태의 영속 기록과 실제 IMAP 식별자 확인은 이관 전 미완료 |
| 2026-09-07 | 첨부 보충 상태·전체 회귀 | 격리 브랜치에 내용 없는 backfill 상태 테이블을 추가. 서버 원문 부재 사유·시도 횟수가 DB 재개방 후 유지되고, 재실패 누적·합성 원문 재확보 성공 후 `complete` 전환 및 본문 보존 PASS. Python 3.9 선택으로 실패했던 문서 시험은 Python 3.14 격리 PATH에서 PASS했고 동일 환경의 전체 Go 회귀 PASS. 실제 Naver·운영 환경 변경 0건 |
| 2026-09-07 | CoolBot credential 전달 준비 | 격리 빌드의 IMAP credential 계약은 canonical source identifier hash 파일명과 password-only JSON, 디렉터리 0700·파일 0600임을 확인. Agent Workspace·Drive 밖에 소유자 전용 수신·shadow token 위치와 격리 ARM64 빌드를 준비. 기존 SSH 수신/Tailscale/저장된 직접 대상은 관찰되지 않아 원격 서비스를 켜지 않고 사용자 승인 AirDrop 1회 전달을 제안. 실제 credential·계정·메일 접근 0건; NAVER-A/B alias 대응은 MacBook 추가 확인 대기 |
| 2026-09-07 | 실제 Naver 제한 시험 | source별 exact 사용자 mailbox 확인과 지정 메일 1건씩의 제한 수집 수행. NAVER-A는 일반 저장·중요 이동·full MIME 전환·complete·중복 방지 PASS이나 원문 attachment part 0으로 첨부 보충 미검증; 사용자가 추가 시험을 스킵했으며 이관 선행조건 아님. NAVER-B는 첨부 보충·complete·중복 방지 PASS. 전체 sync·메일 변경·scheduler 변경 0건 |
| 2026-09-07 | Option 2 실행 준비 | 전체 archive/cursor/attachment store 보존, CoolBot index 재생성 방식 확정. CoolBot owner-only 수신·복원·rollback manifest root 0700/EMPTY 준비. 전달은 quiesced generation의 사용자 AirDrop, 중지 시각은 실행일 07:00 Europe/Berlin, 예상 총 2.5–6시간. 실제 예약 중지·자료 복사 0건 |
| 2026-09-07 | Option 2 일회 실행 예약 | 사용자 보고: 07:00 Europe/Berlin 일회 실행 예약 완료. 현재 MacBook 운영 scheduler는 ACTIVE·미변경이며, pause·drain·source resolution·공간·snapshot·integrity·manifest·bundle은 실행 시 확인 대기. 실패 시 기존 정의 ACTIVE 복원. 별도 중복 automation 생성 없음 |
| 2026-09-07 | Option 2 07:00 실행 | `HOLD_WITH_CLEAN_ROLLBACK`. pause 뒤 drain·source resolution·51.6 GiB 여유공간 PASS. backup repository 초기화 누락으로 snapshot 생성 전에 중단했고 archive/attachment integrity 미실행, manifest·bundle 미생성. 실패 generation 사용 금지. scheduler는 기존 정의 그대로 ACTIVE 복원, 임시 daemon 종료, 원본 변경 없음. 재시도는 repository 선행 초기화·검증 후 새 generation으로 수행 |
| 2026-09-07 | Option 2 snapshot 재시도 | `READY_FOR_COOLBOT_TRANSFER`. 새 generation `PREP-20260907T144624+0200`, 기존 검증 repository 재사용. pause·drain·전체 archive/cursor/attachment snapshot·SQLite/WAL/cursor·attachment hash 검증 PASS. 총 12,628건(+11)은 source별 +8/+3으로 마지막 성공 sync와 일치. 원본 57 pack을 보존했고 검증 복원본은 무손실 23 pack 재배치. credential·로그·cache·analytics·vector DB·workspace 제외. manifest와 약 1.88 GiB owner-only transfer bundle 재열람·SHA-256 PASS. 원본 무변경, MacBook scheduler PAUSED, rollback ACTIVE 정의 READY |
| 2026-09-07 | CoolBot 전체 shadow 복원·재색인 | `FULL_SHADOW_RESTORE_FTS_VECTOR_PASS`. transfer 외곽 SHA-256과 내부 manifest 60/60 hash 일치. 복원 archive 12,628건, source 5,572/7,056, cursor 2·high-water 33, attachment reference 3,065·object 2,039·missing/orphan 0. FTS 12,628/12,628·missing/orphan 0. AFM 0.9.14 loopback 합성 검증 PASS; sqlite-vec active generation은 embedded 12,614·blank 14·missing/failed 0, FTS/vector/hybrid 검색 경로 PASS. 전체 vector 11분 5초. bundle·repository·shadow·FTS/vector 보존, 실제 메일 sync·Drive 게시·CoolBot scheduler 변경 0건. MacBook scheduler 재개 후 생긴 변경분은 cutover 전 새 quiesced snapshot으로 반영 필요 |
| 2026-09-07 | F1 최종 snapshot CoolBot 적용 | `FINAL_SUCCESSOR_SHADOW_PASS`. generation `F1-20260907T204553+0200`의 FULL_FALLBACK bundle 외곽 SHA-256·TAR 안전성·내부 repository 63/63 hash·snapshot 2개 full verify PASS. 최신 snapshot을 기준 shadow를 덮어쓰지 않고 successor에 복원. archive 12,628건, source 5,572/7,056, cursor 2·high-water 33, attachment reference 3,065·object 2,039, missing/broken/orphan·duplicate 0. 기준점 대비 cursor·high-water 변경 0. FTS 12,628/12,628·missing/orphan 0으로 재생성 생략. 검증 vector DB를 successor에 SQLite backup하고 backstop scan 0건; active embedded 12,614·blank 14·missing/failed 0, FTS/vector/hybrid 검색 완전성 PASS. 검증용 daemon·AFM 정상 종료. 메일 sync·Drive 게시·CoolBot scheduler 변경 0건; MacBook scheduler는 최종 전환까지 PAUSED 유지 필요 |
| 2026-09-09 | 2026-09-08 06:00 첫 운영 실행 검증 | `HOLD_WITH_COOLBOT_PAUSED`. 두 source sync 완료 후 index가 실행됐고 신규 30·변경 0·중복 0·실패 0, 전체 12,658건과 FTS 12,658건이 일치했다. vector 증분 30건 성공·실패 0, 전체 mutex·동일 generation 중복 차단·문서 2·manifest 1·complete marker 1의 로컬 게시 hash 검증 PASS. 신규 30건에 attachment event가 없어 중요 mailbox 첨부 분기의 이번 실행 실증은 `NOT_EXERCISED`다. 보호된 실행 로그에서 주소 형식 문자열이 검출되어 log safety FAIL이고 MacBook 최신 generation materialization·열람·검색·freshness는 `NOT_VERIFIED`. 새 generation은 사용 중지하고 마지막 정상 generation을 유지했다. CoolBot 실제 scheduler와 manifest를 함께 PAUSED로 변경했으며 MacBook 기존 scheduler ACTIVE 복원 필요 |
| 2026-09-09 | 첫 운영 로그 안전성 수정 | `FIX_VERIFIED_RETEST_PENDING`. 검출값은 오탐이나 서비스 주소가 아니라 두 source의 실제 계정 식별자였고 IMAP 안내·full-sync 시작 출력에서 총 4회 발생했다. 성공·오류·재시도 경로를 안정적인 불투명 source 참조와 최종 주소 비식별화로 수정했다. 합성 선택적 첨부 경로에서 본문·주소·첨부 payload·첨부명 canary 비노출 PASS, 관련 sync 회귀 PASS. 최초 수정 binary가 운영 build tag 없이 만들어진 것을 PAUSED 상태에서 발견해 사용하지 않고, 기존 PASS 환경과 같은 CGO+FTS5+sqlite-vec 조건으로 재빌드했다. 실제 binary-source 일치·FTS smoke·FTS/vector 관련 전체 패키지 PASS. 시스템 Python 3.9 선택으로 실패했던 문서 스크린샷 시험도 기존 PASS와 같은 Python 3.14 격리 PATH에서 PASS하여 검증 환경 전체 회귀 PASS. 기존 binary·로그·신규 30건·마지막 정상 generation 보존, CoolBot scheduler PAUSED, 운영 재시험 미실행 |
| 2026-09-09 | 로그 수정 후 운영 재시험 | `PENDING_MACBOOK_RETEST_CHECK`. MacBook 기존 scheduler PAUSED·production daemon 정상 drain 확인 뒤 CoolBot 자체 cursor로 두 source 증분 sync 완료. 신규 45·변경 0·중복 0·실패 0, 전체 archive·FTS 12,703건 일치, vector 증분 45/45 성공·실패 0. 전체 mutex·중복 generation 차단·문서 2·manifest 1·complete marker 1·CoolBot Drive materialization hash PASS. 수정 로그에서 주소·메일 헤더·첨부명 패턴 0. 신규 첨부 event가 없어 선택적 첨부 분기는 `NOT_EXERCISED`. 자동 기동 daemon은 공식 종료했고 runner도 후속 실행에서 자동 종료하도록 보강. CoolBot scheduler PAUSED 유지, MacBook source-key 기준·materialization·열람·검색·freshness 검증 대기 |
| 2026-09-09 | 운영 재시험 최종 전환 | `OPERATIONAL_GO`. MacBook에서 문서 2·manifest 1·complete marker 1의 materialization·SHA-256·실제 열람·canary 검색·freshness PASS. source별 MacBook/게시 key 집합은 5,603/5,603 및 7,100/7,100으로 공통 전건, 양쪽 전용 0, 내부 중복 0. 차이 없음. MacBook scheduler·daemon·writer·lock은 PAUSED/0 유지. 검증 후 기존 CoolBot scheduler 한 건만 ACTIVE로 전환하고 실제 scheduler와 manifest를 함께 일치시킴. 실행시각 06:00 Europe/Berlin, CoolBot 단일 writer 확정 |
| 2026-09-09 | 일반 Artifacts 정본 확인 | `GENERAL_ARTIFACTS_ROOT_CONFIRMED`. 기존 `CoolFamDrive/OpenClaw_Output`가 CoolBot 소유·writer이고 개인 역할 계정 Editor임을 확인. 기존 결과물과 로컬 materialization·운영 참조가 있어 재사용하며 새 `CoolBot-Shared`는 만들지 않음. 개인 소유 이메일 KB는 별도 경계로 제외. MacBook의 동일 Cloud item·직접 열람/검색 최종 확인만 대기. 파일·ACL·Shortcut 변경 0건 |
| 2026-09-10 | 일반 Artifacts MacBook 연결 | `GENERAL_ARTIFACTS_ACCESS_PASS`. MacBook에서 기존 정본의 materialization·provider metadata 동일 Cloud item·비민감 결과물 직접 열람과 정확 문자열 검색·Editor 접근 PASS. 단일 진입점은 materialized `OpenClaw_Output` 폴더로 확정. 변경 0건; 개인 소유 이메일 KB는 대상에서 제외 |
| 2026-09-10 | 정기 backup·보존정책 결정 | `DEFERRED_BY_USER`. CoolBot 운영 데이터와 검증된 이관·복구 사본은 유지하되 정기 snapshot·자동 전송·세대 회전 구축은 이번 범위에서 제외하며 후속 작업의 선행조건으로 요구하지 않음. 기존 bundle·repository·shadow·복구 사본 삭제 승인 없음. 외장하드·Drive 원본 backup·추가 파일 암호화는 사용하지 않고 CoolBot ACTIVE·MacBook scheduler PAUSED 유지 |
| 2026-09-10 | 기존 비중요 첨부 정리 inventory | `READ_ONLY_ESTIMATE`. 두 source의 exact 사용자 생성 중요 mailbox가 고유하게 매칭됐고 현재 DB membership 기준 중요 1,799·비중요 10,904·분류 불명 0. attachment object는 중요 전용 767, 비중요 전용 1,162, 양쪽 공유 110. 비중요 전용 pack payload 약 508.2 MB와 비중요 attachment 메시지의 압축 MIME 약 659.5 MB가 재작성·DB 회수의 최대 후보이며, 합계 약 1.09 GiB는 구현 전 추정치다. 공유 object·중요 자료·기존 복구 사본은 보존. 서버 원문 존재 여부 미확인. 삭제·MIME 변환·pack 재작성·DB 축소 0건, scheduler 변경 0건 |
| 2026-09-06 | PRUNE APPLY · Runbook + LLM 안내서 | 사용자 최신 목표와 로컬/링크 메타데이터 확인 반영. 범용 Bridge·샘플 Gate·반복 승인문을 제거하고 결과물 공유 + 이메일 이관으로 재구성. 시스템 변경 없음 |

현재 상태:
- 문서: `OPERATIONAL_GO_RECORDED`; 이전 `HOLD`와 rollback 기록은 적용 이력에 보존
- 일반 결과물 연결: `GENERAL_ARTIFACTS_ACCESS_PASS`. 기존 `CoolFamDrive/OpenClaw_Output`가 CoolBot 소유·writer, 개인 역할 계정 Editor이며 양쪽 materialization·동일 Cloud item·MacBook 열람/검색 확인 완료. 단일 사용자 진입점 확정
- 이메일 shadow 복원·재색인 및 운영 전환: `OPERATIONAL_GO`; log-safe production generation 양쪽 endpoint 검증 PASS
- 두 계정 상시 읽기·보호된 로컬 KB: `DESIGN_AUTHORIZED`
- 선택적 첨부 보존: `NAVER-B_LIMITED_PASS`; NAVER-A 첨부 보충 `NOT_VERIFIED_SKIPPED_BY_USER`이며 선행조건 아님. 이번 운영 신규분 분기는 `NOT_EXERCISED`
- 이메일 Drive 문서: `OPERATIONAL_GO`; 개인 소유 폴더와 CoolBot 생성 개별 파일의 소유권·권한을 구분해 검증했고, 최신 운영 generation의 양쪽 materialization·hash·열람·검색·freshness PASS
- 정기 backup·보존정책: `DEFERRED_BY_USER`; 자동 snapshot·직접 전송·세대 회전은 현재 구축하지 않고 후속 작업의 선행조건으로 요구하지 않음. 검증된 이관 bundle·repository·shadow·복구 사본은 삭제 승인 전까지 보존
- 통합 baseline: 운영 generation 12,703건 양쪽 source-key 전건 일치; CoolBot scheduler `ACTIVE`, MacBook scheduler `PAUSED`, 기존 MacBook ID rollback `READY`
- A·E0 모두 사용자 제공 결과로 수집 완료. CoolBot A의 ‘E0 없음’은 해소됐으며 재실행 불필요.

통합 미완료 항목:

- 일반 Artifacts의 MacBook 접근 연결 범위에는 남은 항목이 없다. 이메일 운영의 별도 관찰·복구 정책은 기존 운영 기록과 해당 rollout 절차를 따른다.

두 결과의 endpoint 변경 0건은 사용자 보고로 기록한다. 기존 MacBook Microsoft source 비활성, 외부 embedding/본문 전송 증거 없음은 보고된 관찰 범위이며, 보존기간·로그 안전성을 대신 증명하지 않는다.

다음 갱신에는 날짜·rollout·endpoint·실제 변경·검증 결과·blocker·rollback 상태·다음 행동만 기록한다. `GO`는 확인된 범위에만 쓰고 미확인은 `NOT_VERIFIED`, 실패·차단은 `HOLD/ROLLBACK` 그대로 남긴다.

## 9. 참고와 이전 안내의 처리

- [CoolFam LLM 실행 안내서](./coolfam-runbook-llm-usage-guide.md)
- [공통 고객 적용 Prompt Set & Runbook](../../02-Implementation/ax-customer-llm-prompt-runbook.md)
- [설계 보고서](../../01-Analysis/ax-consulting-agent-framework-report.md)

이전 Prompt 0/A/B/C/D/H/S2-0/E0의 식별자는 안내서에서 유지한다. 과거에 복사한 Stage 2 실행팩은 폐기하고 최신 안내를 사용한다. 11장 적용 기록 링크는 위 호환 anchor로 연결한다. 제거된 절차는 기존 설계 이력이지 실행 대기 명령이 아니다.
