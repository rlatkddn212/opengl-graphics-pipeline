# opengl-graphics-pipeline
OpenGL 그래픽스 파이프라인을 정리한 문서입니다.



## 그래픽스 파이프라인 개요

- 3D 모델 데이터를 2차원의 모니터 스크린에 랜더링하는 과정
- 3D 모델을 랜더링하는 방법은 여러 가지(레이트레이싱, 레스터라이즈)
- 여기서는 레스터라이즈 방법을 설명함
- 사용자가 직접 프로그래밍 하는 단계와 고정 함수로 이루어진 단계가 있음
- 아래 그림에서 초록색으로 표시한 부분이 사용자가 프로그래밍 가능한 단계
- 아래 그림처럼 스크린에 랜더링되기 까지 여러 단계를 거침



![1565106226877](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565106226877.png)

간단하게 설명하자면, 

1. 모델을 로드하면 정점 데이터를 얻음
2. 정점 데이터를 버텍스 쉐이더에서 이동 변환 시킴
3. 정점 데이터를 연결하여 삼각형을 만듬
4. 삼각형을 레스터라이즈 시켜 2차원 표면에 매칭
5. 프래그먼트 쉐이더에서 각 프래그먼트에 색상을 지정해줌
6. 스크린에 벗어난 프래그먼트를 제거
7. 스크린에 픽셀들을 보간시켜 최종이미지를 만듬



그럼 각 파이프라인 단계를 더 자세히 설명해본다.


-----------
## Vertex Specification 단계

- 말 그대로 정점을 정의하는 단계, 사용자가 프로그래밍 가능한 단계
- 여러 정보를 기반으로 Vertex Shader에 정점들을 전달함
- 여기서 정보란, Stride의 크기, 자료형(float, short, int), Index Buffer를 사용하는지 여부, 삼각형의 연결 방법 (triangle, strip, fan)

Index Buffer를 예를들면 아래 그림처럼 정점 데이터 구성이 달라 질 수 있음

![1565106784543](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565106784543.png)

 위 처럼 사각형을 그릴 때,

 정점 데이터만 사용할 경우 총 6개의 정점데이터,

 인덱스를 사용할 경우 정점 4개에 face 정보 2개를 사용하게 된다.

즉, Index Buffer를 사용하게 될 경우 정점 정보가 다르게 구성되므로 이런 처리들을 Vertex Specification 단계에서 처리해준다.


---------------
## Vertex Shader 단계

- 전달받은 정점을 조작할 수 있는 프로그래밍 가능한 단계

- 주로 world 변환, view(camera) 변환, projection 변환 처리, 애니메이션 처리



### 이동 변환, 스케일 변환, 회전 변환

![1565106981235](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565106981235.png)

![1565106998610](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565106998610.png)

각 정점에 변환 행렬을 곱하여 위 그림처럼 주전자의 정점들을 이동 시킬 수 있다.



### 뷰 변환(카메라 변환)

- 카메라가 보는 방향으로 이동 시키는 변환

![1565107053152](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565107053152.png)



### projection 변환 

- 원근감을 주기 위해 projection 변환을 수행

- 원근 투영, 직교 투영 두 종류의 투영 방법이 있음

- 변환 결과 모델이 NDC 좌표계로 이동됨

- NDC 좌표계란, 각 x,y,z 좌표가 -1.0~1.0 사이에 있는 정육면체 공간

  

아래는 원근 투영 방법

![1565107128493](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565107128493.png)


---------
## Tessellation, Geometry Shader 단계
- 사용자가 프로그래밍 가능한 단계
- Tessellation은 삼각형을 세분화하여 정밀하게 표현할 수 있음
- Geometry Shader는 새로운 프리미티브 형태로 변경할 수 있음



### 테셀레이션

테셀레이션을 통해 아래 그림처럼 삼각형을 세분화할 수 있다. 변위맵을 활용하여 각 버텍스를 조절할 수 있다.

![1565107468285](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565107468285.png)



### Geometry Shader

아래 그림은 쉐이더를 사용하여 구형태의 오브젝트에 노멀값을 랜더링한 결과이다.

각 정점에 Line 프리미티브를 새로 생성하여 랜더링 해보았다.

![1565107561770](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565107561770.png)


------------
## Primitive Assembly 단계

- Primitive 란 점, 선,삼각형을 의미
- 정점들을 연결하여 점, 선, 삼각형을 만듦

그림처럼 각 정점들을 연결하여 삼각형을 만들어 주는 단계이다.

