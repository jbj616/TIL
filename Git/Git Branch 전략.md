## Git Branch 전략

프로젝트를 할 때마다 git flow를 사용하여 개발을 진행하였다고 말하고 다녔는데 정작 본인을 git branch 전략에 대해 자세히 알고 있지 않다고 깨달았다( 단순히 branch 만들고 pr날리고 merge하는 정도...)



### 왜 git branch를 사용해야할까?

#### Decentralized but centralized

![](./img/git_flow_1.png)

origin이라는 곳에 모든 개발자가 push 하고 pull을 받으면서 중앙 집중된 관계를 가질 수 있습니다. 하지만 각 개발자 또한 각자의 이름을 가지고 팀원들과 이런 관계를 유지할 수 있습니다.



#### Main Branches

- master 
  - product 준비 상태
- develop
  - 다음 release의 최신 개발 변경 사항이 반영된 주요지점
  - Integeration branch라고 함

![](./img/git_flow_2.png)



#### Support Branch

- Feature  
- Release
- Hot fix

팀들과 상호작용하면서 기능, 릴리스 준비, 오류 수정 등을 작업할 수 있는 지점

- 한정된 생명 주기를 갖음 ( 작업이 끝난 뒤 사라짐 )
- 각 branch는 특정 목적을 가지고 어떤 브랜치가 어느 부모 브랜치를 가지고 어떤 브랜치에 merge 해야하는지 규칙이 존재해야함



#### Feature branches

- From : develop
- merge back into : develop

Feature branch는 향후 릴리스를 위한 새로운 기능을 개발하는데 사용됩니다. 기능이 개발중인 동안 존재하지만 develop에 새로운 기능을 추가하기위해 합쳐지거나 버려집니다.

![](./img/git_flow_3.png)

1. 브랜치 만들기

   ```bash
   git checkout -b myfeature develop
   ```

2. 기능 통합

   ```bash
   git checkout develop
   git merge --no-ff myfeature
   git branch -d myfeature
   git push origin develop
   ```

   - `--no-ff`옵션은 기분 branch 에 히스토리에 대한 정보가 유실되지 않고 커밋을 그룹화함.

![](./img/git_flow_4.png)



#### Release Branch

- From : develop
- Merge back into: develop 과 master
- Naming convention : release-\*

Release 브랜치는 새로운 product 릴리즈를 준비하기 위함. 사소한 버그나 출시를 위한 버전, 빌드 날짜 등 메타 정보를 작업합니다. 

develop브랜치의 릴리즈 될 기능들은 릴리즈 될때까지 대기 해야함

1. 릴리즈 브랜치 만들기

   ```bash
   git checkout -b release-1.2 develop
   ./bump-version.sh 1.2
   git commit -a -m "Bumped version number to 1.2"
   ```

   bump-version.sh : 일부 파일을 변경하여 새 버전을 반영하는 셸 스크립트

   - 버그 수정은 할 수 있지만 새로운 기능은 추가하면 안됨.

2. 릴리즈 브랜치 마무리하기

   ```bash
   git checkout master
   git merge --no-ff release-1.2
   git tag -a 1.2
   ```

   > You might as well want to use the `-s` or `-u <key>` flags to sign your tag cryptographically.

3. 변경 사항 반영

   ```bash
   git checkout develop
   git merge --no-ff release-1.2
   ```

4. 브랜치 제거

   ```bash
   git branch -d release-1.2
   ```


#### Hotfix branch

- from : master
- merge back into : develop 과 master
- naming convention : hotfix-\*

계획되지 않았지만 원하지 않는 상태에서 즉시 해결해야할 버그가 발생했을 때 활용한다.

- 이 때 develop 브랜치는 작업을 계속 진행할 수 있다.

1. hotfix 브랜치 만들기

   ```bash
   git checkout -b hotfix-1.2.1 master
   ./bump-version.sh 1.2.1
   git commit -a -m "1.2.1"
   ```

2. 수정 후 

   ```bash
   git commit -m "Fixed severe porduction problem"
   ```

3. 마무리하기

   ```bash
   git checkout master
   git merge --no-ff hotfix-1.2.1
   git tag -a 1.2.1
   ```

   > You might as well want to use the `-s` or `-u <key>` flags to sign your tag cryptographically.

4. 병합하기

   ```bash
   git checkout develop
   git merge -no-ff hotix-1.2.1
   ```

   - **수정된 hotfix는 릴리스 브랜치가 존재하면 릴리스 브랜치로 merge 해야함!**

5. branch 삭제

   ```bash
   git branch -d hotfix-1.2.1
   ```



### Summary

![](./img/git_flow_0.png)



### 참고

<https://nvie.com/posts/a-successful-git-branching-model/>