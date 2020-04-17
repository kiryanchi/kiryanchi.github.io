---
layout: post
title: Pyinstaller "pyi_rth_pkgres”, "valueerror" 에러 해결법
date: 2020-04-11
category: python
tags: ['pyinstaller', 'pyi_rth_pkgres', 'valueerror', 'python']
permalink: '/blog/:categories/:title'
---

Pyinstaller "pyi_rth_pkgres”, "valueerror" 에러 해결법
===


### 프로그램 작성 당시 개발환경
> Windows 10  
> Pycharm Professional Edition  
> Python 3.8.2 venv

삼촌을 도와 간단한 알바를 하던 중 [엑셀에 사진을 넣어야 하는 단순 반복 작업](https://github.com/kiryanchi/excelphotoautomatic/)이 있어서 Python을 사용해 프로그램을 만든 후,

Pyinstaller 를 사용해 exe 파일로 만들려고 했다.

```
pyinstaller --onefile main.py
```

그러나 뿜어오는 Error Message

```
Failed to execute script pyi_rth_pkgres
```

구글신의 힘을 빌려 검색을 해보니 pyinstaller 삭제 후, dev 버전을 다운받으라고 했다.

```
pip uninstall pyinstaller
pip install https://github.com/pyinstaller/pyinstaller/archive/develop.zip
```

그러나 이번엔 다른 문제에 부딪혔다.

```
valueerror: can't mix absolute and relative paths
```

대충 해석하자면 절대경로와 상대경로를 섞어쓸 수 없다는 내용인데,

구글에 검색해도 마땅히 해결법을 찾지 못 해 이리저리 자료를 찾던 중 [pyinstaller github](https://github.com/pyinstaller/pyinstaller)에서 정답을 찾았다.

#### 문제는 바로 내 **_파이썬 버전의 문제_**

 현재 2020년 4월 11일자 파이썬 최신 버전은 3.8.2 버전이다.

그러나 pyinstaller 는 파이썬 버전 3.7까지 지원한다.

구글 검색을 통해 파이썬 3.7 버전을 설치한 뒤 파이참에서 새 프로젝트를 파이썬 3.7 버전으로 가상환경을 설치한 뒤 다시 pyinstaller 를 설치하고 exe 파일을 만들었더니 정상 작동을 확인했다.

혹시나 macOS 에서 파이썬 버전을 관리하고 싶다면 homebrew 를 사용하는 편이 낫다.

# 결론  
[pyinstaller github](https://github.com/pyinstaller/pyinstaller) 에 들어가서 지원하는 파이썬 버전을 확인하고 내 파이썬 버전과 비교해보자.
