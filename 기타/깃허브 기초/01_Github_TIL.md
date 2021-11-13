### Github TIL

### 1.TIL?

> - TIL은 Today I Learned의 줄임말로 개발자 사이에서 매일 자신이 학습한 내용을 commit(기록)하는것
> - github, bitbucket, gitlab과 같은 원격 저장소에서 제공하는 1commit-1grass의 흥미 요소 제공

### 2.TIL 세팅

### (1) Git으로 프로젝트 관리 시작 : git init

- 자신이 앞으로 학습한 내용을 기록할 TIL 폴더를 하나 생성한다. 이때 해당 폴더는 최상단에 생성한다.

- git bash 에서 TIL 폴더로 이동한 이후에 아래의 명령어로 git 관리를 시작한다.

```bash
$git init
```

### (2) Commit을 위한 Staging : git add

- 현재 코드 상태의 스냅샷을 찍기 위한 파일 선택 (== Staging Area에 파일 추가)

```
$git add [파일 이름] # .은 모든 변경 사항을 staging area로 올림
```

### (3) 버전 관리를 위한 스냅샷 저장:`git commit`

- 현재 상태에 대한 스냅샷을 commit 하여, 버전 관리를 진행한다.

```
$git commit -m "커밋 메시지"
```

### (4) 원격 저장소 정보 추가 : `git remote`

- Github 원격(remote) 저장소(repository)를 생성하고 `TIL `폴더와 연결한다.

- 새로운 원격 저장소가 추가될 때만 입력한다.

```
$git remote add orgin [github 원격 저장소 주소]
```

### (5) 원격 저장소로 코드 `git push`

- 최종적으로 Github 원격 저장소에 push한다.

```
$git push origin master
```

### (6) 그 외 명령어

- 현재 git 의 상태를 조회 git status

```
$git status
```

- 버전 관리 이력을 조회

```
$git log
```

- `git` 설정 (user.name & user.email) : **최초 1회 설정**

```bash
$git config --global user.name "name"
$git config --global user.email "email@email.net"
```

### 3. `README.md`

> 원격(remote) 저장소(repository)에 대한 정보를 기록하는 마크다운 문서. 일반적으로 해당 프로젝트를 사용하기 위한 방법 등을 기재한다.

### (1)` README.md` 파일 생성

- `README.md` 파일을 `TIL` 폴더(최상단)에 생성한다. 이름은 반드시 **README.md**로 설정한다.

```
$touch README.md
```

### **(2) (자신만의) TIL 원칙에 대한 간단한 내용 추가**

- 마크다운 작성법 pdf에서 배우고 실습한 내용을 토대로 README.md 파일을 작성한다.
- 형식은 자유롭게 작성하되 마크다운 문법(의미론적)을 지켜서 작성한다.

### (3) 저장 후 버전관리 : add , commit , push

- 작성이 완료되면 아래의 명령어를 통해 commit 이력을 남기고 원격 저장소로 push한다.

```
$git add REAMDE.md
$git commit -m "add README.md"
$git push origin master

```

## 4. 추가 학습 내용 관리

**(1) 추가 내용 관리**

- `TIL` 폴더 내에서 학습을 원하는 내용의 폴더를 생성하고 파일들을 생성한 후 작업을 진행한다.

**(2) 변경 사항을 저장하고, 원격저장소로 옮긴다.**

- 업데이트가 완료되면 아래의 명령어를 통해 commit 이력을 남기고 원격 저장소로 push한다.

```
$git add .
$git commit -m "학습내용 추가"
$git push origin master
```

## 기타 정리 사항

- git init 으로 .git 을 만든다.

- GIt 폴더 단위로 코드 관리

- 한폴더에 하나의 .git 있어야함 거의 최상단에만 있어야한다.

- 파일이동,파일명변경 mv 복사 cp 꿀팁 현재 작업폴더에서 mv ~/mark.md .

- 절대경로로 파일이동한다.

- git status 현재 폴더상태 물어보기 계속 쳐봐야한다.

- git commit 으로 스냅샷을 만든다

- visual code 사용시 code . 하면 현재 폴더 기준 비주얼코드 열림

## checkout 으로 과거로 가기

- git log --oneline 로 작업 해시코드 및 로그 확인

- git checkout bd1096c 해시코드로 과거로 가기가 가능.

- git checkout master 현실로 돌아오자

## git 사용 3가지

- push and pull

- fork & PR

- shred repository

### branch

- git branch -> 현재 저장소에 있는 모든 branch 출력 & 현재 브랜치도 보여줌
- git branch [branch명] -> 브랜치 생성
- git checkout -b [branch명] -> 브랜치 생성하고 바로 포인터 이동
- git checkout [branch명] -> 브랜치 이동

> git의 log는 서로 다른 세계의 내용물을 복사하는게 아니라 링크를 걸어 주는 것이기 때문에 링크드 리스트 개념으로 이해하면 된다. head는 어디를 가르키고 있는지 알려줌.

> branch는 독립된 세계로 생각하자.

> 평행 세계의 내용을 가져와서 합친다 = merge

- git merge [브랜치명] -> 목적 브랜치를 병합한다.
  - 병합의 주체가 되는 브랜치로 이동 후 merge 하기.
  - (master 에서 branch 를 머지한다.)

## 머지의 종류랄까

1. fast forwards
2. merge commit
3. merge conflict

**fast forwards**

- 하나의 브랜치에만 커밋이 쌓인 경우이다.
- 하나의 브랜치에만 커밋이 쌓였기 때문에 분기점이 아직 없음.
- 즉 구성도로 봤을 때 한줄이므로 여기서 merge는 포인터가 맨앞으로 이동하는 거와 같다.

**merge commit**

- 둘다 커밋이 쌓이는 경우이다. 포인터 이동으로는 안되고
- git merge 시새로운 커밋이 생성된다.

**merge conflict **

- 같은 파일을 안건드리는게 conflict 방지방법중 최고다.
- 하지만 대체로 발생하게 되어있음. 애매모호할 때.

- 대부분 머지 후 커밋 메시지를 Resolve merge commit 이라 지음.
- 브랜치로 비동기적으로 사용이 가능하기 때문에 동시 작업 가능.

### Shared Repo

- master는 건드리지 않는다.
- 각자의 branch 에서 작업하고 커밋 후 pull request 로 진해한다.
- 머지 작업까지 완료 한 후 다시 새로운 작업을 하고 싶으면 master로 돌아가서
- 지금까지 머지된 프로젝트를 다시 pull 받고 시작한다.

- 상대방의 작업 Repo 에 직접 접근해서 push 를 할려면 상대방에게 invite를 받아야 한다.
- 이것이 불가능하다면 일단 fork 를 하고 내 로컬에서 작업한 후 push 한다음 수정된 사항을 pull request 로 master 에게 요청한다.

- master의 프로젝트를 하나 받고 branch 로 분기하여 작업을 진행하고 pull request 를 진행하면서 머지한다.
