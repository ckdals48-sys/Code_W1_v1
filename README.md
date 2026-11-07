# Code_W1_v1
Repository 생성 1주차_v1

## 터미널 로깅 설정

`../.vscode/settings.json`에 터미널 프로필을 추가해, 이 워크스페이스(Test 폴더)에서 새 터미널을 열면 입력/출력이 자동으로 `../.vscode/terminal-logs/`에 저장되도록 구성했습니다.

- **PowerShell (Logged)**: `Start-Transcript` 기반, 기본 프로필
- **Git Bash (Logged)**: `../.vscode/bash-logging-init.sh` 초기화 스크립트 기반 (`exec > tee` + 명령 히스토리 기록)

## 작업 로그 (2026-11-04)

1. **저장소 클론**
   - `dev-workstation` 저장소 클론 시도 → 실패 (저장소 없음, 404)
   - `Code_W1_v1` 저장소 클론 → 성공

2. **PowerShell 터미널**
   - `app` 디렉터리 생성 (`mkdir app`)
   - `touch index.html` 시도 → PowerShell에는 `touch` 명령이 없어 실패

3. **Git Bash 터미널**
   - `index.html` 파일 생성 (`touch index.html`)
   - `index.html` → `backup.html` 복사
   - `backup.html` → `old.html` 이름 변경
   - `old.html` 삭제
   - 최종적으로 `index.html`만 빈 파일로 남음

### 관련 로그 파일

| 파일 | 내용 |
|---|---|
| `session_20261104_221632_10508.log` | PowerShell 세션 (mkdir/touch 포함, 외부 자동 기록) |
| `session_20261104_221743_10508.log` | 같은 세션의 내부(수동 시작) 기록 |
| `session_20261104_223123_12436.log` | PowerShell 세션 (대기 중, 명령 없음) |
| `session_20261104_manual_copy.log` | git clone 작업 (수동 복사로 저장) |
| `session_20261104_gitbash_manual_copy.log` | Git Bash 파일 작업 (수동 복사로 저장) |

## 작업 로그 (2026-11-07)

1. **저장소 상태 확인 (PowerShell)**
   - `git clone` 재시도 → 실패 (`Code_W1_v1` 폴더가 이미 존재)
   - `cd Code_W1_v1` → `git status` → `main` 브랜치 최신 상태, 변경사항 없음
   - `docker --version` → Docker 29.9.2 (build a82e528) 확인
   - `pwd` → 현재 경로 확인

2. **PowerShell 명령어 오류**
   - `ls -al` 시도 → 실패 (`Get-ChildItem`에는 `-al` 매개변수가 없음)
   - `Get -Childtiem -Force`, `Get -ChildItem -Force` 오타 시도 → 모두 명령어 인식 실패 (올바른 형식: `Get-ChildItem -Force`, 공백 없이 붙여써야 함)

3. **Claude Code 세션 로깅 설정**
   - Claude Code를 통해 실행하는 명령어 출력을 별도로 기록하기 위해 `../logs/` 디렉터리에 세션 로그 생성 시작 (`session_20261107_165812.log`)

### 관련 로그 파일 (2026-11-07)

| 파일 | 내용 |
|---|---|
| `session_20261107_165650_14580.log` | PowerShell 세션 (git clone 재시도, git status, docker 버전 확인, ls/Get-ChildItem 오타 오류) |
| `../../logs/session_20261107_165812.log` | Claude Code 세션을 통해 실행된 명령어 출력 기록 (신규) |
