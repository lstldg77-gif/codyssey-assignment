# 코디세이 개발 환경 구축 실습

> 작성자: 동팔님 (실습 화면 사용자 `donghee1`)  
> 실습일: 2026-10-30  
> 상태: **진행 중 — 현재까지 완료한 부분을 정리한 제출 초안**

## 1. 프로젝트 소개

Windows 11에서 PowerShell, WSL Ubuntu, Docker Desktop을 설치하고 기본 명령어, Linux 파일 권한, Docker 이미지·컨테이너, Nginx 웹서버 포트 연결을 실습했다. 이어서 개인 HTML 페이지, Dockerfile, 마운트·볼륨, Git/GitHub 배포를 실습할 예정이다.

## 2. 환경

| 항목 | 확인된 환경 |
| --- | --- |
| 운영체제 | Windows 11 Pro |
| Windows 터미널 | Windows PowerShell |
| Linux | WSL Ubuntu, 사용자 `donghee1` |
| Docker Desktop | 4.97.0 (화면 확인) |
| Docker CLI | 29.9.2, build a82e528 |
| Docker 엔진 | Docker Desktop / Linux / WSL2 |
| 리눅스 커널 | 6.18.54.1-microsoft-standard-WSL2 (`docker info` 출력) |
| Git / GitHub | 버전·업로드 결과 아직 미확인 |

## 3. 실습 완료 현황

| 항목 | 상태 | 증빙 |
| --- | --- | --- |
| Windows PowerShell 파일·폴더 명령어 | 완료 | 실습 기록 참조 |
| WSL Ubuntu 실행 및 작업 폴더 생성 | 완료 | [WSL 위치 확인](evidence/01-wsl-home.png) |
| 파일 권한 `chmod 600`, `644` | 완료 | [파일 권한 변경](evidence/02-file-permission.png) |
| 폴더 권한 `chmod 700`, `755` | 완료 | [폴더 권한 변경](evidence/03-directory-permission.png) |
| Docker 설치 및 CLI 버전 확인 | 완료 | [버전 확인](evidence/04-docker-version.png) |
| `docker run hello-world` | 완료 | [Hello Docker](evidence/05-hello-world.png) |
| `docker info` | 완료 | [엔진 정보](evidence/06-docker-info.png) |
| Ubuntu Docker 컨테이너에서 `pwd`, `ls` | 완료 | [Ubuntu 컨테이너](evidence/07-ubuntu-container.png) |
| 컨테이너 목록 및 로그 조회 | 완료 | [ps 및 logs](evidence/08-docker-logs.png) |
| Nginx 실행 및 8080:80 포트 연결 | 완료 | [명령어](evidence/09-nginx-terminal.png) / [브라우저](evidence/10-nginx-browser.png) |
| 개인 `index.html` 생성 및 내용 조회 | 완료 | [파일 생성](evidence/11-custom-html.png) |
| 개인 HTML을 8081에서 바인드 마운트 | 완료 | [8081 개인 웹페이지](evidence/12-bind-mount-browser.png) |
| Dockerfile 이미지 빌드 (`codyssey-site:1.0`) | 완료 | [이미지 빌드 및 목록](evidence/13-docker-build.png) |
| Dockerfile 이미지로 웹서버 실행 (8082) | 완료 | [컨테이너 실행](evidence/14-docker-run-terminal.png) / [8082 웹페이지](evidence/15-dockerfile-browser.png) |
| Docker named volume 데이터 보존 | 완료 | [볼륨 생성·저장·재확인](evidence/16-docker-volume.png) |
| Git 초기화, 커밋 및 GitHub 업로드 | **추가 실습 예정** | 추후 추가 |

## 4. 실제로 실행한 주요 명령어

### WSL 및 권한

```powershell
wsl -d Ubuntu -u donghee1 -- bash --noprofile --norc
```

```bash
cd ~
pwd
mkdir codyssey-workspace
cd codyssey-workspace
touch permission-test.txt
chmod 600 permission-test.txt
ls -l permission-test.txt
chmod 644 permission-test.txt
ls -l permission-test.txt
mkdir permission-dir
chmod 700 permission-dir
ls -ld permission-dir
chmod 755 permission-dir
ls -ld permission-dir
```

### Docker

```powershell
docker --version
docker run hello-world
docker images
docker ps -a
docker info
docker run -it --name codyssey-ubuntu ubuntu bash
# 컨테이너 안에서: pwd, ls, exit
docker ps -a
docker logs codyssey-ubuntu
docker run -d --name codyssey-web -p 8080:80 nginx
docker ps
```

브라우저에서 `http://localhost:8080`을 열어 **Welcome to nginx!**가 표시됨을 확인했다.

### 개인 웹페이지 생성

```powershell
cd C:\Users\User\codyssey-workspace
Set-Content -Path index.html -Value '<h1>Codyssey Docker Project</h1><p>Made by Dongpal</p>'
Get-Content index.html
```

이 저장소의 `index.html`은 위 실습과 동일한 내용을 포함한다.

## 5. 문제 해결 기록

1. **Docker 명령어를 인식하지 못함**: Docker Desktop 설치 전 또는 기존 PowerShell 창에서 `docker`가 명령어로 인식되지 않았다. Docker Desktop을 설치·실행한 후 PowerShell을 새로 열었고 `docker --version`에서 `29.9.2`를 확인했다. [오류 화면](evidence/04a-docker-not-found.png)
2. **URL을 PowerShell에 입력하여 오류**: `http://localhost:8080`은 PowerShell 명령어가 아니라 브라우저 주소다. 크롬 주소창에 입력해 Nginx 기본 페이지를 성공적으로 열었다. [터미널 오류](evidence/09-nginx-terminal.png), [정상 화면](evidence/10-nginx-browser.png)
3. **Ubuntu 예시 프롬프트를 명령어로 잘못 입력**: `root@xxxxxxxxxxxx:/#`는 실행 결과 예시이지 명령어가 아니다. 실제 컨테이너 프롬프트에서 `pwd`, `ls`를 실행해 정상 동작을 확인했다. [확인 화면](evidence/07-ubuntu-container.png)

## 6. 제출용 사진 안내

`evidence/` 폴더는 실제 실행 화면을 담는다. README 표에서 각 사진을 누르면 확인할 수 있다. 특히 포트 연결 증빙은 **브라우저 주소와 웹페이지 문구가 함께 보이도록** 촬영했다.

사진에 없는 결과는 성공했다고 주장하지 않으며, 이후 실습이 끝나는 대로 추가한다.

## 7. 이어서 진행할 실습

1. HTML 수정 후 브라우저 새로고침으로 반영 확인
2. Git 버전·사용자 설정 확인, 저장소 생성·커밋, GitHub 업로드
3. 완료 증빙과 정확한 저장소 주소를 이 README에 추가

`NEXT_STEPS.md`에 복사해 실행할 명령어를 정리했다.
