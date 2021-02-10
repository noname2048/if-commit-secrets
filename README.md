# if-commit-secrets
만약 시크릿 키나 API_KEY를 github에 노출시키면 어떻게 해야할까에 대해 실험

비공개 정보를 올렸으나 git rm --cached 로 삭제하여도 해당 커밋에 정보가 남는다

그러니까... 이런일이 닥치면 어떻게 해야할까?



실험조건 (아래 번호당 한개의 커밋)

1. README.md 추가
2. some_project.py 추가
3. secrets.json 추가
4. some_project.py 변경
5. secrets.json 을 git rm 으로 삭제
6. .gitignore 변경

![Imgur](https://i.imgur.com/p9mBIzM.png)



1. ### git filter-branch

   무언가 쉽고 편하게 secrets.json만 바꿀 수 있을 것 같은 느낌이라 열심히 문서를 읽고 써봤는데 명령어 사용할때 부터 git filter-repo를 사용하라고 권장하고, 조심해서 사용하도록 겁을 많이 준다. 또, 공식 문서에는 이미 push된 커밋에 대해서는 사용하지 말라고 되었다. 

   즉, 이미 secrets가 push된 상황에서 고려할 수 있는 쉬운 해결법은 아니였던것 같다. git filter-repo 는 apt 에는 search 되지 않아서 pip install git-filter-repo로 설치 하였는데, 관련 github readme 가 읽기 쉽게 짜여지기는 커녕 공식 문서는 man page 형태로 담겨 있어, 많은 git에 대한 기초 지식을 요구 하였다. 

   정리하면 이 명령어를 사용하면 모든 커밋이 재작성 되는 형태이기 때문에, push를 적용하지 않은 로컬 커밋에서 다시 커밋을 모두 재작성하여, merge해서 사용해야 한다.

   = 사용하기 힘들어보인다. 러닝커브도 높다.

2. ### git reset HEAD~3 & git push -f origin

   오히려 직관적인 방법이 더 잘 통하는 것 같다. 아예 커밋을 secrets.json이 추가되기전으로 되돌린 다음에, secrets key 에 대해 다시 한번 확실히 검사하고 진행하는 편이 빨라 보인다.

   = 직관적

아래는 대략적인 사용했던 명령어들에 대해 정리

git filter-branch와 filter-repo 관련

```bash
> git filter-branch --index-filter 'git rm --cached --ignore-unmatch secrets.json' HEAD
WARNING: git-filter-branch has a glut of gotchas generating mangled history
	 rewrites.  Hit Ctrl-C before proceeding to abort, then use an
	 alternative filtering tool such as 'git filter-repo'
	 (https://github.com/newren/git-filter-repo/) instead.  See the
	 filter-branch manual page for more details; to squelch this warning,
	 set FILTER_BRANCH_SQUELCH_WARNING=1.
Proceeding with filter-branch...

Rewrite 3619dbd9d63facff76af057ad4aa6d9d07b13fc5 (3/6) (0 seconds passed, remaining 0 predicted)    rm 'secrets.json'
Rewrite 083465f030f66674adcad73be61b511003f27c98 (4/6) (0 seconds passed, remaining 0 predicted)    rm 'secrets.json'
Rewrite c65f09c523949b2694195016fce2308854b50506 (6/6) (0 seconds passed, remaining 0 predicted)    
Ref 'refs/heads/try-remove-secrets' was rewritten
```

```
> git filter-repo --invert-paths --path 'secrets.json' --use-base-name
```

vscode extentions의 git graph로 해당 내역을 확인한 결과

![img](https://i.imgur.com/uzdgP2a.png)

처럼 아예 뿌리가 달라진다.



## 결론

1. master에는 결과물을 신중히 올리자.
2. 만약 비공개 정보가 올라가면 로컬에서 브런치로 빠르게 코드를 저장하고
3. main이나 master에 reset 명령어 + -f push로 커밋을 내리고
4. 신중히 merge하여 다시 push 하자

