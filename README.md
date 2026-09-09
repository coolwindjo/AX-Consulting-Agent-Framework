# AX Consulting Agent Framework

10인 이하 업체의 AX 사전진단부터 고객 PC 적용·검증·지식 환류까지 이어 주는 문서 패키지입니다.

## 다섯 기능 위치를 기억하는 법

| 공식 명칭 | 사람이 이해하기 쉬운 역할 | 기본 배치 원칙 |
|---|---|---|
| **Agent Workspace for Automation** | Agent가 자동화를 실행하는 작업실 | 실행 정본·skills·tasks는 로컬 유지 |
| **Cloud library for Community Sharing** | 조직 구성원이 함께 읽고 쓰는 공동 자료 선반 | 공동 원천·배포자료를 관리하되 Agent queue·runtime과 분리 |
| **Cloud workspace for Version control** | 사람 working copy 또는 Agent 변경 후보·승인본을 관리하는 Cloud 작업대 | 목적별로 분리하고 single-writer·승격·충돌·복원 규칙을 두며 Git·backup과 구분 |
| **Obsidian Vault for Organization** | 사람이 지식을 정리하고 읽는 서재 | 실행 runtime이 아닌 조직화·탐색 UI |
| **Context Workspace for Continuity** | LLM이 업무를 이어받는 데 필요한 맥락 꾸러미 | 고객·프로젝트별 최소 문맥과 인계상태 제공 |

문서에서는 위 공식 명칭을 풀어 쓴다. 다섯 위치가 물리적으로 일부 겹치더라도 자동화·공동공유·변경이력·정리·연속성이라는 책임은 구분한다. Cloud provider의 version history는 Git이나 검증된 backup을 대신하지 않는다.

## 어떤 문서를 읽어야 하나요?

| 목적 | 문서 | 한 문장 역할 |
|---|---|---|
| 고객에게 맞는 방안 선택 | [AX 분석 리포트](./01-Analysis/ax-consulting-agent-framework-report.md) | 고객 상황을 진단해 `S1~S4`, `OPTION`, `CLOUD` 조합을 결정합니다. |
| 고객 PC에서 안전하게 실행 | [고객 적용 Prompt Set & Runbook](./02-Implementation/ax-customer-llm-prompt-runbook.md) | 선택안을 LLM이 DRY-RUN·APPLY·검증·복구하도록 안내합니다. |
| CoolFam 링크를 LLM에 입력 | [CoolFam Runbook LLM 사용 안내서](./04-Field-Cases/CoolFam/coolfam-runbook-llm-usage-guide.md) | CoolBot PC·개인 PC의 Codex Prompt, 통합판정, Runbook 반영 순서를 안내합니다. |
| CoolFam CoolBot와 개인 PC 연계 | [CoolFam CoolBot와 조승현 개인 PC 연속성 Runbook](./04-Field-Cases/CoolFam/coolfam-coolbot-personal-pc-continuity-runbook.md) | `coolfam830` 소유 Bridge 1개를 `coolwind` 계정에 공유하고 개인 Direct Workspace의 Shortcut으로 같은 portable Context·Skill·task를 사용하도록 설계합니다. |
| TCEU 작업 실행 | [TCEU Runbook LLM 사용 안내서](./04-Field-Cases/TCEU/tceu-runbook-llm-usage-guide.md) | PC별 연결 확인·일상 작업·재공유 요청·Git 갱신을 안내합니다. |
| TCEU 공용 Agent 노드 적용 | [TCEU 구성원 업무 PC와 공용 OpenClaw PC 연계 Runbook](./04-Field-Cases/TCEU/tceu-two-pc-agent-integration-runbook.md) | CnwC 임시 열람, AgwA 실행, Agent 소유 CllC의 KB·결과 공유를 연결합니다. |
| TCEU 적용상태 확인 | [TCEU Runbook 현재 상태](./04-Field-Cases/TCEU/tceu-two-pc-agent-integration-runbook.md) | 관찰·사용자 설명·미검증을 구분하고 실제 연결 검증 결과를 유지합니다. |
| 그림의 별도 내보내기 파일 | `03-Visuals/` | 향후 PNG·SVG·PDF로 내보낸 도식을 보관합니다. 현재 도식 원본은 두 Markdown 문서의 Mermaid 블록입니다. |
| 폐기하지 않을 이전 판 | `90-Archive/` | 큰 구조 변경 전의 기준본만 날짜와 함께 보관합니다. |

