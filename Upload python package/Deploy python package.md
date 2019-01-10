# 나만의 Python package 

이 가이드는 나만의 python package로 설계하고 PyPI(Python Package Index)에 업로드하는 방법에 대한 가이드이다. 

> PyPI(Python Package Index) : 파이썬 관련 패키지들이 저장된 저장소



## Create package

- Python package로 만들기 위한 프로젝트 폴더는 다음과 같은 구조로 구성된다. 

  ```bash
  <calcpkg> 
    ├── __init__.py
    ├── calculator.py
  LICENSE
  setup.py
  README.md
  ```


- Python에서는 프로젝트 폴더 안에 `__init__.py`  파일이 존재하면 해당 폴더를 패키지로 인식하는데 

  다음과 같이 작성한다.

  ```bash
  __all__ = ['calculator']
  ```

  > Note
  >
  > `__all__` 이라는 목록을 통해 package의 색인을 명시적으로 제공할 수 있다. 즉, 사용자가 `from packge_name import *` 라고 정의할 때, `__all__` 목록에 제공된 모듈을 import 한다. 



- `__init__.py` 파일을 작성하였으면, 모듈 파일을 작성한다. 다음은 간단한 사칙연산이 가능한 예제 코드이다. 

  ```bash
  def add(a,c):
      return a + c
  
  def substract(a,c):
      return a - c
  
  def multiply(a,c):
      return a * c
  
  def divide(a,c):
      return a / c
  
  def calculator(a,b,c):
      value = 0
  
      if b == "+":
          value = add(a,c)
      elif b == "-":
          value = substract(a,c)
      elif b == "*":
          value = multiply(a,c)
      elif b == "/":
          value = divide(a,c)
      return value
  ```




## Deploy package

위에서 생성한 Package를 배포하기 위해서 다음과 같은 절차를 따른다. 



__1. Create setup.py __

Python에서는 `setuptools` 패키지를 통해 프로젝트의 빌드, 배포 과정을 쉽게 관리할 수 있는데, `setup.py` 는 이를 위한 메타데이터가 포함된 스크립트이고 다음은 `setup.py` 파일에 대한 예시이고 유동적으로 작성할 수 있다.  

```python
from setuptools import setup, find_packages

with open("README.md", "r") as fh:
    long_description = fh.read()
   
setup(
    name="calcpkg",
    version="0.1.0",
    author="cyy03576",
    author_email="limban32@nate.com",
    description="Example Package",
    packages=find_packages(),
    license="MIT",
    python_requires='>=2.7,!=3.0.*,!=3.1.*,!=3.2.*,!=3.3.*',
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/Cyy92",
    install_requires=[
        'module_name>=version',
    ]
    classifiers=[
        "License :: OSI Approved :: MIT License",
    ],
)
```

- Metadata

  다음은 setup 모듈에 포함되는 대표적인 metadata에 대한 목록이다. 

  |      Field       |                   Description                   |
  | :--------------: | :---------------------------------------------: |
  |       name       |                  package 이름                   |
  |     version      |                package 배포 버전                |
  |      author      |                 package 작성자                  |
  |   author_email   |              package 작성자 email               |
  |   description    |               package에 대한 설명               |
  |     packages     |       프로젝트에 포함되는 package 리스트        |
  |     license      |               package의 라이센스                |
  | python_requires  |         실행 환경에 필요한 python 버전          |
  |       url        |          package를 대표하는 웹 페이지           |
  | install_requires |    자동으로 설치되는 python dependency 목록     |
  | dependency_links | Pypi에 존재하지 않는 외부 dependency를 위한 URL |


__2. Create LICENSE__

PyPI에 package를 업로드하고 이를 배포하기 위해서는 반드시 라이센스가 필요하다.  다음의 [링크](https://choosealicense.com/)에서 라이센스를 선택한 후, 해당 라이센스에 대한 문구를 복사하여 `LICENSE` 파일을 생성한다. 

```bash
Copyright (c) 2019 Package Example
  
Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```



__3. Create README.md__

Package에 대한 자세한 설명을 mark down으로 생성한 후, `setup.py` 에서 참조할 수 있다. 



__4.  Install setuptools & wheel__

- Python 2.7

  ```bash
  pip install setuptools
  pip install wheel
  ```

- Python 3.x ~

  ```bash
  python3 -m pip install --user --upgrade setuptools wheel
  ```



__5. Build__

- Python 2.7

  ```bash
  python setup.py bdist_wheel
  ```

- Python 3.x ~

  ```bash
  python3 setup.py sdist bdist_wheel
  ```

다음과 같이 빌드를 하게 되면 dist라는 폴더가 생성되고, dist 폴더는 다음과 같은 구조를 가진다. 

- Python 2.7

  ```bash
  <dist>
    ├── calcpkg-0.1.0-py2-none-any.whl
  ```

- Python 3.x ~ 

  ```bash
  <dist>
    ├── calcpkg-0.1.0-py3-none-any.whl
    ├── calcpkg-0.1.0.tar.gz
  ```



__6. Deploy package__

PyPI에 package를 업로드하고 배포하기 위해서는 우선 PyPI 계정이 필요하다. 공식 [홈페이지](https://pypi.org/)에서 계정을 생성한 후, `twine`을 통해 개인 repository에 package를 업로드하면 된다. 

> twine : PyPI에 python package를 업로드할 때, 보안성과 검증 가능성을 높이고자 사용되는 유틸리티



먼저, `twine`을 설치한다. 

- Python 2.7

  ```bash
  pip install twine
  ```

- Python 3.x ~

  ```bash
  python3 -m pip install --user --upgrade twine
  ```



`twine` 설치가 완료되면, 이를 통해 PyPI에 업로드한다. 업로드할 때, PyPI 계정을 필요로 한다.

```bash
twine upload dist/*
Enter your username: 
Enter your password:
```



위의 명령어 실행 후, 다음과 같은 화면이 표시되면 성공적으로 업로드를 마친 것이다. 

```bash
Uploading calcpkg-0.1.0-py2-none-any.whl
100%|                                             | 7.32k/7.32k [00:05<00:00, 1.30kB/s]
```



## Package instructions

package 업로드까지 완료되었으면, 업로드한 package를 설치하여 사용할 수 있고  이를 통해 사칙연산을 진행할 수 있다. 




- Install package

  - Python 2.7

    ```bash
    pip install calcpkg
    ```

  - Python 3.x ~

    ```bash
    python3 -m pip install calcpkg
    ```


- Create main program

  Package의 설치까지 완료되면, 이 package를 import하여 main program을 작성하면 된다.

  ```python
  from calcpkg.calculator import calculator
  
  if __name__ == '__main__':
      addresult = calculator(10,"+",11)
      subresult = calculator(23,"-",9)
      mulresult = calculator(78,"*",29)
      divresult = calculator(48,"/",8)
      print("Add result = %d" % addresult)
      print("Sub result = %d" % subresult)
      print("Mul result = %d" % mulresult)
      print("Div result = %d" % divresult)
  ```

  main 파일 작성 후 실행하면, 다음과 같은 결과를 얻을 수 있다.

  - Python 2.7

    ```bash
    python test.py
    
    -> Add result = 21
       Sub result = 14
       Mul result = 2262
       Div result = 6
    ```

  - Python 3.x ~

    ```bash
    python3 test.py
    
    -> Add result = 21
       Sub result = 14
       Mul result = 2262
       Div result = 6
    ```





