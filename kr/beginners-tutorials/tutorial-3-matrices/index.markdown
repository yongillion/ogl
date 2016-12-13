---
layout: page
status: publish
published: true
title: 'Tutorial 3 : Matrices'
date: '2011-04-09 20:59:15 +0200'
date_gmt: '2011-04-09 20:59:15 +0200'
categories: [tuto]
order: 30
tags: []
---
{:TOC}

> _엔진이 배를 움직이지는 것이 아니다. 배는 그 자리에 가만히 있으며 엔진이 배를 감싸고 있는 세상을 움직이는 것이다._
> 
> Futurama

**이 튜토리얼은 전체 튜토리얼 중에서 가장 중요합니다. 최소 8번 이상 읽으세요.**

# 동차 좌표계 (Homogeneous coordinates)

지금까지 우리는 3D 좌표를 (x,y,z) 세가지로만 다루었습니다. 이제 w를 소개합니다. 앞으로 우리는 (x,y,z,w) 벡터를 사용할 것입니다.

이것에 대해 곧 명확히 알게 되겠지만 지금은 이것만 기억하도록 합니다.

- w == 1이면 벡터 (x,y,z,1)은 공간에서의 위치를 나타냅니다.
- w == 0이면 벡터 (x,y,z,0)는 방향을 나타냅니다.

(사실은 이를 영원히 기억해야 합니다.)

이것이 무슨 차이를 만들어 낼까요? 회전(Rotation) 측면에서는 아무것도 변하지 않습니다. 위치나 방향을 회전한다면 같은 결과를 얻게 됩니다. 하지만 평행이동(Translation, 점을 특정 방향으로 이동하는 것)이라는 측면에서는 다릅니다. "방향을 평행이동한다"라는 것은 무슨 의미일까요? 말이 안됩니다.

동차 좌표계는 두가지 경우를 하나의 수학 공식으로 다룰 수 있게 해줍니다.

# 변환 행렬 (Transformation matrices)


## 행렬의 소개 (An introduction to matrices)


간단히 말해 행렬은 미리 정의된 개수의 행(Row)과 열(Column)만큼의 숫자 배열입니다. 예를 들어 2x3 행렬은 다음과 같이 생겼습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/2X3.png)

3D 그래픽에서는 대부분 4x4 행렬을 사용합니다. 이 행렬은 (x,y,z,w) 정점들을 변환(Transformation)할 수 있게 해줍니다. 정점에 행렬을 곱함으로써 수행할 수 있습니다.

**행렬 x 정점 (이 순서대로 !!) = 변환 된 정점**

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/MatrixXVect.gif)

보기보다 무섭진 않습니다. 왼쪽 손가락을 a에 두고, 오른쪽 손가락을 x에 두세요. 이것이 _ax_ 입니다. 왼쪽 손가락을 다음 숫자인 (b)로 옮기세요. 그리고 오른쪽 손가락을 다음 숫자인 (y)로 옮기세요. 이것이 _by_ 입니다. 다시 한번 _cz_, 또 다시 한번 _dw_. ax + by + cz + dw. 새로운 x를 얻었습니다! 각 줄에 같은 작업을 수행하면 새로운 (x,y,z,w) 벡터를 얻을 수 있습니다.

이제 이것을 계산하기가 지루합니다. 우리는 자주 이 계산을 할 것이므로 컴퓨터가 대신하도록 요청하겠습니다.

**C++에서 GLM을 이용하여:**

``` cpp
glm::mat4 myMatrix;
glm::vec4 myVector;
// fill myMatrix and myVector somehow
glm::vec4 transformedVector = myMatrix * myVector; // 다시 한번 말하지만 반드시 이 순서대로 해야 합니다! 순서가 정말 중요합니다.
```

**GLSL에서:**

``` glsl
mat4 myMatrix;
vec4 myVector;
// myMatrix와 myVector를 어떠한 방법으로 채웁니다.
vec4 transformedVector = myMatrix * myVector; // 네, GLM과 아주 비슷합니다.
```

