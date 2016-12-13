---
layout: page
status: publish
published: true
title: 'Tutorial 4 : A Colored Cube'
date: '2011-04-26 07:55:37 +0200'
date_gmt: '2011-04-26 07:55:37 +0200'
categories: [tuto]
order: 40
tags: []
---

네 번째 튜토리얼에 오신 것을 환영합니다! 이번 튜토리얼에서는 아래와 같은 것들을 배워 보겠습니다.

* 지루한 삼각형 대신 정육면체를 그려보겠습니다.
* 조금 화려한 색상을 입히고
* Z-버퍼가 무엇인지를 배울 겁니다.

# 정육면체 그리기

정육면체는 여섯 개의 정사각형 면으로 이루어집니다. OpenGL은 삼각형만을 받아들이므로 우리는 12개의 삼각형을 그려야 합니다. 각 면당 두 개씩 말이죠. 삼각형을 만들 때와 동일한 방법으로 정점들을 정의합니다.

``` cpp
// Our vertices. Three consecutive floats give a 3D vertex; Three consecutive vertices give a triangle.
// A cube has 6 faces with 2 triangles each, so this makes 6*2=12 triangles, and 12*3 vertices
static const GLfloat g_vertex_buffer_data[] = {
    -1.0f,-1.0f,-1.0f, // triangle 1 : begin
    -1.0f,-1.0f, 1.0f,
    -1.0f, 1.0f, 1.0f, // triangle 1 : end
    1.0f, 1.0f,-1.0f, // triangle 2 : begin
    -1.0f,-1.0f,-1.0f,
    -1.0f, 1.0f,-1.0f, // triangle 2 : end
    1.0f,-1.0f, 1.0f,
    -1.0f,-1.0f,-1.0f,
    1.0f,-1.0f,-1.0f,
    1.0f, 1.0f,-1.0f,
    1.0f,-1.0f,-1.0f,
    -1.0f,-1.0f,-1.0f,
    -1.0f,-1.0f,-1.0f,
    -1.0f, 1.0f, 1.0f,
    -1.0f, 1.0f,-1.0f,
    1.0f,-1.0f, 1.0f,
    -1.0f,-1.0f, 1.0f,
    -1.0f,-1.0f,-1.0f,
    -1.0f, 1.0f, 1.0f,
    -1.0f,-1.0f, 1.0f,
    1.0f,-1.0f, 1.0f,
    1.0f, 1.0f, 1.0f,
    1.0f,-1.0f,-1.0f,
    1.0f, 1.0f,-1.0f,
    1.0f,-1.0f,-1.0f,
    1.0f, 1.0f, 1.0f,
    1.0f,-1.0f, 1.0f,
    1.0f, 1.0f, 1.0f,
    1.0f, 1.0f,-1.0f,
    -1.0f, 1.0f,-1.0f,
    1.0f, 1.0f, 1.0f,
    -1.0f, 1.0f,-1.0f,
    -1.0f, 1.0f, 1.0f,
    1.0f, 1.0f, 1.0f,
    -1.0f, 1.0f, 1.0f,
    1.0f,-1.0f, 1.0f
};
```
표준 함수들(glGenBuffers, glBindBuffer, glBufferData, glVertexAttribPointer)을 이용해 OpenGL 버퍼를 생성하고, 바인드 하고 설정합니다. 튜토리얼 2를 빠르게 떠올려 보시죠. 드로우 콜은 변경되지 않습니다. 그려져야 할 적절한 수의 정점들을 설정했을 뿐이죠.

``` cpp
// Draw the triangle !
glDrawArrays(GL_TRIANGLES, 0, 12*3); // 0부터 시작하는 12*3 개의 인덱스들 -> 12 삼각형 -> 6 정사각형
```

이 코드에 몇 가지 주목할만한 것들이 있습니다.

* 현재 3D 모델은 고정되어 있습니다. 이를 변경하기 위해서는 소스 코드를 변경하여야 하며, 어플리케이션을 다시 컴파일해야 하는 것이 최선입니다. 튜토리얼 7에서 동적으로 모델을 로딩하는 방법을 배우도록 하죠. 
* 각 정점이 최소한 3번 쓰여집니다(위 코드에서 "-1.0f,-1.0f,-1.0f"를 검색해보세요). 이는 심각하게 메모리를 낭비합니다. 이를 다루는 방법에 대해 튜토리얼 9에서 배우도록 하죠. 

