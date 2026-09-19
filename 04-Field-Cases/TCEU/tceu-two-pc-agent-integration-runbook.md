---
title: TCEU 운영 Runbook
type: field-case-runbook
status: partially-applied-promotion-pending
version: 3.0
created: 2026-08-15
last_reviewed: 2026-09-19
---

# TCEU 운영 Runbook

목적은 두 가지다. **System Master가 Agent의 보관 허용된 KB·결과를 쉽게 이용하고, 개인 Skill은 System Master의 검토를 거친 revision만 팀에 배포한다.** 개인 원천은 필요한 작업 동안만 공유하며 Agent의 영구 기억으로 복제하지 않는다.

이 문서는 운영 계약·상태의 정본이다. 실행 문장은 [사용 안내서](./tceu-runbook-llm-usage-guide.md)에 둔다. 과거 설계·상세 시험은 Git 이력과 CllC 인계문서에 보존하고, 완료한 구축을 다시 실행하지 않는다.

## 1. 공간과 권한

| 공간 | 책임 | 사용 범위 |
|---|---|---|
| CnwC | System Master의 개인 업무 공간 | Context·Vault·Version은 기존 root의 논리 역할. 필요한 폴더만 TCEU Manager에게 임시 Viewer |
| AgwA | TCEU Manager의 기존 OpenClaw workspace | 작업 실행과 보관 허용된 RAG. runtime·DB·index·cache는 장치 로컬 |
| CllC | Agent 계정 소유, System Master Editor | 허용된 KB·완성 결과·운영자료·승인된 공용 Skill |
| 각 멤버의 개인 Skill | 해당 멤버 | 개인 작성·시험. 공유 요청 전까지 자동 수집하지 않음 |

System Master는 사람 검토자이며 TCEU Manager는 OpenClaw 별칭이다. Agent가 자신을 승인자로 기록해서는 안 된다. 실제 계정·장치 경로·개인 공유 URL은 공개 Git에 넣지 않는다.

기존 CllC를 재사용한다. 멤버의 공용 Skill 이용에는 **승인된 Skill 배포 영역의 Viewer**만 부여한다. CllC root의 KB·Artifacts까지 자동 공개하지 않으며 제출 권한과 배포본 수정 권한을 분리한다. 멤버별 회사 계정·실행 도구·OS와 실제 ACL을 확인한 뒤 한 번 연결한다. 모든 멤버가 같은 Agent나 OS를 쓴다고 가정하지 않는다.

## 2. KB·결과 사용

- System Master는 기존 CllC 진입점에서 `Knowledge/`와 `Artifacts/`를 연다. 같은 Cloud item인지 확인하고, 로컬 검색을 사용하면 해당 출력 폴더에서 실제 열기·검색을 시험한다.
- KB는 사람이 읽는 문서와 manifest를 제공한다. `version / source_ref / source_updated_at / generated_at / hash / 검증 상태`로 범위·최신성을 확인한다. live DB·vector index를 OneDrive로 열거나 전체 AgwA를 동기화하지 않는다.
- 기존 불변 release와 task별 artifact 구조를 유지한다. 완전한 파일·hash를 확인한 뒤 최신본으로 안내하고 부분 전달이나 실패 시 마지막 정상본을 보존한다.
- 자동 생성 영역의 writer는 Agent 하나다. 사람의 수정 의견은 `notes/` 등 별도 영역에 남긴다. 같은 파일을 두 PC에서 동시에 수정하지 않는다.
- 선정 운영 KB의 공유 완료를 Wiki·Issue 등 모든 업무 KB의 보관·공유 허용으로 확대하지 않는다. 새 KB는 원천별 권한·보존 범위를 먼저 확인한다.

로컬 저장, Cloud 반영, System Master 실제 열람은 별도 증거다. 서비스가 active인 것만으로 최신성이나 다른 PC 접근을 PASS로 판단하지 않는다.

## 3. 임시 개인 원천

현재 작업에 필요한 목적·폴더 별칭·Viewer·기간만 요청한다. 제공된 새 공유는 해당 작업의 접근 권한이며 자동 색인·상시 보관 권한이 아니다.

- 민감 원문과 이를 재현하는 추출문·요약·OCR·임베딩·스크린샷은 AgwA memory·RAG·로그·CllC에 저장하지 않는다. 도구가 transcript에 원문을 남기면 비민감 입력 또는 검증된 다른 처리 경로를 요청한다.
- 기억은 소유자 역할, 비민감 목적/위치 별칭, 재공유 필요 여부만 남긴다. 실제 민감 위치 매핑은 System Master가 관리하고 공유 토큰을 저장하지 않는다.
- 공유 회수 후 과거 URL·다운로드·cache로 접근을 우회하지 않는다. 다시 필요하면 소유자에게 현재 공유를 요청한다.
- 직접 권한·조직 링크·상속 권한을 구분한다. 회수 확인은 Agent 계정의 원본 위치 응답으로 하고, 폐기된 공유 링크의 오류만으로 접근 차단을 단정하지 않는다.
- 공유 해제와 로컬 사본 삭제는 별개다. 제한 검사 미발견은 PC 전체 부재가 아니며, 대상·보존 범위가 정해지지 않은 대량 삭제는 하지 않는다.

