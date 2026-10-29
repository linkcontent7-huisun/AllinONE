# E1-1 — 내 컴퓨터에 개발자용 '작업실' 꾸미기

## 1) 프로젝트 개요
터미널·Docker·Git을 직접 세팅하고, 실행 결과(로그·접속·데이터 유지)를 이 문서에 텍스트로 남긴다.

## 2) 실행 환경
| 항목 | 값 | 확인 명령 |
|---|---|---|
| 호스트 OS | Windows 11 + WSL2 | - |
| 터미널 OS | Ubuntu 26.04 LTS (WSL2) | `cat /etc/os-release` |
| Shell | bash 5.3.9 | `bash --version` |
| Docker | 29.9.2 (Docker Desktop, 엔진 동작 확인) | `docker --version`, `docker info` |
| Git | 2.53.0 (WSL) / 2.55.0 (Windows) | `git --version` |

## 3) 수행 체크리스트
- [x] 터미널 기본 조작 → [로그](logs/01-terminal-permission.txt), 4절
- [x] 권한 실습 (파일 1개, 디렉토리 1개 chmod 전/후) → 4절
- [x] Git 사용자·기본 브랜치 설정 확인 → 5절
- [x] Docker 설치/점검 (`docker --version`, `docker info`) → [로그](logs/02-docker-basic.txt), 7절
- [x] Docker 운영 (images, ps -a, logs, stats) → 7절
- [x] hello-world, ubuntu 진입 (ls, echo), exec 관찰 → 7절
- [x] 커스텀 이미지 (방식 A: nginx:alpine) → [Dockerfile](web/Dockerfile), 8절
- [x] 포트 매핑 접속 2회 (8080, 8081, curl 응답) → [로그](logs/03-image-port-mount-volume.txt), 8절
- [x] 바인드 마운트 (호스트 변경 전/후) → 8절
- [x] 볼륨 영속성 (컨테이너 삭제 전/후) → 8절
- [ ] VSCode GitHub 연동 증거 (직접 로그인 후 스크린샷 첨부 필요)
- [x] 트러블슈팅 2건 이상 (3건 기록) → 6절
- [x] 민감정보 마스킹 (이메일 일부 마스킹)

보너스: [x] Compose 기초 · [x] Compose 멀티 · [x] Compose 운영 명령 · [x] 환경변수 → 9절 · [ ] SSH 키

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
- 해결: Docker Desktop을 직접 실행한 뒤 `docker info`에서 `Server=29.9.2` 확인 ([로그](logs/02-docker-basic.txt)).

**2. WSL 스크립트 경로가 깨짐**
- 문제: Git Bash에서 `wsl bash /mnt/c/...` 실행 시 `C:/Program Files/Git/mnt/c/...: No such file`
- 원인 가설: Git Bash가 `/`로 시작하는 인자를 Windows 경로로 자동 변환한다.
- 확인: 경로 앞에 `C:/Program Files/Git`이 붙은 오류 메시지.
- 해결: `MSYS_NO_PATHCONV=1`을 붙여 변환을 끈 뒤 성공.

**3. 이미지 다운로드 실패: `no such host`**
- 문제: 엔진을 켠 직후 `docker pull hello-world`가 `lookup registry-1.docker.io: no such host`로 실패 ([원본](logs/02a-docker-dns-error.txt))
- 원인 가설: ① PC 인터넷 문제 ② Docker 내부 DNS가 아직 준비되지 않음
- 확인: PC에서 `nslookup`은 성공, `curl https://registry-1.docker.io/v2/`는 401(정상 응답) → PC 네트워크는 정상. 잠시 뒤 `docker run --rm alpine nslookup registry-1.docker.io`가 성공.
- 해결: 엔진 기동 직후의 일시적 DNS 미준비로 판단. 잠시 기다린 뒤 재시도해서 성공.

## 7) 수행 로그 — Docker 설치·운영 (원본: [logs/02-docker-basic.txt](logs/02-docker-basic.txt))
```
$ docker --version
Docker version 29.9.2, build a82e528
$ docker info --format 'Server={{.ServerVersion}} OS={{.OperatingSystem}}'
Server=29.9.2 OS=Docker Desktop
$ docker run --name hello1 hello-world        → "Hello from Docker!" 출력
$ docker run --name ub1 ubuntu:24.04 bash -c 'ls / | head -5; echo hello-from-ubuntu'
bin boot dev etc home / hello-from-ubuntu
$ docker stats --no-stream      (sleeper: CPU 0.00%, MEM 944KiB / 15.41GiB)
```
run과 exec의 차이: `docker run`은 새 컨테이너를 만들어 명령이 끝나면 종료(Exited 0)된다. `docker exec`는 이미 떠 있는 컨테이너(`sleep 600`)에 들어가 명령만 실행하며, 컨테이너는 계속 유지된다.

## 8) 커스텀 이미지·포트·마운트·볼륨 (원본: [logs/03-image-port-mount-volume.txt](logs/03-image-port-mount-volume.txt))
- 베이스: `nginx:alpine`. 커스텀 3가지는 ① LABEL(이미지 식별) ② ENV APP_ENV=dev(설정 분리) ③ COPY site/(기본 페이지 교체).
- 빌드 `docker build -t e1-1-web:1.0 web` 성공, 라벨과 환경변수는 `docker image inspect`로 확인.
- 포트 매핑 2회: 같은 이미지를 `-p 8080:80`, `-p 8081:80`으로 실행하면 둘 다 같은 페이지를 응답한다. 컨테이너 포트는 격리되어 있어서 호스트 포트와 연결(매핑)해야 밖에서 접근할 수 있다.
```
$ curl -s http://localhost:8080   → <h1>E1-1 컨테이너 웹 서버</h1> ...
$ curl -s http://localhost:8081   → 동일
```
- 바인드 마운트(`-v 호스트폴더:/usr/share/nginx/html:ro`): 호스트에서 파일을 고치면 컨테이너 응답이 즉시 바뀐다.
```
전: <h1>bind BEFORE</h1>   →   후: <h1>bind AFTER (host edit)</h1>
```
- 볼륨 영속성: `mydata` 볼륨에 `hello.txt`를 쓰고 컨테이너를 `docker rm -f`로 삭제한 뒤, 새 컨테이너에 같은 볼륨을 연결하자 `hi`가 그대로 읽혔다.
- 참고: 로그 파일에 찍힌 명령은 따옴표가 빠진 표기이고, 실제 실행은 따옴표를 적용했다.
- 브라우저 주소창 스크린샷은 보조 자료이며, 평가용 증거는 위 curl 텍스트 로그다.

## 9) 보너스 — Docker Compose (원본: [logs/04-compose.txt](logs/04-compose.txt))
- 기초: [compose-single/docker-compose.yml](compose-single/docker-compose.yml)로 `up -d --build` → `ps` → `curl`(8090 응답) → `logs` → `down`. `docker run` 옵션을 파일로 문서화하는 것이 핵심이다.
- 환경변수: `HOST_PORT=8095 APP_MODE=prod`를 주입하자 8095 포트 200 응답, 컨테이너 안 `APP_MODE`가 `prod`로 바뀌었다(기본값은 8090/dev).
- 멀티: [compose-multi/docker-compose.yml](compose-multi/docker-compose.yml)에서 web과 client 2개를 함께 실행했고, `client`가 `wget http://web`으로 서비스 이름만으로 웹 서버 응답을 받았다(같은 네트워크의 서비스 디스커버리).
- 운영 명령 `up` `down` `ps` `logs`를 모두 수행했다.
