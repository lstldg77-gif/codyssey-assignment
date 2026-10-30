# 실습 재현 명령어

수행 결과와 증거 링크는 [README.md](README.md)에 정리했다. 아래 명령은 해당 절차를 다시 확인할 때 사용한다. 프로젝트 폴더 경로는 각자 환경에 맞게 바꾼다.

## PowerShell 파일 조작

```powershell
New-Item -ItemType Directory -Force practice | Out-Null
Set-Location practice
Get-Location
New-Item -ItemType File -Name .hidden-note
Set-Content -Path notes.txt -Value 'terminal practice'
Get-ChildItem -Force | Select-Object Name,Length
Get-Content notes.txt
New-Item -ItemType File -Name empty.txt
Copy-Item notes.txt copy.txt
Move-Item copy.txt renamed.txt
Remove-Item renamed.txt
Test-Path renamed.txt
```

`Get-ChildItem -Force`로 숨김 항목을 포함해 확인하고, 마지막 `Test-Path`가 `False`이면 삭제된 것이다.

## Docker 운영 확인

```powershell
docker --version
docker info
docker images
docker ps
docker ps -a
docker logs codyssey-built
docker stats --no-stream
Invoke-WebRequest -UseBasicParsing http://localhost:8082
```

커스텀 이미지가 아직 없다면 프로젝트 루트에서 먼저 빌드한다.

```powershell
docker build -t codyssey-site:1.0 .
docker run -d --name codyssey-built -p 8082:80 codyssey-site:1.0
```

같은 이름의 컨테이너가 이미 있으면 `docker ps -a`로 상태를 확인하고, 데이터가 필요 없는지 확인하기 전에는 삭제하지 않는다.

## Named volume 영속성 재확인

```powershell
docker volume create codyssey-data
docker run --rm -v codyssey-data:/data alpine sh -c "echo codyssey-persistent-data > /data/result.txt"
docker run --rm -v codyssey-data:/data alpine cat /data/result.txt
docker volume inspect codyssey-data
```

읽기 명령의 출력이 `codyssey-persistent-data`인지 확인한다. `docker volume rm`은 저장 데이터도 삭제하므로 재현 확인 중에는 실행하지 않는다.

## GitHub 원격 저장소 확인

```powershell
git --version
git config --get user.name
git config --get init.defaultBranch
git remote -v
git status --short --branch
```

제출 원격 저장소는 [lstldg77-gif/codyssey-assignment](https://github.com/lstldg77-gif/codyssey-assignment)이며 기본 작업 브랜치는 `main`이다. 이메일과 인증정보는 출력·문서에 공개하지 않는다.
