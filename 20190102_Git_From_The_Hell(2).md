# 지옥에서 온 Git 2주차 - Git의 원리

2주차에서는 git의 원리에 대해서 학습하였습니다. 1주차에서 배운 기능들이 내부적으로는 어떤 메커니즘을 통해 동작하는지에 대해서 살펴보았습니다. 



## Gistory 설치

Gistory란 git을 분석하기 위한 도구입니다. 명령을 내렸을 때, git의 내부에서 일어나는 일을 분석하면서 git이 어떻게 동작하는지를 알 수 있게 도와주는 도구입니다. Gistory는 파이썬 환경에서 동작하고, 파이썬 버전에 따른 설치 방법은 다음과 같습니다.

- Python2

  ```bash
  pip install gistory
  ```

- Python3

  ```bash
  pip3 install gistory
  ```



Gistory 설치 후, git 저장소를 생성했던 프로젝트 폴더에서 `.git` 디렉토리로 이동 후, gitstory를 실행합니다.

```bash
cd .git/
gistory
```

Gistory 실행 후, `자신의 서버 IP 주소:8805`를 웹 브라우저에서 실행하면 다음과 같은 화면이 출력됩니다. 

![그림1](https://github.com/Cyy92/user-images/blob/master/git-fth/2nd-week/gistory.PNG?raw=true)

![그림2](https://github.com/Cyy92/user-images/blob/master/git-fth/2nd-week/gistory-web.PNG?raw=true)



## Git add의 원리

Git은 서로 다른 파일명을 가지는 파일들에 대해서 파일의 내용이 같다면 하나의 객체로 파일들을 관리합니다. 즉, git add로 특정 파일들을 등록할 때, 파일의 내용이 같다면 git은 이를 하나의 객체로 인식합니다. 이는 내용이 중복되는 여러 개의 파일들을 제어할 수 있는 git add의 핵심 메커니즘이라고 할 수 있습니다. 세부적으로 확인하기 위해서 다음과 같이 실습하였습니다. 

- 파일 생성 후 파일 등록

  ```bash
  vim f1.txt
  git add f1.txt
  ```

- 파일 복사 후 파일 등록

  ```bash
  cp f1.txt f2.txt
  git add f2.txt
  ```

- Gistory 확인

  ![그림3](https://github.com/Cyy92/user-images/blob/master/git-fth/2nd-week/gistory-same-filecontent.png?raw=true)

  Gistory를 통해 그림과 같이 __index__ 디렉토리와 __objects__ 디렉토리를 확인할 수 있습니다. __index__ 디렉토리에는 위에서 등록한 파일의 이름과 commit ID가 담겨져 있고, __objects__ 디렉토리에는 파일의 내용이 담겨져 있습니다. 

  그림을 살펴보면, 위에서 등록한 두 개의 파일에 대해서 파일명은 다르지만 같은 commit ID를 가지고 있고, 같은 object, 즉 파일 내용을 가리키고 있습니다. 이를 통해, 파일명이 상이해도 파일이 담고 있는 내용이 같다면 git은 하나의 객체로 묶어서 관리한다라는 사실을 알 수 있습니다. 



## Objects 파일명의 원리

Git은 파일의 내용을 기반으로 objects 디렉토리에 하위 디렉토리 및 파일을 생성합니다. Gistory에서 확인할 수 있듯이, objects 디렉토리에는 특정 문자열로 된 디렉토리와 파일이 생성됩니다. git은 이러한 objects 디렉토리의 파일명을 hash 알고리즘을 통해 도출해내는데 메커니즘은 다음과 같습니다.

![그림4](https://github.com/Cyy92/user-images/blob/master/git-fth/2nd-week/sha1-hash-algo.png?raw=true)

1. __해당 [링크](http://www.sha1-online.com/)에서 sha-1 hash 알고리즘을 통해 파일 내용의 hash 값을 도출해냅니다. __
2. __git은 얻어낸 hash 값에서 처음 두 글자를 objects 디렉토리의 하위 디렉토리, 두 글자를 제외한 나머지 문자열을 파일의 이름으로 생성합니다. __



## Commit의 원리

Commit 역시 하나의 객체, 즉 파일로 인식되어 objects 디렉토리에 저장됩니다. Commit의 내부 메커니즘은 다음과 같습니다. 



![그림5](https://github.com/Cyy92/user-images/blob/master/git-fth/2nd-week/git-commit.png?raw=true)

- Stage area에 올려진 파일에 대해서 commit message를 작성한 후 버전을 만들게 되면 gistory에서는 그림과 같이 commit에 대한 objects 디렉토리가 생성된 것을 확인할 수 있습니다.  디렉토리 내부를 살펴보면 __tree__라는 정보가 저장되어 있는데, __tree__란 commit이 일어난 시점에 프로젝트 폴더에 저장된 파일의 이름과 내용을 담고 있는 정보입니다.  



![그림6](https://github.com/Cyy92/user-images/blob/master/git-fth/2nd-week/parent-commit.png?raw=true)

- 새로운 버전을 생성하게 되면 그림과 같이 __parent__라는 정보가 추가로 저장됩니다. Commit의 핵심 메커니즘 중 하나인 __parent__란 그림에서 알 수 있듯이 이전 버전에 대한 정보입니다. 이 값을 통해 이전 버전의 파일에 대한 정보를 알 수 있습니다. 



![그림7](https://github.com/Cyy92/user-images/blob/master/git-fth/2nd-week/commit-diff.png?raw=true)

- 또 하나의 commit 메커니즘은 바로 그림과 같습니다. 앞서 설명했듯이, parent란 정보를 통해 이전 commit을 알 수 있는데, 새로 생선한 commit의 tree와 이전 commit의 tree가 다른 것을 알 수 있습니다. 이는 파일의 내용이 서로 다르기 때문인데 이를 통해 이전 commit과 새로 생성한 commit 간 파일의 차이점을 확인할 수 있습니다. 



## Status의 원리

특정 파일의 상태를 확인하기 위해 `git status`란 명령어를 입력하는데, 다음은 이 명령어가 입력되었을 때, 어떠한 원리로 동작되는지를 알아보기 위함입니다. 



![그림8](https://github.com/Cyy92/user-images/blob/master/git-fth/2nd-week/git-add-to-stagearea.PNG?raw=true)

- 특정 파일을 변경 하게 되면, 그림과 같이 git은 `git add`를 통해 stage area에 올려져 commit 대기 상태가 된 파일을 추적하게 됩니다. 



![그림9](https://github.com/Cyy92/user-images/blob/master/git-fth/2nd-week/git-status-diff.png?raw=true)

- 그 시점에서 gistory의 index 디렉토리와 이전 commit의 objects 디렉토리를 확인해보면 두 디렉토리가 담고 있는 파일의 내용이 다른 것을 확인할 수 있습니다. Git은 이 차이점을 통해 stage area에 올려진 commit 대기 상태의 파일을 인식합니다. 



![그림10](https://github.com/Cyy92/user-images/blob/master/git-fth/2nd-week/same-index-objects.png?raw=true)

- stage area에 올려진 파일에 대해서 commit message 작성 후 commit을 생성하면 그림과 같이 index 디렉토리와 commit 시점에 생성된 objects 디렉토리가 담고 있는 정보가 같은 것을 확인할 수 있습니다. 



![그림11](https://github.com/Cyy92/user-images/blob/master/git-fth/2nd-week/final-status.PNG?raw=true)

- 두 개의 디렉토리가 담고 있는 정보에 차이점이 없기 때문에 git은 그림과 같이 더 이상 commit할 파일이 없다고 인식하게 되는 것입니다. 