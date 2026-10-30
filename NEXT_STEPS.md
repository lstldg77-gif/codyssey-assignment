# 다음 실습 안내 (Windows PowerShell)

아래 명령어는 **실행 예정**이며, 아직 성공을 확인하지 않았다. 프로젝트 폴더(`C:\Users\User\codyssey-workspace`)에서 진행한다.

## A. 바인드 마운트 확인

```powershell
cd C:\Users\User\codyssey-workspace
docker stop codyssey-web
docker run -d --name codyssey-myweb -p 8081:80 -v "${PWD}:/usr/share/nginx/html:ro" nginx
docker ps
```

크롬에서 `http://localhost:8081`을 열어 개인 웹페이지가 나오는지 확인하고 주소창 포함 캡처한다. 이후 `index.html` 내용을 바꾸고 새로고침해 변화를 확인한다.

**주의:** 기존 이름 `codyssey-myweb` 컨테이너가 이미 있다면 같은 이름의 `docker run`은 오류가 난다. 이 경우 먼저 `docker ps -a`를 확인한다.

## B. Dockerfile 빌드 (이 패키지의 `Dockerfile`, `index.html` 사용)

이 패키지의 파일들을 Windows `codyssey-workspace` 폴더에 복사한 후 해당 폴더에서 진행한다.

```powershell
docker build -t codyssey-site:1.0 .
docker run -d --name codyssey-built -p 8082:80 codyssey-site:1.0
docker ps
```

크롬에서 `http://localhost:8082` 확인 후 사진 촬영.

## C. Docker named volume (실행 전 확인)

```powershell
docker volume create codyssey-data
docker run --rm -v codyssey-data:/data alpine sh -c "echo codyssey-persistent-data > /data/result.txt"
docker run --rm -v codyssey-data:/data alpine cat /data/result.txt
docker volume inspect codyssey-data
```

두 번째 `docker run`에서 `codyssey-persistent-data`가 출력되면 컨테이너가 없어져도 데이터가 남는 것을 확인한 것이다.

## D. Git / GitHub (아직 미완료)

```powershell
git --version
git status
git init -b main
git add README.md NEXT_STEPS.md Dockerfile index.html .dockerignore .gitignore evidence
git commit -m "Document Codyssey developer environment practice"
```

Git 사용자 정보가 없으면 본인 이름·GitHub에 사용하는 메일을 `git config --global user.name`, `git config --global user.email`로 설정한다. 원격 저장소 생성과 `git push`는 실제 GitHub 주소가 정해진 뒤 진행한다. 아직 제출 완료로 표시하지 않는다.
