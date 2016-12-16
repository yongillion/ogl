---
layout: page
status: publish
published: true
title: '튜토리얼 6 : 키보드와 마우스'
date: '2011-05-08 08:26:13 +0200'
date_gmt: '2011-05-08 08:26:13 +0200'
categories: [tuto]
order: 60
tags: []
---

여섯 번재 튜토리얼에 오신 것을 환영합니다!

이제 우리는 키보드와 마우스를 이용해 카메라를 움직이는 방법을 배워보겠습니다. 마치 FPS처럼요.

# 인터페이스

튜토리얼 내내 이 코드를 다시 사용할 것이기 때문에, 이 코드를 별도의 파일에 작성하겠습니다. common/controls.cpp에 구현하고 common/controls.hpp에 함수를 선언합니다. 그리고 tutorial06.cpp에서 이를 불러옵니다.

tutorial06.cpp의 코드는 이전 튜토리얼에서 별로 바뀌지 않았습니다. 주요 변경사항은 이전에는 MVP 행렬을 한 번만 계산했지만 이제는 매 프레임마다 계산합니다. 그러므로 이 코드를 메인 루프 안으로 옮기도록 하겠습니다.

``` cpp
do{

    // ...

    // 키보드와 마우스 입력으로부터 MVP 행렬을 계산합니다.
    computeMatricesFromInputs();
    glm::mat4 ProjectionMatrix = getProjectionMatrix();
    glm::mat4 ViewMatrix = getViewMatrix();
    glm::mat4 ModelMatrix = glm::mat4(1.0);
    glm::mat4 MVP = ProjectionMatrix * ViewMatrix * ModelMatrix;

    // ...
}
```

이 코드는 세개의 새로운 함수를 필요로 합니다.

* computeMatricesFromInputs() 은 키보드와 무우스 입력을 읽어들여 투영 행렬과 뷰 행렬을 계산합니다. 이 함수가 모든 마법이 일어나는 곳입니다.
* getProjectionMatrix() 는 투영 행렬을 반환합니다.
* getViewMatrix() 은 뷰 행렬을 반환합니다.

물론 이것은 여러가지 방벚중 하나일 뿐입니다. 이러한 함수들이 마음에 들지 않는다면 변경해도 좋습니다.

conrols.cpp 내부를 들여다보겠습니다.

# The actual code

몇 가지 변수들이 필요합니다.

``` cpp
// position
glm::vec3 position = glm::vec3( 0, 0, 5 );
// 수평 각도 : -Z 방향으로
float horizontalAngle = 3.14f;
// 수직 각도 : 0, 수평선을 바라보도록
float verticalAngle = 0.0f;
// 초기 시야
float initialFoV = 45.0f;

float speed = 3.0f; // 3 units / second
float mouseSpeed = 0.005f;
```
FoV는 줌 레벨을 나타냅니다. 80&deg; = 매우 넓은 각도와 엄청난 왜곡, 60&deg; - 45&deg; : 표준.  20&deg; : 많은 확대

입력에 따라 위치와 position과 horizontalAngle, verticalAngle, FoV를 먼저 계산하도록 할 것입니다. 그리고 이 변수들로부터 뷰 행렬과 투영 행렬을 계산할 것입니다.

## Orientation

마우스 위치를 읽어들이는건 쉽습니다.

``` cpp
// Get mouse position
int xpos, ypos;
glfwGetMousePos(&xpos, &ypos);
```

커서의 위치를 다시 중앙으로 돌려놓을 필요가 있습니다. 그렇지 않으면 곧 커서가 창 밖으로 나가게 되어 더 이상 움직일 수 없게 됩니다.

``` cpp
// Reset mouse position for next frame
glfwSetMousePos(1024/2, 768/2);
```

이 코드는 윈도우의 크기가 1024*768이라고 가정하고 있습니다. 필요하다면 glfwGetWindowSize를 호출해서 윈도우의 크기를 얻어올 수 있습니다.

이제 바라볼 각도를 계산할 수 있습니다.

``` cpp
// Compute new orientation
horizontalAngle += mouseSpeed * deltaTime * float(1024/2 - xpos );
verticalAngle   += mouseSpeed * deltaTime * float( 768/2 - ypos );
```

오른쪽에서 왼쪽으로 읽어보죠.