( 여러분의 코드에 복사/붙여넣기 하셨나요? 자 한번 해보세요.)

## 평행 이동 행렬(Translation matrices)

이것은 이해하기 가장 쉬운 변환 행렬입니다. 평행 이동 행렬은 다음과 같이 생겼습니다. 

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/translationMatrix.png)

X,Y,Z는 위치에 더하고자 하는 값입니다.

그래서 만약 벡터 (10,10,10,1)을 X 방향으로 10만큼 이동시키고자 한다면 다음과 같습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/translationExamplePosition1.png)

(해보세요! 해보오오세요!)

... (20,10,10,1) 동차 벡터를 얻었습니다! 기억하세요. 1은 방향이 아닌 위치를 나타냅니다. 그러므로 이 변환은 위치를 다루고 있다는 사실을 변경하지 않았습니다. 좋군요. 

이제 -z 축으로의 방향을 나타내는 벡터 (0,0,-1,0)의 경우 어떤 일이 일어나는지 살펴보죠.

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/translationExampleDirection1.png)

원래의 방향(0,0,-1,0)입니다. 훌륭하군요. 이전에 말했듯이 방향을 이동한다는 것은 말이 안되기 때문입니다.

그렇다면 이를 어떻게 코드로 나타낼까요?

**C++에서 GLM을 이용하여:**

``` cpp
#include <glm/gtx/transform.hpp> // after <glm/glm.hpp>
 
glm::mat4 myMatrix = glm::translate(10.0f, 0.0f, 0.0f);
glm::vec4 myVector(10.0f, 10.0f, 10.0f, 0.0f);
glm::vec4 transformedVector = myMatrix * myVector; // guess the result
```

**GLSL에서:**

``` glsl
vec4 transformedVector = myMatrix * myVector;
```

사실 GLSL에서는 이 작업을 거의 하지 않습니다. 대부분 여러분은 C++에서 glm::translate()를 이용해 행렬을 계산한 뒤 이를 GLSL로 보내어 곱셈만을 수행합니다. 

## 단위 행렬(The Identity matrix)

이것은 특별합니다. 아무것도 하지 않습니다. 그러나 A 곱하기 1.0은 A라는 사실을 아는 것만큼 중요하기 때문에 언급합니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/identityExample.png)

**C++ 에서:**

``` cpp
glm::mat4 myIdentityMatrix = glm::mat4(1.0f);
```

## 크기 변환 행렬(Scaling matrices)

크기 변환 행렬 역시 매우 쉽습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/scalingMatrix.png)

따라서 벡터(위치든 방향이든 상관 없습니다)의 크기를 모든 방향에 대해 두배 늘리고 싶다면 다음과 같습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/scalingExample.png)

그리고 w는 여전히 변하지 않았습니다. 다음과 같은 질문을 할 수 있습니다. "방향의 크기를 조절한다"는것이 무슨 의미인가요? 글쎄요. 가끔, 별로, 일반적으로는 이렇게 하지 않지만, (아주 드물게) 몇명 경우에는 도움이 될 수 있습니다.

(단위 행렬은 (X,Y,Z) = (1,1,1)인 크기변환행렬의 특별한 경우에 불과하다는 것을 알아두세요. 또한 (X,Y,Z) = (0,0,0)인 평행이동행렬의 특별한 경우이기도 합니다.)

**C++에서 :**

``` cpp
// Use #include <glm/gtc/matrix_transform.hpp> and #include <glm/gtx/transform.hpp>
glm::mat4 myScalingMatrix = glm::scale(2.0f, 2.0f ,2.0f);
```

## 회전 행렬(Rotation matrices)

