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

> _엔진이 배를 움직이는 것이 아니다. 배는 그 자리에 가만히 있으며 엔진이 배를 감싸고 있는 세상을 움직인다._
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

![]({{site.baseurl}}/assets/images/tuto-3-matrix/2X3.png)

3D 그래픽에서는 대부분 4x4 행렬을 사용합니다. 이 행렬은 (x,y,z,w) 정점들을 변환(Transformation)할 수 있게 해줍니다. 정점에 행렬을 곱함으로써 수행할 수 있습니다.

**행렬 x 정점 (이 순서대로 !!) = 변환 된 정점**

![]({{site.baseurl}}/assets/images/tuto-3-matrix/MatrixXVect.gif)

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

![]({{site.baseurl}}/assets/images/tuto-3-matrix/translationMatrix.png)

X,Y,Z는 위치에 더하고자 하는 값입니다.

그래서 만약 벡터 (10,10,10,1)을 X 방향으로 10만큼 이동시키고자 한다면 다음과 같습니다.

![]({{site.baseurl}}/assets/images/tuto-3-matrix/translationExamplePosition1.png)

(해보세요! 해보오오세요!)

... (20,10,10,1) 동차 벡터를 얻었습니다! 기억하세요. 1은 방향이 아닌 위치를 나타냅니다. 그러므로 이 변환은 위치를 다루고 있다는 사실을 변경하지 않았습니다. 좋군요. 

이제 -z 축으로의 방향을 나타내는 벡터 (0,0,-1,0)의 경우 어떤 일이 일어나는지 살펴보죠.

![]({{site.baseurl}}/assets/images/tuto-3-matrix/translationExampleDirection1.png)

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

![]({{site.baseurl}}/assets/images/tuto-3-matrix/identityExample.png)

**C++ 에서:**

``` cpp
glm::mat4 myIdentityMatrix = glm::mat4(1.0f);
```

## 크기 변환 행렬(Scaling matrices)

크기 변환 행렬 역시 매우 쉽습니다.

![]({{site.baseurl}}/assets/images/tuto-3-matrix/scalingMatrix.png)

따라서 벡터(위치든 방향이든 상관 없습니다)의 크기를 모든 방향에 대해 두배 늘리고 싶다면 다음과 같습니다.

![]({{site.baseurl}}/assets/images/tuto-3-matrix/scalingExample.png)

그리고 w는 여전히 변하지 않았습니다. 다음과 같은 질문을 할 수 있습니다. "방향의 크기를 조절한다"는것이 무슨 의미인가요? 글쎄요. 가끔, 별로, 일반적으로는 이렇게 하지 않지만, (아주 드물게) 몇명 경우에는 도움이 될 수 있습니다.

(단위 행렬은 (X,Y,Z) = (1,1,1)인 크기변환행렬의 특별한 경우에 불과하다는 것을 알아두세요. 또한 (X,Y,Z) = (0,0,0)인 평행이동행렬의 특별한 경우이기도 합니다.)

**C++에서 :**

``` cpp
// Use #include <glm/gtc/matrix_transform.hpp> and #include <glm/gtx/transform.hpp>
glm::mat4 myScalingMatrix = glm::scale(2.0f, 2.0f ,2.0f);
```

## 회전 행렬(Rotation matrices)

회전 행렬은 매우 복잡합니다. 사용하는데 있어 정확한 레이아웃을 아는 것이 중요하지는 않기 때문에 건너 뛰도록 하겠습니다. 더 자세하게 알고 싶다면 [Matrices and Quaternions FAQ]({{site.baseurl}}/assets/faq_quaternions/index.html)를 참조하세요(유명한 자료). [Rotations tutorials]({{site.baseurl }}{{intermediate-tutorials/tutorial-17-quaternions}}) 를 참조해도 좋습니다.

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