아래 그림에서 Patch란 테셀레이션을 통과했을 때 프리미티브를 의미한다.

![1565107673378](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565107673378.png)


-------
## Resterization 단계

- 레스터라이즈하기 전 클리핑, 컬링, viewport 변환을 실행

- 레스터라이즈는 삼각형을 2차원의 각 픽셀에 대응

- 삼각형 정보들은 보간되어 처리



레스터라이즈 전 클리핑, 컬링, ViewPort 변환이 실행된다.

### 컬링

각 삼각형이 CW인지 CCW인지를 확인하여 제거해준다. (컬링을 CW로 할 경우 CW 삼각형 제거)

![1565107841801](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565107841801.png)



### 뷰포트 변환

뷰포트 변환은 Projection 변환을 통해 NDC 좌표로 옮겨진 좌표계를 2차원 표면 좌표계로 이동 시킨다.

![1565107951993](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565107951993.png)

### 레스터라이즈

모델이 2차원 표면에 어느 부분을 차지하는지 지정해주는 단계이다.

viewport 변환이 이뤄지면 정점이 2차원 표면으로 이동하게 되는데, 2차원 표면들은 사각형의 프래그먼트(픽셀과 거의 동일)들로 이루어져 있다. 삼각형 안에 있는 프래그먼트를 체크하게된다.

지정된 프래그먼트는 삼각형 정점들로 보간되어진 정보를 가지게된다.

![1565108027782](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565108027782.png)


--------
## Fragment Shader 단계

- 사용자가 프로그래밍 가능한 단계
- 보간되어 전달된 프래그먼트에 정보를 바탕으로 색상을 조작

- Light 효과를 줄 수 있음 (ex. 퐁쉐이딩)

- shadow 효과를 줄 수 있음

아래 그림처럼 색상을 입힐 수 있다.

![1565108459212](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565108459212.png)




----------
## Fragment Processing 단계
- 여러 테스트를 통해 Fragment를 사용할 것인지 결정

- 소유권 테스트, 가위 테스트,  깊이테스트, 스텐실 테스트가 있음

### 가위 테스트

실제로 가위 처럼 잘라 낸다.

![1565108520174](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565108520174.png)

### 깊이 테스트

어떤 삼각형이 화면에 가까이 있는지를 체크하기 위한 깊이 버퍼(z 버퍼)를 사용하여 다른 삼각형에 의해 가려진 부분을 제거한다.

![1565108573278](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565108573278.png)



### 스텐실 테스트

스텐실 버퍼를 사용하여 스텐실 버퍼에서 1로 저장된 부분은 Rendering 하고 0으로 된 부분은 제거하는 테스트를 진행한다.

![1565108674800](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565108674800.png)




---------
## Pixel Processing 단계
- 프래그먼트가 테스트를 통과하게 되면 실제로 스크린에 사용될 픽셀이 됨
- 마지막 단계로 이 픽셀들에 색상을 블랜딩하거나, 디더링, 샘플링을 통해 이미지를 자연스럽게 표현



### 디더링

디더링은 아래 그림처럼 bit 수 제한 때문에 색상 표현이 제한될 경우 사용된다. 

![1565108791800](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565108791800.png)



### 멀티 샘플링

아래 그림 처럼 픽셀에 샘플러를 둬서 몇개의 샘플을 차지하냐에 따라 색상에 차이를 둔다.

멀티 셈플링을 통해 계단현상을 줄일 수 있다

![1565108895158](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565108895158.png)




------------
## Double Buffering

- 하나의 버퍼를 사용하면 video memory에 픽셀을 랜더링하는 동안 화면에 제대로 표시하지 못함
- 버퍼를 2~3개를 사용하여 해결

![1565108968108](https://github.com/rlatkddn212/opengl-graphics-pipeline/blob/master/assets/1565108968108.png)





---------------
## 참고 자료 이미지
https://www.shadertoy.com/view/Xds3zN
https://www.lighthouse3d.com/tutorials/glsl-tutorial/primitive-assembly/
http://glasnost.itcarlow.ie/~powerk/GeneralGraphicsNotes/HSR/backfaceculling.html
http://alex-charlton.com/posts/Dithering_on_the_GPU/
https://learnopengl.com/Advanced-OpenGL/Anti-Aliasing
http://stonebird.github.io/Project4-CUDA-Rasterizer/
https://learnopengl.com/Advanced-OpenGL/Stencil-testing
https://en.wikibooks.org/wiki/Cg_Programming/Vertex_Transformations