이제 흰색 정육면체를 그릴 모든 준비가 되어 있습니다. 셰이더를 동작시켜보죠 ! 어서요. 시도라도 해보세요 :)

# 색상 추가하기

색상는 개념적으로 위치와 동일합니다. 그저 데이터일뿐이죠. OpenGL 용어로 이를 "어트리뷰트"라고 부릅니다. 사실 우리는 이미  glEnableVertexAttribArray()와 glVertexAttribPointer()를 사용한 적이 있죠. 이제 새로운 어트리뷰트를 추가해봅시다. 코드가 매우 비슷할 거에요.

먼저 색상들을 선언합니다. 정점당 하나의 RGB 값(세개의 GLfloat 값)이 필요한데 여기에 제가 랜덤하게 생성한 값들이 있습니다. 결과가 사실 보기 좋지는 않을 겁니다. 여러분이 더 좋게 만들 수 있겠죠. 예를 들어 각 정점마다 고유의 색상을 가지도록요.

``` cpp
// One color for each vertex. They were generated randomly.
static const GLfloat g_color_buffer_data[] = {
    0.583f,  0.771f,  0.014f,
    0.609f,  0.115f,  0.436f,
    0.327f,  0.483f,  0.844f,
    0.822f,  0.569f,  0.201f,
    0.435f,  0.602f,  0.223f,
    0.310f,  0.747f,  0.185f,
    0.597f,  0.770f,  0.761f,
    0.559f,  0.436f,  0.730f,
    0.359f,  0.583f,  0.152f,
    0.483f,  0.596f,  0.789f,
    0.559f,  0.861f,  0.639f,
    0.195f,  0.548f,  0.859f,
    0.014f,  0.184f,  0.576f,
    0.771f,  0.328f,  0.970f,
    0.406f,  0.615f,  0.116f,
    0.676f,  0.977f,  0.133f,
    0.971f,  0.572f,  0.833f,
    0.140f,  0.616f,  0.489f,
    0.997f,  0.513f,  0.064f,
    0.945f,  0.719f,  0.592f,
    0.543f,  0.021f,  0.978f,
    0.279f,  0.317f,  0.505f,
    0.167f,  0.620f,  0.077f,
    0.347f,  0.857f,  0.137f,
    0.055f,  0.953f,  0.042f,
    0.714f,  0.505f,  0.345f,
    0.783f,  0.290f,  0.734f,
    0.722f,  0.645f,  0.174f,
    0.302f,  0.455f,  0.848f,
    0.225f,  0.587f,  0.040f,
    0.517f,  0.713f,  0.338f,
    0.053f,  0.959f,  0.120f,
    0.393f,  0.621f,  0.362f,
    0.673f,  0.211f,  0.457f,
    0.820f,  0.883f,  0.371f,
    0.982f,  0.099f,  0.879f
};
```

이전에 했던 방법과 완전히 동일한 방법으로 버퍼를 만들고 바인딩한 후 채웁니다. 

``` cpp
GLuint colorbuffer;
glGenBuffers(1, &colorbuffer);
glBindBuffer(GL_ARRAY_BUFFER, colorbuffer);
glBufferData(GL_ARRAY_BUFFER, sizeof(g_color_buffer_data), g_color_buffer_data, GL_STATIC_DRAW);
```

설정도 역시 동일하게요.

``` cpp
// 2nd attribute buffer : colors
glEnableVertexAttribArray(1);
glBindBuffer(GL_ARRAY_BUFFER, colorbuffer);
glVertexAttribPointer(
    1,                                // attribute. No particular reason for 1, but must match the layout in the shader.
    3,                                // size
    GL_FLOAT,                         // type
    GL_FALSE,                         // normalized?
    0,                                // stride
    (void*)0                          // array buffer offset
);
```

이제 정점 셰이더에서 추가된 이 버퍼를 접근할 수 있습니다.

``` glsl
// Notice that the "1" here equals the "1" in glVertexAttribPointer
layout(location = 1) in vec3 vertexColor;
```

{: .highlightglslvs }

우리는 정점 셰이더에서 아무것도 하지 않을 겁니다. 그저 단순히 프래그먼트 셰이더로 넘기도록 하겠습니다.

``` glsl
// Output data ; will be interpolated for each fragment.
out vec3 fragmentColor;

void main(){

    [...]

    // The color of each vertex will be interpolated
    // to produce the color of each fragment
    fragmentColor = vertexColor;
}
```
{: .highlightglslvs }