행렬-행렬간의 곱셈은 행렬-벡터 간의 곱셈과 매우 유사합니다. 그러므로 자세한 내용은 생략하겠습니다. 필요하다면 [Matrices and Quaternions FAQ]({{site.baseurl}}/assets/faq_quaternions/index.html#Q11)를 참조하세요. 이제 컴퓨터에게 이 작업을 수행하도록 간단히 요청해보겠습니다. 

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

# The Model, View and Projection matrices

_For the rest of this tutorial, we will suppose that we know how to draw Blender's favourite 3d model : the monkey Suzanne._

The Model, View and Projection matrices are a handy tool to separate transformations cleanly. You may not use this (after all, that's what we did in tutorials 1 and 2). But you should. This is the way everybody does, because it's easier this way.

## The Model matrix

This model, just as our beloved red triangle, is defined by a set of vertices. The X,Y,Z coordinates of these vertices are defined relative to the object's center : that is, if a vertex is at (0,0,0), it is at the center of the object.

![]({{site.baseurl}}/assets/images/tuto-3-matrix/model.png)

We'd like to be able to move this model, maybe because the player controls it with the keyboard and the mouse. Easy, you just learnt do do so : `translation*rotation*scale`, and done. You apply this matrix to all your vertices at each frame (in GLSL, not in C++!) and everything moves. Something that doesn't move will be at the _center of the world_.

![]({{site.baseurl}}/assets/images/tuto-3-matrix/world.png)

Your vertices are now in _World Space_. This is the meaning of the black arrow in the image below : _We went from Model Space (all vertices defined relatively to the center of the model) to World Space (all vertices defined relatively to the center of the world)._

![]({{site.baseurl}}/assets/images/tuto-3-matrix/model_to_world.png)

We can sum this up with the following diagram :

![]({{site.baseurl}}/assets/images/tuto-3-matrix/M.png)

## The View matrix

Let's quote Futurama again :

> _The engines don't move the ship at all. The ship stays where it is and the engines move the universe around it._

![]({{site.baseurl}}/assets/images/tuto-3-matrix/camera.png)

When you think about it, the same applies to cameras. It you want to view a moutain from another angle, you can either move the camera... or move the mountain. While not practical in real life, this is really simple and handy in Computer Graphics.

So initially your camera is at the origin of the World Space. In order to move the world, you simply introduce another matrix. Let's say you want to move your camera of 3 units to the right (+X). This is equivalent to moving your whole world (meshes included) 3 units to the LEFT ! (-X). While you brain melts, let's do it :

``` cpp
// Use #include <glm/gtc/matrix_transform.hpp> and #include <glm/gtx/transform.hpp>
glm::mat4 ViewMatrix = glm::translate(-3.0f, 0.0f ,0.0f);
```

Again, the image below illustrates this : _We went from World Space (all vertices defined relatively to the center of the world, as we made so in the previous section) to Camera Space (all vertices defined relatively to the camera)._

![]({{site.baseurl}}/assets/images/tuto-3-matrix/model_to_world_to_camera.png)

Before you head explodes from this, enjoy GLM's great glm::lookAt function:

``` cpp
glm::mat4 CameraMatrix = glm::lookAt(
    cameraPosition, // the position of your camera, in world space
    cameraTarget,   // where you want to look at, in world space
    upVector        // probably glm::vec3(0,1,0), but (0,-1,0) would make you looking upside-down, which can be great too
);
```

Here's the compulsory diagram :

![]({{site.baseurl}}/assets/images/tuto-3-matrix/MV.png)

This is not over yet, though.

## The Projection matrix

We're now in Camera Space. This means that after all theses transformations, a vertex that happens to have x==0 and y==0 should be rendered at the center of the screen. But we can't use only the x and y coordinates to determine where an object should be put on the screen : its distance to the camera (z) counts, too ! For two vertices with similar x and y coordinates, the vertex with the biggest z coordinate will be more on the center of the screen than the other.

This is called a perspective projection :

![]({{site.baseurl}}/assets/images/tuto-3-matrix/model_to_world_to_camera_to_homogeneous.png)

And luckily for us, a 4x4 matrix can represent this projection[^projection] :

``` cpp
// Generates a really hard-to-read matrix, but a normal, standard 4x4 matrix nonetheless
glm::mat4 projectionMatrix = glm::perspective(
    FoV,         // The horizontal Field of View, in degrees : the amount of "zoom". Think "camera lens". Usually between 90° (extra wide) and 30° (quite zoomed in)
    4.0f / 3.0f, // Aspect Ratio. Depends on the size of your window. Notice that 4/3 == 800/600 == 1280/960, sounds familiar ?
    0.1f,        // Near clipping plane. Keep as big as possible, or you'll get precision issues.
    100.0f       // Far clipping plane. Keep as little as possible.
);
```

One last time :

_We went from Camera Space (all vertices defined relatively to the camera) to Homogeneous Space (all vertices defined in a small cube. Everything inside the cube is onscreen)._

And the final diagram :

![]({{site.baseurl}}/assets/images/tuto-3-matrix/MVP.png)

Here's another diagram so that you understand better what happens with this Projection stuff. Before projection, we've got our blue objects, in Camera Space, and the red shape represents the frustum of the camera : the part of the scene that the camera is actually able to see.

![]({{site.baseurl}}/assets/images/tuto-3-matrix/nondeforme.png)

Multiplying everything by the Projection Matrix has the following effect :

![]({{site.baseurl}}/assets/images/tuto-3-matrix/homogeneous.png)

In this image, the frustum is now a perfect cube (between -1 and 1 on all axes, it's a little bit hard to see it), and all blue objects have been deformed in the same way. Thus, the objects that are near the camera ( = near the face of the cube that we can't see) are big, the others are smaller. Seems like real life !

Let's see what it looks like from the "behind" the frustum :

![]({{site.baseurl}}/assets/images/tuto-3-matrix/projected1.png)

Here you get your image ! It's just a little bit too square, so another mathematical transformation is applied (this one is automatic, you don't have to do it yourself in the shader) to fit this to the actual window size :

![]({{site.baseurl}}/assets/images/tuto-3-matrix/final1.png)

And this is the image that is actually rendered !

## Cumulating transformations : the ModelViewProjection matrix

... Just a standard matrix multiplication as you already love them !

``` cpp
// C++ : compute the matrix
glm::mat4 MVPmatrix = projection * view * model; // Remember : inverted !
```

``` glsl
// GLSL : apply it
transformed_vertex = MVP * in_vertex;
```
{: .highlightglslfs }

# Putting it all together

* First step : generating our MVP matrix. This must be done for each model you render.

  ``` cpp
  // Projection matrix : 45° Field of View, 4:3 ratio, display range : 0.1 unit <-> 100 units
  glm::mat4 Projection = glm::perspective(glm::radians(45.0f), (float) width / (float)height, 0.1f, 100.0f);
  
  // Or, for an ortho camera :
  //glm::mat4 Projection = glm::ortho(-10.0f,10.0f,-10.0f,10.0f,0.0f,100.0f); // In world coordinates
  
  // Camera matrix
  glm::mat4 View = glm::lookAt(
      glm::vec3(4,3,3), // Camera is at (4,3,3), in World Space
      glm::vec3(0,0,0), // and looks at the origin
      glm::vec3(0,1,0)  // Head is up (set to 0,-1,0 to look upside-down)
      );
  
  // Model matrix : an identity matrix (model will be at the origin)
  glm::mat4 Model = glm::mat4(1.0f);
  // Our ModelViewProjection : multiplication of our 3 matrices
  glm::mat4 mvp = Projection * View * Model; // Remember, matrix multiplication is the other way around
  ```

* Second step : give it to GLSL

  ``` cpp
  // Get a handle for our "MVP" uniform
  // Only during the initialisation
  GLuint MatrixID = glGetUniformLocation(program_id, "MVP");
  
  // Send our transformation to the currently bound shader, in the "MVP" uniform
  // This is done in the main loop since each model will have a different MVP matrix (At least for the M part)
  glUniformMatrix4fv(mvp_handle, 1, GL_FALSE, &mvp[0][0]);
  ```

* Third step : use it in GLSL to transform our vertices

  ``` glsl
  // Input vertex data, different for all executions of this shader.
  layout(location = 0) in vec3 vertexPosition_modelspace;
  
  // Values that stay constant for the whole mesh.
  uniform mat4 MVP;
  
  void main(){
    // Output position of the vertex, in clip space : MVP * position
    gl_Position =  MVP * vec4(vertexPosition_modelspace,1);
  }
  ```
  {: .highlightglslvs }

* Done ! Here is the same triangle as in tutorial 2, still at the origin (0,0,0), but viewed in perspective from point (4,3,3), heads up (0,1,0), with a 45° field of view.

![]({{site.baseurl}}/assets/images/tuto-3-matrix/perspective_red_triangle.png)

In tutorial 6 you'll learn how to modify these values dynamically using the keyboard and the mouse to create a game-like camera, but first, we'll learn how to give our 3D models some colour (tutorial 4) and textures (tutorial 5).

# Exercises

*   Try changing the glm::perspective
*   Instead of using a perspective projection, use an orthographic projection (glm::ortho)
*   Modify ModelMatrix to translate, rotate, then scale the triangle
*   Do the same thing, but in different orders. What do you notice ? What is the "best" order that you would want to use for a character ?

_Addendum_

[^projection]: [...]luckily for us, a 4x4 matrix can represent this projection : Actually, this is not correct. A perspective transformation is not affine, and as such, can't be represented entirely by a matrix. After beeing multiplied by the ProjectionMatrix, homogeneous coordinates are divided by their own W component. This W component happens to be -Z (because the projection matrix has been crafted this way). This way, points that are far away from the origin are divided by a big Z; their X and Y coordinates become smaller; points become more close to each other, objects seem smaller; and this is what gives the perspective. This transformation is done in hardware, and is not visible in the shader.