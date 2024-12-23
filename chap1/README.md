# 실행결과

- outline 검출

https://www.youtube.com/watch?v=vO5_AskTKCA

- inline 검출

https://www.youtube.com/watch?v=zr1YfN2--Fc

# <vision.cpp>

## 설명
https://github.com/lsy0727/jetson_linetracer/blob/c17fe10008568a9095b55729a9b66723a896d8a7/chap1/vision.cpp#L6-L22

* preprocess()함수 : 영상 전처리 함수

line 14~15 : 영상의 밝기 보정 (평균 100으로)

line 16 : 임계값 너무 작으면 노이즈가 생기고, 높으면 라인이 사라짐. 너무높지만 않으면 130보다 좀더 높아도 괜찮음.


https://github.com/lsy0727/jetson_linetracer/blob/066bfa827bbd73ece02550065d6da75e55f1d83d/chap1/vision.cpp#L25-L55

* findObjects()함수 : 객체 검출 함수

현재 검출된 객체(라인)들과 이전 프레임에서의 메인라인의 무게중심 점의 차이를 이용해 메인라인을 업데이트함.

거리는 150이하인 경우로 설정

이유 : (100일 때는 회전시에 급격한 변화가 생기면 라인을 놓치는 경우가 생기고, 150이 넘으면 다른 라인을 메인라인으로 변경해버리는 경우가 생김)

line 44~47 :  현재 영상에서 검출된 객체 중 가장 거리가 짧고, 이전 객체와 거리가 150이하인경우 객체가 있다고 판단( -> min_index 업데이트 )

1. 메인 라인이 업데이트 되는 경우(min_index가 업데이트 된 경우) 검출된 메인 라인의 좌표를 tmp_pt에 저장하고, 다음 라인검출시 비교용도로 사용

2. 객체가 없는 경우는 업데이트되지 않아, 사라지기 직전 프레임의 위치를 기준으로 찾음.


https://github.com/lsy0727/jetson_linetracer/blob/52029a6928be30dbb33f1d82258b6eeb6368e3d7/chap1/vision.cpp#L58-L76

* drawObjects()함수 : 객체와 기준점을 표시해주는 함수

tmp_pt : 메인라인의 무게중심 좌표가 저장되어있음

메인라인에는 빨간색 바운딩박스와 점을 찍고, 나머지 라인에는 파란색 바운딩박스와 점을 찍음.


https://github.com/lsy0727/jetson_linetracer/blob/fb40dde10008374f52f945060cc801049d7f6a36/chap1/vision.cpp#L79-L81

* getError()함수 : 위치오차를 계산해주는 함수

error = (로봇 중앙 x좌표) - (메인라인 중앙 x좌표)

(-)인 경우 : 라인이 오른쪽에 있음

(+)인 경우 : 라인이 왼쪽에 있음


# <main.cpp>

## 설명

https://github.com/lsy0727/jetson_linetracer/blob/7ef5a14f2f97c535510f72687339519513aead9a/chap1/main.cpp#L17-L18

영상을 불러옴

https://github.com/lsy0727/jetson_linetracer/blob/915247fd4fa228f9e15e1d91068828fb530be84a/chap1/main.cpp#L58-L59

영상 전처리

https://github.com/lsy0727/jetson_linetracer/blob/264c3858304777b9c9f430937db3807fb0978d0a/chap1/main.cpp#L62-L65

최초 실행시 기준 설정

https://github.com/lsy0727/jetson_linetracer/blob/b4731212bccaa30cfebd02233539394ce138941a/chap1/main.cpp#L67-L70

최초 실행시의 기준 라인을 메인라인으로 생각했을 때

1. 객체(라인)을 모두 찾음
  
2. 메인라인은 빨간색 바운딩 박스와 중심점, 나머지 라인은 파란색 바운딩 박스와 중심점 표시

error = getError(thresh, tmp_pt);

현 시점 error 계산

https://github.com/lsy0727/jetson_linetracer/blob/b50295ef34b53ead141986eba736ce2ba3aa1249/chap1/main.cpp#L56

https://github.com/lsy0727/jetson_linetracer/blob/b50295ef34b53ead141986eba736ce2ba3aa1249/chap1/main.cpp#L81-L83

line 56 : 반복문 시작시 시간계산 시작

line 81 : 시간계산 종료

line 82 : 걸린 시간 계산(sec)

https://github.com/lsy0727/jetson_linetracer/blob/90f333524c48705ea3510b53cd21a9898dcd1cda/chap1/main.cpp#L86-L88

frame : 기본 영상

gray : 흑백 영상

thresh : 최종 결과 영상


# 고찰

시간을 0.03sec로 고정해뒀더니 총 걸리는 시간이 0.03sec + (라인검출코드 실행시간)으로 나와 일정한 딜레이가 나오지 않았다.

-> 라인검출 시간을 계산해 usleep()시간을 동적인 값으로 조정되게 만들어야한다.
