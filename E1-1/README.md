# E1-1 — 내 컴퓨터에 개발자용 '작업실' 꾸미기

## 1) 프로젝트 개요
터미널·Docker·Git을 직접 세팅하고, 실행 결과(로그·접속·데이터 유지)를 이 문서에 텍스트로 남긴다.

## 2) 실행 환경
| 항목 | 값 | 확인 명령 |
|---|---|---|
| 호스트 OS | Windows 11 + WSL2 | - |
| 터미널 OS | Ubuntu 26.04 LTS (WSL2) | `cat /etc/os-release` |
| Shell | bash 5.3.9 | `bash --version` |
| Docker | Client 29.6.2 (Docker Desktop) | `docker --version` |
| Git | 2.53.0 (WSL) / 2.55.0 (Windows) | `git --version` |

## 3) 수행 체크리스트
- [x] 터미널 기본 조작 → [로그](logs/01-terminal-permission.txt), 아래 4절
- [x] 권한 실습 (파일 1개, 디렉토리 1개 chmod 전/후) → 아래 4절
- [x] Git 사용자·기본 브랜치 설정 확인 → 아래 5절
- [ ] Docker 설치/점검 (`docker --version` 완료, `docker info` 데몬 미기동 → 트러블슈팅 1)
- [ ] Docker 운영 (images, ps -a, logs, stats)
- [ ] hello-world, ubuntu 진입 (ls, echo), attach/exec 차이
- [ ] 커스텀 이미지 (A: nginx / B: ubuntu·alpine)
- [ ] 포트 매핑 접속 (주소창 포함 화면 또는 curl)
- [ ] 바인드 마운트 (호스트 변경 전/후)
- [ ] 볼륨 영속성 (컨테이너 삭제 전/후)
- [ ] VSCode GitHub 연동 증거
- [ ] 트러블슈팅 2건 이상 (현재 2건 기록, 해결 확인 후 완료)
- [x] 민감정보 마스킹 (이메일 일부 마스킹)

보너스: [ ] Compose 기초 · [ ] Compose 멀티 · [ ] Compose 운영 명령 · [ ] 환경변수 · [ ] SSH 키

## 4) 수행 로그 — 터미널 조작 + 권한 (WSL Ubuntu)
전체 원본: [logs/01-terminal-permission.txt](logs/01-terminal-permission.txt)

핵심 발췌:
```
$ mkdir -p ~/codyssey/practice
$ cd ~/codyssey/practice && pwd
/home/nhs/codyssey/practice
$ touch .hidden empty.txt
$ ls -la          # 숨김 파일(.hidden) 포함 목록
$ cp note.txt docs/note-copy.txt
$ mv docs/note-copy.txt docs/note-renamed.txt
$ realpath docs/note-renamed.txt      # 절대경로
/home/nhs/codyssey/practice/docs/note-renamed.txt   # 상대경로는 docs/note-renamed.txt
```
권한 변경 전/후 (파일 `perm.txt`, 디렉토리 `permdir`):
```
전: 644 -rw-r--r-- perm.txt   /  755 drwxr-xr-x permdir
$ chmod 600 perm.txt && chmod 700 permdir
후: 600 -rw------- perm.txt   /  700 drwx------ permdir
$ chmod +x perm.txt  →  755 -rwxr-xr-x perm.txt
```
해석: 숫자는 소유자·그룹·기타 순서로 r=4, w=2, x=1을 더한 값이다. 644는 소유자 rw, 나머지 r. 755는 소유자 rwx, 나머지 r-x.

## 5) Git 설정
```
$ git config --global --list   (발췌)
user.name=linkcontent7-huisun
user.email=li***@gmail.com
init.defaultbranch=main
```
GitHub 로그인은 `gh auth status`로 linkcontent7-huisun 계정 확인.

## 6) 트러블슈팅
**1. Docker 데몬 연결 실패**
- 문제: `docker info`의 Server 항목이 `failed to connect to the docker API at npipe:////./pipe/dockerDesktopLinuxEngine`
- 원인 가설: Docker Desktop(엔진)이 실행되지 않았다.
- 확인: `docker --version`은 성공(클라이언트만 설치), `docker ps -a`도 같은 연결 오류 → 클라이언트는 정상, 엔진만 꺼져 있음.
- 해결: (Docker Desktop 실행 후 갱신 예정)

**2. WSL 스크립트 경로가 깨짐**
- 문제: Git Bash에서 `wsl bash /mnt/c/...` 실행 시 `C:/Program Files/Git/mnt/c/...: No such file`
- 원인 가설: Git Bash가 `/`로 시작하는 인자를 Windows 경로로 자동 변환한다.
- 확인: 경로 앞에 `C:/Program Files/Git`이 붙은 오류 메시지.
- 해결: `MSYS_NO_PATHCONV=1`을 붙여 변환을 끈 뒤 성공.
