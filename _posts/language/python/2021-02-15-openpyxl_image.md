---
layout: post
title: openpyxl 원하는 사진 삭제
date: 2021-02-15
category: python
tags: ['openpyxl', 'image', 'python']
permalink: '/blog/:categories/:title'
---

## 계기

Python 에서 엑셀을 다루기 위해 openpyxl 이라는 라이브러리가 존재한다.

엑셀을 다루기 위한 거의 모든 기능이 구현되어 있지만 한가지 문제를 발견했다.

바로 원하는 셀의 사진을 삭제할 수 없다는 것이다.

## 정보

그래서 파워구글링을 해서 찾아낸 정보는 다음과 같다.

```python
wb['worksheet']._images

del wb['worksheet']._images [1]

wb.save('wb.xlsx')
```

출처 - [https://stackoverflow.com/questions/54574398/find-images-and-delete-images-in-excel-using-openpyxl](https://stackoverflow.com/questions/54574398/find-images-and-delete-images-in-excel-using-openpyxl)

즉, sheet 안에 _images 배열이 있고 그 안에 이미지 정보가 들어있다는 것이다.

그리고 공식문서에서 다음 정보를 발견했다.

```python
...
class Image(object):
    """Image in a spreadsheet"""

    _id = 1
    _path = "/xl/media/image{0}.{1}"
    anchor = "A1"

    def __init__(self, img):

        self.ref = img
...
```

출처 - [https://openpyxl.readthedocs.io/en/stable/_modules/openpyxl/drawing/image.html#Image](https://openpyxl.readthedocs.io/en/stable/_modules/openpyxl/drawing/image.html#Image)

보면 anchor 멤버에 cell 정보가 있다. 이를 조합해서 바로 기능을 구현해봤다.

## 구현

### 첫 시도

```python
import openpyxl

wb = openpyxl.load_workbook('test.xlsx')

sheet = wb['Sheet1']

# 시트에 들어있는 이미지들을 복사해서 리스트로 만든다.
# 이미지를 삭제할 때, 리스트가 변경되어 for 문에 영향을 주는것을 방지하기 위함
copy_sheet_images = sheet._images[:]

# 어느 셀에 무슨 사진이 있는지 쉽게 알 수 있게 {key: 셀, value: 이미지 정보} 인 딕셔너리를 만듬
for image in copy_sheet_images:
    anchor = image.anchor
		print(anchor)
...

TypeError: can only concatenate tuple (not "list") to tuple
```

?? 분명 anchor 가 있는데 TypeError 가 뜬다.

어떻게 돌아가는지 라이브러리를 뜯어봤다.

오랜 시간이 흐르고 (하루를 다 썼다) 별 소득없이 하루를 마무리하기위해 맥북앞에 앉았는데

openpyxl.drawing.spreadsheet_drawing 에서 해답을 발견했다.

```python
...
def _check_anchor(obj):
    """
    Check whether an object has an existing Anchor object
    If not create a OneCellAnchor using the provided coordinate
    """
    anchor = obj.anchor
    if not isinstance(anchor, _AnchorBase):
        row, col = coordinate_to_tuple(anchor.upper())
        anchor = OneCellAnchor()

				# 이 아래 부분
        anchor._from.row = row -1
        anchor._from.col = col -1
				# 이 윗 부분

        if isinstance(obj, ChartBase):
            anchor.ext.width = cm_to_EMU(obj.width)
            anchor.ext.height = cm_to_EMU(obj.height)
        elif isinstance(obj, Image):
            anchor.ext.width = pixels_to_EMU(obj.width)
            anchor.ext.height = pixels_to_EMU(obj.height)
    return anchor
...
```

anchor._from.row 와 anchor._from.col 을 사용한게 눈에보였다.

바로 anchor._from 을 print 해봤다.

```python
import openpyxl

wb = openpyxl.load_workbook('test.xlsx')

sheet = wb['Sheet1']

# 시트에 들어있는 이미지들을 복사해서 리스트로 만든다.
# 이미지를 삭제할 때, 리스트가 변경되어 for 문에 영향을 주는것을 방지하기 위함
copy_sheet_images = sheet._images[:]

# 어느 셀에 무슨 사진이 있는지 쉽게 알 수 있게 {key: 셀, value: 이미지 정보} 인 딕셔너리를 만듬
for image in copy_sheet_images:
    anchor = image.anchor._from
		print(anchor)
...

<openpyxl.drawing.spreadsheet_drawing.AnchorMarker object>
Parameters:
col=7, colOff=38100, row=29, rowOff=114300
<openpyxl.drawing.spreadsheet_drawing.AnchorMarker object>
Parameters:
col=9, colOff=241300, row=5, rowOff=177800
<openpyxl.drawing.spreadsheet_drawing.AnchorMarker object>
Parameters:
col=0, colOff=406400, row=66, rowOff=165100
<openpyxl.drawing.spreadsheet_drawing.AnchorMarker object>
Parameters:
col=0, colOff=508000, row=22, rowOff=0
<openpyxl.drawing.spreadsheet_drawing.AnchorMarker object>
Parameters:
col=0, colOff=901700, row=2, rowOff=88900
```

아래와같이 정상적으로 나오는 걸 확인했다. ( 사진을 5장 A열에 3장, H, J 열에 각각 1장씩 넣었다. ) 

사진에서 유추할 수 있듯 col 은 열을 의미하고 row 는 행을 의미하는 듯 하다.

또한 A3 에 사진을 넣엇는데 col=0, row=2로 나타난 것으로 보아 row + 1 을 해야 정상적인 row 가 나오는 듯 하다.

## 최종코드

```python
import openpyxl
import string

wb = openpyxl.load_workbook('test.xlsx')

sheet = wb['Sheet1']

# 시트에 들어있는 이미지들을 복사해서 리스트로 만든다.
# 이미지를 삭제할 때, 리스트가 변경되어 for 문에 영향을 주는것을 방지하기 위함
copy_sheet_images = sheet._images[:]

# 어느 셀에 무슨 사진이 있는지 쉽게 알 수 있게 {key: 셀, value: 이미지 정보} 인 딕셔너리를 만듬
_images = {}
for image in copy_sheet_images:
		row = image.anchor._from.row + 1
		col = string.ascii_uppercase[image.anchor._from.col] # string.ascii_uppercase 는 대문자를 다 모아둔 리스트
		cell = f'{col}{row}'

		_images[cell] = image
		
		# 참고. 만약 사진을 저장하고 싶다면, _data 를 담으면 된다.
		# image._data 를 리스트에 담은 이유는 _data 메소드의 return 값이 BytesIO().read() 값이다.
		# 즉, 이미지 정보의 binary 값을 return 하게 되고 이를 Pillow 를 이용해서 읽을 계획
		# _images[cell] = image._data

# A3의 사진을 삭제한다.
delete_image = _images['A3']
sheet._images.remove(delete_image)

wb.save('out.xlsx')
```

![example](/assets/2021-02-15-openpyxl_image/example.png)

사진이 정상적으로 삭제된 모습 (손은 우리 아빠 손이다)
