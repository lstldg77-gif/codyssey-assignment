# 코디세이 개발 환경 구축 및 Docker 실습 (보너스 과제 포함)

> 실습일: 2026-10-30  
> 제출 저장소: [lstldg77-gif/codyssey-assignment](https://github.com/lstldg77-gif/codyssey-assignment)

## 1. 프로젝트 개요

Windows PowerShell과 WSL Ubuntu에서 터미널·파일 권한을 연습하고, Docker 이미지와 컨테이너를 운영했다. NGINX 기반 커스텀 이미지를 직접 빌드하고 포트 매핑·바인드 마운트·named volume을 검증했다. 마지막으로 Git 로컬 저장소를 GitHub 원격 저장소에 연결해 `main` 브랜치로 업로드했다.

## 2. 실행 환경

| 항목 | 확인 결과 |
| --- | --- |
| OS / 터미널 | Windows 11 Pro, Windows PowerShell |
| Linux | WSL Ubuntu |
| Docker Desktop | 4.97.0 (실습 화면 확인) |
| Docker CLI | `Docker version 29.9.2, build a82e528` |
| Docker 엔진 | Docker Desktop, Server 29.9.2 |
| Git | `git version 2.56.0.windows.4` |
| 저장소 / 브랜치 | GitHub 저장소 `codyssey-assignment`, `main` |

버전과 데몬은 `docker --version`, `docker info`로 확인했다. 이 문서의 Docker 리소스 수치와 사용량은 확인 당시 값이며, 실행 시점에 따라 달라질 수 있다.

## 3. 수행 체크리스트

| 영역 | 상태 | 결과 및 증거 |
| --- | --- | --- |
| 터미널 파일·폴더 조작 | 완료 | 아래 [터미널 로그](#4-터미널-조작-로그) |
| 파일·디렉터리 권한 | 완료 | [파일 권한](evidence/02-file-permission.png), [디렉터리 권한](evidence/03-directory-permission.png) |
| Docker 설치·데몬 확인 | 완료 | [Docker 버전](evidence/04-docker-version.png), [Docker 정보](evidence/06-docker-info.png) |
| 이미지·컨테이너·로그·리소스 | 완료 | [컨테이너 및 로그](evidence/08-docker-logs.png), 아래 [Docker 운영](#6-docker-기본-운영과-컨테이너) |
| `hello-world`, Ubuntu 컨테이너 | 완료 | [hello-world](evidence/05-hello-world.png), [Ubuntu 컨테이너](evidence/07-ubuntu-container.png) |
| Dockerfile 커스텀 이미지 | 완료 | [Dockerfile](Dockerfile), [이미지 빌드](evidence/13-docker-build.png), [실행](evidence/14-docker-run-terminal.png) |
| 포트 매핑·웹 응답 | 완료 | [8080 NGINX](evidence/09-nginx-terminal.png), [8080 브라우저](evidence/10-nginx-browser.png), [8082 브라우저](evidence/15-dockerfile-browser.png) |
| 바인드 마운트 | 완료 | [8081 브라우저](evidence/12-bind-mount-browser.png) |
| named volume 영속성 | 완료 | [볼륨 데이터 확인](evidence/16-docker-volume.png) |
| Git 설정·GitHub 연결 | 완료 | 아래 [Git 및 GitHub](#10-git-설정과-github-연동), [GitHub 저장소](https://github.com/lstldg77-gif/codyssey-assignment) |
| **보너스 과제: Compose 단일 서비스** | **완료** | 아래 [Compose 실습](#14-보너스-과제-docker-compose-단일-서비스) |

## 4. 터미널 조작 로그

PowerShell에서 임시 연습 폴더를 만들어 현재 위치 확인, 숨김 파일을 포함한 목록 조회, 파일 생성·내용 확인·복사·이름 변경·삭제를 실행했다. 출력의 사용자별 임시 경로는 `<임시 폴더>`로 표시했다.

```powershell
PS> Get-Location
<임시 폴더>\codyssey-cli-practice

PS> New-Item -ItemType File -Name .hidden-note
.hidden-note
PS> New-Item -ItemType File -Name notes.txt
notes.txt
PS> Set-Content -Path notes.txt -Value 'terminal practice'
PS> Get-ChildItem -Force | Select-Object Name,Length
.hidden-note      0
notes.txt        19

PS> Get-Content notes.txt
terminal practice

PS> New-Item -ItemType File -Name empty.txt
empty.txt
PS> Copy-Item notes.txt copy.txt
PS> Get-Item copy.txt | Select-Object Name,Length
copy.txt         19
PS> Move-Item copy.txt renamed.txt
PS> Get-Item renamed.txt | Select-Object Name,Length
renamed.txt      19
PS> Remove-Item renamed.txt
PS> Test-Path renamed.txt
False
```

`Get-ChildItem -Force`는 숨김 파일까지 보여준다. `Copy-Item`은 원본을 보존한 채 복사하고, `Move-Item`은 파일을 옮기거나 이름을 바꾼다. `Test-Path` 결과 `False`로 삭제를 확인했다. 연습 폴더는 임시 폴더에서 정리했다.

### 절대 경로와 상대 경로

- 절대 경로는 루트부터 위치를 지정한다. 예: `C:\Users\User\codyssey-workspace\index.html`
- 상대 경로는 현재 위치를 기준으로 지정한다. 프로젝트 루트에서 `index.html`, 하위 폴더는 `evidence\01-wsl-home.png`처럼 쓴다.
- 재현할 때는 먼저 프로젝트 폴더로 이동하고 상대 경로 명령을 실행한다.
- 위 `C:\Users\User\...`는 경로 형식 예시다. 실제 Windows 계정명과 프로젝트 위치에 맞게 바꾼다.

## 5. 파일 권한 실습

WSL에서 `ls -l`/`ls -ld`로 변경 전·후 권한을 확인하고 `chmod`로 파일과 디렉터리 권한을 변경했다.

| 대상 | 변경 전 | 변경 후 | 증거 |
| --- | --- | --- | --- |
| 파일 `permission-test.txt` | `600` (`rw-------`) | `644` (`rw-r--r--`) | [권한 변경 화면](evidence/02-file-permission.png) |
| 디렉터리 `permission-dir` | `700` (`rwx------`) | `755` (`rwxr-xr-x`) | [권한 변경 화면](evidence/03-directory-permission.png) |

권한 숫자는 소유자·그룹·기타 사용자 순서다. 각 자리의 `r=4`, `w=2`, `x=1`을 더한다. 따라서 `7=4+2+1`은 읽기·쓰기·실행, `5=4+1`은 읽기·실행, `6=4+2`는 읽기·쓰기다. 디렉터리의 `x`는 해당 디렉터리로 진입하거나 내부 항목을 탐색할 수 있는 권한이다.

```bash
ls -l permission-test.txt
chmod 600 permission-test.txt
ls -l permission-test.txt
chmod 644 permission-test.txt
ls -l permission-test.txt

ls -ld permission-dir
chmod 700 permission-dir
ls -ld permission-dir
chmod 755 permission-dir
ls -ld permission-dir
```

## 6. Docker 기본 운영과 컨테이너

### 설치·데몬 및 이미지 확인

```powershell
PS> docker --version
Docker version 29.9.2, build a82e528

PS> docker info --format 'Server={{.ServerVersion}} OS={{.OperatingSystem}} Containers={{.Containers}} Images={{.Images}}'
Server=29.9.2 OS=Docker Desktop Containers=5 Images=5

PS> docker images
IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
alpine:latest        be2656061b53   13MB         3.94MB
codyssey-site:1.0    97eaa1be1cad   93.6MB       26.3MB         U
hello-world:latest   c567b231f656   25.9kB       9.49kB         U
nginx:latest         600f7cd45e27   243MB        66.8MB         U
ubuntu:latest        6a06111c3b0a   163MB        45.9MB         U
```

이미지 크기와 ID는 확인 시점의 로컬 결과다. 이미지 상세 정보와 데몬 정보는 [Docker 정보 화면](evidence/06-docker-info.png)에서도 확인할 수 있다.

### 기본 이미지·컨테이너 명령

```powershell
docker run hello-world
docker run -it --name codyssey-ubuntu ubuntu bash
# 컨테이너 안에서 실행
pwd
ls
echo "hello from ubuntu"
exit

docker ps
docker ps -a
docker logs codyssey-ubuntu
docker stats --no-stream
```

`hello-world`의 성공 메시지는 [실행 증거](evidence/05-hello-world.png), Ubuntu 내부 명령은 [컨테이너 증거](evidence/07-ubuntu-container.png)에서 확인할 수 있다. `docker ps`는 실행 중인 컨테이너, `docker ps -a`는 종료된 컨테이너까지 표시한다. `docker logs`는 컨테이너의 표준 출력·오류 로그를 조회한다. `docker stats --no-stream`은 한 번의 CPU·메모리 사용량을 보여준다.

운영 상태를 재확인한 결과, 실행 중이던 컨테이너는 `codyssey-built`, `codyssey-myweb`이었고 종료 상태인 컨테이너에는 `codyssey-web`, `codyssey-ubuntu`, `hello-world`가 있었다. `docker stats --no-stream`에서도 실행 중인 두 컨테이너의 CPU·메모리 항목을 확인했다. 컨테이너 ID와 자원 사용량은 실행 시점마다 달라진다. 기존 로그·목록 확인 화면은 [컨테이너 운영 증거](evidence/08-docker-logs.png)다.

### 종료와 유지의 차이

- `docker run -it ubuntu bash`처럼 대화형 셸이 PID 1인 컨테이너는 `exit`하면 주 프로세스가 끝나 컨테이너도 종료된다. `docker ps -a`로 종료 상태를 확인할 수 있다.
- `docker run -d ... nginx`는 주 프로세스인 NGINX가 계속 동작하므로 터미널에서 분리되어도 컨테이너가 실행 상태로 유지된다.
- 실행 중인 컨테이너에 `docker exec -it <컨테이너> sh`를 사용하면 별도 셸을 연다. 그 셸을 종료해도 원래 주 프로세스가 살아 있으면 컨테이너는 계속 실행된다.
- `docker stop <컨테이너>`는 주 프로세스에 종료를 요청한다. 중지된 컨테이너를 다시 시작할 때는 `docker start <컨테이너>`를 사용한다.

## 7. Dockerfile 기반 커스텀 이미지

기존 베이스 이미지 `nginx:alpine`을 사용했다. 직접 작성한 [Dockerfile](Dockerfile)은 정적 페이지를 NGINX 기본 문서 경로에 복사하고 컨테이너 포트 `80`을 문서화한다.

| 커스텀 항목 | 목적 |
| --- | --- |
| `FROM nginx:alpine` | 경량 NGINX 베이스 이미지 사용 |
| `COPY index.html /usr/share/nginx/html/index.html` | 개인 페이지를 이미지에 포함 |
| `EXPOSE 80` | 컨테이너가 제공하는 HTTP 포트 표시 |

```powershell
docker build -t codyssey-site:1.0 .
docker run -d --name codyssey-built -p 8082:80 codyssey-site:1.0
docker ps
docker logs codyssey-built
```

빌드·이미지 증거: [빌드 화면](evidence/13-docker-build.png). 실행 증거: [컨테이너 실행](evidence/14-docker-run-terminal.png), [8082 브라우저 응답](evidence/15-dockerfile-browser.png). 재확인 시 `http://localhost:8082`에서 `index.html` 응답을 받았다.

## 8. 포트 매핑과 마운트

포트 매핑은 호스트에서 들어온 요청을 컨테이너의 서비스 포트로 전달한다. 예를 들어 `-p 8082:80`은 호스트 `8082` 요청을 컨테이너 `80`으로 연결한다. 매핑하지 않으면 호스트 브라우저에서 해당 컨테이너 포트로 직접 접속할 수 없다.

| 방식 | 실행 예 | 검증 |
| --- | --- | --- |
| NGINX 기본 페이지 | `docker run -d --name codyssey-web -p 8080:80 nginx` | [명령](evidence/09-nginx-terminal.png), [브라우저 `localhost:8080`](evidence/10-nginx-browser.png) |
| 호스트 파일 바인드 마운트 | `docker run -d --name codyssey-myweb -p 8081:80 -v "${PWD}:/usr/share/nginx/html:ro" nginx` | [브라우저 `localhost:8081`](evidence/12-bind-mount-browser.png) |
| 커스텀 이미지 | `docker run -d --name codyssey-built -p 8082:80 codyssey-site:1.0` | [브라우저 `localhost:8082`](evidence/15-dockerfile-browser.png) |

바인드 마운트는 호스트의 특정 경로를 컨테이너에 연결한다. 위 예의 `:ro`는 컨테이너에서 호스트 파일을 읽기 전용으로 연결한다. `${PWD}`는 명령을 실행한 현재 폴더이므로 `index.html`이 있는 폴더에서 실행해야 한다.

## 9. Docker named volume 영속성

named volume은 Docker가 관리하는 저장 공간이다. 컨테이너 수명과 분리되어 컨테이너를 삭제해도 볼륨을 지우지 않는 한 데이터가 유지된다.

```powershell
docker volume create codyssey-data
docker run --rm -v codyssey-data:/data alpine sh -c "echo codyssey-persistent-data > /data/result.txt"
docker run --rm -v codyssey-data:/data alpine cat /data/result.txt
codyssey-persistent-data
docker volume inspect codyssey-data
```

첫 컨테이너가 종료된 후 두 번째 컨테이너에서 같은 볼륨의 파일을 읽어 `codyssey-persistent-data`를 확인했다. 컨테이너는 `--rm`으로 제거되며 named volume은 남는다. 볼륨을 직접 삭제하는 `docker volume rm codyssey-data`는 데이터도 지우므로 재현 확인 전에는 실행하지 않는다. [생성·저장·재확인 증거](evidence/16-docker-volume.png).

## 10. Git 설정과 GitHub 연동

Git은 로컬 변경 이력과 커밋을 관리하고, GitHub는 원격 저장소를 제공한다. 사용자 이름은 `Dongpal`로 설정되어 있으며, 이메일은 개인정보 보호를 위해 아래 출력에서 마스킹했다. 새 저장소 기본 브랜치는 `main`이다.

```powershell
PS> git --version
git version 2.56.0.windows.4

PS> git config --get user.name
Dongpal
PS> git config --get user.email
<redacted>
PS> git config --get init.defaultBranch
main

PS> git config --list
user.name=Dongpal
user.email=<redacted>
init.defaultBranch=master
init.defaultBranch=main
remote.origin.url=https://github.com/lstldg77-gif/codyssey-assignment.git

PS> git remote -v
origin  https://github.com/lstldg77-gif/codyssey-assignment.git (fetch)
origin  https://github.com/lstldg77-gif/codyssey-assignment.git (push)

PS> git status --short --branch
## main...origin/main
```

`git config --list`에서는 인증 정보나 개인 이메일을 공개하지 않도록 마스킹했다. `git remote -v`와 `git status`로 원격 연결 및 추적 브랜치를 확인했다. `git push -u origin main`이 성공해 원격 `main`이 생성됐고, 이후 로컬 `main`이 `origin/main`을 추적한다. 현재 제출 저장소는 [GitHub에서 확인](https://github.com/lstldg77-gif/codyssey-assignment)할 수 있다. HTTPS 인증 토큰이나 비밀번호는 문서·로그에 기록하지 않는다.

Git 설치 시 기본 설정 파일의 `master` 값 위에 사용자 설정 `main`을 지정했다. `git config --get init.defaultBranch`의 최종 유효값은 `main`이다.

## 11. 처음부터 재현하기

Docker Desktop과 WSL이 실행 중이고 저장소를 내려받은 상태에서 프로젝트 루트에서 진행한다.

```powershell
# 이미지 빌드 및 커스텀 웹 컨테이너 실행
docker build -t codyssey-site:1.0 .
docker run -d --name codyssey-built -p 8082:80 codyssey-site:1.0
docker ps
Invoke-WebRequest -UseBasicParsing http://localhost:8082

# named volume 데이터 쓰기·읽기
docker volume create codyssey-data
docker run --rm -v codyssey-data:/data alpine sh -c "echo codyssey-persistent-data > /data/result.txt"
docker run --rm -v codyssey-data:/data alpine cat /data/result.txt

# 컨테이너 로그·리소스·종료 상태 확인
docker logs codyssey-built
docker stats --no-stream
docker stop codyssey-built
docker ps -a
```

이미 같은 이름의 컨테이너가 있는 경우 먼저 `docker ps -a`로 상태를 확인한다. 기존 컨테이너나 볼륨을 삭제하기 전에는 필요한 데이터가 없는지 확인한다. `docker run`은 컨테이너 이름이 중복되면 실패하므로, 기존 컨테이너를 임의로 삭제하지 말고 다른 이름을 사용한다.

## 12. 보안 및 개인정보

README에는 이메일을 마스킹했고 인증 토큰·비밀번호·개인키는 기록하지 않았다. 스크린샷을 추가할 때도 터미널 프롬프트, 브라우저, Git 설정에 민감정보가 노출되지 않는지 확인한다.

## 13. 문제 해결 기록

1. Docker 설치 전 또는 기존 PowerShell에서 `docker`가 인식되지 않았다. Docker Desktop을 실행한 뒤 새 PowerShell에서 버전을 확인했다. [오류 화면](evidence/04a-docker-not-found.png), [정상 버전 확인](evidence/04-docker-version.png)
2. `http://localhost:8080`은 PowerShell 명령이 아니라 브라우저 주소다. 브라우저에서 NGINX 기본 페이지를 확인했다. [터미널](evidence/09-nginx-terminal.png), [브라우저](evidence/10-nginx-browser.png)
3. `root@...:/#`는 Ubuntu 컨테이너 프롬프트 예시이지 실행 명령이 아니다. 컨테이너 내부에서 `pwd`, `ls`를 실행했다. [Ubuntu 컨테이너](evidence/07-ubuntu-container.png)

## 14. 보너스 과제: Docker Compose 단일 서비스

`compose.yaml`은 기존 Dockerfile로 웹 이미지를 빌드해 `web` 서비스로 실행한다. 호스트 포트 `8083`을 컨테이너 포트 `80`에 연결한다. Compose를 사용하면 실행 설정을 파일로 저장해 같은 명령으로 재현할 수 있다.

```yaml
services:
  web:
    build: .
    ports:
      - "8083:80"
```

프로젝트 루트에서 실행하고 서비스 상태, 로그, HTTP 응답을 확인한다.

```powershell
docker compose up --build -d
docker compose ps
docker compose logs web
Invoke-WebRequest -UseBasicParsing http://localhost:8083
docker compose down
docker compose ps
```

`up`은 이미지를 빌드하고 서비스를 시작하며, `ps`와 `logs`는 상태와 로그를 확인하고, `down`은 Compose 컨테이너와 네트워크를 정리한다. 이 서비스의 실행·응답·종료는 아래 검증 로그에서 확인할 수 있다. 포트 `8083`이 이미 사용 중이면 Compose 설정의 호스트 포트를 비어 있는 포트로 변경한다.

실행 검증 결과:

```text
PS> docker compose up --build -d
Image codyssey-assignment-web Built
Container codyssey-assignment-web-1 Started

PS> docker compose ps
NAME                        SERVICE   STATUS                  PORTS
codyssey-assignment-web-1   web       Up Less than a second   0.0.0.0:8083->80/tcp

PS> docker compose logs web
web-1 | ... Configuration complete; ready for start up

PS> Invoke-WebRequest -UseBasicParsing http://localhost:8083
<h1>Codyssey Docker Project</h1>
<p>Made by Dongpal</p>

PS> docker compose down
Container codyssey-assignment-web-1 Removed
Network codyssey-assignment_default Removed

PS> docker compose ps
NAME    SERVICE   STATUS   PORTS
```

## 과제 목표 요약

1. **절대·상대 경로:** 절대 경로는 루트부터 지정하는 전체 주소이고, 상대 경로는 현재 위치를 기준으로 한 주소다.
2. **파일 권한:** `r/w/x`는 읽기·쓰기·실행 권한이며, 각각 `4/2/1`로 더해 `755`(소유자 전체 권한, 그룹·기타 읽기/실행)나 `644`(소유자 읽기/쓰기, 그룹·기타 읽기)처럼 나타낸다. 디렉터리의 `x`는 내부 항목에 접근하는 권한이다.
3. **커스텀 이미지:** Dockerfile에서 `nginx:alpine`을 베이스로 사용하고 개인 `index.html`을 복사해 `codyssey-site:1.0` 이미지를 만들었다. `EXPOSE 80`은 컨테이너 포트를 문서화하며, 외부 접속에는 별도의 포트 매핑이 필요하다.
4. **포트 매핑:** `-p 8082:80`은 호스트의 8082 포트를 컨테이너의 80 포트에 연결해 브라우저에서 웹 서버에 접속할 수 있게 한다.
5. **Docker 볼륨:** named volume은 컨테이너와 별도로 데이터를 보관해 컨테이너를 삭제한 뒤에도 데이터를 유지한다.
6. **Git과 GitHub:** Git은 로컬 변경 이력과 커밋을 관리하고, GitHub는 저장소를 원격으로 보관·공유하는 플랫폼이다. `git push`로 로컬 커밋을 원격 저장소에 올린다.

보너스 과제로 Docker Compose 단일 웹 서비스를 구성해 `up`, `ps`, `logs`, `down`으로 실행·확인·종료했다. 자세한 설명은 별도 문서 [목표요약.md](목표요약.md)에서도 확인할 수 있다.
