# 지옥에서 온 Git 1주차 - 버전관리의 본질

1주차에서는 버전관리의 본질에 대해서 학습하였습니다. 버전관리 시스템을 관통하는 본질적인 기능, 즉 Git을 설치하는 방법부터 시작해서 기본적인 명령어 숙지 및 사용법에 대해서 학습하였습니다. 



## 1. Git 설치 & 실습

- Linux & Unix 기반

  OS에 맞게 다음의 명령어를 통해 설치합니다.



  __Ubuntu__

  ```bash
  sudo apt-get install git
  ```

  __Cent OS__

  ```bash
  sudo yum install git
  ```



- 다음의 [링크](https://codeonweb.com/)에서 회원 가입 후, 실습을 진행할 수 있습니다.

  ![그림1](https://github.com/Cyy92/user-images/blob/master/git-fth/1st-week/practice-img.png?raw=true)

__실습 절차__

1. `실습` 카테고리 선택 후, 
2. 하단에 `git` 언어 및 버전을 선택합니다.
3. 언어 및 버전 선택까지 완료하면, 터미널에 명령어를 입력해 실습을 진행합니다.



## Git 저장소 만들기

버전 관리 시, 여러 가지 정보가 담기는 디렉토리를 만들어주어야 하는데, 이 때 생성되는 디렉토리를 우리는 저장소라고 명칭할 수 있습니다. 



- 프로젝트 폴더 생성

  다음의 명령어를 통해 git 저장소를 생성할 프로젝트 폴더를 만들어줍니다. 

  ```bash
  mkdir gitfth
  ```

- git 저장소 생성

  프로젝트 폴더로 이동 후, `git init`이라는 명령어를 통해 저장소를 생성해서 git에게 해당 폴더의 버전을 관리하겠다는 것을 알려줍니다.

  ```bash
  cd gitfth
  git init
  ```

  명령어를 통해 저장소를 생성하였으면, `ls -al`이라는 명령어를 통해 저장소의 생성 여부를 확인할 수 있습니다.

  ![그림 2](https://github.com/Cyy92/user-images/blob/master/git-fth/1st-week/git-init.PNG?raw=true)



## Git이 관리할 대상으로 파일 등록

Git은 기본적으로 새로운 파일을 관리하지 않습니다. 파일을 관리하기 위해서는 관리 대상으로 파일을 등록해야 합니다.  이는 프로젝트 개발 중, 개발에 꼭 필요한 핵심 파일들, 즉 버전 관리되어야 하는 파일들과 테스트 등의 목적으로 생성된 임시적인 파일들을 명확하게 구분하기 위해서입니다. 

1. 먼저 다음의 명령어를 통해 임의의 텍스트 파일을 생성하고 파일 안에 내용을 작성해줍니다.

   ```bash
   vim f1.txt
   ```

2. `git status`를 통해 프로젝트 폴더의 상태를 확인합니다. 

   ```bash
   git status
   ```

   ![그림 3](https://github.com/Cyy92/user-images/blob/master/git-fth/1st-week/git-status.PNG?raw=true)

   그림에서처럼, Untracked files에 1번에서 생성한 f1.txt 파일이 빨간색으로 표시되는 것을 확인할 수 있습니다. Git은 버전 관리되고 있는 디렉토리에 특정 파일이 생성되었을 때, 이를 추적하도록 명령 받기 전까지는 이 파일의 존재를 무시합니다. 우리는 1번에서 생성한 파일의 추적을 명령하지 않았기 때문에 그림과 같이 생성한 파일은 빨간색으로 표시되는 것입니다. 

3. Git이 파일을 추적하도록 명령합니다.

   ```bash
   git add f1.txt
   ```

   ![그림 4](https://github.com/Cyy92/user-images/blob/master/git-fth/1st-week/git-add.PNG?raw=true)

   파일을 추적하도록 명령하면, 그림과 같이 Git은 f1.txt 파일을 버전 관리되고 있는 프로젝트 폴더의 새로운 파일로 인식합니다. 이와 같이 우리는 `git addd`명령어를 통해 git이 관리할 대상으로 파일을 등록할 수 있습니다. 



## 버전 만들기 (commit)

다음은 버전을 실제로 생성하고 이를 저장하는 방법입니다. 또한 이를 통해 과거의 버전으로 돌아갈 수 있고 파일 변경 사항의 이력을 확인할 수 있습니다. 

> 버전이란, 특정 작업의 완결된 상태를 뜻합니다.



- 버전을 만들기 전에, 버전을 만든 사람에 대한 정보를 설정합니다. 이는 사용자에게 버전 정보를 제공하기 위함입니다. 

  ```bash
  git config --global user.name "자신의 닉네임"
  git config --global user.email "자신의 이메일"
  ```

- 버전 정보를 설정한 후에, 다음의 명령어를 통해 버전을 만들어줍니다. 

  ```bash
  git commit
  ```

  이 명령어를 실행하면, 그림과 같이 vim이 실행되고 우리는 commit 메세지를 작성할 수 있습니다.

  > 버전의 변화, 즉 파일이 변경된 이유를 명시하기 위한 메세지를 commit 메세지라고 합니다.

  ![그림 5](https://github.com/Cyy92/user-images/blob/master/git-fth/1st-week/commit-message.PNG?raw=true)

  commit 메세지 작성 후 저장하면, 다음과 같이 버전이 생성됩니다.

  ![그림 6](https://github.com/Cyy92/user-images/blob/master/git-fth/1st-week/create-version.PNG?raw=true)

  생성한 버전에 대한 정보를 확인하기 위해선 다음의 명령어를 실행합니다. 

  ```bash
  git log
  ```

- `git commit`을 통해 버전을 생성한 후, 특정 파일을 수정한 경우 다음과 같이 git은 해당 파일을 무시합니다.

  ![그림 7](https://github.com/Cyy92/user-images/blob/master/git-fth/1st-week/modified-files.PNG?raw=true)

  때문에 반드시 위에서 언급했던 `git add`명령어를 통해 수정된 파일을 추적하도록 명령해주어야 합니다. Git에게 추적 명령을 내리면 git은 다음과 같이 수정된 파일을 다시 추적하게 됩니다.

  ![그림 8](https://github.com/Cyy92/user-images/blob/master/git-fth/1st-week/modified-files(2).PNG?raw=true)

  수정된 파일을 추적하기 시작하면, `git commit`명령어를 통해 commit 메세지 작성 후, 새로운 버전을 생성할 수 있고 `git log`를 통해 새로운 버전에 대한 정보를 확인할 수 있습니다. 



## Stage area

Git은 commit 전에 반드시 add를 꼭 해야 합니다. `git add`를 통해 특정 파일을 commit 대기 상태로 만들고,  `git commit`을 통해 대기 상태인 파일을 버전에 포함시킵니다. 그리고 우리는 commit 대기 상태를 __Stage area__라고 명칭합니다.

즉, `git add f1.txt`라는 명령어를 입력하면, __f1.txt__ 파일은 __Stage__에 올라가게 되고, __Stage__에 올려진 __f1.txt__ 파일은 `git commit`을 통해 저장소에 저장됩니다. 

![그림9](https://github.com/Cyy92/user-images/blob/master/git-fth/1st-week/stage-architecture.png?raw=true)



## 변경사항 확인하기

버전 관리를 하는 가장 중요한 효용은 수정된 내용을 추적해서 문제해결을 하는데 이용하기 위해서라고 할 수 있습니다. 버전관리의 효용은 크게 두 가지로 나눌 수 있는데 그 중 하나는 버전간의 차이점을 확인할 수 있다는 것입니다. 다음은 이를 확인하기 위한 방법에 대한 설명입니다.



1. __로그에서 출력되는 버전 간 차이점__

   위에서 살펴본 `git log` 명령어에 `-p` 옵션을 통해 확인할 수 있습니다.

   ```bash
   git log -p
   ```

   ![그림10](https://github.com/Cyy92/user-images/blob/master/git-fth/1st-week/git-log-p.png?raw=true)

   그림에서처럼 위의 명령어를 통해 구체적인 버전 간 차이점을 확인할 수 있습니다. 

   - 로그를 통해 __---__와 __+++__로 현재 시점의 버전과 이전의 버전을 구분할 수 있습니다. 
   - `--- a/f2.txt`는 이전 버전에 대한 텍스트 파일이고, `+++ b/f2.txt`는 현재 버전에 대한 텍스트 파일이며 세부적으로는 - / + 로 어떤 내용이 바뀌었는지 구분할 수 있습니다. 즉, 이전 버전 파일의 내용인 __source : 2__가 __f2.txt : 2__로 바뀐 것을 확인할 수 있습니다. 

2. __로그를 사용하지 않은 버전 간 차이점__

   ![그림11](https://github.com/Cyy92/user-images/blob/master/git-fth/1st-week/git-diff.png?raw=true)

   Commit은 고유한 자신의 ID가 존재합니다. 다음은 commit의 ID를 이용하여 버전 간 차이점을 확인할 수 있는 방법입니다. 

   ```bash
   git diff '버전 id1'..'버전 id2' 
   ```

   ![그림12](https://github.com/Cyy92/user-images/blob/master/git-fth/1st-week/git-diff-res.PNG?raw=true)

   뿐만 아니라 commit을 하기 전, 이전 commit과의 차이점을 확인하고자 할 경우 버전 id를 제외한 `git diff` 명령어를 사용할 수 있습니다. 이는 수정된 내용이 방대한 경우, 버전 관리를 조금 더 효율적으로 하기 위함입니다.  



## 과거의 버전으로 돌아가기

버전 관리의 두 번째 효용은 과거의 상태로 돌아갈 수 있다는 점입니다. commit을 취소하여 과거의 상태로 돌아갈 수 있는데, 크게 두 가지 방법이 있습니다. 

1. __Reset__

   ![그림12](https://github.com/Cyy92/user-images/blob/master/git-fth/1st-week/git-reset.PNG?raw=true)

   돌아가고자하는 commit의 id를 이용해 다음의 명령어를 입력하면 과거의 버전으로 되돌릴 수 있습니다. 

   ```bash
   git reset '버전 id' --hard
   ```

   예를 들어 그림에서처럼 다섯 번째 commit에서 세 번째 commit으로 되돌리고자 할 때, 되돌리고자 하는 commit의 id인 "1578c856...3ee38f" 를 '버전 id'에 기입하면 됩니다. 

2. __Revert__

   Revert는 reset처럼 commit을 취소하여 삭제하는 것과는 달리 취소한 commit을 새로운 버전으로 만드는 명령어입니다. 

   ```bash
   git revert '버전 id'
   ```



## 명령의 빈도와 메뉴얼 보는 방법

- 명령의 빈도수

  ![그림13](https://github.com/Cyy92/user-images/blob/master/git-fth/1st-week/frequency.PNG?raw=true)

- 메뉴얼 보는 방법

  Git 명령어 사용이 익숙하지 않은 경우, 다음의 옵션을 통해 자세한 메뉴얼을 확인할 수 있습니다. 메뉴얼을 확인하는 것은 어떠한 문제에 직면해서 그 문제를 해결하고자 할 때, 핵심적인 요소로 작용될 수 있습니다. 

  ```bash
  git '명령어' --help
  ```