회전 행렬은 매우 복잡합니다. 사용하는데 있어 정확한 레이아웃을 아는 것이 중요하지는 않기 때문에 건너 뛰도록 하겠습니다. 더 자세하게 알고 싶다면 [Matrices and Quaternions FAQ](http://www.opengl-tutorial.org/assets/faq_quaternions/index.html)를 참조하세요(유명한 자료). [Rotations tutorials]({{site.baseurl }}{{intermediate-tutorials/tutorial-17-quaternions}}) 를 참조해도 좋습니다.

**C++에서:**

``` cpp
// Use #include <glm/gtc/matrix_transform.hpp> and #include <glm/gtx/transform.hpp>
glm::vec3 myRotationAxis( ??, ??, ??);
glm::rotate( angle_in_degrees, myRotationAxis );
```

## 변환을 누적하기 (Cumulating transformations)

이제 벡터에 대해 회전과 이동, 크기를 변환하는 방법을 배웠습니다. 이러한 변환들을 조합한다면 멋질 것 같습니다. 이는 행렬들을 함께 곱하면 됩니다. 예를 들어.

``` cpp
TransformedVector = TranslationMatrix * RotationMatrix * ScaleMatrix * OriginalVector;
```
**!!! 주의 !!!** 이 라인은 실제로 크기 변환을 먼저 수행하며, 그 다음 회전, 그리고 평행 이동을 수행합니다. 이는 행렬 곱셈이 동작하는 방식입니다.

다른 순서로 수행한다면 같은 결과를 얻을 수 없을 것입니다. 직접 해보세요.

- 한걸음 앞으로 이동한 뒤 왼쪽으로 도세요.
- 왼쪽으로 돈 뒤 한걸음 앞으로 이동하세요.

사실 이 순서는 게임 캐릭터 및 다른 아이템에 일반적으로 필요한 순서입니다. 먼저 필요한 경우 크기를 변환하고, 방향을 설정한 뒤 이동을 시키세요. 예를 들어 선박 모형이 주어진다면(단순화를 위해 회전을 제거했습니다).

* 잘못된 방법 :
	- 배를 (10,0,0)만큼 평행 이동합니다. 이제 배의 중심은 원점에서 10만큼 떨어져있습니다. 
	- 배의 크기를 2배 확대합니다. 모든 좌표가 _원점에 대해 상대적으로_ 2가 곱해집니다. 종잡을 수 없이... 결국 큰 배를 얻겠지만 중심도 2*10=20이 됩니다. 원하는 바가 아닙니다.

* 옳은 방법 :
	- 배를 2배 확대합니다. 큰 배를가 되었고, 중심이 원점에 있습니다.
	- 배를 평행 이동합니다. 여전히 동일한 크기를 유지하면서도 옳바른 위치를 가집니다.

행렬-행렬간의 곱셈은 행렬-벡터 간의 곱셈과 매우 유사합니다. 그러므로 자세한 내용은 생략하겠습니다. 필요하다면 [Matrices and Quaternions FAQ](http://www.opengl-tutorial.org/assets/faq_quaternions/index.html#Q11)를 참조하세요. 이제 컴퓨터에게 이 작업을 수행하도록 간단히 요청해보겠습니다. 

**C++에서 GLM을 이용하여:**

``` cpp
glm::mat4 myModelMatrix = myTranslationMatrix * myRotationMatrix * myScaleMatrix;
glm::vec4 myTransformedVector = myModelMatrix * myOriginalVector;
```

**GLSL에서:**

``` glsl
mat4 transform = mat2 * mat1;
vec4 out_vec = transform * in_vec;
```

# 모델, 뷰, 투영 행렬 (The Model, View and Projection matrices)

_이 튜토리얼의 나머지에서는 블렌더의 유명한 3D 원숭이 모델인 수잔을 그리는 법을 이미 알고 있다고 가정하겠습니다_

모델, 뷰, 투영 행렬은 변환을 깔끔히 불리하는 편리한 도구입니다. 이걸 사용하지 않을 수도 있지만(우리가 튜토리얼 1과 2에서 한 것처럼요) 사용해야 합니다. 왜냐하면 이 방법이 가장 쉽기 때문에 모두가 사용하기 때문이에요.

## 모델 행렬 (The Model matrix)

이 모델은 우리가 좋아했던 삼각형처럼 여러 정점들의 집합으로 정의됩니다. 이 정점들의 X,Y,Z 좌표는 오브젝트의 중심으로부터 상대적인 위치를 나타냅니다. 즉 정점 (0,0,0)은 오브젝트의 중심을 나타내지요.

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/model.png)

플레이어가 키보드나 마우스로 이 모델을 제어하기 때문에 어쩌면 이 모델을 이동하고 싶을 수도 있습니다. 쉽습니다. 이미 배운 것처럼 `translation*rotation*scale`를 수행하면 되지요. 매 프레임마다 이 행렬을 모든 정점에 적용하면 (GLSL에서요, C++이 아니라) 모든 것이 움직일 것입니다. 움직이지 않는 것은 _월드의 중심(Center of the world)_입니다. 
![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/world.png)

정점들은 이제 _월드 공간(World Space)_에 존재합니다. 이것은 아래 이미지에서 검은색 화살표를 의미합니다. _우리는 모델 공간(모든 정점이 모델의 중심으로부터 상대적인 위치)에서 월드 공간(모든 정점이 월드의 중심으로부터 상대적인 위치)으로 옮겨 온 것입니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/model_to_world.png)

