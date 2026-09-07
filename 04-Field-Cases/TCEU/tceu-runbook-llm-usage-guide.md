---
title: TCEU Runbook 링크를 LLM으로 실행하는 방법
type: field-case-usage-guide
status: stage-1-hold-remediation-ready
version: 1.9
created: 2026-08-20
last_reviewed: 2026-08-21
---

# TCEU Runbook 링크를 LLM으로 실행하는 방법

## 1. 이 문서로 무엇을 하는가

이 문서는 [TCEU Runbook 실행 정본](https://drive.google.com/file/d/1BewvqzpXnE2ef66GRJ-XS7OlErtKUmRw/view?usp=drive_link)을 어느 PC의 Codex에 어떤 Prompt로 전달할지 정한다. 전체 패키지는 [AX Consulting Agent Framework 폴더](https://drive.google.com/drive/folders/10tK0B_IT514L95hUiw1HkTXNv5YOkjD5)에서 찾되, 실행할 때는 Runbook 파일 하나부터 지정한다.

> **최초 적용:** System Master가 자기 업무 PC에서 Prompt A를, 공용 OpenClaw PC에서 Prompt B를 직접 실행한다. System Master 업무 PC의 Codex가 Prompt C로 통합 판정하고 Prompt D로 비민감 결과를 TCEU Runbook에 반영한다.

## 2. 어느 LLM에 입력하는가

| 위치                  | 사용할 LLM      | 담당                                                             | 맡기지 않는 일                              |
| ------------------- | ------------ | -------------------------------------------------------------- | ------------------------------------- |
| System Master 업무 PC | **Codex**    | 최초 개인 endpoint 점검, 두 PC 통합판정, Runbook write-back               | 공용 OpenClaw PC의 실제 상태 추정              |
| 이후 구성원 업무 PC        | **Codex**    | 자기 PC와 자기 Context Workspace의 확장 Stage 점검                       | 공용 OpenClaw PC 또는 다른 구성원 PC의 실제 상태 추정 |
| 공용 OpenClaw PC      | **Codex**    | Gateway·Node·Telegram·Agent Workspace·Community Sharing 의존성 점검 | 대표 업무 PC 상태 추정, 사람 승인 대행              |
| Telegram 전담 Agent   | **OpenClaw** | Stage 2 이후 승인된 `task_id`의 제한 실행과 상태 반환                         | Runbook 전체 해석, Stage 판정, OS·폴더 재구성    |
| 다른 LLM              | 선택사항         | 읽기 전용 2차 검토                                                    | 별도 정본·동시 수정                           |

`tceu.manager`는 Agent용 Microsoft 365 service identity다. System Master 또는 사람 승인자가 아니다. TCEU System Master는 조승현이며, 통합 판정은 System Master가 사용하는 대표 업무 PC의 Codex에서 수행한다.

현재 적용 흐름은 `공용 PC read-only resolver → 업무 PC Prompt R → 업무 PC Prompt A → 공용 PC Prompt B → 업무 PC Prompt C 통합 → Prompt D 기록`이다. 하나라도 미확인·실패이면 HOLD하고 기존 업무를 유지한다.

## 3. Prompt 전에 공통값과 local 경로를 정한다

두 PC에서 같은 값을 사용한다.

| 값 | 예시 | 원칙 |
|---|---|---|
| `RUNBOOK_URL` | 위 TCEU Runbook 파일 링크 | 폴더가 아닌 파일 링크 |
| `ROLLOUT_ID` | `TCEU-ROLLOUT-2026-001` | 두 PC와 모든 Stage에서 동일 |
| `STAGE_GATE` | `1`, 이후 `2-A`, `2-B`, `2-C`, `3-A`, `3-B` | 한 번에 하나만 실행 |
| `EVIDENCE_TARGET` | `CURRENT_CODEX_TASK_ONLY` | Stage 1 기본값. 점검 대상 파일에는 쓰지 않음 |
| `RUNBOOK_EDIT_TARGET` | 위 TCEU Runbook 파일 링크 또는 승인된 local sync 경로 | Prompt D가 원본을 갱신할 실제 대상 |
| `COMMUNITY_TEST_ITEM_REF` | 승인된 비민감 기존 표본 1개의 상대경로 또는 share reference | 두 Community Sharing view의 동일 Cloud 원천 검증용 |

그다음 **해당 PC에서만 유효한 실제 절대경로**를 다섯 역할 모두에 대해 넣는다. 역할이 없으면 필드를 생략하지 않고 `NOT_APPLICABLE`로 쓴다.

| 공식 역할 | 각 구성원 업무 PC | 공용 OpenClaw PC |
|---|---|---|
| Agent Workspace for Automation | `NOT_APPLICABLE` | 공용 OpenClaw local workspace 실제 경로 |
| Cloud library for Community Sharing | 해당 PC에서 보이는 조직 공용폴더 local sync 경로 | 해당 PC에서 보이는 같은 조직 공용폴더의 local sync 경로 |
| Cloud workspace for Version control | 자기 결합 root 실제 경로 | `NOT_APPLICABLE` |
| Obsidian Vault for Organization | `SAME_AS_VERSION_WORKSPACE_PATH` | `NOT_APPLICABLE` |
| Context Workspace for Continuity | `SAME_AS_VERSION_WORKSPACE_PATH` | `NOT_APPLICABLE` |

업무 PC의 장치별 후보는 Codex loader `%USERPROFILE%\.agents\skills`, runtime `%USERPROFILE%\.local\skill-runtimes`, 임시 output `%USERPROFILE%\.local\agent-outputs\<workspace-id>`다. Cloud Context에는 portable `skills/` 정본과 최종 project output만 두며 Junction으로 local 경로를 연결하지 않는다.

업무 PC는 Community Sharing과 개인 결합 root 두 경로만 실제로 입력하고 Version·Vault·Context는 같은 root로 표시한다. 공용 PC는 Agent Workspace와 Community Sharing 두 경로를 입력한다. 반대 역할은 `NOT_APPLICABLE`이며 새 root를 만들지 않는다.

절대경로는 `LOCAL-ONLY`로 유지하고 공유 결과에는 `PATH_RESOLVED`, `PATH_MISSING`, `PATH_ROLE_MISMATCH`, `NOT_APPLICABLE`과 비민감 reference만 남긴다. Stage 1 evidence는 기본적으로 `CURRENT_CODEX_TASK_ONLY`다.

### Prompt 0 — placeholder 없는 최초 실행 Prompt 만들기

System Master는 A~D 템플릿을 직접 수정하기 전에 아래 Prompt 0을 한 번 실행한다. 이 Prompt는 시스템을 점검하거나 변경하지 않고 필요한 값을 한 번에 받아, 실행 가능한 A~D Prompt를 생성한다.

```text
TCEU 최초 Stage 1 실행 Prompt를 준비하세요.

RUNBOOK_URL: https://drive.google.com/file/d/1BewvqzpXnE2ef66GRJ-XS7OlErtKUmRw/view?usp=drive_link
RUNBOOK_EDIT_TARGET_DEFAULT: https://drive.google.com/file/d/1BewvqzpXnE2ef66GRJ-XS7OlErtKUmRw/view?usp=drive_link
DEFAULT_ROLLOUT_ID: TCEU-ROLLOUT-2026-001
DEFAULT_STAGE_GATE: 1
EVIDENCE_TARGET: CURRENT_CODEX_TASK_ONLY

확정된 입력값:
- MEMBER_COMMUNITY_SHARING_PATH_STATUS: RESOLVED_USER_CONFIRMED_LOCAL_VALUE
- MEMBER_VERSION_WORKSPACE_PATH_STATUS: RESOLVED_USER_CONFIRMED_LOCAL_VALUE
- SHARED_COMMUNITY_SHARING_PATH_STATUS: RESOLVED_USER_CONFIRMED_LOCAL_VALUE
- LOCAL_PATH_ADAPTER_REF: tceu-storage-role-paths
- LOCAL_PATH_ADAPTER_BOOTSTRAP: OPTIONAL_FOR_READ_ONLY_RESOLVER
- APPROVER_REF: TCEU_SYSTEM_MASTER
- APPROVER_IDENTITY_TYPE: HUMAN_BUSINESS_APPROVER
- SERVICE_ACCOUNT_REF: tceu.manager
- CHANGE_WINDOW: 2026-08-24 18:00–19:00 Europe/Berlin
- RTO_TARGET_MINUTES: 30
- ROLLBACK_OWNER_REF: SYSTEM_MASTER
- RUNBOOK_LOCAL_SYNC_PATH_STATUS: NONE_USE_DRIVE_LINK
- RUNBOOK_EDIT_TARGET: https://drive.google.com/file/d/1BewvqzpXnE2ef66GRJ-XS7OlErtKUmRw/view?usp=drive_link
- RUN_PROMPT_R: YES
- MEMBER_RERUN_READY: NO
- SHARED_RERUN_READY: NO
- OPERATIONAL_DRIFT: UNRESOLVED
- EXACT_SEQUENCE: Shared resolver → Prompt R → A → B → C → D

아직 필요한 입력만 짧은 체크리스트로 요청하세요.
- 공용 OpenClaw PC resolver가 확인한 AGENT_WORKSPACE_PATH와 runtime match 결과
- 양쪽 local view에 이미 존재하는 승인된 비민감 COMMUNITY_TEST_ITEM_REF 1개
- SAME_CLOUD_ITEM·ACL 역할 증거, Gateway 노출·인증, session/task 귀속, 운영 drift 분류

규칙:
1. 확정된 값을 다시 질문하거나 service account를 사람 승인자로 해석하지 마세요.
2. 절대 Windows 경로는 Cloud 문서에 기록하지 말고 그 PC의 실행 Prompt에서만 사용하세요. local-only `tceu-storage-role-paths` adapter가 있으면 우선 읽되, 최초 resolver 단계에서 adapter가 없다는 이유만으로 중단하지 마세요.
3. 미확정 값이 모두 채워질 때까지 Stage 판정·시스템 점검·Runbook 갱신을 시작하지 마세요. 공용 Agent Workspace는 실행 중인 OpenClaw config·process·workspace 설정을 read-only로 대조하고 넓은 디스크 검색, 임의 확정, 새 폴더 생성을 금지하세요.
4. 미확정 항목이 남으면 공용 PC용 read-only resolver Prompt와 `INPUTS_MISSING`만 반환하세요. resolver에는 다음 bootstrap 규칙을 넣으세요: adapter가 있으면 경로를 읽고, 없으면 실행 중인 OpenClaw config·service/process command line·working directory·workspace 설정으로 Agent Workspace를 확인한다. Community Sharing은 이미 확정된 local 값을 현재 공용 PC task에서 한 번만 입력받아 존재·역할만 검증한다. adapter 생성·저장은 승인된 변경창 전에 금지한다. adapter 부재를 terminal `LOCAL_ADAPTER_MISSING`으로 판정하지 말고, 확정된 Community Sharing local 값만 필요하면 `LOCAL_PATH_INPUT_REQUIRED`를 반환하세요. A·B·C·D나 변경 Prompt를 출력하지 마세요.
5. 공용 resolver 결과와 공통 시험 항목이 준비되면 변경창 전에는 read-only 검증만 하고, 실제 변경은 승인된 변경창에서 `Prompt R`부터 시작하세요.
6. 모든 값이 준비되면 동일 rollout_id를 사용한 Prompt R, A, B, C, D를 `EXACT_SEQUENCE`대로 출력하세요.
7. 출력되는 실행 Prompt에는 `<...>`, `PLACEHOLDER`, 빈 값이 하나도 없어야 합니다. Prompt D는 Prompt C와 같은 task에서 직전 응답을 사용하도록 작성하세요.
8. A·B에는 양쪽에 이미 존재한다고 확인된 COMMUNITY_TEST_ITEM_REF, 구체적인 read-only 회귀점검과 승인·복귀 참조를 포함하세요.
9. 마지막에 READY_TO_RUN 또는 INPUTS_MISSING만 반환하세요.
```

## 4. 가장 간단한 사용 순서

1. System Master 업무 PC에서 **Prompt 0**을 실행한다. 현재는 공용 resolver 결과가 없으므로 `INPUTS_MISSING`이 정상이다.
2. Prompt 0이 출력한 read-only resolver를 공용 OpenClaw PC에서 실행하고, 양쪽에 이미 존재하는 비민감 공통 시험 항목을 확정한다.
3. resolver 결과를 Prompt 0 task에 반환해 `READY_TO_RUN`과 placeholder 없는 실행 Prompt를 받는다.
4. 승인된 변경창에 System Master 업무 PC에서 **Prompt R**을 실행하고 회귀검증을 통과한다.
5. 두 PC에서 Runbook 링크를 열어 파일명·`version`·`last_reviewed`를 확인한 뒤, 같은 rollout으로 업무 PC **Prompt A**와 공용 PC **Prompt B**를 실행한다.
6. System Master 업무 PC의 같은 Codex task에서 **Prompt C**로 통합하고 **Prompt D**로 Runbook 17장에 반영한다.
7. 통합 결과가 `GO`이고 `RUNBOOK_UPDATED`가 확인된 경우에만 다음 Gate를 별도 지시한다. `HOLD`면 기존 업무를 유지하고 Prompt H로 돌아간다.

### 4.1 현재 HOLD의 처리 순서

현재 HOLD는 문서 반영 실패가 아니라 Gate가 실제 미완료임을 뜻한다.

```text
Prompt H로 blocker 분류
  ├─ member Node/OneDrive 문제 남음 → Prompt R → 회귀검증
  ├─ shared Agent Workspace 미확정 → 실제 runtime 경로 확인
  ├─ 공통 시험 항목 미선정 → 양쪽에 이미 존재하는 승인 표본으로 정정
  └─ queue/index 변화 → rollout 인과관계 분류
        ↓
입력·회귀·승인/복귀 정보가 모두 준비됨
        ↓
Prompt A → Prompt B → Prompt C → Prompt D
```

업무 중인 공용 PC의 queue나 index가 계속 변한다는 이유로 시스템을 멈추지 않는다. 변화는 `ROLLOUT_CAUSED`, `EXPLAINED_BASELINE_DRIFT`, `UNRESOLVED`로 분류하며, 적용이 만든 유해 변화 또는 원인을 설명할 수 없는 중대 변화만 HOLD 사유로 유지한다.

## 5. 복사해서 쓰는 Prompt

아래 A~D는 Prompt 0이 생성할 내용을 검토하기 위한 정본 템플릿이다. **`<...>`가 하나라도 남은 템플릿을 직접 실행하지 않는다.** 처음 적용은 `Stage 1 APPLY`지만 실제 시스템 변경이 아니라 두 PC read-only audit와 현재 Codex task의 결과 기록만 수행한다.

### Prompt A — 최초 System Master 업무 PC의 Codex

```text
TCEU Runbook 실행 정본:
https://drive.google.com/file/d/1BewvqzpXnE2ef66GRJ-XS7OlErtKUmRw/view?usp=drive_link

ROLLOUT_ID: <ROLLOUT_ID>
STAGE_GATE: 1
ENDPOINT: system-master-work-pc
MEMBER_REF: <조직이 승인한 비민감 구성원 참조키>
EXECUTION_OWNER: SYSTEM_MASTER
ROLE: SYSTEM_MASTER_INITIAL_AUDITOR
SYSTEM_MASTER_COORDINATOR: true
MODE: STAGE-1 APPLY
EVIDENCE_TARGET: CURRENT_CODEX_TASK_ONLY
AGENT_WORKSPACE_PATH: NOT_APPLICABLE
COMMUNITY_SHARING_PATH: <이 PC에서 보이는 조직 공용폴더 실제 local sync 경로; LOCAL-ONLY>
VERSION_WORKSPACE_PATH: <이 PC 사용자의 실제 결합 root 절대경로; LOCAL-ONLY>
OBSIDIAN_VAULT_PATH: SAME_AS_VERSION_WORKSPACE_PATH
CONTEXT_WORKSPACE_PATH: SAME_AS_VERSION_WORKSPACE_PATH
COMMUNITY_TEST_ITEM_REF: <승인된 비민감 기존 표본의 상대경로 또는 share reference>

0. `<...>`, PLACEHOLDER 또는 빈 필수값이 남으면 변경·판정 없이 `INPUT_TEMPLATE_INCOMPLETE`와 누락 필드만 반환하세요.
1. 링크의 파일명·version·last_reviewed를 확인하고 접근 불가이면 추측 없이 `ACCESS_BLOCKED`와 필요한 첨부/local 사본을 반환하세요.
2. 지정한 Community Sharing과 결합 root의 존재·역할, Version=Vault=Context를 read-only 확인하세요. 불일치면 `PATH_MISSING/PATH_ROLE_MISMATCH`로 HOLD하고 상위폴더 순회·새 root 생성을 금지합니다.
3. Runbook 6·8·8.2·9장 범위에서 이 PC만 점검하세요. 생성·저장·이동·download·pin·설정·권한 변경은 금지하고 결과는 현재 task에만 반환하세요.
4. 승인 문서 read, Obsidian 검색, OneDrive 무오류를 실제 검증하고 PASS/FAIL 근거를 남기세요. 다른 PC는 추정하지 마세요.
5. 표본의 상대경로·가용성·timestamp·size·cloud identity를 확인하세요. materialize하지 말고 한쪽에 없으면 `TEST_ITEM_NOT_COMMON/CLOUD_NOT_MATERIALIZED`로 분류하세요.
6. 절대경로 대신 다섯 역할 상태와 비민감 reference를 쓰고 다음 순서로 반환하세요.
   ACCESS / OBSERVED_DOCUMENT / ENDPOINT_BASELINE / FIVE_ROLE_MAPPING /
   FIVE_ROLE_PATH_STATUS / COMMUNITY_SHARING_IDENTITY / BUSINESS_REGRESSION /
   DEVICE_BOUND_EXCLUSION / BLOCKERS /
   UNEXPECTED_CHANGES / VERDICT(GO|HOLD) / NEXT_ACTION
증거가 없거나 필수값이 미확인이면 HOLD하며 다음 Stage를 실행하지 마세요.
```

다른 구성원 PC를 나중에 확대할 때만 `ENDPOINT: member-work-pc`, `ROLE: MEMBER_LOCAL_AUDITOR`, `SYSTEM_MASTER_COORDINATOR: false`로 바꾼다. 최초 두-PC 적용에서는 System Master 값으로 고정한다.

### Prompt B — 공용 OpenClaw PC의 Codex

```text
TCEU Runbook 실행 정본:
https://drive.google.com/file/d/1BewvqzpXnE2ef66GRJ-XS7OlErtKUmRw/view?usp=drive_link

ROLLOUT_ID: <ROLLOUT_ID>
STAGE_GATE: 1
ENDPOINT: shared-openclaw-pc
EXECUTION_OWNER: SYSTEM_MASTER
ROLE: SHARED_OPENCLAW_LOCAL_AUDITOR
MODE: STAGE-1 APPLY
EVIDENCE_TARGET: CURRENT_CODEX_TASK_ONLY
AGENT_WORKSPACE_PATH: <공용 OpenClaw local workspace 실제 절대경로; LOCAL-ONLY>
COMMUNITY_SHARING_PATH: <이 PC에서 보이는 조직 공용폴더 실제 local sync 경로; LOCAL-ONLY>
VERSION_WORKSPACE_PATH: NOT_APPLICABLE
OBSIDIAN_VAULT_PATH: NOT_APPLICABLE
CONTEXT_WORKSPACE_PATH: NOT_APPLICABLE
COMMUNITY_TEST_ITEM_REF: <Prompt A와 동일한 승인 표본 reference>

0. `<...>`, PLACEHOLDER 또는 빈 필수값이 남으면 변경·판정 없이 `INPUT_TEMPLATE_INCOMPLETE`와 누락 필드만 반환하세요.
1. 링크의 파일명·version·last_reviewed를 확인하고 접근 불가이면 추측 없이 `ACCESS_BLOCKED`와 필요한 첨부/local 사본을 반환하세요.
2. Agent Workspace와 Community Sharing의 존재·역할을 read-only 확인하세요. 불일치면 `PATH_MISSING/PATH_ROLE_MISMATCH`로 HOLD하고 상위폴더 순회·새 workspace 생성을 금지합니다.
3. Runbook 6·8·8.2·9장 범위에서 이 PC만 점검하세요. restart·config/skill/folder/sync/권한 변경과 materialization을 금지하고 결과는 현재 task에만 반환하세요.
4. 공용 노드의 service identity와 사람 역할을 분리하고, 공용 1+1 mapping 및 OpenClaw 참조경로를 분류하세요. `UNKNOWN`은 무변경입니다.
5. Gateway·Node, Telegram 연결, 승인 표본 metadata를 실제 검증하세요. 표본이 없으면 `TEST_ITEM_NOT_COMMON/CLOUD_NOT_MATERIALIZED`로 분류하고 다른 PC는 추정하지 마세요.
6. 절대경로 대신 다섯 역할 상태와 비민감 reference를 쓰고 다음 순서로 반환하세요.
   ACCESS / OBSERVED_DOCUMENT / ENDPOINT_BASELINE / OPENCLAW_HEALTH /
   FIVE_ROLE_PATH_STATUS / COMMUNITY_SHARING_IDENTITY /
   COMMUNITY_SHARING_DEPENDENCY_MAP / IDENTITY_BOUNDARY /
   BUSINESS_REGRESSION / BLOCKERS / UNEXPECTED_CHANGES /
   VERDICT(GO|HOLD) / NEXT_ACTION
증거가 없거나 필수값이 미확인이면 HOLD하며 다음 Stage를 실행하지 마세요.
```

최초 적용의 Prompt B는 System Master가 공용 OpenClaw PC에서 직접 실행한다. `tceu.manager`는 service identity일 수 있지만 사람 실행자나 System Master를 대신하지 않는다.

### Prompt C — System Master 업무 PC Codex의 통합 판정

Prompt 아래에 두 PC의 결과 전문을 각각 붙인다.

```text
TCEU Runbook 실행 정본:
https://drive.google.com/file/d/1BewvqzpXnE2ef66GRJ-XS7OlErtKUmRw/view?usp=drive_link

ROLLOUT_ID: USE_MATCHING_ROLLOUT_ID_FROM_A_AND_B_RESULTS
STAGE_GATE: 1
MODE: STAGE-1 INTEGRATE-ONLY

아래 MEMBER_WORK_PC_RESULT와 SHARED_OPENCLAW_PC_RESULT를 Runbook 8장과 8.2 기준으로 비교하세요.

규칙:
- 결과 placeholder가 남으면 `INPUT_TEMPLATE_INCOMPLETE`로 중단합니다.
- rollout·문서 version·endpoint·baseline·blocker를 대조하고, 적용 역할 `PATH_RESOLVED`, 개인 세 역할 same-root, 같은 Community cloud identity를 확인합니다. 절대경로는 복제하지 않습니다.
- 사실 충돌은 임의 선택하지 않고 `CONFLICT`, 한쪽 HOLD·접근불가·미검증은 통합 HOLD입니다. 부분 GO와 다음 Stage 실행을 금지합니다.
- 운영 변화는 시각·process/task·audit 변경목록으로 `ROLLOUT_CAUSED`, `EXPLAINED_BASELINE_DRIFT`, `UNRESOLVED`로 분류하며 근거 없는 중대 변화는 HOLD합니다.
- 시스템·파일·설정은 변경하지 않습니다.

반환 형식:
DOCUMENT_MATCH / ENDPOINT_COVERAGE / REQUIRED_DECISIONS /
CROSS_PC_CONFLICTS / UNEXPECTED_CHANGES /
INTEGRATED_VERDICT(GO|HOLD) / EXACT_NEXT_ACTION

--- MEMBER_WORK_PC_RESULT ---
<System Master 업무 PC Codex 결과 붙여넣기>

--- SHARED_OPENCLAW_PC_RESULT ---
<공용 OpenClaw PC Codex 결과 붙여넣기>
```

### Prompt D — System Master의 TCEU Runbook 반영

Prompt C의 통합 결과가 만들어지면 System Master가 아래 Prompt를 실행한다. Runbook을 직접 수정할 수 있는 Codex에는 승인된 local sync 경로를, 링크만 읽을 수 있는 Codex에는 Drive 파일 링크를 `RUNBOOK_EDIT_TARGET`으로 준다.

```text
RUNBOOK_EDIT_TARGET: https://drive.google.com/file/d/1BewvqzpXnE2ef66GRJ-XS7OlErtKUmRw/view?usp=drive_link
ROLLOUT_ID: USE_FROM_PREVIOUS_PROMPT_C_RESULT
STAGE_GATE: USE_FROM_PREVIOUS_PROMPT_C_RESULT
EXECUTION_OWNER: SYSTEM_MASTER
MODE: RUNBOOK-WRITEBACK APPLY
WRITEBACK_SCOPE: FRONTMATTER_VERSION_STATUS_AND_SECTION_17_ONLY
INTEGRATED_RESULT: USE_PREVIOUS_ASSISTANT_RESPONSE_IN_THIS_CODEX_TASK

직전 Prompt C 통합 결과를 TCEU Runbook의 17장 적용 기록에 반영하세요.

규칙:
1. 파일명·version·last_reviewed가 Prompt C와 다르면 `VERSION_MISMATCH`로 중단하세요.
2. 17.2는 교체하고 17.3은 같은 `rollout_id+gate` 행을 갱신하거나 없을 때만 추가하세요. PATH 상태·Cloud identity·회귀·blocker·운영 변화·verdict·다음 행동을 보존하세요.
3. 절대경로·ID·credential·원문·secret은 제외하고 HOLD/ROLLBACK도 그대로 기록하세요. 다음 Gate는 실행하지 마세요.
4. frontmatter 날짜·status·patch version과 17장만 바꾸고 구조를 검증하세요.
5. 원본 편집이 불가능하면 사본 없이 `RUNBOOK_WRITEBACK_BLOCKED`와 최소 UPDATE_PACKET을 반환하세요. 성공은 `RUNBOOK_UPDATED`로 끝내세요.

```

`RUNBOOK_UPDATED`가 확인되어야 해당 Gate의 문서 반영이 완료된다. `UPDATE_PACKET`만 나온 경우 System Master가 원본 Runbook을 직접 열 수 있는 Codex에서 같은 Prompt D를 다시 실행한다.

### Prompt H — 현재 통합 HOLD 해소 준비

이 Prompt는 **System Master 업무 PC의 Codex**에 그대로 입력한다. 시스템을 변경하는 Prompt가 아니라 최신 HOLD를 실행 가능한 작은 작업으로 분해한다. 이전 A·B·C 결과가 같은 task에 있으면 함께 사용하고, 없으면 Runbook 17장의 비민감 기록만 기준으로 시작한다.

```text
TCEU Runbook 실행 정본:
https://drive.google.com/file/d/1BewvqzpXnE2ef66GRJ-XS7OlErtKUmRw/view?usp=drive_link

ROLLOUT_ID: TCEU-ROLLOUT-2026-001
STAGE_GATE: 1-H
EXECUTION_OWNER: SYSTEM_MASTER
MODE: HOLD-REMEDIATION-PREPARE READ-ONLY

최신 Runbook의 frontmatter와 17.2·17.3을 읽고 현재 통합 HOLD를 해소할 순서를 준비하세요. 파일·설정·OneDrive materialization·OpenClaw runtime을 변경하지 말고 다음 Stage도 실행하지 마세요.

현재 blocker를 다음 네 종류로 분류하세요.
1. INPUT_OR_PATH: shared Agent Workspace, 공통 Community 시험 항목, Cloud identity·ACL, 승인자·변경창·RTO·rollback 담당
2. MEMBER_RUNTIME: Cloud `.agents`·`outputs`·node_modules·Junction, local Codex loader/runtime·임시 output, OneDrive·Obsidian·대표 Skill 회귀
3. SHARED_RUNTIME_BASELINE: Gateway bind·authentication, 구성원별 session/task 귀속, OpenClaw health
4. OPERATIONAL_DRIFT: queue, index, task-state와 같은 운영 중 변화

판정 규칙:
- Agent Workspace는 실행 중인 OpenClaw 설정에서 read-only 확인하고 새 workspace를 만들지 마세요.
- 시험 항목은 양쪽에 이미 존재해야 합니다. 한쪽 누락은 `TEST_ITEM_NOT_COMMON/CLOUD_NOT_MATERIALIZED`, Cloud identity는 `SAME_CLOUD_ITEM/IDENTITY_UNRESOLVED`로 기록하고 생성·download·pin과 실제 ID 공유를 금지합니다.
- Context node_modules 또는 Cloud `.agents` Junction 문제가 남으면 Prompt R입니다. local loader discovery·import·대표실행·cache·Context node_modules=0·Cloud `.agents` 잔존 항목=0·OneDrive 회귀가 필요하며 `outputs` 실제 이동은 Stage 2-B입니다.
- queue·index·task-state는 `ROLLOUT_CAUSED/EXPLAINED_BASELINE_DRIFT/UNRESOLVED`로 분류합니다. Gateway 외부노출·무인증이면 별도 변경창 전까지 HOLD합니다.
- System Master는 기술 rollback을 맡을 수 있으나 service account는 사람 승인자가 아닙니다. 업무 승인자·변경창·RTO를 확인하세요.

먼저 사용자에게 아직 필요한 값만 짧게 요청하세요. 이전 결과에 있는 값을 다시 묻지 마세요. 경로를 모르면 해당 endpoint Codex에 붙일 read-only resolver Prompt를 출력하세요.

모든 입력이 준비되면 다음을 반환하세요.
- BLOCKER_MATRIX: 항목 / observed·inferred / 해결 owner / 작업 / PASS evidence / 현재 상태
- RUN_PROMPT_R: YES 또는 NO와 근거
- MEMBER_RERUN_READY: YES 또는 NO
- SHARED_RERUN_READY: YES 또는 NO
- OPERATIONAL_DRIFT_CLASSIFICATION
- Prompt A와 Prompt B에 넣을 확정값 목록. 실제 절대경로는 화면에 반복하지 말고 local reference로 표시
- EXACT_SEQUENCE: 필요한 경우 Prompt R → A → B → C → D
- READY_TO_RERUN 또는 INPUTS_MISSING

HOLD를 GO로 바꾸거나 Runbook을 수정하지 마세요. 실제 A·B 재실행 결과만 다음 통합판정의 근거입니다.
```

### Prompt R — System Master 업무 PC의 Node Skill runtime 복구

이 Prompt는 Stage 1에서 Context Workspace 안의 `node_modules` Junction이 OneDrive sync를 방해한다고 확인된 경우에만 사용한다. 아래 다섯 경로를 해당 PC의 실제 local 값으로 채우고, 공통 Prompt Set Runbook의 `PROMPT-39` 전체 계약을 실행한다. 미치환값이 남아 있으면 실행하지 않는다.

```text
TCEU Runbook 실행 정본:
https://drive.google.com/file/d/1BewvqzpXnE2ef66GRJ-XS7OlErtKUmRw/view?usp=drive_link

실행할 공통 계약: ax-customer-llm-prompt-runbook.md의 PROMPT-39
ROLLOUT_ID: TCEU-ROLLOUT-2026-001
STAGE_GATE: 1-R
ENDPOINT: member-work-pc
EXECUTION_OWNER: SYSTEM_MASTER
MODE: APPLY

CONTEXT_WORKSPACE: <이 PC의 Context Workspace 실제 절대경로; LOCAL-ONLY>
SKILL_DISCOVERY_PATH: <이 프로젝트에서 Codex가 읽는 Skill 탐색 위치; LOCAL-ONLY>
LOCAL_SKILL_RUNTIME_ROOT: <이 PC의 Skill별 local runtime root; LOCAL-ONLY>
LOCAL_SHARED_NODE_RUNTIME: <이 PC의 임시 Node 작업용 shared runtime; LOCAL-ONLY>
LOCAL_AGENT_OUTPUTS_ROOT: <이 PC의 임시 output root; LOCAL-ONLY>

Prompt 전체에 `<...>`, PLACEHOLDER 또는 빈 필수값이 남아 있으면 변경하지 말고 INPUT_TEMPLATE_INCOMPLETE로 중단하세요.
PROMPT-39의 inventory, portable definition, 일반파일 loader, explicit ESM resolver, 무자동설치, Junction 제거 전 검증, rollback과 완료 보고를 빠짐없이 따르세요.
SKILL_DISCOVERY_PATH는 기본적으로 `%USERPROFILE%\.agents\skills` 사용자 범위를 사용하고 TCEU loader에는 `tceu-` prefix를 적용하세요. local discovery와 대표 Skill 실행이 PASS하기 전 Cloud `.agents`를 제거하지 마세요.
Cloud `outputs`는 내구성 있는 최종 결과와 임시 산출물로 분류만 하세요. 최종 결과는 기존 project에 유지하고 임시 output의 실제 local 전환은 Stage 2-B에서 별도 적용합니다.
Cloud workspace 안에 `node_modules`, package cache Junction·symlink 또는 장치별 절대 runtime 경로를 새로 만들지 마세요.
Context Workspace·Skill discovery·local runtime과 관련 없는 경로는 조사하거나 변경하지 마세요.
완료 후에도 다음 Stage를 실행하지 말고, System Master가 같은 rollout의 Prompt A→B→C→D를 다시 수행할 수 있는 checkpoint만 반환하세요.
```

Windows 후보는 `%USERPROFILE%\.agents\skills`, `%USERPROFILE%\.local\skill-runtimes`, `%USERPROFILE%\.local\agent-runtimes\shared-node`, `%USERPROFILE%\.local\agent-outputs\<workspace-id>`다. 실제 존재·쓰기 가능성과 회사 정책을 확인하기 전 자동 생성하지 않으며 절대경로는 공유 기록에 남기지 않는다.

## 6. Stage 2와 Stage 3은 이렇게 호출한다

Stage 1 통합 `GO` 뒤에도 Gate를 한 번에 하나씩 호출한다. 아래 공통 Prompt에서 `<STAGE_GATE>`와 `<ENDPOINT_ROLE>`만 바꾸고, 해당 Gate의 사전판정 참조를 붙인다.

```text
TCEU Runbook 실행 정본:
https://drive.google.com/file/d/1BewvqzpXnE2ef66GRJ-XS7OlErtKUmRw/view?usp=drive_link

ROLLOUT_ID: <ROLLOUT_ID>
STAGE_GATE: <2-A | 2-B | 2-C | 3-A | 3-B>
ENDPOINT: <member-work-pc | shared-openclaw-pc>
MODE: APPLY
PRIOR_GATE_RESULT: <직전 통합 GO 결과 참조>
AGENT_WORKSPACE_PATH: <shared-openclaw-pc이면 실제 local 경로, 아니면 NOT_APPLICABLE>
COMMUNITY_SHARING_PATH: <이 endpoint의 실제 local sync 경로>
VERSION_WORKSPACE_PATH: <member-work-pc이면 실제 결합 root, 아니면 NOT_APPLICABLE>
OBSIDIAN_VAULT_PATH: <member-work-pc이면 SAME_AS_VERSION_WORKSPACE_PATH, 아니면 NOT_APPLICABLE>
CONTEXT_WORKSPACE_PATH: <member-work-pc이면 SAME_AS_VERSION_WORKSPACE_PATH, 아니면 NOT_APPLICABLE>
LOCAL_CODEX_SKILLS_ROOT: <member-work-pc의 2-B이면 실제 local 경로, 아니면 NOT_APPLICABLE>
LOCAL_AGENT_OUTPUTS_ROOT: <member-work-pc의 2-B이면 실제 local 경로, 아니면 NOT_APPLICABLE>

Prompt 전체에 `<...>`, PLACEHOLDER 또는 빈 필수값이 남아 있으면 INPUT_TEMPLATE_INCOMPLETE로 중단하고 적용하지 마세요.
Runbook 8장의 지정된 Gate 하나만 실행하세요. 다른 Gate를 준비하거나 함께 적용하지 마세요.
다섯 역할 필드의 존재·same-root·not-applicable과 endpoint 역할 일치를 먼저 확인하고 다른 root로 범위를 넓히지 마세요. 실제 절대경로는 공유 evidence나 Telegram에 기록하지 마세요.
실행 전 pre-check, 허용 변경, 금지 변경과 rollback을 먼저 요약하고, 현재 승인범위가 불완전하면 HOLD하세요.
실행 후 8.2 형식으로 expected_changes, observed_changes, unexpected_changes, 업무 회귀, rollback 가능성과 GO/HOLD/ROLLBACK을 반환하세요.
정상 내부 문서 저장에는 반복 승인을 추가하지 말고, 외부발송·공유확대·권한·credential·파괴적 변경은 별도 승인 없이는 실행하지 마세요.
```

권장 순서는 `2-A → 통합판정·Prompt D → 2-B → 통합판정·Prompt D → 2-C → 통합판정·Prompt D → 3-A → 통합판정·Prompt D → 3-B → 최종판정·Prompt D`다. 앞 Gate가 통합 `GO`가 아니면 다음 Prompt를 보내지 않는다.

## 7. Telegram OpenClaw에는 언제 무엇을 보내는가

Telegram은 Runbook 전체를 읽히는 곳이 아니다. Stage 2의 승인된 shadow task부터 다음처럼 짧게 보낸다.

```text
[TCEU task]
rollout_id: <ROLLOUT_ID>
stage_gate: <2-C | 3-A>
task_id: <TASK_ID>
담당 Agent: 현재 Telegram 토픽에 이미 정의된 전담 Agent
승인된 입력: <Runbook이 허용한 task reference>
허용 작업: read / local draft / validation
금지 작업: external send / permission change / source overwrite
완료 기준: <한 문장>
반환: claimed / review_required / blocked 중 하나와 blocker·evidence reference
```

Telegram의 Agent 성격은 이 안내서에서 새로 정의하지 않는다. 현재 토픽의 승인된 정의와 [OpenClaw Parallel specialist lanes](https://docs.openclaw.ai/concepts/parallel-specialist-lanes)를 따른다.

## 8. 링크가 열리지 않을 때

LLM마다 Drive 브라우저 접근능력과 로그인 세션이 다르다. 파일 링크가 열리지 않는 경우 다음 순서만 허용한다.

1. 사람이 해당 PC의 브라우저에서 권한 있는 회사 계정으로 링크가 열리는지 확인한다.
2. 공유범위나 권한을 임의로 바꾸지 않는다.
3. Codex에 Runbook Markdown 파일 하나를 직접 첨부하거나, 이미 승인된 local sync 사본의 경로를 지정한다.
4. Codex가 읽은 파일명·`version`·`last_reviewed`를 반환하게 한다.
5. Prompt의 다섯 역할 필드를 모두 채우고 적용되는 local path가 해당 endpoint 역할과 일치하는지 확인한다. 경로를 모르면 추정하지 않고 사용자에게 그 PC의 실제 경로를 요청한다.
6. 링크·첨부·local 사본의 버전이 다르면 `VERSION_MISMATCH`로 `HOLD`한다.

다음 문장만으로 접근 실패를 안전하게 중단할 수 있다.

```text
이 링크를 직접 읽을 수 없으면 기억이나 일반지식으로 내용을 보완하지 말고 ACCESS_BLOCKED로 중단하세요. 필요한 입력은 최신 Runbook Markdown 첨부 또는 승인된 local sync 사본입니다.
```

## 9. 피해야 할 입력

| 입력 | 문제 | 대신 사용할 방식 |
|---|---|---|
| “이 링크 보고 전부 적용해줘” | PC·Stage·다섯 역할 경로·변경범위·rollback이 없음 | Prompt A 또는 B |
| 폴더 링크만 전달 | 어떤 파일이 정본인지 불명확 | Runbook 파일 링크 지정 |
| 한 PC의 Codex에 두 PC 점검 요청 | 다른 PC 상태를 추정하게 됨 | PC별 독립 점검 후 Prompt C |
| Telegram에 Runbook 전체 적용 요청 | 메시징·실행·판정 책임이 섞임 | Stage 2 이후 승인 task만 전달 |
| `Stage 1 GO` 뒤 “계속 진행” | 여러 Gate가 한 번에 실행될 수 있음 | 정확한 `2-A APPLY`부터 별도 호출 |

## 10. 한 줄 기억법

**System Master가 두 PC를 순서대로 확인하고, 자기 Codex에서 합쳐 Runbook에 기록한 뒤, OpenClaw는 승인된 task만 실행한다.**

## Related

- [TCEU 구성원 업무 PC와 공용 OpenClaw PC 연계 Runbook](https://drive.google.com/file/d/1BewvqzpXnE2ef66GRJ-XS7OlErtKUmRw/view?usp=drive_link)
- [공통 고객 적용 Prompt Set & Runbook](../../02-Implementation/ax-customer-llm-prompt-runbook.md)
- [패키지 시작 안내](../../README.md)