* 1024/2 - xpos : 마우스가 윈도우의 중심으로부터 얼마나 떨어져있는지를 나타냅니다. 이 값이 클수록 카메라를 더 많이 돌리게 됩니다.
* float(...)은 곱셈이 잘 수행될 수 있도록 실수 값으로 변환합니다.
* mouseSpeed 는 회전을 빠르게 하거나 느리게 하는데 사용됩니다. 이 값을 잘 조정하거나 혹은 유저가 선택할 수 있게 합니다.
* += : 마우스를 움직이지 않았다면 "1024/2-xpos" 가 0이 되어 "horizontalAngle+=0"은 horizontalAngle을 변경하지 않을 것입니다. 만약 "="를 사용해 버리면 매 프레임마다 강제로 원점으로 돌아와버릴 것입니다. 좋지 않습니다.

이제 바라보는 방향을 월드 공간에서 표현하는 벡터를 계산할 수 있습니다..

``` cpp
// 방향 : 구면 좌표계를 직교 좌표계로 변환
glm::vec3 direction(
    cos(verticalAngle) * sin(horizontalAngle),
    sin(verticalAngle),
    cos(verticalAngle) * cos(horizontalAngle)
);
```

이 코드는 표준화된 계산입니다. 만약 코사인과 사인에 대해서 모른다면 여기 간단한 설명이 있습니다.

<img class="alignnone whiteborder" title="Trigonometric circle" src="http://www.numericana.com/answer/trig.gif" alt="" width="150" height="150" />

위 공식은 3D로 일반화한 것 뿐입니다.

이제 우리는 "up" 벡터를 안정적으로 계산하고 싶습니다. "up"이 항상 +Y 방향을 나타내는 것은 아니라는 것을 명심하세요. 만약 아래를 내려다 보고 있다면 "up" 벡터는 사실 수평일 겁니다. 여기 카메라가 같은 위치에서 같은 방향을 바라보지만 "up" 벡터가 다른 경우의 예가 있습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-6-mouse-keyboard/CameraUp.png)

우리의 경우 유일하게 변하지 않는 건 카메라의 오른쪽을 향하는 벡터가 항상 수평이라는 사실입니다. 팔을 오른쪽으로 뻗은 후 위, 아래 아무 방향이나 바라보면 이해하기 쉬울 것입니다. 그러므로 "right" 벡터를 먼저 정의하겠습니다. 수평이기 때문에 Y 좌표는 0입니다. 그리고 X와 Z 좌표는 위 그림과 같지만 각도를 90&deg; 또는 Pi/2 라디안 만큼 회전합니다.

``` cpp
// Right vector
glm::vec3 right = glm::vec3(
    sin(horizontalAngle - 3.14f/2.0f),
    0,
    cos(horizontalAngle - 3.14f/2.0f)
);
```

이제 우리는 "right" 벡터와 "directon" 또는 "front" 벡터를 가지고 있습니다. "up" 벡터는 이 두 벡터에 모두 직각인 벡터입니다. 이를 계산하기 쉽게 만들어주는 유용한 수학도구가 있는데 바로 외적(cross product)입니다.

``` cpp
// Up vector : perpendicular to both direction and right
glm::vec3 up = glm::cross( right, direction );
```
외적이 무엇을 하는지 기억하는 아주 쉬운 방법이 있습니다. 튜토리얼 3에서 본 오른손 법칙을 다시 떠올려 보세요. 첫 번째 벡터가 엄지, 두 번째 벡터가 검지라면 결과는 중지입니다. 아주 편리합니다.

## 위치

코드는 꽤나 직관적입니다. 그런데 awsd 대신 위/아래/오른쪽/왼쪽 키를 사용했는데 그 이유는 제 azerty 키보드에서 awsd는 실제로 zqsd이기 때문입니다.또한 qwerZ 키보드도 다를 것이고 한국어 키보드도 다를 것입니다. 한국인들이 어떤 키보드를 사용하는지는 모르겠지만 아마도 다를 것이라고 생각합니다.

``` cpp
// 앞으로 이동
if (glfwGetKey( GLFW_KEY_UP ) == GLFW_PRESS){
    position += direction * deltaTime * speed;
}
// 뒤로 이동
if (glfwGetKey( GLFW_KEY_DOWN ) == GLFW_PRESS){
    position -= direction * deltaTime * speed;
}
// 오른쪽으로 이동
if (glfwGetKey( GLFW_KEY_RIGHT ) == GLFW_PRESS){
    position += right * deltaTime * speed;
}
// 왼쪽으로 이동
if (glfwGetKey( GLFW_KEY_LEFT ) == GLFW_PRESS){
    position -= right * deltaTime * speed;
}
```