## 4. 개인 Skill → System Master 검토 → 팀 이용

### 4.1 승격 계약

1. **개인 작성:** System Master와 멤버는 자기 환경에서 Skill을 만들고 시험한다.
2. **후보 제출:** 작성자가 공용으로 쓸 별도 비민감 제출본만 준비한다. 개인 원본·memory·설정 전체를 복사하지 않는다. `tceu-shared-<name>` 이름, 상대 참조, 용도·의존성을 확인하고 검토 대기함에 snapshot ZIP·파일 hash를 둔다. 후보는 loader와 송신 watcher 밖에 둔다.
3. **사람 검토:** System Master가 목적, 내용, 민감정보, 필요한 도구·권한, OS/의존성, 표본 결과와 정확한 bundle hash를 검토한다. 자기 작성 Skill도 본인이 확인한 뒤 승격하며 Agent가 승인을 대신하지 않는다.
4. **승격:** 검토한 revision과 실제 사람 승인 근거를 연결한 뒤 게시한다. 승인 후 내용이 바뀌면 새 후보로 다시 검토한다. 기존 “공용 원본에 저장하면 무조건 배포” 안내는 이 계약으로 대체한다.
5. **자동 이용:** 한 번 연결된 각 멤버 환경이 승인된 revision을 자동 수신·검증·로딩하고 다음 요청에서 사용한다. 승격 후 멤버별 수동 복사·반복 설치·재승인은 요구하지 않는다.

“즉시”는 온라인·동기화·loader·의존성 준비가 끝난 환경에서 승인본 도착 후 다음 요청에 반영한다는 뜻이다. 전송시간, 오프라인, 미설치 의존성은 별도 표시한다. 모든 멤버의 실제 연결·호출 증거가 모이기 전 전체 이용 가능으로 표시하지 않는다.

### 4.2 정본·승인·배포 경계

| 영역 | 의미 | 변경 권한 |
|---|---|---|
| 개인 Skill | 작성자 작업본 | 작성자 |
| `Skill-Candidates/` | 검토 중 snapshot·검토 결과, 자동 로딩 금지 | 지정 제출자·검토자, 후보 간 덮어쓰기 방지 |
| `Shared-Skills/` | 검토를 통과한 공용 정본 | System Master 및 승인된 게시 작업 |
| `Shared-Skills-Delivery/` | 불변 패키지·manifest와 명시적 활성 revision 선택 | 승인된 게시 작업; 멤버는 읽기 |
| 각 PC의 로컬 배포 영역 | 검증된 실행 사본 | receiver; 직접 편집 금지 |

위 표는 목표 ACL이다. 폴더 생성이나 JSON의 `approved` 필드만으로 권한·사람 승인이 강제됐다고 판단하지 않는다. 현재 승인·배포 ACL과 멤버 연결은 아직 미검증이다.

공용 PC가 CllC 소유자라는 사실은 TCEU Manager에게 임의 승격 판단을 맡긴다는 뜻이 아니다. 멤버가 후보·배포본을 수정해 검토를 우회하지 못하도록 실제 원격 쓰기 권한과 게시 경로를 확인해야 한다.

### 4.3 전달과 복귀

기존 ZIP·파일 inventory·SHA-256 검증을 재사용한다. 패키지·manifest의 Cloud 도착 순서를 가정하지 않고 불완전하거나 충돌한 revision은 활성화하지 않는다. 새 의존성·권한 확대는 자동 설치하지 않고 `NEEDS_SETUP`으로 표시한다.

**수정시각으로 최신 revision을 추정하지 않는다.** 승격·원복 모두 정확한 revision과 package hash를 지정한 게시 선택 기록을 사용해야 한다. 원복은 기존 불변 패키지를 다시 선택하는 새 게시 행위이며 파일 mtime 변경이나 과거 패키지 삭제로 흉내 내지 않는다. 이 선택 계약의 기존 송신·수신부 연결은 아래 미완료 항목이다.

각 대상 상태는 `PENDING_REVIEW / APPROVED / SYNCING / READY(revision) / NEEDS_SETUP / ERROR`로 구분한다. READY는 로딩 준비이며 호출 성공과 다르다. 이름 충돌, 일부 파일 누락, 잘못된 hash, 오프라인·재연결, 이전 revision 원복을 검증한다.