다음과 같은 다이어그램으로 요약할 수 있습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/M.png)

## 뷰 행렬 (The View matrix)

Futurama를 다시 인용해보죠.

> _엔진이 배를 움직이지는 것이 아니다. 배는 그 자리에 가만히 있으며 엔진이 배를 감싸고 있는 세상을 움직이는 것이다._

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/camera.png)

잘 생각해보면 카메라에도 동일하게 적용할 수 있습니다. 만약 다른 각도에서 산을 바라보고 싶으면, 카메라를 움직일 수 있겠지만... 또는 산을 움직일 수도 있겠죠. 실생활에서는 통용되지 않겠지만 컴퓨터 그래픽에서는 정말 쉽고 유용합니다.

처음에 카메라는 월드 공간의 원점에 위치합니다. 월드를 이동하기 위해서는 단지 새로운 행렬을 이용하면 됩니다. 카메라를 오른쪽(+X)으로 3 유닛만큼 이동한다고 해보죠. 이는 전체 월드(메쉬를 포함해)를 왼쪽(-X)으로 3 유닛 이동하는 것과 동일합니다! 잊어 버리기 전에 한번 해보죠.

``` cpp
// Use #include <glm/gtc/matrix_transform.hpp> and #include <glm/gtx/transform.hpp>
glm::mat4 ViewMatrix = glm::translate(-3.0f, 0.0f ,0.0f);
```
또 한번 아래 이미지는 이렇게 표현됩니다. _우리는 월드 공간(모든 정점이 월드의 중심으로부터 상대적인 위치)에서 카메라 공간(모든 정점이 카메라로부터 상대적인 위치)으로 옮겨 왔습니다._

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/model_to_world_to_camera.png)

머리가 폭발하기 전에 GLM의 멋진 glm::lookAt() 함수를 살펴보도록 하죠.

``` cpp
glm::mat4 CameraMatrix = glm::lookAt(
    cameraPosition, // 월드 공간에서의 카메라 위치.
    cameraTarget,   // 월드 공간에서 바라보는 방향.
    upVector        // 아마 glm::vec3(0,1,0)이겠지만, (0,-1,0)로 설정하면 위쪽을 아래쪽으로 가게 할수 있어요. 멋집니다.
);
```

여기 다이어그램이 있습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/MV.png)

아직 끝이 아닙니다.

## 투영 행렬 (The Projection matrix)

이제 우리는 카메라 공간에 있습니다. 이는 모든 변환이 완료된 후 x==0, y==0를 가지게 된 정점들이 화면 가운데에 렌더링 되어야 한다는 것을 의미합니다. 하지만 오브젝트가 화면의 어디에 그려져야 하는지 결정하는 데에 x와 y만을 사용할 수는 없습니다. 카메라와의 거리 (z) 역시 고려해야 합니다! 비슷한 x, y 좌표를 가지는 두 정점 중에 더 큰 z 좌표를 가지는 정점이 화면의 중심에 더 가깝게 그려져야 합니다.

