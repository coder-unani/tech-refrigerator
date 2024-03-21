# Git 명령어 활용

## git init

```bash
# git local repository 초기화
git init

# git 원격 저장소 추가
git remote add origin git@github.com:coder-unani/tech.refrigerator.git

# git 원격 저장소 확인
git remote -v
```

## git branch

```bash
# 브랜치 생성
git branch unani

# 해당 브랜치로 이동
git checkout unani

# 해당 브랜치의 상세정보
git branch -v 

# 모든 브랜치 리스트
git branch
(확인 후 :q 누르고 종료)

# 원격 브랜치 목록
git branch -a

# 브랜치 삭제
git branch -d

# 원격 브랜치 삭제
git push origin -d unani

# git master branch가 github에 올라가지 않을 경우
git reset --hard origin/master 
```

## git push/pull

```bash
# git push
git push origin master

# git pull 
git pull origin master

# 강제로 push/pull
git (push or pull) origin master --force 
```

## git add

```bash
# add
git add .
git add <파일 이름>
```

## git commit

```bash
# git commit
git commit -m "메세지 입력"
```

## git merge

```bash
# master에 체크아웃 
git checkout master 

# unani 브랜치를 master에 합침
git merge unani

# git branch끼리 연결이 끊어져 history 문제로 merge가 안 될 경우
git merge unani --allow-unrelated-histories

# 현재 진행중인 merge 내역 초기화
git reset --merge
```

```bash
# merge 충돌 해결하기
# git은 충돌 내용을 하단과 같이 코드 상에 보여준다.
&&<<<<<<< HEAD
{현재 브랜치의 다른 파일 내용}
=======
{충돌나는 브랜치명 또는 commit에서의 다른 파일 내용}
>>>>>>> 충돌나는 브랜치명 또는 commmit 아이디
# 예시
<<<<<<< HEAD
master content -> 현재 브랜치[master]에서 수정된 내용 
=======
test content -> 머지할 브랜치[test]에서 수정한 내용 
>>>>>>> 충돌나는 브랜치명 또는 commmit 아이디

## 충동 안나도록 merge 하는 요령 ##
# 대상 브랜치로 이동
git checkout master

# 대상 브런치의 로컬 최신화
git pull origin master

# 다시 내 작업 브랜치로 이동
git checkout unani

# 머지 요청
git merge master

# 수정 후, add, commit, push 진행 
```

## git rebase

```bash
# git commit 내역 확인
git log --pretty=format:"%h = %an , %ar : %s" --graph

# git commit 내역 수정
git rebase -i -r <해시값>

# git 모든 커밋내역 선택 수정
git rebase -i -r --root

# 수정 할 커밋 해시값 앞에 pick => edit 또는 e로 수정
# :wq 저장하고 종료

# 작성자,이메일 수정시
git commit --amend --author "UNANI [coder.unani@gmail.com](<mailto:coder.unani@gmail.com>)"

# 필요한 내용 수정하고 :wq 저장하고 종료

# 선택한 다음 commit 수정
git rebase --continue
```

## git tag

```bash
# git tag 추가 
git tag -a v1.0 -m "tag messages" 

# git tag 원격저장소 추가 
(모든 tag) 
git push origin master --tags  
git push origin --tags  
(특정 tag) 
git push origin v1.0(태그명)
```



# 