실행 중인 요청의 revision 고정/전환 대기는 별도 수용 조건이다. 현재 일반 OpenClaw execution lease는 미검증이며, watcher 1개·수동 lock 시험·Gateway 정지만으로 모든 실행을 보호한다고 주장하지 않는다. 네트워크 장애를 퇴역 지시로 해석하지 않고 마지막 정상본을 보존한다.

## 5. 현재 적용 상태

검토일은 2026-09-19다. 최신 직접 점검과 과거 인계, 사용자 설명을 구분한다.

| 항목 | 근거·상태 | 다음 행동 |
|---|---|---|
| CllC 존재·선정 KB | 09-19 로컬 직접 점검: 기존 root·KB manifest/release/LATEST hash 일치 | 기존 경로 사용. 새로 만들지 않음 |
| KB 자동화 | 09-19 직접 점검: path·5분 timer active/enabled | 원천별 최신성 확인 |
| CllC 원격 접근 | 09-10 인계: Owner/Editor·양쪽 Cloud 열람 PASS | 현재 ACL은 원격 재검증 필요 |
| CnwC 현재 공유 | 09-19 사용자 설명: 새 임시 Viewer 공유 | 이번 작업에 원문 불필요하여 열람·복사하지 않음. 현재 ACL 미검증 |
| 과거 개인 공유 회수 | 09-10 원본 위치 차단·CllC 이용·smoke 호출 PASS | 새 임시 공유와 구분. 로컬 잔존 전체 부재는 미검증 |
| 공용 Skill receiver | 09-19 직접 점검: active/enabled | 실행 중 보호가 검증됐다는 뜻은 아님 |
| Skill 원복 선택 | **FAIL** — 09-19 원본 revision `e9ecb710840a3bc24c8e`와 실제 active/선택 revision `3073d04058a677ebc5a2` 불일치. mtime 선택·기존 manifest 재사용 문제를 격리 합성 재현 | 명시적 승인 revision 선택으로 송신·수신 계약 수정 후 원복 시험 |
| System Master 송신 자동 시작 | 09-10 마지막 인계: 재로그온 후 watcher 0·지속성 FAIL; 이번에 해당 PC 재확인 안 함 | 실제 Action/runtime·로그 준비부터 수정·재시험 |
| 검토 후보 도구 | 09-19 로컬 적용: 별도 대기함과 ZIP/hash 생성기, 합성시험 6건 PASS | 실제 후보는 System Master 검토 후 처리 |
| 전 멤버 배포 | 설계 확정 전: 멤버 계정·도구·OS·ACL 미확정 | 대상 명단과 실행 환경 확인 후 연결 |
| 전체 | **부분 적용** | 아래 순서로 계속 진행 |

## 6. 남은 적용 순서

1. **현재 공용 PC:** KB·기존 서비스 재사용, 후보 대기함·검증 도구 준비. 이 단계는 완료했다. 실제 승격은 하지 않았다.
2. **System Master:** 멤버·도구·배포 Viewer 범위와 검토자를 확정하고 한 비민감 후보를 검토한다. 후보 hash와 승인을 연결하며 승인/배포 영역의 실제 ACL을 확인한다.
3. **구현 PC:** 기존 송신·수신부에 명시적 승인 revision 선택·원복을 연결하고 격리 시험 후 안전한 변경 시점에 적용한다. 과거 임시 revision을 바로 삭제하거나 서비스 중지로 실행 보호를 추정하지 않는다.
4. **System Master PC:** 진단 준비를 확인한 뒤 송신기 최소 수정·수동 시험 → 사용자의 실제 재로그온 → 단일 watcher 10분 지속·자동 패키징을 검증한다.
5. **각 대상 PC:** 승인 후보 하나의 같은 revision/hash 수신·실제 호출·정상 원복을 확인한다. 미접속 멤버, 의존성 부족, 실행 중 보호는 별도 미완료로 남긴다.

[CoolFam 실제 적용 교훈](../CoolFam/coolfam-coolbot-personal-pc-continuity-runbook.md)을 유지한다: 준비가 실패한 채 동일 시험을 반복하지 않고, 실제 예약 환경을 수동 환경과 대조하며, 수동 성공과 자동 성공을 분리하고, 상대 PC 열람/호출까지 확인한다. CoolFam의 메일 보관 권한·경로·7회 관찰·백업 제외 결정은 TCEU에 이식하지 않는다.

변경 전 대상·사본·복귀 방법을 정한다. 실패 시 관련 신규 배포만 멈추고 정상 결과·기존 업무·개인 원본을 보존한다. 인계는 `task / PC / 실제 시각 / 변경 / revision·hash / 검증 / blocker / 복귀 상태 / 다음 PC·Prompt`만 남긴다. 공개 Git에는 비민감 요약만, 실제 위치와 상세 증거는 CllC의 제한된 운영 자료에 둔다.