```mermaid
flowchart LR
    A["고객 Discovery"] --> B["분석 리포트<br/>상황 진단·솔루션 선택"]
    B --> C["Prompt Set & Runbook<br/>DRY-RUN·APPLY·검증"]
    C --> D["고객 PC 현장 Evidence"]
    D --> E["재사용 가능한 Insight"]
    E --> B
```

CoolFam 사례와 TCEU 사례는 서로 독립된 field case입니다. CoolFam Runbook은 `coolfam830` Cloud library for Community Sharing의 `CoolFam-Agent-Bridge/` 원본을 `coolwind` 개인 Cloud workspace for Version control에 `Agent` Shortcut으로 연결합니다. Agent Workspace와 개인 Drive root 자체는 Sync하지 않고 Bridge의 portable Context·Skill·task·output만 동기화하며, Bridge 밖 background crawl은 금지합니다.

TCEU 사례는 **CnwC 임시 열람 → AgwA 작업 → CllC 결과 공유**를 기본 흐름으로 사용합니다. 개인 민감 원문과 이를 재현하는 파생물은 Agent 메모리·RAG·로그에 보관하지 않고, 접근이 회수되면 필요한 폴더만 다시 공유받습니다. 정상적인 비민감 결과는 저장·동기화로 인계합니다.

운영 기준은 Git의 [TCEU Runbook](./04-Field-Cases/TCEU/tceu-two-pc-agent-integration-runbook.md), 실행 방법은 [사용 안내서](./04-Field-Cases/TCEU/tceu-runbook-llm-usage-guide.md)에 유지합니다. CllC에는 검증된 열람 사본과 공유 가능한 업무 결과를 두며, 실제 계정·개인 공유 링크·기기 경로는 공개 Git에 기록하지 않습니다.

공용 Skill은 System Master가 CnwC에서 연결된 CllC 정본을 편집하면 자동 검증·배포를 거쳐 AgwA의 다음 요청에서 사용하도록 설계했습니다. 수동 복사·매번 승인 없이 쓰는 방법과 아직 미구현인 최초 연결 항목은 TCEU Runbook 5장에 있습니다.

기존 아이디어 중 개인 3역할의 같은-root 사용, portable core, Skill 3 Scope, single writer, 두 PC 검증·복귀, 전문 Agent의 권한 분리, 검증된 교훈 환류를 유지했습니다. 현재 상태 확인 → 비민감 시험 → 제한 실사용 순서로 필요한 기능만 검증하며, 문서 갱신을 시스템 적용 완료로 취급하지 않습니다.

## 기준본과 로컬 사본

- 이 패키지의 Markdown 문서가 사람이 지속 개선하는 **Cloud 기준본**입니다.
- OpenClaw 운영을 위해 공통 Prompt Set만 아래 로컬 경로에 **읽기용 사본**을 둡니다.
  - `/Users/coolbot_macmini/.openclaw/workspace/docs/manuals/openclaw-codex-framework-options.md`
- 수정 순서는 `Cloud 기준본 수정 → 검증 → Cloud에서 로컬로 단방향 갱신 → SHA-256 일치 확인`입니다.
- 로컬 사본을 독립적으로 수정해 두 개의 원본을 만들지 않습니다.
- OpenClaw의 Agent Workspace for Automation과 memory·skills·runtime은 계속 로컬에 둡니다. 이 패키지는 실행 workspace가 아닙니다.

## 개인정보·기기 종속 데이터 제외

- `DEVICE-BOUND`: credential, session, cookie, DB, browser profile, 기기 종속 runtime은 평문 Cloud에 저장하지 않습니다.
- `PERSONAL-DRIVE-ONLY`: 사용자·가족 자료의 원문·추출문·개인 파일명·공유 URL은 이 패키지에 기록하지 않습니다. 현재 요청과 그때 제공된 링크가 함께 있을 때만 one-shot으로 접근합니다.
- 고객 사례는 비식별 프로파일과 검증 결과만 기록합니다.

## 다음 개선 실행

```text
/evolve_agent_framework_report [이 폴더의 분석 리포트 절대경로]
```

옵션 적용 전에는 기본적으로 `DRY-RUN`을 사용합니다. 시스템 변경은 명시적인 `APPLY`와 각 승인 경계가 모두 충족된 경우에만 수행합니다.