프래그먼트 셰이더에서 fragmentColor를 선언합니다.

``` glsl
// Interpolated values from the vertex shaders
in vec3 fragmentColor;
```
{: .highlightglslfs }

... 그리고 이걸 최종 출력 색상으로 복사합니다.

``` glsl
// Ouput data
out vec3 color;

void main(){
    // Output color = color specified in the vertex shader,
    // interpolated between all 3 surrounding vertices
    color = fragmentColor;
}
```
{: .highlightglslfs }

우리가 얻은 결과물입니다.

![]({{site.baseurl}}/assets/images/tuto-4-colored-cube/missing_z_buffer.png)

으악. 괴상하네요. 무슨 일이 벌어졌는지 이해하기 위해 "먼" 삼각형과 "가까운" 삼각형을 그릴 때 어떤 일이 발생하는지 살펴보도록 하죠.

![]({{site.baseurl}}/assets/images/tuto-4-colored-cube/FarNear.png)

좋습니다. 이제 "먼" 삼각형을 나중에 그리면 어떻게 될까요?

![]({{site.baseurl}}/assets/images/tuto-4-colored-cube/NearFar.png)

"가까운" 것보다 위에 그려져 버렸습니다. 더 뒤에 있는데도 불구하고 말이죠. 이것이 바로 우리 정육면체에 발생한 일입니다. 일부 면은 가려지기를 바랬지만 이 면들이 더 나중에 그려지는 바람에 앞에 보여지게 된 것입니다. 이를 해결하기 위해 Z-버퍼를 호출해 보죠!

*퀵 노트 1*: 만약 이러한 문제를 보지 못한다면 카메라 위치를 (4,3,-3)로 변경해 보세요.

*퀵 노트2:  색상이 위치와 동일하게 어트리뷰트라면 왜 색상을 위해서 out vec3 fragmentColor 와 in vec3 fragmentColor를 선언해야 할까요? 위치에서는 할 필요가 없고요. 그 이유는 위치는 조금 특별하기 때문입니다. 위치는 유일하게 필수적인 어트리뷰트입니다(그렇지 않다면 OpenGL은 삼각형을 어디에 그려야 하는지 알지 못할테니까요). 그리하여 정점 셰이더에서 gl_Position은 "내장된" 변수입니다.

# Z-버퍼

이 문제를 해결하는 방법은 각 프래그먼트의 깊이(즉 Z) 요소를 버퍼에 저장하는 것입니다. 그리고 프래그먼트를 그리고 싶은 매 순간 그릴 필요가 있는지 확인합니다(즉, 새로운 프래그먼트가 이전 프래그먼트보다 더 가까이 있는지를 확인합니다). 

이 작업을 직접 할 수도 있겠지만 하드웨어가 스스로 수행하도록 요청하는 게 훨씬 간단하겠네요.

``` cpp
// Enable depth test
glEnable(GL_DEPTH_TEST);
// Accept fragment if it closer to the camera than the former one
glDepthFunc(GL_LESS);
```

이제 매 프레임마다 색상뿐 아니라 깊이도 지워줘야 합니다.

``` cpp
// Clear the screen
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```

이 모든 문제를 해결하는데 이 정도면 충분합니다.

![]({{site.baseurl}}/assets/images/tuto-4-colored-cube/one_color_per_vertex.png)

# 연습

* 정육면체와 삼각형을 다른 위치에 그리세요. 두 개의 MVP 행렬과, 메인 루프에서 두 번의 드로우 콜 호출이 필요하지만 셰이더는 하나만 있으면 됩니다. 

* 색상 값을 직접 생성해보세요. 예를 들면, 실행할 때마다 랜덤하게 변경되게, 또는 정점의 위치에 따라, 두개의 색상을 섞는 등 창의적인 아이디어를 발휘해보세요 :) C를 모르신다면 아래 구문을 참고하세요.

``` cpp
static GLfloat g_color_buffer_data[12*3*3];
for (int v = 0; v < 12*3 ; v++){
    g_color_buffer_data[3*v+0] = your red color here;
    g_color_buffer_data[3*v+1] = your green color here;
    g_color_buffer_data[3*v+2] = your blue color here;
}
```

* 그 다음엔 매 프레임마다 색상이 변경되게 해보세요. 각 프레임마다 glBufferData를 호출해야 할 것입니다. 먼저 적절한 버퍼가 바인딩(glBindBuffer)되어야 한다는 것을 명심하세요.

