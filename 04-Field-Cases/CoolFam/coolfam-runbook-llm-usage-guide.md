---
aliases:
  - CoolFam Runbook을 LLM으로 실행하는 방법
tags: [coolfam, llm-guide, openclaw]
type: field-llm-usage-guide
status: rollout-complete-operational
version: "2.4"
created: 2026-08-20
last_reviewed: 2026-09-19
---

# CoolFam LLM 실행 안내서

[운영 Runbook](./coolfam-coolbot-personal-pc-continuity-runbook.md)이 구조·권한·상태의 정본이다. CoolFam rollout은 완료됐고 현재 blocker는 없다. 이 안내서는 **평소 사용 → 문제 확인 → 필요한 범위만 조치** 순으로 사용한다. A·E0·C·F1 이관 Prompt는 완료된 구축 절차의 이력이며 새 rollout이나 명시적 재점검 요청 없이 재실행하지 않는다. 문서 갱신은 시스템 변경 승인이 아니다.

## 평소 사용

1. **이메일 KB:** MacBook에서 개인 소유 Email RAG KB 폴더를 열고, 최신 generation의 문서 2개·manifest·complete marker가 같은 generation인지 확인한 뒤 문서를 열거나 문구를 검색한다.
2. **중요 첨부:** 첨부를 지속 보관할 메일은 해당 계정의 사용자 생성 `중요` mailbox에 둔다. 별표·flag·하위 mailbox는 판정 기준이 아니다. 일반 메일은 본문만 보관하며, 나중에 중요 mailbox로 옮긴 메일은 서버 원문이 남아 있는 범위에서 첨부를 보충한다.
3. **일반 Artifacts:** MacBook의 materialized `CoolFamDrive/OpenClaw_Output` 폴더를 단일 진입점으로 사용한다. Email RAG KB는 별도 개인 소유 폴더이므로 섞지 않는다.
4. **개인 자료:** 필요한 현재 작업에서만 현재 링크·범위를 CoolBot 계정에 임시 Viewer로 공유하고, 작업 후 해제한다. 과거 링크로 background 탐색하지 않는다.
5. **갱신 문제:** 중복 실행이나 scheduler 재생성을 먼저 하지 않는다. 기대 generation, 문서 2개·manifest·marker, hash·freshness, CoolBot scheduler 최근 상태를 읽기 전용으로 확인하고 sync·index·게시 중 실패 단계만 보고한다.

## 1. 문제 확인 시작 방법

각 PC의 기존 작업에 최신 Runbook과 이 안내서를 열거나 첨부한다. 새 task·별도 evidence 파일은 필수가 아니다. 문서 접근에 사용할 기존 파일 링크:

[CoolFam 운영 Runbook — Google Drive](https://drive.google.com/file/d/1IdFdfceNPM-AN-AkFHrwo3sG_7wv3_gj/view?usp=drive_link)

두 PC에서 같은 revision과 검증일을 읽었는지 확인한다. 링크를 읽지 못하면 최신 Markdown 첨부 또는 이미 접근 가능한 local sync 파일을 사용한다. 문서가 없으면 내용을 추측하지 않는다.

- 일상 문제는 위 `평소 사용` 5번으로 시작한다.
- 다른 endpoint 상태가 필요하면 원격으로 추정하지 말고 비식별 결과만 기존 MacBook E0 작업과 CoolBot 작업 사이에 전달한다.
- Prompt E0·A·C는 이관 완료 이력이다. 새 rollout, endpoint 교체, 구조적 오류 진단을 명시한 경우에만 필요 부분을 재사용한다.

개인 CnwC의 과거 절대경로·Drive 링크는 고정 입력으로 재배포하지 않는다. MacBook에서는 현재 열린 workspace와 실제 scheduler 설정으로 root를 확인한다. CoolBot에서 개인 원천 Drive를 읽을 때는 그 작업의 새 요청·링크·임시 Viewer 공유가 필요하다. 개인 소유 이메일 KB 출력 폴더는 별도로 지정된 Editor 공유 범위에서 예약 게시하며 원천 탐색에 사용하지 않는다.

## 2. Prompt E0 — MacBook의 이메일 실행 현황 확인

**완료된 이관 baseline Prompt.** 개인 PC용 `Prompt B`도 이 E0을 사용했다. 현재 일상 운영에서 E0 전체를 재실행하지 않고, MacBook endpoint 정보가 실제로 변경됐거나 특정 항목의 읽기 전용 추가 점검이 필요할 때만 해당 부분을 사용한다.

```text
HISTORICAL_COMPLETED_DO_NOT_EXECUTE
최신 CoolFam 운영 Runbook을 읽고 현재 MacBook의 이메일 RAG 이관 준비를 읽기 전용으로 점검하세요.
ROLLOUT_ID: COOLFAM-EMAIL-RAG-2026-001
MODE: READ_ONLY_BASELINE

대상은 Personal Email RAG KB — Daily Sync와 사용자가 지정한 두 Naver 계정의 기존 일일 build입니다.
현재 workspace·기존 scheduler·실행 정의를 근거로 경로를 확인하세요. 과거 경로를 단정하거나 Drive 전체를 검색하지 마세요.
정확한 대상이 식별되지 않으면 위치만 질문하세요. 미구성 Agent/Bridge/loader는 이 점검의 blocker가 아닙니다.

확인할 것:
- 실제 runner·source access 방식(직접 mailbox / local mail DB / 혼합), dependency, scheduler 종류와 활성 상태, 시각·시간대
- KB 저장 형식, 열람 문서와 실행 DB의 구분, 증분 cursor, 동시 실행·중복 방지
- 대상 mailbox·기간·제외범위·보존정책의 설정 존재 여부
- 외부 embedding/LLM 전송 여부, 원문이 transcript·log로 유입될 가능성
- CoolBot으로 옮길 최소 코드·설정의 비식별 manifest와 로컬 rollback 방법
- MacBook에서 공유 문서를 직접 열고 검색할 때 필요한 실제 접근 방식

금지: 메일·첨부·실제 KB 본문 열람, credential 값 출력, Skill 실행/복사/수정, scheduler·공유 변경.
제한자료가 설정 파일에 섞여 있으면 내용을 출력하지 않고 안전하게 확인할 방법을 찾으세요.
개인 원문·파일명·절대경로·공유 URL은 task 결과와 공용 문서에 남기지 마세요.
비민감 구현 경로만 endpoint 로컬 계획에 쓰고, 공유 요약은 역할·익명 ID로 표시하세요.

반환: DOCUMENT_REVISION / ENDPOINT / SOURCE_MODE / SCHEDULER / KB_FORMAT /
PORTABILITY / PRIVACY_AND_EXPORT / BACKUP_AND_ROLLBACK / BLOCKERS / NEXT_ACTION.
상태는 READY_FOR_PLAN 또는 NEEDS_INPUT이며, 구현 완료나 기존 Gate의 GO로 표시하지 마세요.
```

## 3. Prompt A — CoolBot 준비 확인

**완료된 이관 baseline Prompt.** CoolBot runtime·Drive·ACL·scheduler baseline은 이미 수집·적용됐다. 현재 일상 상태 확인은 위 `평소 사용` 5번을 우선한다.

```text
HISTORICAL_COMPLETED_DO_NOT_EXECUTE
최신 CoolFam 운영 Runbook을 읽고 CoolBot의 결과물 공유 및 이메일 이관 준비만 읽기 전용으로 점검하세요.
ROLLOUT_ID: COOLFAM-EMAIL-RAG-2026-001
MODE: READ_ONLY_BASELINE
AGENT_WORKSPACE: /Users/coolbot_macmini/.openclaw/workspace

Runbook의 CoolFamDrive parent와 제안된 CoolBot-Shared 경로를 확인하세요.
기존 같은 목적 폴더·writer·참조가 있으면 재사용 후보로 보고하고 생성·rename하지 마세요.
CoolBot 계정, Drive materialization, 대상·부모 ACL과 상속, 개인 계정 Editor 공유 가능성을 확인하세요.
권한을 읽을 수 없으면 미검증으로 표시하고 이메일 격리를 단정하지 마세요.

이메일 내용 없이 runtime·보호된 로컬 저장소 후보·OS credential store 사용 가능성·실제 scheduler를 점검하세요.
로컬 cron manifest와 실제 활성 scheduler를 구분하고 이름 검색만으로 작업 부재를 단정하지 마세요.
AgwA 밖의 DB·cache·index와 비식별 상태 보고가 가능한지 확인하세요.
FileVault·변경분 backup·30분 rollback, 독립 backup 부재 시 실제 손실 범위를 확인하세요.

메일 접근, credential 읽기/등록, 폴더·권한·cron·manifest 변경은 하지 마세요.
개인 Drive 내용은 이번 점검에 필요하지 않습니다. 보이는 공유 폴더를 탐색하지 마세요.
미구성 Bridge나 Skill loader를 이유로 이 현황 점검 자체를 중단하지 마세요.

반환: DOCUMENT_REVISION / ENDPOINT / SHARED_OUTPUT_PRESTATE / ACL_BOUNDARY /
LOCAL_RUNTIME / LIVE_SCHEDULER_AND_MANIFEST / PRIVACY / BACKUP_AND_ROLLBACK /
BLOCKERS / NEXT_ACTION. 마지막은 READY_FOR_PLAN 또는 NEEDS_INPUT입니다.
```

## 4. Prompt C — 두 결과를 하나의 변경계획으로 통합

**완료된 이관 계획 Prompt.** 기존 A·E0의 비식별 요약을 통합해 현재 운영 구조를 확정했다. 아래 블록은 구축 이력·설계 참조용이며, 현재 운영을 다시 계획 단계로 돌리거나 이미 확정된 경로·권한·사용자 결정을 재질문하지 않는다. raw 로그·개인 파일명·메일을 합친 새 파일은 만들지 않는다.

```text
HISTORICAL_COMPLETED_DO_NOT_EXECUTE
현재 작업의 CoolBot Prompt A와 MacBook Prompt E0 결과를 최신 CoolFam Runbook 기준으로 통합하세요.
없는 endpoint 증거만 요청하고, 이미 확인된 경로·권한·사용자 결정을 반복 질문하지 마세요.

결정된 요구:
- AgwA·CnwC 전체 sync 없음. CoolBot 결과물 정본을 MacBook에서 이용.
- 개인 원천 Drive는 현재 요청·링크에 한정한 임시 공유. 이메일 KB 출력은 개인 소유 폴더를 CoolBot Editor로 지정 공유하며, 일일 게시 동안 권한을 유지.
- 사용자 지정 두 Naver 계정 읽기와 AgwA 밖 보호된 로컬 KB 상시 보관은 설계 승인됨.
- MacBook은 KB 문서를 직접 열고 검색.
- 이메일 KB 출력은 coolwind@hotmail.co.kr 소유, coolfam830@gmail.com Editor인 전용 Drive 폴더. 일반 CoolBot 소유 Artifacts와 분리.
- 별도 파일·backup 암호화는 추가하지 않음. 기존 FileVault·Keychain은 유지. 정기 백업 구축은 사용자 결정으로 rollout 범위에서 제외했으며 필수 후속 작업이 아님.
- 개인 소유 Email RAG KB 전용 폴더와 CoolBot Editor 권한, 양 endpoint materialization·열람·검색은 확정·검증 완료. 기존 개인 원천 링크나 일반 Artifacts 폴더로 대체하지 않음.
- MacBook은 게시 문서를 직접 열고 검색. 별도 원격 검색 서비스를 선행조건으로 만들지 않음.
- 원문 전체 아카이브·첨부·credential·live DB·vector index·cache·로그는 Drive 게시 제외. 열람 문서의 필드·보존 및 대상 ACL은 계획에서 확정.
- A·E0 모두 제공됨. A의 E0 없음은 해소됐으므로 재실행 요구 금지.
- DIRECT_MAILBOX/IMAPS, 06:00 Europe/Berlin과 runner drift는 E0 보고값으로 사용. 두 계정 sync 성공 후 index 조건 유지.
- 두 source의 첨부 중요 판정은 source별로 읽기 전용 확인한 exact 사용자 생성 `중요` IMAP mailbox identifier membership만 사용. 표시 이름·별표·중요 flag·하위 mailbox 추론 금지.
- 중요 mailbox 소속은 전체 MIME·첨부를 보관. 그 외는 본문·필수 메타데이터와 첨부 payload·파일명을 제거한 수정 MIME만 보관하고 attachment store에 저장하지 않음. 수정본을 원본 또는 원본 무결성 보존본으로 표시하지 않음.
- 여러 mailbox에 나타나는 동일 메시지는 membership 전체를 먼저 평가해 중요 소속이 하나라도 있으면 첨부를 보존. 이후 중요 mailbox 이동은 원문이 서버에 있을 때만 멱등 backfill하고, 없으면 복구 불가 상태로 표시. 중요에서 나가도 기존 첨부 자동 삭제 금지; 공유 object의 다른 참조 보존.
- 일반 결과물 공유와 이메일 접근·실행을 별도 작업으로 취급.

공유 결과물 연결 / 이메일 수동 shadow / 예약 전환을 구분한 최소 변경계획을 만드세요.
각 계획에 확인된 exact target의 endpoint 로컬 참조, 변경분 manifest, 검증, rollback을 포함하세요.
보존·메일 범위·외부 전송·시각/시간대가 미정이면 기존 설정에서 확인하고 필요한 새 결정만 질문하세요.
정확한 값이 없는 실행 명령이나 placeholder 승인문은 생성하지 마세요.
runner drift·전체 mutex·로그 누출·기간/보존·일관된 DB backup/30분 restore를 각각 검증 항목으로 두세요.
실제 scheduler에만 3건, manifest에만 1건은 기존 불일치로 대조하되 무관한 작업을 수정하거나 전체 manifest를 덮어쓰지 마세요.
일반 결과물은 기존 후보의 writer·ACL을 확인한 뒤 재사용을 우선하고, 대상 ACL과 Shortcut 검색은 연결 후 시험하세요.
이메일 영역은 개인 소유자와 CoolBot Editor 두 계정만 접근하도록 상속 권한까지 검증하세요. 새 대상과 실제 생성 파일 소유권을 확인하고 기존 일반 출력 ACL 결과를 재사용하지 마세요.
공유 해제 시 게시를 중단하고 재공유를 요청하세요. 원천·상위·형제 폴더 탐색이나 임의 게시 경로 우회는 금지합니다. 넓은 부모 권한을 가진 후보를 그대로 사용하지 마세요.
완성된 문서 세대·비식별 manifest·해시로 부분 sync를 판별하고 MacBook의 실제 문서 열람·합성 검색·freshness를 검증하는 계획을 포함하세요.

shadow는 별도 출력·cursor를 사용하고 production을 덮어쓰지 않습니다.
수동 검증이 끝나기 전 MacBook 예약을 중지하거나 CoolBot 예약을 활성화하지 않습니다.
전체 mutex는 게시 완료까지 유지하고, MacBook은 완전성 검사에 통과한 세대만 사용합니다.
실제 이메일 전에는 격리된 합성 IMAP/MIME fixture로 mailbox 식별자 오판, 다중 membership, 첨부 제거 MIME, filename 비보관, FTS/vector 재색인, 승격 backfill·원문 부재, 강등 보존, 공유 object 참조, 중단·재실행·복구·로그 canary를 검증합니다. 기존 MacBook DB·첨부는 합성 개발에서 읽거나 변환하지 않습니다.
전환은 MacBook 예약 중지 → 진행 작업 종료 확인 → 최종 일관된 backup 확보·검증 → CoolBot 단일 writer 활성화 순서입니다.
실제 cron과 config/cron-jobs.json을 함께 갱신하는 범위를 포함하세요.
메일 발송·삭제·이동·읽음 변경은 금지합니다.

이 블록은 완료된 계획 이력입니다. 정상 운영에서 재실행하거나 새 공유·credential·메일 접근·cron 승인으로 해석하지 마세요.
반환: CONSISTENCY / DECISIONS / VERIFIED_TARGETS / MINIMAL_CHANGESETS /
ACCEPTANCE_TESTS / ROLLBACK / OPEN_QUESTIONS / EXACT_NEXT_ACTION.
```

## 5. 완료된 계획 실행 이력

아래 1~4는 모두 완료된 구축 절차의 요약이다. 현재 작업 목록이 아니며, 정상 운영 중 반복 실행하지 않는다. 새 기능·endpoint 교체·복구가 필요한 별도 작업에서는 사용자가 구체적 범위를 다시 승인해야 한다.

1. 결과물 연결: 폴더·공유·MacBook 진입점을 구성하고 비민감 문서의 양쪽 열기·검색·동일 정본을 검증한다.
2. 이메일 수동 shadow: 최소 실행 정의·credential을 준비하고 합성 데이터, 승인된 표본 순으로 실행한다. 기간·문서 필드·보존·로그 안전성·복구 범위를 확정한다. 검증된 두 계정 전용 Drive 영역에 완성된 KB 문서만 게시하고 MacBook 열람·검색·freshness를 확인한다.
3. 예약 전환: 수동 결과·복구가 통과한 뒤 기존 예약을 가역 중지하고 CoolBot cron을 켠다. 실패 시 CoolBot을 먼저 멈추고 MacBook을 복원한다.
4. 운영: 첫 7회 예약 결과를 검증했다. 메일 내용 없이 성공시각·건수·freshness·오류 유형만 보고하는 계약은 계속 유지한다.

‘계획 작성’은 ‘실행’이 아니며 ‘설계 승인’은 credential 등록이나 cron 활성화 완료가 아니다. 일반 산출물 저장마다 별도 publish 승인을 반복하지 않는다.

## 5A. Prompt F1 — 최종 증분 snapshot과 production cutover 실행팩 완료 이력

**이전·cutover에 사용한 완료 절차이며 재실행하지 않는다.** 기준점 `PREP-20260907T144624+0200`과 최종 successor, F1·bundle·repository·shadow·복구 사본은 보존 상태다. 아래 실행팩은 복구 감사와 이력 참조용이며, 정상 scheduler와 production에 다시 적용하지 않는다.

MacBook의 기존 E0 작업에 다음 한 문장 블록을 전달한다.

```text
HISTORICAL_COMPLETED_DO_NOT_EXECUTE
최신 CoolFam 운영 Runbook의 Prompt F1을 실행하세요.
ROLLOUT_ID: COOLFAM-EMAIL-RAG-2026-001
MODE: FINAL_INCREMENTAL_SNAPSHOT_ONLY
BASE_GENERATION: PREP-20260907T144624+0200
SCHEDULER_ID: personal-email-rag-kb-daily-sync

이미 확인한 E0 baseline·runner·repository·backup 결과를 재실행하지 말고, 현재 실제 scheduler 상태와 최근 성공 실행만 확인하세요. 기존 scheduler 정의와 상태를 rollback manifest에 보존한 뒤 해당 scheduler를 PAUSED로 전환하세요.

관련 Codex task, sync, embedding run, building generation, wrapper/index process, DB writer, write-lock이 모두 0일 때만 drain PASS로 판정하세요. 상주 read-only daemon handle은 writer가 아니지만, 실제 쓰기·lock 여부를 별도로 확인하세요. drain 실패 시 snapshot을 시작하지 말고 기존 scheduler 상태를 복원하세요.

기존에 검증된 backup repository를 재사용하고 재초기화하지 마세요. canonical production archive, DB 내부 cursor/checkpoint, attachment store·pack index를 한 quiesced generation으로 snapshot하세요. credential·token·로그·cache·analytics·vector DB·workspace·개인 파일은 제외하고 기존 전체 기간·MIME·첨부를 정리하거나 변환하지 마세요.

새 snapshot에 대해 SQLite quick_check·integrity_check, WAL 일관성, source별 cursor와 마지막 completed run 일치, folder high-water mark, 전체/source별 메일 건수, 중복 source key, attachment reference·unique object, pack CRC·content hash, missing·broken·orphan 0을 검증하세요. 메일 원문·주제·주소·첨부명·계정 비밀은 출력하지 마세요.

기준 snapshot과 같은 repository의 incremental snapshot이면 먼저 저장소 자체의 full verify를 통과시키세요. 전송팩이 base generation에 대한 종속성과 새 snapshot의 모든 필요 blob·index·manifest dependency closure를 자동 검증하고 CoolBot의 기존 검증 repository에 멱등 병합할 수 있을 때만 증분 bundle을 만드세요. 이 조건을 증명하는 공식/기존 공구가 없으면 최신 snapshot을 독립적으로 복원·검증할 수 있는 새 전체 bundle을 만드세요. 임의 rsync·tar delta나 실행 중 DB 직접 복사를 증분으로 간주하지 마세요.

bundle은 owner-only 권한으로 생성하고 generation·base generation·snapshot ID·생성시각·전체/source별 건수·cursor/high-water 요약·attachment 집계·제외목록·파일 크기·SHA-256을 manifest에 남기세요. bundle을 재열람해 manifest 파일 수·크기·hash와 일치하는지 검증하세요.

어느 단계든 실패하면 실패 generation·bundle을 사용 금지로 표시하고 기존 scheduler를 시작 전 상태로 복원하세요. 성공하면 CoolBot 최종 전환이 끝날 때까지 MacBook scheduler를 PAUSED로 유지하고 원본 DB·vector·cursor·attachment store를 변경·삭제하지 마세요.

반환: GENERATION / BASE_GENERATION / PRESTATE / RECENT_SUCCESS / PAUSE / DRAIN /
SNAPSHOT / ARCHIVE_INTEGRITY / CURSOR_AND_HIGH_WATER / ATTACHMENT_INTEGRITY /
BUNDLE_TYPE(INCREMENTAL_VERIFIED 또는 FULL_FALLBACK) / DEPENDENCY_CLOSURE /
MANIFEST / TRANSFER_BUNDLE / SCHEDULER_FINAL_STATE / ROLLBACK / VERDICT / EXACT_NEXT_ACTION.
```

CoolBot은 수신 bundle의 외곽·내부 hash와 base 종속성을 검증한 후 다음을 적용한다.

1. 기존 검증 bundle·repository·`PREP-20260907T144624+0200` shadow·FTS·vector를 보존한다. 증분 bundle은 기존 repository의 복사본에만 멱등하고 repository full verify를 다시 통과시킨다. 불완전 delta, 알 수 없는 base, hash 충돌은 즉시 HOLD다.
2. 현재 shadow를 in-place 덮어쓰지 않고 새 successor shadow에 최신 snapshot을 복원한다. generation manifest로 중복 적용을 거부하고 source별 cursor·folder high-water·최근 completed run을 기준점과 비교한다.
3. 새 archive에 포함된 FTS의 행 수·missing·orphan·FTS5 integrity를 확인한다. 통과하면 기존 전체 FTS를 다시 만들지 않고, 최종 snapshot의 일관된 FTS를 사용한다. 누락·stale이 있을 때만 재생성한다.
4. CoolBot의 검증된 vector DB를 successor의 별도 복사본으로 복제하고, 동일 fingerprint에서 backstop 증분 scan으로 추가·변경·누락 embedding만 보완한다. active generation·watermark·archive `embed_gen`·실제 vector row coverage를 함께 대조하고 missing/failed 0이 아니면 활성 후보로 삼지 않는다.
5. 전체/source별 건수, 기간, 중복 source key, SQLite quick/integrity, cursor/high-water, attachment reference/object·pack CRC/hash·missing/orphan, FTS/vector/hybrid 합성 검색을 비식별 비교한다. 원문이나 검색 결과 내용은 출력하지 않는다.
6. 검증된 최종 KB 열람 문서 generation을 개인 소유 Email RAG Drive 폴더에 게시한다. 완성 문서·manifest·complete marker 순으로 게시하고 폴더 소유자와 개별 파일 소유자·ACL을 구분해 확인한다. 원문 archive·첨부·credential·DB·vector·cache·로그는 게시하지 않는다.
7. MacBook에서 파일 수·SHA-256·manifest·marker를 대조하고 실제 materialization, 문서 열람, 합성 문구 검색, freshness를 통과해야 게시 PASS다. marker 생성 시각만으로 동기화 완료를 판정하지 않는다.
8. 위 검증이 모두 통과한 후에만 CoolBot의 실제 scheduler와 `config/cron-jobs.json`의 해당 항목 하나를 같은 변경으로 갱신한다. `06:00 Europe/Berlin`을 실제 scheduler timezone과 manifest 모두에 명시하고 canonical runner를 exact local reference로 고정한다. 무관한 scheduler·manifest 불일치는 수정하거나 전체 manifest를 덮어쓰지 않는다.
9. CoolBot 단일 writer·전체 mutex·실제 예약 한 건을 확인한 뒤 활성화한다. 최종 성공 후 MacBook scheduler는 PAUSED로 유지한다. 실패 시 CoolBot scheduler를 먼저 중지하고 writer·lock 0을 확인한 뒤 MacBook의 보존된 정의를 ACTIVE로 복원한다.

## 6. Prompt D — 결과 기록

```text
현재 작업의 검증된 CoolFam 결과를 운영 Runbook의 '현재 적용 기록'에 간결하게 반영하세요.
문서만 수정하고 endpoint·공유·scheduler는 변경하지 마세요.
날짜, rollout, endpoint, 변경 범위, 실제 검증, blocker, rollback 상태, 다음 행동을 기록하세요.
같은 사건은 중복 행 대신 현재 상태를 갱신하세요. 제안·사용자 보고·직접 관찰을 구분하세요.
HOLD/ROLLBACK과 미검증을 숨기지 말고, 전체 구현 완료로 확대하지 마세요.
원문·개인 파일명·개인 경로·계정 비밀·공유 URL·raw 로그를 기록하지 마세요.
설계가 바뀐 경우 LLM 안내서의 해당 입력도 정합성만 맞추세요.
정본을 편집할 수 없으면 새 사본을 만들지 말고 비식별 UPDATE_PACKET만 반환하세요.
반환: UPDATED 또는 WRITEBACK_BLOCKED / VERIFIED_SCOPE / NEXT_ACTION.
```

## 7. 개인 자료가 다시 필요할 때

```text
현재 작업에 필요한 개인 자료를 목적 별칭과 필요한 범위로 설명하고,
개인 소유자에게 해당 폴더의 현재 링크와 CoolBot 계정 Viewer 임시 공유를 요청하세요.
과거 링크·개인 파일명·원문을 기억에서 복원하거나 registry로 background 탐색하지 마세요.
작업 목적·종료 조건·저장 금지·공유 해제 필요를 함께 알려주세요.
자료 내용이 task/memory/log에 남지 않는 도구가 없으면 읽지 말고 안전한 대안을 요청하세요.
작업 후 비민감 완료 상태만 남기고 재사용 시 다시 공유 요청이 필요하다고 기록하세요.
```

## 8. 이전 Prompt와의 호환

| 기존 ID | 현재 용도 |
|---|---|
| Prompt 0 | 이 안내서 1절의 문서·endpoint 확인. 긴 A–D 템플릿 생성 단계 제거 |
| Prompt A | 완료된 CoolBot baseline 이력. 새 구조적 재점검에서만 필요 부분 사용 |
| Prompt B / E0 | 완료된 MacBook baseline 이력. 특정 원격 상태 추가 확인에서만 사용 |
| Prompt C / F1 | 완료된 이관 계획·cutover 절차. 정상 운영에서 재실행 금지 |
| Prompt D | 검증된 상태 변경이 있을 때의 정본 기록 |
| Prompt H | 실패 원인을 입력 누락·runtime·privacy·권한·복구로 나누고 필요한 조치만 계획. 자동 변경 없음 |
| Prompt S2-0 | 퇴역. 과거 Bridge/PS-014 실행팩을 실행하지 말고 최신 A·E0→C 사용 |

현재 [운영 Runbook의 구축 순서](./coolfam-coolbot-personal-pc-continuity-runbook.md#5-구축-순서와-완료-조건) 1~5단계와 첫 7회 자동 관찰은 완료됐다. 정기 백업 구축은 `DEFERRED_BY_USER`로 범위 제외이며 필수 후속 작업이 아니다. 기존 Stage 1 GO와 이관 Prompt는 이력이며 새 권한·credential·scheduler 변경의 승인을 대신하지 않는다.
