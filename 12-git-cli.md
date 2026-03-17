---
layout: default
title: 12. Git CLI 사용법
nav_order: 13
---

# 12. Git CLI 사용법

[← 목차로 돌아가기](index)

---

## Git이란?

**Git**은 소스 코드의 변경 이력을 관리하는 **분산 버전 관리 시스템**입니다.
파일의 변경 내용을 추적하고, 여러 사람이 동시에 작업할 수 있게 해줍니다.

```
작업 흐름 개요

  작업 디렉토리       스테이징 영역         로컬 저장소        원격 저장소
  (Working Dir)  →   (Staging Area)  →   (Local Repo)  →  (Remote Repo)
                      git add              git commit        git push

  변경한 파일을       커밋할 파일를           변경 이력을         GitHub 등에
  수정하는 곳         선택하는 곳            기록하는 곳          업로드하는 곳
```

---

## 목차
- [기본 설정](#기본-설정)
- [저장소 초기화 / 복제](#저장소-초기화--복제)
- [ADD — 변경 파일 스테이징](#add--변경-파일-스테이징)
- [COMMIT — 변경 이력 저장](#commit--변경-이력-저장)
- [PUSH — 원격 저장소에 업로드](#push--원격-저장소에-업로드)
- [PULL / FETCH — 원격 변경사항 가져오기](#pull--fetch--원격-변경사항-가져오기)
- [BRANCH — 브랜치 관리](#branch--브랜치-관리)
- [CHECKOUT / SWITCH — 브랜치 이동](#checkout--switch--브랜치-이동)
- [MERGE — 브랜치 합치기](#merge--브랜치-합치기)
- [CHERRY-PICK — 특정 커밋만 가져오기](#cherry-pick--특정-커밋만-가져오기)
- [STASH — 작업 임시 저장](#stash--작업-임시-저장)
- [LOG — 커밋 이력 확인](#log--커밋-이력-확인)
- [DIFF — 변경 내용 비교](#diff--변경-내용-비교)
- [RESET / REVERT — 되돌리기](#reset--revert--되돌리기)
- [REBASE — 커밋 정리](#rebase--커밋-정리)
- [TAG — 버전 태그](#tag--버전-태그)
- [실전 워크플로우](#실전-워크플로우)

---

## 기본 설정

처음 Git을 사용할 때 이름과 이메일을 등록합니다.

```bash
# 사용자 이름 설정
git config --global user.name "홍길동"

# 이메일 설정
git config --global user.email "hong@example.com"

# 기본 브랜치 이름을 main으로 설정
git config --global init.defaultBranch main

# 설정 확인
git config --list

# 줄바꿈 설정 (Windows)
git config --global core.autocrlf true
# 줄바꿈 설정 (Mac/Linux)
git config --global core.autocrlf input
```

---

## 저장소 초기화 / 복제

```bash
# 새 저장소 초기화 (빈 폴더를 Git 저장소로 만들기)
git init
git init my-project        # 폴더 생성 + 초기화 동시에

# 원격 저장소 복제 (GitHub → 내 컴퓨터로 가져오기)
git clone https://github.com/user/repo.git

# 특정 폴더 이름으로 복제
git clone https://github.com/user/repo.git my-folder

# 특정 브랜치만 복제
git clone -b develop https://github.com/user/repo.git

# 원격 저장소 연결 (init 후 GitHub와 연결)
git remote add origin https://github.com/user/repo.git

# 연결된 원격 저장소 확인
git remote -v
```

---

## ADD — 변경 파일 스테이징

`git add`는 커밋할 파일을 스테이징 영역에 올리는 명령어입니다.

```bash
# 특정 파일만 추가
git add index.html
git add src/main.py

# 여러 파일 한 번에 추가
git add index.html style.css main.js

# 특정 폴더 전체 추가
git add src/
git add docs/

# 현재 디렉토리의 변경된 파일 전부 추가
git add .

# 모든 변경 파일 추가 (삭제된 파일 포함)
git add -A

# 변경된 내용을 대화형으로 선택하여 추가 (일부만 스테이징)
git add -p index.html

# 스테이징 상태 확인
git status
```

```
상태 표시 읽는 법:
  Changes to be committed     → 스테이징된 파일 (초록색)
  Changes not staged          → 수정됐지만 미스테이징 (빨간색)
  Untracked files             → Git이 추적하지 않는 새 파일 (빨간색)
```

---

## COMMIT — 변경 이력 저장

`git commit`은 스테이징된 변경사항을 로컬 저장소에 저장합니다.

```bash
# 기본 커밋 (메시지 입력 창이 열림)
git commit

# 메시지를 바로 입력하여 커밋
git commit -m "로그인 기능 추가"

# 스테이징 + 커밋 동시에 (새 파일 제외, 추적 중인 파일만)
git commit -am "버그 수정: 로그인 오류 해결"

# 직전 커밋 수정 (메시지 또는 파일 추가)
git add forgotten-file.txt
git commit --amend -m "로그인 기능 추가 (누락 파일 포함)"
```

### 좋은 커밋 메시지 작성법

```bash
# ✅ 좋은 예 — 무엇을 왜 했는지 명확히
git commit -m "fix: 로그인 시 세션 만료 오류 수정

- 토큰 갱신 로직에서 race condition 발생 문제 해결
- 만료 10분 전 자동 갱신 추가"

# 타입 접두사 활용
git commit -m "feat: 다크모드 지원 추가"
git commit -m "fix: 결제 금액 계산 오류 수정"
git commit -m "docs: README 설치 가이드 업데이트"
git commit -m "refactor: 인증 모듈 구조 개선"
git commit -m "test: 장바구니 유닛 테스트 추가"
git commit -m "chore: 패키지 의존성 업데이트"

# ❌ 나쁜 예
git commit -m "수정"
git commit -m "asdf"
git commit -m "작업중"
```

---

## PUSH — 원격 저장소에 업로드

`git push`는 로컬 커밋을 원격 저장소(GitHub 등)에 올립니다.

```bash
# 기본 push (현재 브랜치 → 연결된 원격 브랜치)
git push

# 처음 push할 때 원격 브랜치와 연결하며 push
git push -u origin main
git push -u origin feature/login

# 특정 브랜치를 원격에 push
git push origin main
git push origin develop

# 로컬 브랜치를 다른 이름으로 push
git push origin local-branch:remote-branch

# 원격 브랜치 삭제
git push origin --delete feature/old-branch

# 태그 push
git push origin v1.0.0       # 특정 태그
git push origin --tags       # 모든 태그
```

> **주의**: `git push --force`는 원격 이력을 덮어쓰므로 팀 프로젝트에서는 사용하지 마세요.

---

## PULL / FETCH — 원격 변경사항 가져오기

### FETCH — 가져오되 합치지 않기

```bash
# 원격 저장소의 변경사항을 가져오기만 함 (내 파일은 그대로)
git fetch origin

# 특정 브랜치만 fetch
git fetch origin main

# 모든 원격 저장소 fetch
git fetch --all

# fetch 후 차이 확인
git diff origin/main
git log origin/main..HEAD   # 내가 push 안 한 커밋 확인
```

### PULL — 가져오고 바로 합치기

```bash
# fetch + merge를 한 번에 (기본)
git pull

# 특정 원격/브랜치에서 pull
git pull origin main

# merge 대신 rebase 방식으로 pull (깔끔한 이력 유지)
git pull --rebase origin main

# pull 전 로컬 변경사항이 있으면 stash 후 pull
git stash
git pull
git stash pop
```

```
fetch vs pull 차이:

  git fetch   →  원격 변경사항 확인만 (안전)
  git pull    →  fetch + 자동 merge (편리)

  협업 시 fetch로 먼저 확인 후 merge하는 것이 안전합니다.
```

---

## BRANCH — 브랜치 관리

브랜치는 독립적인 작업 공간입니다. 기능 개발, 버그 수정을 격리하여 진행합니다.

```bash
# 브랜치 목록 보기
git branch              # 로컬 브랜치
git branch -r           # 원격 브랜치
git branch -a           # 로컬 + 원격 전체

# 새 브랜치 생성
git branch feature/login
git branch bugfix/cart-error

# 브랜치 생성 + 즉시 이동
git checkout -b feature/login
git switch -c feature/login   # 최신 방식

# 브랜치 이름 변경
git branch -m old-name new-name

# 로컬 브랜치 삭제
git branch -d feature/login        # merge된 경우만 삭제
git branch -D feature/login        # 강제 삭제

# 원격 브랜치 삭제
git push origin --delete feature/login
```

### 브랜치 전략 (Git Flow)

```
main          ─────●─────────────────●──────  (배포)
                   │                 ↑
develop       ─────●──●──●───────────●──────  (통합)
                      │  ↑           ↑
feature/login ────────●──●           │        (기능 개발)
                                     │
hotfix        ───────────────────────●──────  (긴급 수정)
```

---

## CHECKOUT / SWITCH — 브랜치 이동

```bash
# 브랜치 이동 (구 방식)
git checkout main
git checkout develop
git checkout feature/login

# 브랜치 이동 (신 방식, Git 2.23+)
git switch main
git switch develop

# 이전 브랜치로 돌아가기
git checkout -
git switch -

# 브랜치 생성 + 이동
git checkout -b feature/signup
git switch -c feature/signup

# 원격 브랜치를 로컬로 가져와서 이동
git checkout -b feature/login origin/feature/login
git switch --track origin/feature/login

# 특정 커밋 시점으로 파일 복구
git checkout abc1234 -- src/main.py   # 특정 커밋의 파일로 복구
git checkout HEAD -- src/main.py      # 최신 커밋 상태로 복구
```

---

## MERGE — 브랜치 합치기

```bash
# feature 브랜치를 main에 합치기
git checkout main
git merge feature/login

# 병합 커밋 없이 선형으로 합치기 (Fast-forward)
git merge --ff-only feature/login

# 항상 병합 커밋 생성 (이력 추적에 유리)
git merge --no-ff feature/login

# 충돌(conflict) 발생 시 해결 과정
# 1. 충돌 파일 확인
git status

# 2. 파일을 열어 충돌 부분 수동 수정
# <<<<<<< HEAD
# 내 코드
# =======
# 상대방 코드
# >>>>>>> feature/login

# 3. 수정 후 스테이징
git add 충돌파일.py

# 4. 병합 완료
git commit

# 병합 취소 (충돌 해결 중 포기할 때)
git merge --abort
```

---

## CHERRY-PICK — 특정 커밋만 가져오기

다른 브랜치의 **특정 커밋 하나**만 현재 브랜치에 적용합니다.
hotfix를 main과 develop 양쪽에 적용할 때 자주 사용합니다.

```bash
# 커밋 해시 확인
git log --oneline

# 특정 커밋 하나만 가져오기
git cherry-pick abc1234

# 여러 커밋 가져오기
git cherry-pick abc1234 def5678

# 범위로 가져오기 (start 제외, end 포함)
git cherry-pick abc1234..def5678

# 커밋하지 않고 스테이징 상태로만 가져오기
git cherry-pick --no-commit abc1234

# 충돌 발생 시
git cherry-pick abc1234
# → 충돌 파일 수정 →
git add .
git cherry-pick --continue

# cherry-pick 취소
git cherry-pick --abort
```

### cherry-pick 활용 예시

```
상황: develop 브랜치의 버그 수정 커밋을 main에도 적용해야 할 때

1. develop 브랜치에서 커밋 해시 확인
   git log --oneline develop
   → abc1234 fix: 결제 오류 수정

2. main 브랜치로 이동
   git checkout main

3. 해당 커밋만 가져오기
   git cherry-pick abc1234

4. main에 push
   git push origin main
```

---

## STASH — 작업 임시 저장

커밋하기는 싫고, 브랜치를 이동해야 할 때 작업 내용을 임시로 저장합니다.

```bash
# 현재 변경사항 임시 저장
git stash

# 메시지와 함께 저장
git stash save "로그인 기능 작업 중"

# 새 파일(untracked)도 포함하여 저장
git stash -u
git stash --include-untracked

# stash 목록 보기
git stash list
# stash@{0}: On main: 로그인 기능 작업 중
# stash@{1}: On develop: 임시 저장

# 가장 최근 stash 복원 (stash는 삭제됨)
git stash pop

# 특정 stash 복원
git stash pop stash@{1}

# stash 적용만 (stash는 유지)
git stash apply stash@{0}

# stash 삭제
git stash drop stash@{0}   # 특정 stash 삭제
git stash clear            # 전체 삭제

# stash 내용 확인
git stash show stash@{0}
git stash show -p stash@{0}  # 상세 diff
```

---

## LOG — 커밋 이력 확인

```bash
# 기본 로그
git log

# 한 줄로 간략히 보기
git log --oneline

# 브랜치 그래프와 함께
git log --oneline --graph --all

# 특정 작성자의 커밋만
git log --author="홍길동"

# 특정 파일의 변경 이력
git log --oneline -- src/main.py

# 날짜 범위 지정
git log --after="2025-01-01" --before="2025-03-01"

# 커밋 메시지로 검색
git log --grep="로그인"

# 최근 N개만 보기
git log -5

# 변경된 파일 목록도 함께 표시
git log --stat

# 상세 diff도 함께 표시
git log -p
```

```bash
# 예쁜 로그 출력 (커스텀 포맷)
git log --oneline --graph --all --decorate

# 결과 예시:
# * abc1234 (HEAD -> main) feat: 다크모드 추가
# * def5678 fix: 로그인 오류 수정
# | * ghi9012 (feature/signup) feat: 회원가입 기능
# |/
# * jkl3456 init: 초기 커밋
```

---

## DIFF — 변경 내용 비교

```bash
# 스테이징 전 변경사항 확인 (작업 디렉토리 vs 마지막 커밋)
git diff

# 스테이징된 변경사항 확인 (스테이징 vs 마지막 커밋)
git diff --staged
git diff --cached

# 특정 파일만 비교
git diff src/main.py

# 두 브랜치 비교
git diff main..feature/login

# 두 커밋 비교
git diff abc1234 def5678

# 변경된 파일 이름만 보기
git diff --name-only
git diff --name-status     # 상태(M/A/D)도 함께
```

---

## RESET / REVERT — 되돌리기

### RESET — 로컬에서 되돌리기 (주의 필요)

```bash
# 스테이징만 취소 (파일 내용은 유지)
git reset HEAD index.html
git restore --staged index.html   # 최신 방식

# 마지막 커밋을 취소하고 스테이징 상태로
git reset --soft HEAD~1

# 마지막 커밋을 취소하고 작업 디렉토리 상태로 (파일 내용 유지)
git reset --mixed HEAD~1
git reset HEAD~1          # mixed가 기본값

# ⚠️ 마지막 커밋과 파일 내용 모두 삭제 (되돌리기 불가)
git reset --hard HEAD~1

# 특정 커밋 시점으로 되돌리기
git reset --hard abc1234
```

### REVERT — 협업 시 안전하게 되돌리기

```bash
# 특정 커밋의 변경사항을 되돌리는 새 커밋 생성
git revert abc1234

# 커밋 메시지 입력 창 없이 바로 revert
git revert abc1234 --no-edit

# 여러 커밋 revert
git revert abc1234 def5678

# revert 결과를 커밋하지 않고 스테이징만
git revert --no-commit abc1234
```

```
reset vs revert 선택 기준:

  git reset   →  아직 push하지 않은 로컬 커밋을 지울 때
  git revert  →  이미 push한 커밋을 안전하게 되돌릴 때 (협업 필수)
```

---

## REBASE — 커밋 정리

```bash
# main 브랜치를 기준으로 현재 브랜치 커밋 재정렬
git rebase main

# 충돌 발생 시
git rebase main
# → 충돌 파일 수정 →
git add .
git rebase --continue

# rebase 취소
git rebase --abort

# 최근 3개 커밋을 대화형으로 정리 (squash, 순서 변경 등)
git rebase -i HEAD~3
```

```
대화형 rebase 명령:
  pick   → 커밋 유지
  reword → 커밋 메시지 수정
  squash → 이전 커밋에 합치기 (메시지 선택)
  fixup  → 이전 커밋에 합치기 (메시지 버림)
  drop   → 커밋 삭제
```

---

## TAG — 버전 태그

배포 버전 등 중요한 시점을 태그로 표시합니다.

```bash
# 태그 목록 보기
git tag

# 현재 커밋에 태그 추가 (lightweight)
git tag v1.0.0

# 메시지 있는 태그 (annotated, 권장)
git tag -a v1.0.0 -m "정식 배포 버전 1.0.0"

# 특정 커밋에 태그 추가
git tag -a v1.0.0 abc1234

# 태그 정보 보기
git show v1.0.0

# 태그 원격에 push
git push origin v1.0.0       # 특정 태그
git push origin --tags       # 모든 태그

# 태그 삭제
git tag -d v1.0.0                     # 로컬
git push origin --delete v1.0.0       # 원격
```

---

## 실전 워크플로우

### 1. 새 기능 개발 흐름

```bash
# 1. 최신 main 가져오기
git checkout main
git pull origin main

# 2. 기능 브랜치 생성
git checkout -b feature/dark-mode

# 3. 작업 후 커밋
git add .
git commit -m "feat: 다크모드 토글 버튼 추가"
git add .
git commit -m "feat: 다크모드 CSS 스타일 적용"

# 4. 원격에 push
git push -u origin feature/dark-mode

# 5. GitHub에서 Pull Request 생성 → 리뷰 → merge

# 6. 작업 완료 후 브랜치 정리
git checkout main
git pull origin main
git branch -d feature/dark-mode
```

---

### 2. 긴급 버그 수정 (Hotfix) 흐름

```bash
# 1. main에서 hotfix 브랜치 생성
git checkout main
git checkout -b hotfix/login-crash

# 2. 수정 후 커밋
git add .
git commit -m "fix: 로그인 시 앱 크래시 수정"

# 3. main에 merge
git checkout main
git merge --no-ff hotfix/login-crash
git push origin main

# 4. develop에도 적용 (cherry-pick)
git checkout develop
git cherry-pick $(git log hotfix/login-crash --oneline -1 | awk '{print $1}')
git push origin develop

# 5. 브랜치 정리
git branch -d hotfix/login-crash
```

---

### 3. 자주 쓰는 명령어 요약

```bash
# 현재 상태 확인 (가장 자주 사용)
git status
git log --oneline -10

# 변경 → 스테이징 → 커밋 → push
git add .
git commit -m "메시지"
git push

# 브랜치 생성 + 이동
git checkout -b feature/기능명

# 원격 변경사항 가져오기
git pull origin main

# 작업 임시 저장 후 브랜치 이동
git stash
git checkout other-branch
git stash pop

# 실수한 커밋 되돌리기 (push 전)
git reset --soft HEAD~1
```

---

### 4. 자주 하는 실수와 해결법

| 상황 | 해결 방법 |
|------|----------|
| 파일을 잘못 스테이징했을 때 | `git restore --staged 파일명` |
| 커밋 메시지를 잘못 썼을 때 (push 전) | `git commit --amend -m "새 메시지"` |
| 잘못된 브랜치에 커밋했을 때 | `git cherry-pick`으로 올바른 브랜치에 적용 후 `git reset` |
| .gitignore에 추가할 파일을 이미 커밋했을 때 | `git rm --cached 파일명` 후 커밋 |
| pull 충돌이 났을 때 | 충돌 파일 수정 후 `git add .` → `git commit` |
| 원격 브랜치가 삭제됐는데 로컬에 남아있을 때 | `git fetch --prune` |

---

[← 이전: BigQuery 함수](11-functions) | [← 목차로 돌아가기](index)