이를 원근 투영(Perspective Projection)이라고 부릅니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/model_to_world_to_camera_to_homogeneous.png)

다행히도 4x4 행렬은 이 투영을 표현할 수 있습니다.[^projection]

``` cpp
// Generates a really hard-to-read matrix, but a normal, standard 4x4 matrix nonetheless
glm::mat4 projectionMatrix = glm::perspective(
    FoV,         // The horizontal Field of View, in degrees : the amount of "zoom". Think "camera lens". Usually between 90° (extra wide) and 30° (quite zoomed in)
    4.0f / 3.0f, // Aspect Ratio. Depends on the size of your window. Notice that 4/3 == 800/600 == 1280/960, sounds familiar ?
    0.1f,        // Near clipping plane. Keep as big as possible, or you'll get precision issues.
    100.0f       // Far clipping plane. Keep as little as possible.
);
```

마지막으로 한번 더,

_우리는 카메라 공간(모든 정점이 카메라로부터 상대적인 위치를 나타냄)으로부터 동차 공간(Homogeneous Space. 모든 정점이 작은 정육면체 안에 정의됩니다. 정육면체 안의 모든 것은 화면상에 출력됩니다)으로 옮겨 왔습니다._

최종 다이어그램은,

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/MVP.png)

여기에 투영에 대해 더 잘 이해할 수 있는 또 다른 그림이 있습니다. 투영 이전에 우리는 카메라 공간에서 파란 오브젝트들을 얻습니다. 그리고 붉은 모형은 카메라의 절두체(frustum)를 표현합니다. 카메라가 실제로 볼 수 있는 장면의 일부를 나타내죠.

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/nondeforme.png)

모든 것을 투영 행렬과 곱하면 다음과 같은 효과를 가집니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/homogeneous.png)

이 그림에서 이제 절두체는 완전한 정육면체(보기가 좀 어렵지만, 모든 축의 값이 -1, 1 사이입니다)이며, 모든 파란색 오브젝트들이 같은 방법으로 변형되었습니다. 그러므로 카메라에 가까운(=보이지 않는 정육면체의 앞면과 가까운) 오브젝트들은 커지고, 그렇지 않은 오브젝트들은 작아집니다. 현실과 비슷합니다!

절두체의 "뒤"에서 보면 다음과 같이 보일 겁니다. 

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/projected1.png)

이제 원하는 이미지를 얻었습니다. 이 이미지는 너무 정사각형이기 때문에 실제 창 크기에 맞출 수 있게 또다른 수학적 변환을 적용합니다(이것은 자동으로 일어나기 때문에 셰이더에서 직접 할 필요는 없습니다).

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/final1.png)

이제 이 이미지가 실제로 렌더링 될 이미지 입니다!

## 변환을 누적하기 : 모델-뷰-투영 행렬

... 너무나도 사랑스러운 표준 행렬 곱셈입니다.

``` cpp
// C++ : 행렬을 계산합니다.
glm::mat4 MVPmatrix = projection * view * model; // 기억하세요: 반대입니다!
```

``` glsl
// GLSL : 행렬을 적용합니다.
transformed_vertex = MVP * in_vertex;
```
{: .highlightglslfs }

# 모두 합치기

* 1단계 : MVP 행렬을 만듭니다. 이는 그리려고 하는 모든 모델에 대해 수행되어야 합니다.

  ``` cpp
  // 투영 행렬 : 45° Field of View, 4:3 ratio, display range : 0.1 unit <-> 100 units
  glm::mat4 Projection = glm::perspective(glm::radians(45.0f), (float) width / (float)height, 0.1f, 100.0f);
  
  // 또는 정사영(orthographic) 카메라 :
  //glm::mat4 Projection = glm::ortho(-10.0f,10.0f,-10.0f,10.0f,0.0f,100.0f); // In world coordinates
  
  // 카메라 행렬
  glm::mat4 View = glm::lookAt(
      glm::vec3(4,3,3), // Camera is at (4,3,3), in World Space
      glm::vec3(0,0,0), // and looks at the origin
      glm::vec3(0,1,0)  // Head is up (set to 0,-1,0 to look upside-down)
      );
  
  // 모델 행렬 : 단위 행렬 (모델은 원점에 위치)
  glm::mat4 Model = glm::mat4(1.0f);
  // MVP(모델-뷰-프로젝션) : 세 행렬의 곱.
  glm::mat4 mvp = Projection * View * Model; // Remember, matrix multiplication is the other way around
  ```