여기서 특별한 건 deltaTime 뿐입니다. 다음과 같은 간단한 이유로 매 프레임마다 1 유닛씩 이동하는 것은 바람직하지 않을 것입니다.

* 만약 컴퓨터가 빨라서 60fps로 동작한다면 초당 60 * 속도 유닛만큼 이동할 것입니다.
* 만약 컴퓨터가 느려서 20fps로 동작한다면 초당 20 * 속도 유닛만큼 이동할 것입니다.

좋은 컴퓨터를 가지고 있는 것이 꼭 빠르게 동작하는 것을 보장하는 것은 아니므로 "마지막 프레임으로부터 지난 시간" 또는 "deltaTime"을 이용해 이동할 거리를 조절해야만 합니다.

* 만약 컴퓨터가 빨라서 60fps로 동작한다면 초당 1/60 * 속도 유닛만큼 이동할 것입니다.
* 만약 컴퓨터가 느려서 20fps로 동작한다면 초당 1/20 * 속도 유닛만큼 이동할 것입니다.

훨씬 낫네요. deltaRime은 아주 쉽게 구할 수 있습니다.

``` cpp
double currentTime = glfwGetTime();
float deltaTime = float(currentTime - lastTime);
```

## 시야각 (Field Of View)

재미로 마우스 휠을 시야각과 엮어보도록 하죠. 싸구려 줌을 가지게 됩니다.

``` cpp
float FoV = initialFoV - 5 * glfwGetMouseWheel();
```

## 행렬 계산

행렬들을 계산하는건 이제 어렵지 않습니다. 이전과 완전히 동일한 함수를 사용하지만 새로운 값을 넘깁니다.

``` cpp
// Projection matrix : 45&deg; Field of View, 4:3 ratio, display range : 0.1 unit <-> 100 units
ProjectionMatrix = glm::perspective(FoV, 4.0f / 3.0f, 0.1f, 100.0f);
// Camera matrix
ViewMatrix       = glm::lookAt(
    position,           // Camera is here
    position+direction, // and looks here : at the same position, plus "direction"
    up                  // Head is up (set to 0,-1,0 to look upside-down)
);
```

# 결과

![](http://www.opengl-tutorial.org/assets/images/tuto-6-mouse-keyboard/moveanim.gif)


## 후면 제거 (Backface Culling)

이제 자유롭게 움직일 수 있습니다. 정육면체 안으로 들어가보면 폴리곤들이 여전히 보이는 것을 알 수 있습니다. 명백해 보이지만 사실 이는 최적화의 기회가 열려있다는 것을 의미합니다. 실제로 대부분의 어플리케이션에서 정육면체 _안으로_ 들어갈 일이 없을 겁니다.

아이디어는 GPU에게 카메라가 삼각형의 뒤에 있는지 혹은 앞에 있는지를 검사하도록 하는 것입니다. 앞에 있다면 삼각형을 출력합니다. 만약 메쉬가 닫혀 있으며, 카메라가 삼각형의 뒤에 있고, 메쉬 안에 있지 않다면, 카메라 앞에 또 다른 삼각형이 있을 겁니다. 결과적으로 출력해야할 삼각형이 평균 2배 적어져서 모든 것이 빨라졌다는 것을 제외하고는 아무도 알아채지 못할 것입니다!

아주 좋은 것은 이를 검사하는 것이 매우 쉽다는 것입니다. GPU는 삼각형의 법선을 계산(외적을 이용해서요. 기억나나요?)해서 법선이 카메라 쪽을 향해 있는지 확인합니다.

불행이도 이를 위해 약간의 노력이 필요합니다. 삼각형의 방향은 암시적입니다. 이는 버퍼의 두 정점이 뒤집어져 있다면, 구멍이 생기게 될 것이라는 것을 의미합니다. 그러나 약간의 추가작업을 할 가치가 있습니다. 종종 3D 모델러에서 "법선 뒤집기"를 클릭하기만 하면 됩니다(이 작업은 사실 정점을 뒤집습니다. 따라서 법선도 뒤집어집니다). 모든 것이 좋습니다.

후면 제거를 켜는 것은 식은 죽 먹기 입니다.

``` cpp
// Cull triangles which normal is not towards the camera
glEnable(GL_CULL_FACE);
```

# 연습문제

* verticalAngle을 제한해서 위쪽이 아래쪽으로 올 수 없도록 합니다.
* 오브젝트 주변을 도는 카메라를 만듭니다. ( position = ObjectCenter + ( radius * cos(time), height, radius * sin(time) ) ); radius/height/time을 키보드/마우스 등과 연결하세요.
* 즐기세요!
