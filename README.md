# GitHub Actions 실습 (01~05)

이 저장소는 GitHub Actions의 기본 트리거부터 `GITHUB_TOKEN`(내장 토큰) 활용, Variables/Secrets까지 단계적으로 실습하는 예제입니다.

> 워크플로 파일들은 `.github/workflows/` 아래에 있으며, 파일명 앞의 번호(`01~05`)가 실습 순서입니다.

## 사전 준비 (공통)

- GitHub 저장소에 push 되어 있어야 합니다.
- GitHub 저장소 상단 메뉴 **Actions**가 활성화되어 있어야 합니다.
- 워크플로 실행 로그 확인: **Actions → 해당 워크플로 → Run 클릭 → Jobs/Steps 로그 확인**

## 실습 목록

### 01. 수동 실행 + 입력값 (`workflow_dispatch`)

- 파일: `.github/workflows/01-dispatch-echo.yml`
- 의미
  - 가장 단순한 **수동 실행 워크플로**를 만들어, `inputs`를 받아 로그로 출력하는 흐름을 익힙니다.
- GitHub에서 할 일
  - **Actions → 01 워크플로 선택 → Run workflow**
  - 입력값 `message`를 넣거나 기본값으로 실행 후 `echo` 로그 확인

### 02. push 시 브랜치 조건 분기 (`if`, `github.ref`)

- 파일: `.github/workflows/02-push-branch-echo.yml`
- 의미
  - `push` 이벤트에서 브랜치가 `main`인지 아닌지에 따라 **Step 단위로 조건 분기**하는 방법을 익힙니다.
  - `github.ref`(예: `refs/heads/main`)와 `GITHUB_REF_NAME`(예: `feature/login`) 차이를 체감합니다.
- GitHub에서 할 일
  - `main`에 push 한 번, `main`이 아닌 브랜치에 push 한 번 해보고 로그 비교

### 03. 스케줄 실행 (`schedule`, `cron`)

- 파일: `.github/workflows/03-schedule-echo.yml`
- 의미
  - `cron`을 이용해 **주기 실행 워크플로**를 구성합니다.
  - `date -u`로 UTC 기준 실행 시각을 확인합니다.
- GitHub에서 할 일
  - 기다리면 자동 실행됩니다.
  - 참고: GitHub Actions `schedule`은 요청한 주기(예: 1분)대로 정확히 실행되지 않을 수 있고, 실질적으로는 지연되어 15~30분 간격처럼 보일 때도 있습니다.
  - 즉시 확인하고 싶으면 이 워크플로는 `workflow_dispatch`도 열어둔 상태입니다.

### 04. 혼합 트리거 + echo + 이슈 생성 (`workflow_dispatch/push/pull_request/schedule` + `GITHUB_TOKEN`)

- 파일: `.github/workflows/04-mixed-echo-and-issue.yml`
- 의미
  - 하나의 워크플로에서 여러 트리거를 함께 다룹니다.
  - 내장 토큰 `GITHUB_TOKEN`으로 **GitHub API(Issues)** 를 호출하여 이슈를 생성합니다.
  - `permissions`로 토큰 권한을 최소화하면서 `issues: write`가 필요함을 배웁니다.
- GitHub에서 할 일
  1. **Actions → 04 워크플로 → Run workflow**
     - `create_issue: true`로 실행하면 이슈가 생성됩니다.
  2. 생성된 이슈 확인: **Issues 탭**
- 추가 설명
  - `schedule`에서 이슈를 매번 만들면 이슈가 폭증합니다.
  - 그래서 기본값은 `CREATE_ISSUE_ON_SCHEDULE: "false"` 입니다(필요할 때만 `true`).

### 05. Variables + Secrets + Slack 웹훅 (push 시 Slack 전송)

- 파일: `.github/workflows/05-vars-secrets-slack-webhook.yml`
- 의미
  - **Variables**(`vars.*`)로 “민감하지 않은 값(메시지)”을 관리합니다.
  - **Secrets**(`secrets.*`)로 “민감정보(웹훅 URL)”를 안전하게 관리합니다.
  - Slack 전송은 `slackapi/slack-github-action` 액션으로 간결하게 구현합니다.
- GitHub에서 할 일(필수 설정)
  1. Repository Variable 추가
     - Settings → Secrets and variables → Actions → **Variables**
     - Name: `SLACK_MESSAGE`
     - Value: 예) `main에 push 되었습니다!`
  2. Repository Secret 추가
     - Settings → Secrets and variables → Actions → **Secrets**
     - Name: `SLACK_WEBHOOK_URL`
     - Value: Slack Incoming Webhook URL (`https://hooks.slack.com/services/...`)
  3. 이후 아무 브랜치든 push
     - Actions 로그에서 Slack 전송 Step 실행 확인
- 주의
  - Secret은 로그에 출력하지 마세요.
  - Webhook URL이 잘못되면 Slack 전송이 실패합니다.

## 자주 겪는 문제(트러블슈팅)

- **`schedule`이 1분마다 안 도는 것 같아요**
  - 정상일 수 있습니다. GitHub Actions 스케줄은 지연/배치 실행될 수 있습니다.
  - 즉시 확인하려면 `workflow_dispatch`가 있는 워크플로는 수동 실행으로 테스트하세요.

- **`Invalid workflow file` / YAML syntax 오류**
  - `run: echo "..."` 한 줄에서 문자열 안에 `": "`(콜론+공백) 조합이 있으면 YAML이 깨질 수 있습니다.
  - 안전하게 `run: |` 블록으로 작성하는 것을 권장합니다.

- **이슈 생성이 403으로 실패해요**
  - 워크플로에 `permissions: issues: write`가 필요한 케이스입니다.
  - (조직 정책/저장소 설정에 의해 `GITHUB_TOKEN` 권한이 제한될 수도 있습니다.)
