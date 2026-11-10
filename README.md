# Code_W1_v1

Linux / Docker / Git 실습 기술 문서. 본 문서는 `logs/` 디렉토리에 기록된 실제 터미널 입출력 로그를 분석하여 작성되었으며, 모든 수행 결과와 증거는 이 문서와 `logs/` 내 파일로 확인할 수 있다.

> 범례: ✅ 완료 · 🟡 진행 중(일부 완료) · ⬜ 미완료(TODO)

## 1. 프로젝트 개요

Windows + WSL2(Ubuntu) + Docker Desktop 환경을 구성하고, 터미널 기본 조작/권한 실습, Docker 설치 점검과 기본 운영 명령 수행, 컨테이너 실행 실습, 기존 Dockerfile 기반 커스텀 이미지 제작, 포트 매핑/접속 확인, 볼륨 영속성 검증, Git/GitHub 연동까지 일련의 실습을 수행하고 그 과정을 터미널 로그로 증명하는 것이 이 미션의 목표다.

모든 터미널 세션은 `~/.bashrc`에 추가한 자동 로깅 블록(`script -q -f`)을 통해 새 WSL 세션이 열릴 때마다 입력/출력이 그대로 `logs/*.log`에 기록되도록 구성했다 (상세는 [15. 로그 기록 체계](#15-로그-기록-체계-부록) 참고).

## 2. 실행 환경

| 항목 | 값 | 확인 명령 |
|---|---|---|
| Host OS | Windows 11 Pro 10.0.26300 | `cmd.exe /c ver` |
| WSL 배포판 | Ubuntu 26.04.1 LTS (Resolute Raccoon) | `cat /etc/os-release` |
| WSL 커널 | 6.18.54.1-microsoft-standard-WSL2 | `uname -r` |
| 쉘 | `/bin/bash` (WSL), PowerShell (Windows 측) | `echo $SHELL` |
| 터미널 | WSL(Ubuntu) 통합 터미널, TTY `/dev/pts/*` | `script` 메타데이터 |
| Docker | Docker Desktop, Client/Server 29.9.2 (build a82e528) | `docker --version`, `docker info` |
| Git | 2.53.0 | `git --version` |

```text
$ cat /etc/os-release
PRETTY_NAME="Ubuntu 26.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="26.04"
VERSION_CODENAME=resolute

$ uname -r
6.18.54.1-microsoft-standard-WSL2

$ cmd.exe /c ver
Microsoft Windows [Version 10.0.26300.9711]

$ docker --version
Docker version 29.9.2, build a82e528

$ git --version
git version 2.53.0
```

증거: [logs/wsl_manual_20261107_182732.log](logs/wsl_manual_20261107_182732.log) (사람이 읽기 쉬운 버전: [.readable.txt](logs/wsl_manual_20261107_182732.readable.txt))

## 3. 수행 항목 체크리스트

| 분류 | 항목 | 상태 | 비고 |
|---|---|---|---|
| 터미널 | 위치 확인/목록(숨김 포함)/이동/생성/복사/이동·이름변경/삭제 | ✅ | [5번](#5-터미널-조작-로그) 참고 |
| 터미널 | 파일 내용 확인, 빈 파일 생성 | ✅ | `cat`, `touch` |
| 권한 | 파일 1개 권한 변경 (전/후 비교) | ✅ | [6번](#6-권한-실습-및-증거) 참고 |
| 권한 | 디렉토리 1개 권한 변경 (전/후 비교) | ✅ | [6번](#6-권한-실습-및-증거) 참고 |
| Docker | `docker --version` | ✅ | [7번](#7-docker-설치-및-기본-점검) |
| Docker | `docker info` (데몬 동작 확인) | ✅ | [7번](#7-docker-설치-및-기본-점검) |
| Docker | 이미지 다운로드/목록 (`docker images`, `docker pull`) | ✅ | [8번](#8-docker-기본-운영-명령) |
| Docker | 컨테이너 실행/중지/목록 (`docker ps`, `docker ps -a`) | ✅ | [8번](#8-docker-기본-운영-명령) |
| Docker | 로그/리소스 확인 (`docker logs`, `docker stats`) | ✅ | [8번](#8-docker-기본-운영-명령) |
| 컨테이너 | `hello-world` 실행 성공 | ✅ | [9번](#9-컨테이너-실행-실습) |
| 컨테이너 | `ubuntu` 컨테이너 진입 후 `ls`/`echo` 수행 | ✅ | [9번](#9-컨테이너-실행-실습) |
| 컨테이너 | attach/exec 차이 관찰·정리 | ✅ | [9번](#9-컨테이너-실행-실습) |
| Dockerfile | 기존 베이스 기반 커스텀 이미지 빌드/실행 | 🟡 | [10번](#10-커스텀-이미지-제작-dockerfile-기반) — 정적 콘텐츠 작성 중, `docker build` 미실행 |
| 포트 | 포트 매핑 + 브라우저/`curl` 접속 증거 | ⬜ | 10번 작업 완료 후 진행 예정 |
| 볼륨 | 볼륨 생성/연결 + 컨테이너 삭제 전후 영속성 검증 | ⬜ | 미착수 |
| Git | `git config --list`, 기본 브랜치 설정 | 🟡 | [13번](#13-git-설정-및-github-연동) — user.name/email 미설정 확인됨 |
| GitHub | 로그인/저장소 연동 + 증거(스크린샷) | 🟡 | 원격 연동(`git remote -v`)은 확인됨, 로그인 스크린샷 미첨부 |
| 보안 | 민감정보 마스킹 점검 | ✅ | [14번](#14-보안-및-개인정보-보호) |

## 4. 검증 방법 및 결과 위치

| 검증 대상 | 사용 명령 | 결과 위치 |
|---|---|---|
| 자동 로깅 동작 여부 | 새 WSL 세션을 열고 `logs/`에 새 `.log` 파일이 생성·갱신되는지 확인 | [logs/wsl_20261107_182821_2526.log](logs/wsl_20261107_182821_2526.log) (정상 종료, `Script done` 포함) |
| 터미널 기본 조작 | `pwd`, `ls -al`, `cd`, `mkdir`, `cp`, `mv`, `rm`, `cat`, `touch` | [logs/wsl_manual_20261107_182732.log](logs/wsl_manual_20261107_182732.log) |
| 권한 변경(파일/디렉토리) | `chmod` 전/후 `ls -l` / `ls -ld` | 위 로그 + [logs/perm_directory_test.log](logs/perm_directory_test.log) |
| Docker 설치/데몬 | `docker --version`, `docker info` | [logs/wsl_20261107_182821_2526.log](logs/wsl_20261107_182821_2526.log) |
| Docker 운영 명령 | `docker images`, `docker pull`, `docker ps [-a]`, `docker logs`, `docker stats` | 동일 로그 |
| 컨테이너 실습 | `docker run hello-world`, `docker run -it ubuntu bash`, `docker attach`, `docker exec` | 동일 로그 + [logs/wsl_20261107_185353_3114.log](logs/wsl_20261107_185353_3114.log) |
| Git/GitHub 연동 | `git status`, `git remote -v`, `git config --list` | 본 문서 13번 섹션 (직접 실행 결과) |

가독성을 위해 터미널 제어 문자를 제거한 사본을 각 로그 파일명에 `.readable.txt` 접미사로 함께 두었다 (원본 `.log`가 1차 증거, `.readable.txt`는 열람용).

## 5. 터미널 조작 로그

작업 위치 확인, 숨김 파일 포함 목록, 이동, 생성, 복사, 이동/이름변경, 삭제, 내용 확인, 빈 파일 생성을 모두 수행했다.

```bash
$ pwd
/mnt/c/Users/ckdals485583/Test/Code_W1_v1

$ mkdir TEST
$ ls -al
drwxrwxrwx 1 ckdals485583 ckdals485583 4096 Nov  7 17:43 .
drwxrwxrwx 1 ckdals485583 ckdals485583 4096 Nov  7 16:58 ..
drwxrwxrwx 1 ckdals485583 ckdals485583 4096 Nov  7 17:34 .git
-rwxrwxrwx 1 ckdals485583 ckdals485583    6 Nov  7 17:34 .gitignore
-rwxrwxrwx 1 ckdals485583 ckdals485583   30 Nov  7 17:26 README.md
drwxrwxrwx 1 ckdals485583 ckdals485583 4096 Nov  7 17:43 TEST
...

$ cd TEST
$ touch TEST.txt                 # 빈 파일 생성
$ cat TEST.txt                   # 내용 확인 (빈 파일이므로 출력 없음)

$ cp TEST.txt TEST2.txt          # 복사
$ mkdir TEST-In                  # 디렉토리 생성
$ mv TEST2.txt TEST-In           # 이동
$ cd TEST-In
$ mv TEST2.txt TEST3.txt         # 이름변경
$ cp TEST3.txt TEST-Copy.txt     # 복사
$ rm TEST-Copy.txt               # 삭제
$ rm TEST3.txt                   # 삭제
$ cd -
/mnt/c/Users/ckdals485583/Test/Code_W1_v1/TEST
$ rmdir TEST-In                  # 디렉토리 삭제
$ ls -al
drwxrwxrwx 1 ckdals485583 ckdals485583 4096 Nov  7 17:50 .
drwxrwxrwx 1 ckdals485583 ckdals485583 4096 Nov  7 17:43 ..
-rwxrwxrwx 1 ckdals485583 ckdals485583    0 Nov  7 17:45 TEST.txt
```

전체 원문: [logs/wsl_manual_20261107_182732.log](logs/wsl_manual_20261107_182732.log)

## 6. 권한 실습 및 증거

### 6-1. 파일 권한 변경 (`/mnt/c` 경로 vs. 네이티브 Linux 경로)

처음에는 Windows 드라이브가 마운트된 `/mnt/c/...` 경로(DrvFs)에서 `chmod`를 시도했는데, 변경이 반영되지 않는 것을 관찰했다.

```bash
$ ls -l TEST.txt
-rwxrwxrwx 1 ckdals485583 ckdals485583 0 Nov  7 17:45 TEST.txt
$ chmod 644 TEST.txt
$ ls -l TEST.txt
-rwxrwxrwx 1 ckdals485583 ckdals485583 0 Nov  7 17:45 TEST.txt   # 변경 반영 안 됨 (DrvFs 특성)
```

동일한 실험을 WSL 네이티브 파일시스템(`~`, ext4)에서 반복하자 권한 변경이 정상적으로 반영되었다.

```bash
$ cd ~ && mkdir TEST && cd TEST
$ touch TEST.txt
$ ls -l TEST.txt
-rw-r--r-- 1 ckdals485583 ckdals485583 0 Nov  7 17:56 TEST.txt
$ chmod 755 TEST.txt
$ ls -l TEST.txt
-rwxr-xr-x 1 ckdals485583 ckdals485583 0 Nov  7 17:56 TEST.txt   # 644 -> 755 변경 확인
```

**관찰 정리:** Windows 드라이브를 마운트한 `/mnt/c/...` 경로는 DrvFs라 유닉스 권한 비트가 실질적으로 적용되지 않고 항상 `777`처럼 보인다. 반면 WSL 홈 디렉토리(`~`, ext4 네이티브)에서는 `chmod`가 정상적으로 동작한다. 실습/배포 목적의 파일은 네이티브 Linux 경로에 두는 것이 권한 제어 측면에서 안전하다.

### 6-2. 디렉토리 권한 변경

```bash
$ cd ~/TEST
$ mkdir -p PermDirTest
$ ls -ld PermDirTest
drwxr-xr-x 2 ckdals485583 ckdals485583 4096 Nov  7 19:28 PermDirTest
$ chmod 700 PermDirTest
$ ls -ld PermDirTest
drwx------ 2 ckdals485583 ckdals485583 4096 Nov  7 19:28 PermDirTest   # 755 -> 700 변경 확인
$ chmod 755 PermDirTest
$ ls -ld PermDirTest
drwxr-xr-x 2 ckdals485583 ckdals485583 4096 Nov  7 19:28 PermDirTest   # 원복
```

증거: [logs/perm_directory_test.log](logs/perm_directory_test.log)

## 7. Docker 설치 및 기본 점검

```bash
$ docker --version
Docker version 29.9.2, build a82e528

$ docker info
Client:
 Version:    29.9.2
 Context:    default
...
Server:
 Containers: 1
  Running: 0
  Paused: 0
  Stopped: 1
 Images: 1
 Server Version: 29.9.2
 Storage Driver: overlayfs
 Kernel Version: 6.18.54.1-microsoft-standard-WSL2
 Operating System: Docker Desktop
 OSType: linux
 Architecture: x86_64
 CPUs: 24
 Total Memory: 15.25GiB
```

`docker info`의 `Server:` 블록이 정상 출력되는 것으로 Docker 데몬이 동작 중임을 확인했다.

증거: [logs/wsl_20261107_182821_2526.log](logs/wsl_20261107_182821_2526.log)

## 8. Docker 기본 운영 명령

```bash
# 이미지 목록
$ docker images
IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest   c567b231f656       25.9kB         9.49kB

# 이미지 다운로드
$ docker pull ubuntu
Status: Downloaded newer image for ubuntu:latest

# 컨테이너 목록
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

$ docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED              STATUS                      PORTS     NAMES
197fc43d7481   ubuntu        "bash"     About a minute ago   Exited (0) 12 seconds ago             pedantic_taussig
4a5701c7b5ac   hello-world   "/hello"   7 minutes ago        Exited (0) 7 minutes ago              recursing_yonath

# nginx 컨테이너 실행 후 로그/리소스 확인
$ docker run -d --name TEST_nginx nginx
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
07bce4820056   nginx     "/docker-entrypoint.…"   16 seconds ago   Up 16 seconds   80/tcp    TEST_nginx

$ docker logs TEST_nginx
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/11/07 09:41:08 [notice] 1#1: nginx/1.31.8
2026/11/07 09:41:08 [notice] 1#1: start worker processes
...

$ docker stats TEST_nginx
CONTAINER ID   NAME         CPU %     MEM USAGE / LIMIT     MEM %     NET I/O         BLOCK I/O     PIDS
07bce4820056   TEST_nginx   0.00%     22.23MiB / 15.25GiB   0.14%     1.17kB / 126B   0B / 12.3kB   25
```

증거: [logs/wsl_20261107_182821_2526.log](logs/wsl_20261107_182821_2526.log)

## 9. 컨테이너 실행 실습

### 9-1. hello-world

```bash
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

### 9-2. ubuntu 컨테이너 진입 및 명령 수행

```bash
$ docker run -it ubuntu bash
root@197fc43d7481:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
root@197fc43d7481:/# echo "Hello Docker"
Hello Docker
root@197fc43d7481:/# cat /etc/os-release
PRETTY_NAME="Ubuntu 26.04.1 LTS"
...
root@197fc43d7481:/# exit
```

### 9-3. attach / exec / start 차이 관찰

실제로 `docker run -it --name Test-ubuntu ubuntu bash`로 컨테이너의 메인 프로세스를 `bash`로 띄운 뒤 아래 순서로 실험했다.

```bash
$ docker ps
CONTAINER ID   IMAGE    COMMAND   CREATED         STATUS         PORTS     NAMES
b4390b3f3d13   ubuntu   "bash"    4 minutes ago   Up 2 minutes             Test-ubuntu

$ docker attach Test-ubuntu
root@b4390b3f3d13:/# exit
exit
$ docker ps          # 컨테이너가 사라짐 (메인 프로세스가 종료되어 컨테이너 자체가 정지)
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

$ docker start Test-ubuntu
Test-ubuntu
$ docker exec -it Test-ubuntu bash
root@b4390b3f3d13:/# exit
exit
$ docker ps          # 컨테이너가 계속 실행 중 (exec는 별도 프로세스이므로 메인 프로세스에 영향 없음)
CONTAINER ID   IMAGE    COMMAND   CREATED         STATUS          PORTS     NAMES
b4390b3f3d13   ubuntu   "bash"    7 minutes ago   Up 23 seconds             Test-ubuntu
```

**관찰 정리:**
- `docker attach`는 컨테이너의 **메인(PID 1) 프로세스**에 다시 연결한다. 메인 프로세스가 쉘이었다면, 그 쉘에서 `exit`하는 순간 메인 프로세스가 종료되어 **컨테이너 자체가 정지**한다.
- `docker exec -it ... bash`는 이미 실행 중인 컨테이너 안에 **별도의 새 프로세스**를 띄우는 것이라서, 그 쉘에서 `exit`해도 메인 프로세스와 컨테이너는 영향을 받지 않고 계속 실행된다.
- 따라서 컨테이너를 "유지"하면서 들여다보고 싶다면 `exec`를, 메인 프로세스 자체와 상호작용(및 그 종료로 컨테이너를 멈추는 것)이 목적이라면 `attach`를 사용해야 한다.

증거: [logs/wsl_20261107_182821_2526.log](logs/wsl_20261107_182821_2526.log), [logs/wsl_20261107_185353_3114.log](logs/wsl_20261107_185353_3114.log)

## 10. 커스텀 이미지 제작 (Dockerfile 기반)

**상태: 🟡 진행 중 — 아직 `docker build`/실행까지 완료되지 않음.**

- 선택한 방식: **(A) 웹 서버 베이스 이미지 활용** — `nginx` 공식 이미지를 베이스로, 정적 콘텐츠(`index.html`)만 교체하는 방식으로 진행 중.
- 커스텀 포인트(계획): `nginx` 기본 정적 페이지를 직접 작성한 `index.html`로 교체하여 "내가 만든 페이지가 컨테이너로 서빙되는 것"을 보여준다.
- 현재까지 진행된 작업:
  - 작업 디렉토리 `~/Code_W1_v1/Docker-Web/App` 생성
  - `index.html` 초안 작성 (여러 차례 `heredoc` 입력 중 오타가 발생하여 재작성 중, 현재 내용은 완성본이 아님)
  - `Dockerfile` 작성, `docker build`, 컨테이너 실행, 포트 매핑, 접속 확인은 **아직 수행하지 않음**
- 참고: 이 작업은 WSL 홈 디렉토리 사본(`~/Code_W1_v1`, `cp -r /mnt/c/.../Code_W1_v1 ~/`로 만든 복사본)에서 진행되고 있어, **Git 저장소 경로(`/mnt/c/.../Code_W1_v1`)로 옮겨 커밋해야** 최종 제출물에 포함된다.

**다음 단계(TODO):**
1. `~/Code_W1_v1/Docker-Web/App/index.html` 내용을 깔끔하게 재작성
2. 같은 위치(또는 저장소 경로)에 `nginx:alpine` 베이스의 `Dockerfile` 작성 (`COPY index.html /usr/share/nginx/html/index.html`)
3. `docker build -t custom-nginx .` 로 빌드
4. `docker run -d -p 8080:80 custom-nginx` 로 포트 매핑 실행
5. `curl http://localhost:8080` 또는 브라우저 접속 화면을 캡처해 11번 섹션에 첨부
6. 결과물을 저장소 경로로 옮기고 `git add/commit/push`

## 11. 포트 매핑 및 접속 증거

⬜ **미완료.** 10번 작업(커스텀 이미지 빌드/실행)이 끝난 뒤 `-p <host>:<container>` 매핑과 `curl`/브라우저 접속 화면을 이 섹션에 추가할 예정이다.

<img width="1982" height="761" alt="image" src="https://github.com/user-attachments/assets/74513ef1-fe4c-4793-906b-d3ca7e576c49" />
<img width="1574" height="463" alt="image" src="https://github.com/user-attachments/assets/df425147-4121-4ab8-aa00-534ce6c8496e" />


## 12. Docker 볼륨 영속성 검증

⬜ **미착수.** 다음 절차로 진행할 예정이다.

```bash
# (예정) 볼륨 생성 및 연결
docker volume create my-vol
docker run -d --name vol-test -v my-vol:/data busybox sleep 3600
docker exec vol-test sh -c "echo hello > /data/test.txt"

# (예정) 컨테이너 삭제 전/후 데이터 확인
docker rm -f vol-test
docker run --rm -v my-vol:/data busybox cat /data/test.txt   # 삭제 후에도 데이터가 남아있는지 확인
```

## 13. Git 설정 및 GitHub 연동

```bash
$ git status
On branch main
Your branch is up to date with 'origin/main'.
nothing to commit, working tree clean

$ git remote -v
origin  https://github.com/ckdals48-sys/Code_W1_v1.git (fetch)
origin  https://github.com/ckdals48-sys/Code_W1_v1.git (push)

$ git config --list
core.autocrlf=true
pull.rebase=false
credential.helper=manager
init.defaultbranch=master
remote.origin.url=https://github.com/ckdals48-sys/Code_W1_v1.git
branch.main.remote=origin
branch.main.merge=refs/heads/main
```

- 저장소는 GitHub(`ckdals48-sys/Code_W1_v1`)와 정상적으로 연동되어 있고, `main` 브랜치가 원격과 동기화되어 있다(`git status`).
- 과거 커밋 히스토리(`git log`)에는 `ckdals48-sys <ckdals48@gmail.com>` 및 `unknown <ckdals485583@gnia.kr>` 두 가지 작성자 정보가 혼재되어 있고, **현재 이 머신에는 `user.name`/`user.email`이 전역/로컬 어디에도 설정되어 있지 않음**을 확인했다(`git config --list`에 두 값이 없음).
- 🟡 **TODO:** `git config --global user.name "<이름>"`, `git config --global user.email "<이메일>"`로 사용자 정보를 명시적으로 설정해야 요구사항을 완전히 만족한다.
- ⬜ **TODO:** GitHub 로그인/연동을 보여주는 스크린샷 첨부 (현재는 CLI 자격 증명으로만 연동되어 있음을 `git remote -v`/`git status`로 간접 확인).

## 14. 보안 및 개인정보 보호

- 본 문서와 `logs/` 내 모든 로그를 검토한 결과, 토큰/비밀번호/개인키/인증 코드 등 민감정보가 포함된 명령·출력은 발견되지 않았다.
- `git config --list`에 출력되는 `credential.helper=manager`는 자격 증명 저장 방식(Windows 자격 증명 관리자 연동)만을 나타내며 실제 비밀값은 포함하지 않는다.
- 향후 토큰/비밀번호가 포함될 수 있는 명령(예: `docker login`, `git clone` with PAT)을 실행할 경우, 해당 로그 라인은 커밋 전에 마스킹(`****`) 처리할 것.

## 15. 로그 기록 체계 (부록)

모든 WSL 대화형 세션은 `~/.bashrc`에 추가된 아래 블록을 통해 `script` 명령으로 자동 기록된다 (세션 시작 시 자동 실행, 중복 래핑 방지를 위해 `IN_SCRIPT_LOG` 가드 사용).

```bash
# --- Auto session logging (added by Claude Code) ---
if [ -z "$IN_SCRIPT_LOG" ] && [[ $- == *i* ]]; then
    export IN_SCRIPT_LOG=1
    LOGDIR="/mnt/c/Users/ckdals485583/Test/Code_W1_v1/logs"
    mkdir -p "$LOGDIR"
    LOGFILE="$LOGDIR/wsl_$(date +%Y%m%d_%H%M%S)_$$.log"
    exec script -q -f "$LOGFILE"
fi
# --- end auto session logging ---
```

`logs/` 디렉토리 구성:

| 파일 | 설명 |
|---|---|
| `wsl_manual_20261107_182732.log` | 자동 로깅 적용 "이전"에 열려 있던 세션을 사용자가 직접 붙여넣어 보존한 로그 (터미널 기본 조작 + 권한 실습 일부 포함) |
| `wsl_20261107_182821_2526.log` | 자동 로깅 적용 후 첫 완결 세션 (Docker 설치 점검, 운영 명령, hello-world/ubuntu/nginx 컨테이너 실습) |
| `wsl_20261107_185353_3114.log` | 커스텀 이미지(`Docker-Web/App`) 작업 중인 세션 (진행 중) |
| `perm_directory_test.log` | 디렉토리 권한 변경 실습 (전/후) |
| `*.readable.txt` | 위 `.log` 파일에서 터미널 제어 문자를 제거한 열람용 사본 |

각 `.log`는 `script`의 표준 typescript 형식이며 터미널 제어 시퀀스(커서 이동, 색상 등)가 포함되어 있어 GitHub에서 그대로 보면 다소 지저분하게 보일 수 있다 — 이 경우 같은 이름의 `.readable.txt`를 참고할 것.
수정확인
