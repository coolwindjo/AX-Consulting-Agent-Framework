---
title: TCEU Runbook 사용 안내
type: field-case-usage-guide
status: partially-applied-promotion-pending
version: 3.0
created: 2026-08-20
last_reviewed: 2026-09-19
---

# TCEU Runbook 사용 안내

기준은 [운영 Runbook](./tceu-two-pc-agent-integration-runbook.md)이다. 기존 CllC·KB·receiver는 재사용한다. 현재는 **후보 준비 → System Master 검토 → 승인본 배포 → 멤버별 확인**을 연결하는 단계다. 과거 설치를 처음부터 반복하지 않는다.

## 1. 평소 KB·결과 이용

System Master PC의 기존 CllC 진입점에서 Knowledge·Artifacts를 연다. 최신 version·manifest·hash·갱신 시각과 실제 열람을 확인한다. 문제 시 중복 실행보다 원천→생성→Cloud→열람 중 실패한 단계부터 확인한다.

TCEU Manager PC에 입력:

```text
TCEU Runbook에 따라 현재 요청의 보관·공유 허용 범위만 처리하세요.
완성 결과는 기존 CllC Knowledge 또는 Artifacts 구조에 저장하세요.
개인 원천 접근이 필요하면 목적·폴더 별칭·필요 기간을 제시해 임시 Viewer를 요청하세요.
민감 원문·파생물·개인 경로·공유 토큰을 AgwA memory/RAG/로그나 CllC에 저장하지 마세요.
로컬 저장·Cloud 반영·System Master 열람을 분리 보고하고 다음 PC용 Prompt를 제공하세요.
```

새 개인 공유 링크는 현재 요청에만 사용한다. 이번 설정 작업에 개인 원문은 필요하지 않다.

## 2. 개인 Skill의 공용 후보 제출

실행 PC: 작성자의 개인 PC. 개인 Skill의 준비된 비민감 제출본만 대상으로 한다.

```text
이 개인 Skill을 TCEU 공용 검토 후보로 준비하세요.
개인 원본은 유지하고, 사용자 지정 Skill만 별도 비민감 제출본으로 만드세요.
이름은 tceu-shared- 접두어로 정하고 상대 참조·목적·필요 권한·의존성·지원 OS를 확인하세요.
원문·개인 경로·자격증명·개인 memory·실행 cache는 포함하지 마세요.
기존 Operations/Skill-Promotion-Setup 도구로 ZIP·파일 hash를 고정하고 Skill-Candidates에 제출하세요.
검토 대기함을 loader나 배포 watcher에 등록하지 마세요.
패턴 검사 통과를 사람 승인으로 해석하지 말고 PENDING_SYSTEM_MASTER_REVIEW로 남기세요.
후보 revision/hash·합성 표본 결과·제한사항과 System Master 검토용 Prompt를 제공하세요.
```

도구는 실제 후보의 개인정보 적합성을 보증하지 않는다. 제출 권한이 없으면 기존 허용 경로로 전달할 검토 묶음만 준비하고 CllC 전체 공유를 요구하지 않는다.

## 3. System Master 검토와 대상 확정

실행 PC: System Master PC.

```text
TCEU Runbook v3.0과 현재 CllC 인계문서를 읽고 공용 Skill 승격을 준비하세요.
기존 CllC를 재사용하세요. 초기 대상은 CllC 인계문서에 지정된 Sales 멤버 1명입니다.
이미 지정된 계정을 다시 묻지 말고 해당 멤버의 사용 도구·OS와 실제 권한을 확인하세요.
추가 멤버는 추후 사용자 지정 전까지 포함하지 마세요.
Skill 배포 Viewer와 후보 제출 쓰기 권한을 분리하고 CllC의 업무 KB 전체를 멤버에게 공유하지 마세요.
현재 원격 owner·직접 권한·링크·상속을 확인하고 구체적인 변경 대상만 기록하세요.

지정 후보의 목적·전체 파일·민감정보·도구 권한·의존성·지원 환경·합성 결과를 검토하세요.
정확한 bundle hash와 revision을 제시해 사람 System Master가 해당 내용을 확인하도록 하세요.
Agent가 승인 여부를 임의 작성하거나 아직 검토하지 않은 후보를 Shared-Skills에 넣지 마세요.
승인받은 후 내용이 바뀌면 새 후보로 다시 검토하세요.

현재 수신기에 mtime 기반 원복 선택 문제가 있습니다.
승인 revision을 명시적으로 선택하는 게시/수신 계약이 검증되기 전 새 후보를 live 배포하지 마세요.
반환: 검토 후보·hash / 사람 결정 또는 검토 대기 / 대상 환경·ACL / 필요한 변경 / 다음 PC·Prompt.
```

