# GitHub 팀 협업 완전 가이드

> **대상**: GitHub을 처음 사용하는 팀원 7명
> **목표**: Git 설치부터 PR 머지까지, 팀 협업의 전체 흐름을 익힌다

---

## 목차

1. [사전 준비](#1-사전-준비)
2. [Git의 핵심 개념 이해하기](#2-git의-핵심-개념-이해하기)
3. [GitHub 저장소 만들고 연결하기](#3-github-저장소-만들고-연결하기)
4. [브랜치 전략 — 팀 협업의 핵심](#4-브랜치-전략--팀-협업의-핵심)
5. [일상적인 작업 흐름 (Daily Workflow)](#5-일상적인-작업-흐름-daily-workflow)
6. [Pull Request(PR) 완전 정복](#6-pull-requestpr-완전-정복)
7. [충돌(Conflict) 해결하기](#7-충돌conflict-해결하기)
8. [팀 협업 규칙 (컨벤션)](#8-팀-협업-규칙-컨벤션)
9. [자주 발생하는 문제와 해결법](#9-자주-발생하는-문제와-해결법)
10. [명령어 치트시트](#10-명령어-치트시트)

---

## 1. 사전 준비

### 1-1. Git 설치

**Windows**
1. https://git-scm.com 에서 다운로드
2. 설치 시 모든 옵션 **기본값(Next)** 으로 진행
3. 설치 확인:
```bash
git --version
# 출력 예: git version 2.43.0
```

**Mac**
```bash
# 터미널에서 실행
xcode-select --install
# 또는 Homebrew 사용
brew install git
```

### 1-2. GitHub 계정 만들기

1. https://github.com 접속
2. **Sign up** 클릭
3. 이메일, 비밀번호, 사용자명 입력
4. 이메일 인증 완료

### 1-3. Git 초기 설정 (모든 팀원 필수)

```bash
# 본인 이름과 이메일 설정 (커밋에 표시됨)
git config --global user.name "홍길동"
git config --global user.email "gildong@example.com"

# 기본 브랜치 이름을 main으로 설정
git config --global init.defaultBranch main

# 한글 파일명 깨짐 방지
git config --global core.quotepath false
```

### 1-4. GitHub 인증 설정 (SSH 키)

매번 비밀번호를 입력하지 않기 위해 SSH 키를 등록합니다.

```bash
# 1) SSH 키 생성
ssh-keygen -t ed25519 -C "gildong@example.com"
# 엔터 3번 (기본 경로, 비밀번호 없음)

# 2) 생성된 공개키 복사
#    Windows:
cat ~/.ssh/id_ed25519.pub | clip
#    Mac:
cat ~/.ssh/id_ed25519.pub | pbcopy
```

3. GitHub 접속 → 우측 상단 프로필 → **Settings**
4. 좌측 메뉴 **SSH and GPG keys** → **New SSH key**
5. 복사한 키 붙여넣기 → **Add SSH key**

```
┌─────────────────────────────────────────────────┐
│  GitHub.com  →  Settings  →  SSH and GPG keys   │
│                                                  │
│  Title: [ 내 노트북 키        ]                  │
│  Key:   [ ssh-ed25519 AAAA... ]                  │
│                                                  │
│         [ Add SSH key ]                          │
└─────────────────────────────────────────────────┘
```

6. 연결 확인:
```bash
ssh -T git@github.com
# 출력: Hi 사용자명! You've successfully authenticated
```

---

## 2. Git의 핵심 개념 이해하기

### 2-1. Git이란?

Git은 **코드의 변경 이력을 관리**하는 도구입니다.
GitHub는 이 이력을 **온라인에 저장하고 팀원과 공유**하는 서비스입니다.

```
비유하자면...

  Git    = 내 컴퓨터의 "변경 기록 장치"
  GitHub = 팀이 함께 쓰는 "클라우드 공유 폴더 + 변경 기록"
```

### 2-2. Git의 3가지 영역

파일이 거치는 3단계를 이해하면 Git의 절반은 마스터한 것입니다.

```
  ┌──────────────────────────────────────────────────────────┐
  │                     내 컴퓨터 (Local)                     │
  │                                                          │
  │   ┌─────────────┐    ┌─────────────┐    ┌────────────┐  │
  │   │  작업 디렉토리 │───→│ 스테이징 영역 │───→│   저장소    │  │
  │   │ (Working Dir)│    │  (Staging)  │    │ (Repository)│  │
  │   │             │    │             │    │            │  │
  │   │ 파일을 수정  │    │ 커밋할 파일을 │    │  변경 이력  │  │
  │   │ 하는 곳     │    │ 골라두는 곳  │    │  저장소     │  │
  │   └─────────────┘    └─────────────┘    └────────────┘  │
  │         │                   │                  │         │
  │       git add           git commit          git push     │
  │     (파일 선택)        (이력 저장)       (GitHub에 업로드)  │
  └──────────────────────────────────────────────────────────┘
                                                    │
                                                    ▼
                                          ┌──────────────┐
                                          │    GitHub     │
                                          │  (원격 저장소)  │
                                          │              │
                                          │ 팀원 모두가   │
                                          │ 접근 가능     │
                                          └──────────────┘
```

### 2-3. 핵심 용어 정리

| 용어 | 뜻 | 비유 |
|------|-----|------|
| **Repository (저장소)** | 프로젝트 폴더 + 변경 이력 | 프로젝트 바인더 |
| **Commit (커밋)** | 변경 사항의 스냅샷 저장 | "세이브 포인트" |
| **Branch (브랜치)** | 독립된 작업 공간 | "평행 우주" |
| **Merge (머지)** | 브랜치를 합치는 것 | 평행 우주 합치기 |
| **Pull Request (PR)** | 머지 전 코드 리뷰 요청 | "이거 합쳐도 될까요?" |
| **Clone (클론)** | 원격 저장소를 내 컴퓨터로 복사 | 프로젝트 다운로드 |
| **Push (푸시)** | 내 커밋을 GitHub에 업로드 | 클라우드에 백업 |
| **Pull (풀)** | GitHub의 최신 내용을 내려받기 | 최신 버전 동기화 |
| **Conflict (충돌)** | 같은 부분을 다르게 수정했을 때 | 동시 편집 충돌 |

---

## 3. GitHub 저장소 만들고 연결하기

### 3-1. 팀 리더: 저장소 생성

1. GitHub 접속 → 우측 상단 **+** → **New repository**

```
┌──────────────────────────────────────────────┐
│          Create a new repository             │
│                                              │
│  Repository name: [ ai-project            ]  │
│  Description:     [ AI 개발 프로젝트       ]  │
│                                              │
│  ● Public   ○ Private                        │
│                                              │
│  ☑ Add a README file                         │
│  Add .gitignore: [ Python ▼ ]                │
│                                              │
│         [ Create repository ]                │
└──────────────────────────────────────────────┘
```

> **.gitignore** 는 Git이 무시할 파일 목록입니다.
> AI 프로젝트라면 **Python** 템플릿을 선택하면
> `__pycache__/`, `.env`, `*.pyc` 등이 자동으로 제외됩니다.

### 3-2. 팀 리더: 팀원 초대

1. 저장소 페이지 → **Settings** 탭
2. 좌측 **Collaborators** → **Add people**
3. 팀원의 GitHub 사용자명 또는 이메일로 초대

```
┌──────────────────────────────────────────────┐
│  Settings  →  Collaborators                  │
│                                              │
│  Manage access                               │
│  ┌────────────────────────────────────────┐  │
│  │  [ Add people ]                        │  │
│  │                                        │  │
│  │  팀원A (gildong)        ✅ Accepted     │  │
│  │  팀원B (chulsoo)        ⏳ Pending      │  │
│  │  팀원C (younghee)       ⏳ Pending      │  │
│  │  ...                                   │  │
│  └────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
```

4. 초대받은 팀원은 **이메일** 또는 **GitHub 알림**에서 수락

### 3-3. 모든 팀원: 저장소 Clone (다운로드)

```bash
# 프로젝트를 저장할 폴더로 이동
cd ~/projects

# 저장소 복제 (SSH 주소 사용)
git clone git@github.com:팀리더계정/ai-project.git

# 프로젝트 폴더로 이동
cd ai-project
```

이제 모든 팀원의 컴퓨터에 동일한 프로젝트가 준비되었습니다!

---

## 4. 브랜치 전략 — 팀 협업의 핵심

### 4-1. 왜 브랜치가 필요한가?

7명이 하나의 파일을 동시에 수정하면 충돌이 끊임없이 발생합니다.
**브랜치**를 사용하면 각자 독립된 공간에서 작업한 뒤 안전하게 합칠 수 있습니다.

```
브랜치가 없을 때 (위험!)
─────────────────────────

  팀원A 수정 ──┐
  팀원B 수정 ──┼──→  충돌! 충돌! 충돌!
  팀원C 수정 ──┘


브랜치가 있을 때 (안전!)
─────────────────────────

  main ─────────────────────────────────────────→
    │                          ↑          ↑
    ├── feature/login ────────→┘ (PR+머지)│
    │                                     │
    └── feature/model-training ──────────→┘ (PR+머지)
```

### 4-2. 7인 팀 추천 브랜치 구조

```
main (배포용, 항상 안정적인 상태)
 │
 └── develop (개발 통합 브랜치)
      │
      ├── feature/data-preprocessing    (팀원A: 데이터 전처리)
      ├── feature/model-training        (팀원B: 모델 학습)
      ├── feature/api-server            (팀원C: API 서버)
      ├── feature/frontend              (팀원D: 프론트엔드)
      ├── feature/database              (팀원E: DB 설계)
      ├── feature/auth                  (팀원F: 인증 기능)
      └── feature/deployment            (팀원G: 배포 설정)
```

**규칙:**
- `main` : 배포 가능한 안정 코드만 존재. **직접 커밋 금지!**
- `develop` : 팀원들의 기능이 합쳐지는 통합 브랜치
- `feature/*` : 각 팀원이 기능을 개발하는 브랜치

### 4-3. 브랜치 보호 설정 (팀 리더)

main과 develop 브랜치에 실수로 직접 push하는 것을 방지합니다.

GitHub 저장소 → **Settings** → **Branches** → **Add branch protection rule**

```
┌───────────────────────────────────────────────────────┐
│  Branch protection rule                               │
│                                                       │
│  Branch name pattern: [ main ]                        │
│                                                       │
│  ☑ Require a pull request before merging              │
│    ☑ Require approvals: [ 1 ▼ ]                       │
│                                                       │
│  ☑ Require status checks to pass before merging       │
│                                                       │
│  ☑ Do not allow bypassing the above settings          │
│                                                       │
│         [ Create ]                                    │
└───────────────────────────────────────────────────────┘
```

→ `develop` 브랜치에도 동일하게 설정합니다.

---

## 5. 일상적인 작업 흐름 (Daily Workflow)

매일 코드를 작성할 때 따라야 하는 순서입니다.
아래 흐름을 **반복**하면 됩니다.

```
 ┌─────────────────────────────────────────────────────────────┐
 │                    하루의 작업 흐름                           │
 │                                                             │
 │   ① 최신 코드 받기 (pull)                                    │
 │          │                                                  │
 │          ▼                                                  │
 │   ② 내 브랜치에서 작업                                       │
 │          │                                                  │
 │          ▼                                                  │
 │   ③ 변경 사항 저장 (add → commit)                            │
 │          │                                                  │
 │          ▼                                                  │
 │   ④ GitHub에 올리기 (push)                                   │
 │          │                                                  │
 │          ▼                                                  │
 │   ⑤ 기능 완성 시 → PR 생성                                   │
 │          │                                                  │
 │          ▼                                                  │
 │   ⑥ 코드 리뷰 → 승인 → 머지                                  │
 └─────────────────────────────────────────────────────────────┘
```

### 단계별 명령어

#### ① 최신 코드 받기
```bash
# develop 브랜치의 최신 변경 사항 가져오기
git checkout develop
git pull origin develop
```

#### ② 내 브랜치 만들기 / 이동하기
```bash
# 새 기능 브랜치 만들기 (처음 한 번만)
git checkout -b feature/model-training

# 이미 만든 브랜치로 이동
git checkout feature/model-training

# develop의 최신 내용을 내 브랜치에 반영
git merge develop
```

#### ③ 작업 후 변경 사항 저장
```bash
# 어떤 파일이 변경되었는지 확인
git status

# 변경된 파일 선택 (스테이징)
git add train.py                # 특정 파일만
git add src/                    # 특정 폴더 전체
git add train.py utils.py       # 여러 파일

# 커밋 (저장)
git commit -m "feat: 학습 모델에 Early Stopping 추가"
```

#### ④ GitHub에 올리기
```bash
git push origin feature/model-training
```

#### ⑤~⑥ PR은 다음 섹션에서 자세히 다룹니다!

---

## 6. Pull Request(PR) 완전 정복

### 6-1. PR이란?

PR은 **"내가 작업한 코드를 develop(또는 main)에 합쳐주세요"** 라는 요청입니다.
바로 합치지 않고, **다른 팀원이 코드를 검토(리뷰)** 한 뒤 합칩니다.

```
PR이 필요한 이유
────────────────

  혼자 코드를 합치면...          PR을 거치면...
  ┌──────────────┐              ┌──────────────┐
  │   버그가      │              │   팀원이      │
  │   그대로      │              │   버그를      │
  │   들어감!     │              │   발견!       │
  └──────────────┘              └──────────────┘

  "이 변수 이름이                 "여기 에러 처리가
   왜 a인지 아무도                 빠져있네요~"
   모름..."                      "변수명 바꾸면
                                  더 좋겠어요!"
```

### 6-2. PR 생성 방법

**방법 1: GitHub 웹에서 생성 (추천)**

push 후 GitHub 저장소 페이지에 노란 배너가 나타납니다:

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ 🟡 feature/model-training had recent pushes          │  │
│  │                                                      │  │
│  │              [ Compare & pull request ]               │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
│  ───────────── 클릭하면 아래 화면이 나옵니다 ─────────────   │
│                                                            │
│  Open a pull request                                       │
│                                                            │
│  base: develop  ←  compare: feature/model-training         │
│  (합칠 대상)         (내 브랜치)                              │
│                                                            │
│  Title:                                                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ feat: 학습 모델에 Early Stopping 기능 추가             │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
│  Description:                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ ## 변경 사항                                          │  │
│  │ - EarlyStopping 콜백 구현                              │  │
│  │ - patience, min_delta 파라미터 지원                     │  │
│  │ - 테스트 코드 추가                                     │  │
│  │                                                      │  │
│  │ ## 테스트                                             │  │
│  │ - [x] 단위 테스트 통과                                 │  │
│  │ - [x] 기존 학습 파이프라인과 호환 확인                   │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
│  Reviewers: [ 팀원A, 팀원B ]                                │
│                                                            │
│          [ Create pull request ]                           │
└────────────────────────────────────────────────────────────┘
```

**방법 2: 터미널에서 생성**

```bash
# GitHub CLI(gh)가 설치되어 있다면
gh pr create --base develop --title "feat: Early Stopping 추가" --body "설명..."
```

### 6-3. 코드 리뷰 하는 법

PR 페이지 → **Files changed** 탭에서 변경된 코드를 확인합니다.

```
┌──────────────────────────────────────────────────────────────┐
│  Files changed (3)                                           │
│                                                              │
│  src/training/early_stopping.py                              │
│  ─────────────────────────────────────────                   │
│    10  │ + class EarlyStopping:                               │
│    11  │ +     def __init__(self, patience=5):                │
│    12  │ +         self.patience = patience                   │
│    13  │ +         self.counter = 0                           │
│    14  │ +         self.best_loss = None   ← 💬 클릭해서      │
│        │                                     코멘트 작성!     │
│                                                              │
│  리뷰 코멘트 예시:                                             │
│  ┌──────────────────────────────────────────────────────┐    │
│  │ best_loss 초기값을 float('inf')로 하면                │    │
│  │ None 체크 없이 바로 비교할 수 있을 것 같아요!           │    │
│  │                                                      │    │
│  │  ○ Comment  ● Request changes  ○ Approve             │    │
│  │                                                      │    │
│  │         [ Submit review ]                            │    │
│  └──────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
```

**리뷰 옵션 설명:**
| 옵션 | 의미 | 언제 사용? |
|------|------|-----------|
| **Comment** | 일반 의견 | 가벼운 질문이나 제안 |
| **Request changes** | 수정 요청 | 반드시 고쳐야 할 문제 발견 시 |
| **Approve** | 승인 | 문제없이 합쳐도 될 때 |

### 6-4. PR 머지 (합치기)

리뷰어가 **Approve** 하면 머지할 수 있습니다.

```
┌─────────────────────────────────────────────────┐
│                                                 │
│  ✅ 1 approving review                          │
│  ✅ All checks passed                           │
│                                                 │
│  ┌─────────────────────────────────────┐        │
│  │  [ Squash and merge ▼ ]             │        │
│  │                                     │        │
│  │    ● Squash and merge   (추천!)     │        │
│  │    ○ Create a merge commit          │        │
│  │    ○ Rebase and merge               │        │
│  └─────────────────────────────────────┘        │
│                                                 │
│  ☑ Delete branch after merge                    │
│                                                 │
└─────────────────────────────────────────────────┘
```

> **Squash and merge** 를 추천합니다!
> 여러 개의 작은 커밋이 하나로 합쳐져서 이력이 깔끔해집니다.

### 6-5. PR 전체 흐름 요약

```
  ┌──────┐     ┌──────┐     ┌──────┐     ┌──────┐     ┌──────┐
  │      │     │      │     │      │     │      │     │      │
  │ Push │────→│  PR  │────→│ 리뷰  │────→│ 승인  │────→│ 머지  │
  │      │     │ 생성  │     │      │     │      │     │      │
  └──────┘     └──────┘     └──────┘     └──────┘     └──────┘
                  │              │
                  │         수정 요청 시
                  │              │
                  │         ┌──────┐
                  │         │ 코드  │
                  └─────────│ 수정  │
                            │ 후   │
                            │ Push │
                            └──────┘
```

---

## 7. 충돌(Conflict) 해결하기

### 7-1. 충돌은 왜 발생하나?

두 사람이 **같은 파일의 같은 줄**을 다르게 수정하면 Git이 어느 쪽을 선택해야 할지 모릅니다.

```
  팀원A가 수정:                    팀원B가 수정:
  ──────────────                  ──────────────
  learning_rate = 0.001           learning_rate = 0.01
       │                               │
       └──────────┬────────────────────┘
                  │
                  ▼
           Git: "둘 중 뭘 써야 하지?!"
                  │
                  ▼
              ⚠️ CONFLICT!
```

### 7-2. 충돌 해결 순서

```bash
# 1) develop 최신 내용을 내 브랜치에 합치기 시도
git merge develop

# 2) 충돌 발생 시 이런 메시지가 나옴
# CONFLICT (content): Merge conflict in src/config.py
# Automatic merge failed; fix conflicts and then commit the result.
```

충돌이 발생한 파일을 열면 이렇게 표시됩니다:

```python
# src/config.py

<<<<<<< HEAD (내 브랜치의 코드)
learning_rate = 0.001
=======
learning_rate = 0.01
>>>>>>> develop (develop 브랜치의 코드)
```

**해결 방법**: 둘 중 하나를 선택하거나, 합쳐서 수정합니다.

```python
# 수정 후 (팀원과 상의하여 결정)
learning_rate = 0.001  # 팀 회의에서 결정한 값
```

> `<<<<<<<`, `=======`, `>>>>>>>` 표시를 **모두 제거**해야 합니다!

```bash
# 3) 수정 완료 후
git add src/config.py
git commit -m "fix: config.py 충돌 해결 (learning_rate 값 통일)"
```

### 7-3. 충돌 예방 팁

```
  ┌──────────────────────────────────────────────────┐
  │              충돌을 줄이는 습관                     │
  │                                                  │
  │  1. 매일 아침 develop의 최신 코드를 pull 하기       │
  │  2. 작은 단위로 자주 커밋 & PR 하기                 │
  │  3. 같은 파일을 동시에 수정하지 않도록 역할 분담      │
  │  4. 설정 파일(config)은 한 사람이 관리              │
  │  5. 작업 시작 전 팀 채팅방에서 "OO 파일 수정합니다"  │
  │     라고 알리기                                   │
  └──────────────────────────────────────────────────┘
```

---

## 8. 팀 협업 규칙 (컨벤션)

### 8-1. 커밋 메시지 규칙

일관된 커밋 메시지는 나중에 이력을 찾기 쉽게 해줍니다.

**형식:**
```
<타입>: <제목>

<본문 (선택)>
```

**타입 목록:**
| 타입 | 의미 | 예시 |
|------|------|------|
| `feat` | 새로운 기능 | `feat: 로그인 API 구현` |
| `fix` | 버그 수정 | `fix: 학습 시 메모리 누수 해결` |
| `docs` | 문서 변경 | `docs: README에 설치 방법 추가` |
| `style` | 코드 포맷팅 | `style: 들여쓰기 통일` |
| `refactor` | 리팩토링 | `refactor: 데이터 로더 구조 개선` |
| `test` | 테스트 추가 | `test: 모델 정확도 테스트 추가` |
| `chore` | 설정, 빌드 | `chore: requirements.txt 업데이트` |

**좋은 예 vs 나쁜 예:**
```
❌ "수정"
❌ "update"
❌ "asdfasdf"

✅ "feat: 사용자 회원가입 폼 구현"
✅ "fix: 이미지 업로드 시 확장자 검증 누락 수정"
✅ "docs: API 엔드포인트 목록 문서화"
```

### 8-2. 브랜치 이름 규칙

```
feature/기능명         →  feature/login-page
fix/버그설명           →  fix/memory-leak
docs/문서내용          →  docs/api-documentation
refactor/대상          →  refactor/data-loader
```

### 8-3. PR 규칙

팀에서 정할 것들:

```
  ┌──────────────────────────────────────────────────┐
  │               우리 팀의 PR 규칙                    │
  │                                                  │
  │  1. PR 크기: 한 PR에 변경 300줄 이하 권장           │
  │     (너무 크면 리뷰가 어렵습니다)                    │
  │                                                  │
  │  2. 리뷰어: 최소 1명 이상 Approve 필요              │
  │                                                  │
  │  3. PR 설명에 반드시 포함할 것:                     │
  │     - 무엇을 변경했는지                            │
  │     - 왜 변경했는지                                │
  │     - 어떻게 테스트했는지                           │
  │                                                  │
  │  4. 리뷰 응답 시간: 24시간 이내                     │
  │                                                  │
  │  5. 본인 PR은 본인이 머지                           │
  │     (리뷰 승인 받은 후)                             │
  └──────────────────────────────────────────────────┘
```

### 8-4. 프로젝트 폴더 구조 (AI 프로젝트 예시)

```
ai-project/
├── README.md                  # 프로젝트 소개 및 시작 가이드
├── requirements.txt           # Python 패키지 목록
├── .gitignore                 # Git 제외 파일 목록
├── .env.example               # 환경 변수 예시 (.env는 Git 제외!)
│
├── data/                      # 데이터 폴더 (대용량은 Git 제외)
│   ├── raw/                   # 원본 데이터
│   └── processed/             # 전처리된 데이터
│
├── src/                       # 소스 코드
│   ├── data/                  # 데이터 처리 (팀원A)
│   │   ├── preprocessing.py
│   │   └── augmentation.py
│   ├── models/                # 모델 정의 (팀원B)
│   │   ├── base_model.py
│   │   └── train.py
│   ├── api/                   # API 서버 (팀원C)
│   │   ├── routes.py
│   │   └── middleware.py
│   └── utils/                 # 공통 유틸리티
│       └── logger.py
│
├── tests/                     # 테스트 코드
│   ├── test_data.py
│   └── test_model.py
│
├── notebooks/                 # Jupyter 노트북 (실험용)
│   └── exploration.ipynb
│
└── configs/                   # 설정 파일
    ├── model_config.yaml
    └── server_config.yaml
```

### 8-5. .gitignore 필수 항목

```gitignore
# Python
__pycache__/
*.pyc
*.pyo
venv/
.venv/

# 환경 변수 (API 키 등 민감 정보!)
.env

# 데이터 (대용량)
data/raw/
*.csv
*.parquet

# 모델 파일 (대용량)
*.pt
*.pth
*.h5
*.onnx
models/checkpoints/

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db

# Jupyter
.ipynb_checkpoints/
```

> **주의!** API 키, 비밀번호 등 민감 정보는 **절대** Git에 올리지 마세요!
> `.env` 파일에 저장하고, `.env.example` 에는 형식만 올립니다.

---

## 9. 자주 발생하는 문제와 해결법

### Q1. "커밋 메시지를 잘못 썼어요"

```bash
# 직전 커밋 메시지만 수정 (아직 push 안 했을 때)
git commit --amend -m "feat: 올바른 커밋 메시지"
```

### Q2. "커밋하면 안 되는 파일을 커밋했어요"

```bash
# .env 파일을 실수로 커밋한 경우
git rm --cached .env
echo ".env" >> .gitignore
git add .gitignore
git commit -m "fix: .env 파일을 Git 추적에서 제거"
```

### Q3. "작업 중인데 급하게 브랜치를 바꿔야 해요"

```bash
# 현재 작업을 임시 저장
git stash

# 다른 브랜치로 이동
git checkout develop

# 작업 후 돌아와서 임시 저장한 내용 복원
git checkout feature/my-feature
git stash pop
```

### Q4. "내 브랜치가 develop보다 너무 뒤처졌어요"

```bash
git checkout feature/my-feature
git merge develop
# 충돌이 있으면 7장의 방법으로 해결
```

### Q5. "push가 rejected 됐어요"

```bash
# 원격에 내 로컬에 없는 커밋이 있을 때 발생
# 먼저 pull로 최신 내용을 받은 뒤 다시 push
git pull origin feature/my-feature
git push origin feature/my-feature
```

### Q6. "방금 한 커밋을 취소하고 싶어요"

```bash
# 커밋은 취소하되, 변경한 코드는 유지 (안전한 방법)
git reset --soft HEAD~1
```

### Q7. "다른 팀원의 브랜치를 내 컴퓨터에서 보고 싶어요"

```bash
git fetch origin
git checkout feature/팀원의-브랜치
```

---

## 10. 명령어 치트시트

작업 상황별로 필요한 명령어를 빠르게 찾아보세요.

### 처음 시작할 때

```bash
git clone git@github.com:계정/저장소.git   # 저장소 복제
cd 저장소                                   # 폴더 이동
```

### 매일 반복하는 명령어

```bash
git status                                  # 현재 상태 확인 (자주 사용!)
git pull origin develop                     # 최신 코드 받기
git checkout -b feature/기능명              # 새 브랜치 만들기
git add 파일명                              # 파일 스테이징
git commit -m "feat: 설명"                  # 커밋
git push origin feature/기능명              # GitHub에 올리기
```

### 브랜치 관련

```bash
git branch                                  # 브랜치 목록 보기
git checkout 브랜치명                        # 브랜치 이동
git checkout -b 새브랜치명                   # 새 브랜치 만들고 이동
git branch -d 브랜치명                       # 브랜치 삭제
git merge develop                           # develop을 현재 브랜치에 합치기
```

### 확인 / 조회

```bash
git log --oneline                           # 커밋 이력 한 줄씩 보기
git log --oneline --graph                   # 브랜치 그래프로 보기
git diff                                    # 변경 내용 확인
git diff --staged                           # 스테이징된 변경 확인
```

### 되돌리기 (주의해서 사용)

```bash
git stash                                   # 작업 임시 저장
git stash pop                               # 임시 저장 복원
git reset --soft HEAD~1                     # 마지막 커밋 취소 (코드 유지)
git checkout -- 파일명                       # 파일 수정 취소 (저장 전)
```

---

## 부록: 빠른 시작 체크리스트

아래 체크리스트를 팀원 모두가 완료하면 협업 준비 끝!

```
팀원 개인 체크리스트
────────────────────
[ ] Git 설치 완료
[ ] GitHub 계정 생성
[ ] git config (이름, 이메일) 설정
[ ] SSH 키 생성 및 GitHub에 등록
[ ] 저장소 clone 완료
[ ] 테스트 브랜치 만들어보기
[ ] 테스트 커밋 & push 해보기
[ ] PR 만들어보기

팀 리더 체크리스트
────────────────────
[ ] GitHub 저장소 생성
[ ] 팀원 7명 초대 완료
[ ] develop 브랜치 생성
[ ] main, develop 브랜치 보호 규칙 설정
[ ] .gitignore 설정
[ ] README.md 작성
[ ] 폴더 구조 초기 설정
[ ] 팀 컨벤션 문서 공유
```

---

## 부록: 유용한 도구

| 도구 | 용도 | 링크 |
|------|------|------|
| **GitHub Desktop** | Git을 GUI로 사용 | desktop.github.com |
| **VS Code** | 에디터 + Git 통합 | code.visualstudio.com |
| **GitLens (VS Code 확장)** | 코드에서 바로 Git 이력 확인 | VS Code 확장 마켓 |

> **VS Code** 를 사용하면 터미널 명령어 없이도
> 왼쪽 사이드바의 Source Control 패널에서 Git 작업을 할 수 있습니다.
> 처음에 명령어가 어려우면 VS Code의 GUI를 활용하세요!

---

**이 가이드에 대한 질문이 있으면 팀 채팅방에서 언제든 물어보세요!**
**처음엔 누구나 헷갈립니다. 실수해도 괜찮아요 — Git은 되돌릴 수 있으니까요.**
