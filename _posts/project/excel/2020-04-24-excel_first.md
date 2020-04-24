---
layout: post
title: [PyQt5] 엑셀 자동 작업 프로그램 - gui
date: '2020-04-24'
category: excel
tags: ['Python', 'Excel', 'PyQt5']
permalink: '/blog/:categories/:title'
---

앞서 말했듯이 pyqt5 사용법을 익히느라 많은 고생을 했다. 우선 이렇게 완성한 첫 프로그램이다.

![](/assets/2020-04-24-excel_first/2020-04-24-excel_first_092756.png)

openpyxl 라이브러리에 원하는 셀의 사진을 특정하거나 지우는 방법을 아직 찾지 못 한건지 아니면 없는건지 프로그램에 넣지 못 했다. 그래서 밑의 *경고* 를 집어넣었다.

## 1. 기능

### 엑셀 파일

작업 전, 작업 후, 전주번호는 모두 같은 기능을 하지만 엑셀의 다른 열에 사진을 집어넣는 차이밖에 없으므로 작업 전만 설명하겠다.

특정인을 위한 프로그램이다보니 많은 다수를 생각할 필요가 없어서 생각보다 편했다.

파일 열기를 누르면 엑셀 파일을 선택할 수 있는 창이 뜬다.

```python
# 파일 오픈
self.fileopen_btn.clicked.connect(self.openExcel)

def openExcel(self):
    setLabelText(self.progress_lbl, '[진행중] 파일을 여는 중...')
    # 실질적으로 파일을 불러오는 함수
    fname = QFileDialog.getOpenFileName(self, '엑셀 파일 선택', BASE_DIR, "Excel files (*.xlsx)")

    if fname[0]:
        if os.path.isfile(fname[0]):
            name = fname[0].split('/')[-1]
            self.file_name = fname[0]
            # 엑셀 파일을 분석한다. openpyxl
            self.wb = openpyxl.load_workbook(self.file_name)
            try:    # 시트 이름은 무조건 작업사진이기때문에 바로 이렇게 처리했다.
                self.sheet = self.wb['작업사진']
            except KeyError:
                setLabelText(self.progress_lbl, "[에러 1] '작업사진' 워크시트를 열지 못 했습니다. 엑셀파일을 확인해주세요.")
            else:
                row = (self.sheet.max_row - 1) // 19    # 마찬가지로 무조건 사진하나에 19칸이기때문에 이렇게 처리
                self.spinBox.setRange(1, row)
                setLabelText(self.rowlabel, '원하는 행 (1 ~ ' + str(row) + ')')
                setLabelText(self.fileopen_lbl, name)
                setLabelText(self.progress_lbl, '[완료] 파일을 성공적으로 열었습니다.')
    else:
        setLabelText(self.progress_lbl, '[에러 0] 파일이 잘못 선택됐습니다. 다시 선택해주세요')
```

PyQt 는 시그널과 슬롯으로 신호를 주고받는다. 쉽게 설명하면

```python
self.fileopen_btn.clicked.connect(self.openExcel)
self.filopen_btn # 이라는 이름을 가진 버튼이
.clicked # 클릭되면
.connect # 연결해라
(self.openExcel) # openExcel 함수에
```

이렇게 구분할 수 있겠다. 처음에 나도 이해하는 데 오래걸렸지만 한 번 이해하고 나니 간단했다.

참고로 setLabetText함수는 내가 임의로 만든 함수이다. 라벨을 수정하는 일이 많아서 한 줄로 할 수 있게 만들었다.

```python
def setLabelText(label, text):
    label.setText(text)
    label.adjustSize()
```

### 사진

![](/assets/2020-04-24-excel_first/2020-04-24-excel_first_120152.png)

불러오기를 누르면 사진을 목록에 추가하고 삽입하기를 누르면 그 사진을 

![](/assets/2020-04-24-excel_first/2020-04-24-excel_first_120221.png)

원하는 행을 선택한 뒤 삽입하기를 누르면 사진이 들어간다.

목록에서 사진을 클릭하면 옆에 사진이 뜬다.

![](/assets/2020-04-24-excel_first/2020-04-24-excel_first_120403.png)


## 수정할 점

아직 완성본도 아니지만 막상 사용해보니 그렇게 편리하다는 느낌을 받지 못 했다. 추후 개선을 통해 아래와 같이 바꿀 생각이다.

- 전, 후, 전주 번호를 하나의 창에 모은다.
- 사진을 드래그 & 드랍으로 넣을 수 있게 한다.
- 목록에서 사진을 선택하기보다는 사진이 바로바로 떠서 수정할 수 있도록 해본다.

우선 이정도인데 PyQt 의 기능을 다 모르기도 하지만 내 실력도 많이 낮아서 최대한 열심히 해봐야겠다.