최초 승인 후 멤버마다 다시 사람 승인을 받는 절차를 만들지 않는다. **새 내용의 승격 결정은 System Master가 한 번 하고, 수신은 각 환경에서 자동화**한다.

## 4. 구현과 수신 검증

실행 PC: TCEU Manager PC. System Master의 대상·검토 결과를 인계받은 후 진행한다.

```text
TCEU 공용 Skill의 승인본 선택·전달을 기존 구현에 연결하세요.
System Master가 검토한 정확한 revision/hash와 실제 게시 경로의 쓰기 권한을 먼저 확인하세요.
후보 파일의 approved 값이나 요청자의 자기 역할 주장만으로 승인자를 인증하지 마세요.
멤버 계정·도구가 미확정이면 해당 대상의 권한·loader 설치는 보류하세요.

기존 불변 패키지를 유지하고, 최신 파일 mtime 대신 명시적 게시 선택을 사용하도록 수정하세요.
이전 revision 재선택, 미승인 후보, 후보 수정, 부분 전송, hash 불일치, 중복 수신을 격리 시험하세요.
검증된 선택이 없을 때 과거 임시 패키지로 자동 fallback하지 마세요.
실행 중인 요청 보호와 receiver 전환의 안전한 경계를 확인한 뒤 live 변경하세요.
Gateway 정지·receiver 수동 lock을 일반 execution lease PASS로 확대하지 마세요.

각 환경에서 같은 revision/hash·유효 Skill 목록·새 요청의 비민감 호출 결과를 대조하세요.
오프라인·의존성 부족·연결 미완료 멤버를 별도 표시하고 전체 READY를 추정하지 마세요.
원복 결과와 다음 PC용 Prompt를 기록하세요.
```

현재 준비된 후보 도구는 승인·게시·receiver 변경을 수행하지 않는다. 위 단계는 앞으로 수행할 구현이며 이번 문서 개정으로 완료된 것이 아니다.

## 5. System Master 송신기 복구

실행 PC: System Master PC. 마지막 실패는 09-10 인계 기준이며 최신 수정 여부부터 확인한다.

```text
기존 TCEU Shared Skills Packager의 자동 시작 실패를 이어서 진단·수정하세요.
기존 Action·인수·계정·작업 디렉터리·실제 Python 경로와 수동 성공 환경을 대조하세요.
lifecycle 코드 self-test 성공과 실제 예약 경로의 로그 기록은 구분하세요.
Operational 로그 활성화는 과거 관리자 권한 부족으로 미완료였습니다.
현재 활성 상태와 쓰기를 확인하고 권한 부족이면 사용자가 실행할 최소 조치만 안내하세요.

실제 원인에 필요한 최소 수정과 복귀 방법을 준비하고 예약 작업 경로로 수동 시험하세요.
준비 실패 상태로 재로그온을 반복하지 마세요.
준비 통과 후 사용자가 직접 재로그온하도록 안내하고 자동 로그오프·재부팅은 하지 마세요.
재로그온 후 수동 task 시작 없이 trigger·wrapper start·단일 watcher·동일 PID 10분 지속을 확인하세요.
승인된 비민감 시험본의 패키징과 원복을 검증하고 수동/자동 결과를 분리하세요.
후보를 직접 live 경로에 넣거나 검토 전 변경을 배포하지 마세요.
실제 시각·revision/hash·원복·남은 일과 TCEU Manager 후속 Prompt를 제공하세요.
```

## 6. 결과 반영

실행 PC: Git checkout이 있는 TCEU Manager PC.

```text
$evolve-agent-framework-report
이번 실행의 비민감 사실을 TCEU 문서에 반영하고 로컬 커밋을 만드세요.
사용자 설명·직접 관찰·과거 인계를 구분하고 현재 blocker와 다음 행동만 유지하세요.
```

Git 게시가 필요할 때:

```text
$evolve-agent-framework-report sync
```

게시 후 CllC 사본을 비교·갱신하고 로컬 hash와 Cloud 반영을 구분한다. 공유 URL·계정·장치 경로·원시 로그는 공개 Git에 넣지 않는다. 양쪽 PC의 인계는 한 writer씩 갱신한다.
