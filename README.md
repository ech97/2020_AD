## 차선인식 프로그램 알고리즘

### 목차

​	

1. 카메라 왜곡 보정(구현 예정)

2. 버드아이뷰

3. 라인검출

   3-1) line_detect()

   3-2) post_line_detect()

4. 라인그리기

5. 회전각도 조절



-----------

### 2. 버드아이뷰



#### 1) ROI 영역 설정



#### 2) 왜곡 영역 설정





------

### 3-1. 라인검출_ line_detect



### 1) 색공간 분리

1. **버드아이뷰 이미지를 HLS 색공간과, GRAY 색공간으로 분리**

    - HLS 색공간: 흰색과 노란색 라인 검출시 이용 (이후, R채널에 대해서 흰선, B채널에 대해서 노란선 구별 예정)

      > - inRange 함수를 이용해 검출하여 이진공간으로 옮김
      >
      > - erode와 dilate 함수를 이용해 노이즈 제거

    - GRAY 색공간: Canny 함수를 이용하여 테두리 검출



#### 2) 색공간 결합

1. **차선이 있는 두개의 이진 공간을 OR 논리 연산을 통해 결합**



#### 3) 픽셀 검출

1. **히스토그램 제작**

   - 행방향(axis = 0)으로 np.sum 함수 사용

     

2. **히스토그램의 피크값으로 라인 구별**

   - 히스토그램의 중간지점을 기준으로 왼쪽, 오른쪽 구별

   - 왼쪽, 오른쪽 각각의 공간에서 피크값이 있는 곳의 위치를 파악

     > 피크값의 위치를 반환하는 argmax() 함수 사용

     

3. **슬라이딩 윈도우 제작**

   - 히스토그램에서의 피크값 위치를 중심으로 첫번째 윈도우 제작

   - 첫번째 윈도우 안에있는 픽셀 검출

     > - nonzero() 함수를 통해 '색공간 결합'에서의 이미지에서 흰색인 부분의 좌표 추출
     >
     > - x좌표(nonzero[1])와 y좌표(nonzero[0])로 분리
     >
     > - 윈도우의 가로선과 세로선보다 안에있는지 **관계연산** 진행
     >
     > - 관계연산 결과중 참인 값들의 인덱스 집합을 제작 (관계연산.nonzero()[0])
     >
     > - 인덱스의 수가 곧 윈도우 안의 흰색 픽셀

   - 검출된 픽셀들의 x값의 평균을 중심으로 두번째 윈도우 제작

     > - 이전에 구했던 인덱스를 nonzero_x에 입력하면, 흰색인 x값의 좌표가 나옴
     >
     > - 직전의 과정을 통해 x값의 좌표들을 모아서 평균을 계산(np.mean)
     >
     > - 직전의 계산값을 중심으로하는 두번째 윈도우 제작
     >
     > 
     >
     > ** 이때 y값은 윈도우의 수와 화면 높이에 맞춰서 자동설정

   - for문을 통해 원하는 수만큼 윈도우 제작

     > **반복될수록 이전의 x값들이 쌓여 평균값의 차이가 무뎌짐**

     

#### 4) 곡선 방정식 제작

1. **'슬라이딩 윈도우'에서 검출한 픽셀의 좌표 소환**

   > - 윈도우 만큼의 for문으로 생성되어, 리스트에 append된 모든 인덱스들을 
   >
   >   np.concatenate()함수로 병합
   >
   > - 모든 인덱스값을 nonzero_x와 nonzero_y에 넣어 (x, y) 좌표 도출

2. **픽셀값을 통한 2차방정식 계수 도출**

   > polyfit 함수를 사용하여, 픽셀들의 (x, y)좌표를 입력하여 방정식의 계수 도출

3. **촘촘한 그래프 제작을 위해 y값에 따른 x값 도출**

   > - linspace를 활용해 y값을 촘촘하게 제작
   >
   > - 생성된 y값을 이전에 도출한 방정식에 대입
   >
   > - y값에 해당하는 x값 도출
   >
   > - 계산값 append를 이용해 리스트에 저장

4. **x값들의 평균을 이용하여, 평균 곡선 제작**

   > - 계산된 x값의 집합들이 프레임마다 쌓임
   >
   > ​	: 리스트의 형식으로 쌓여서 [[.....], [.....], [.....] ....] 형식으로 쌓여있음
   >
   > - n번의 프레임 이후, n개의 x값의 집합이 생성
   >
   > - np.squeeze()를 이용하여 배열 형태로 변경 [[....] [....] [....] ...]
   >
   > - 최근값들이 있는 뒤에서부터, 원하는 만큼 빼서 평균을 냄 [[....]]
   >
   >   (enumerate와 reversed 함수를 사용하여 뒤에서부터 뺌)
   >
   > - 평균 x값 저장 (추후 라인 그릴때 사용)




### 3-2. 라인검출_ post_line_detect

1. **3-1에서의 x값 가져오기**
   - x값을 가져온뒤, 윈도우 마진을 더해 왼쪽, 오른쪽 차선의 범위형성



2. 픽셀 검출
   - 1번 과정에서 얻어낸 x값들의 범위안에 있는 흰색 픽셀 검출
   - 







-----

### 4. 라인 그리기



#### 1) 좌표 설정

1. **x, y 좌표 묶기**

   - (x1, y)제작

     > 행벡터 x1과 y를 np.vstack을 이용하여 세로로 붙이기 

   - (x2, y) 제작

     > - vstack을 이용하여 세로로 붙이기
     > - transpose를 이용하여 전치 시키기
     > - filpud를 이용하여 위아래 뒤집기

   - 묶기

     > hstack을 이용하여 가로로 묶기

2. **색칠하기**

   - 라인, 면 색칠

     > fillpoly 함수를 이용하여, 선 및 면 색칠

3. 원본이미지와 병합

   - 색칠한 이미지와 원본이미지 병합

     > addWeighted 함수를 이용하여 원본이미지와 병합