* 2단계: GLSL로 넘깁니다.

  ``` cpp
  // "MVP" 유니폼에 대한 핸들을 얻어 옵니다.
  // 초기화 단계에서만 수행하면 됩니다.
  GLuint MatrixID = glGetUniformLocation(program_id, "MVP");
  
  // "MVP" 유니폼을 통해 변환을 현재 바인딩된 셰이더로 전달합니다.
  // 각 모델마다 다른 MVP 행렬을 가질 것이기 때문에 (최소한 M 이라도요) 이 작업은 메인 루프에서 수행됩니다.
  glUniformMatrix4fv(mvp_handle, 1, GL_FALSE, &mvp[0][0]);
  ```

* 3단계: MVP 행렬을 정점들을 변환하는데 사용합니다.

  ``` glsl
  // 입력되는 정점 데이터. 셰이더가 실행될 때마다 다른 값입니다.
  layout(location = 0) in vec3 vertexPosition_modelspace;
  
  // 모든 메쉬에 대해 동일한 값을 유지합니다.
  uniform mat4 MVP;
  
  void main(){
    // 잘려진 공간으로 정점의 위치를 출력합니다. MVP * position
    gl_Position =  MVP * vec4(vertexPosition_modelspace,1);
  }
  ```
  {: .highlightglslvs }

* 끝입니다! 여기에 튜토리얼 2에서 본 것과 동일한 삼각형이 있습니다. 여전히 원점 (0,0,0)에 있지만 point (4,3,3), heads up (0,1,0), 45° field of view 의 시점에서 바라본 것입니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-3-matrix/perspective_red_triangle.png)

In tutorial 6 you'll learn how to modify these values dynamically using the keyboard and the mouse to create a game-like camera, but first, we'll learn how to give our 3D models some colour (tutorial 4) and textures (tutorial 5).

튜토리얼 6에서 게임에 사용되는 카메라를 생성하기 위해, 이 값들을 키보드와 마우스를 통해 동적으로 변경하는 방법에 대해서 배울 것입니다. 그보다 앞서 3D 모델에 색상을 주고(튜토리얼 4), 텍스쳐를 입히는 방법(튜토리얼 5)를 배우도록 하겠습니다. 

# Exercises

*   glm::perspective를 바꿔보세요.
*   원근 투영을 사용하는 데신 정사영 투영(glm::ortho)을 사용해 보세요.
*   삼각형을 평행 이동하고, 회전하고, 크기를 변경하도록 모델 행렬을 수정해보세요.
*   동일한 작업을 다른 순서로 해보세요. 무엇을 깨닫게 되나요? 무엇이 캐릭터에 사용하기에 가장 "최적"의 순서인가요?

_Addendum_

[^projection]: [...]다행히도 4x4 행렬은 이 투영을 표현할 수 있습니다. 사실 이 말은 옳지 않습니다. 원근 변환은 아핀이 아니기 때문에 행렬로 완전히 표현할 수 없습니다. 투영 행렬과 곱해진 이후에 동차 좌표들은 그들의 W 성분으로 나누어집니다. 이 W 성분은 바로 -Z 입니다(투영 행렬이 이러한 방법으로 작성되었기 때문에). 이러한 방법으로 원점에서 멀리 떨어진 점일 수록 더 큰 Z로 나누어지게 되어, 이러한 점들의 X와 Y 좌표는 더 작아집니다. 점들이 서로 더 가까워지고 오브젝트들은 더 작아 보입니다. 이러한 방법으로 원근이 이루어 집니니다. 이러한 변환은 하드웨어에 의해 일어나며 셰이더에서는 보이지 않습니다.