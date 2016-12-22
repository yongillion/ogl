---
layout: page
status: publish
published: true
title: '튜토리얼 11 : 2D 텍스트'
date: '2011-05-16 22:38:44 +0200'
date_gmt: '2011-05-16 22:38:44 +0200'
categories: [tuto]
order: 30
tags: []
---

이 튜토리얼에에서 우리는 3D 컨텐츠 위에 2D 텍스트를 그리는 방법을 배울 것입니다. 간단한 타이머를 만들어보겠습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-11-2d-text/clock.png)


# API

간단한 인터페이스를 구현해보겠습니다(common/text2D.h).

``` cpp
void initText2D(const char * texturePath);
void printText2D(const char * text, int x, int y, int size);
void cleanupText2D();
```

코드가 640*480과 1080p 모두에서 동작하게 하기 위해 x와 y좌표를 [0-800][0-600]사이로 하겠습니다. 정점 셰이더는 이를 실제 화면의 사이즈로 변환할 것입니다.
In order for the code to work at both 640*480 and 1080p, x and y will be coordinates in [0-800][0-600]. The vertex shader will adapt this to the actual size of the screen.

완전한 구현은 "common/text2D.cpp"을 보십시오.

# 텍스처

initText2D는 단순히 텍스처와 몇개의 셰이더를 읽어옵니다. 특별한건 없지만 텍스처를 한 번 살펴 보겠습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-11-2d-text/fontalpha.png)

이 텍스처는 폰트로부터 텍스처를 생성해주는 많은 툴들중 하나인 [CBFG](http://www.codehead.co.uk/cbfg/)를 이용해 생성하였습니다. 붉은 색 배경의 Paint.NET으로 불러들인 모습입니다(붉은 배경은 시각적인 목적입니다. 붉은 색으로 보이는 것은 모두 투명한 부분입니다.).

printText2D의 목적은 적적한 화면 위치와 텍스처 좌표를 가지는 쿼드(quad, 사각형)를 생성하는 것입니다.

# 그리기

아래 버퍼들을 채울 것입니다.

``` cpp
std::vector<glm::vec2> vertices;
std::vector<glm::vec2> UVs;
```
각 문자마다 쿼드를 정의하는 네 개의 정점에 대한 좌표를 계산해서, 두 개의 삼각형을 버퍼에 넣을 것입니다.

``` cpp
for ( unsigned int i=0 ; i<length ; i++ ){

    glm::vec2 vertex_up_left    = glm::vec2( x+i*size     , y+size );
    glm::vec2 vertex_up_right   = glm::vec2( x+i*size+size, y+size );
    glm::vec2 vertex_down_right = glm::vec2( x+i*size+size, y      );
    glm::vec2 vertex_down_left  = glm::vec2( x+i*size     , y      );

    vertices.push_back(vertex_up_left   );
    vertices.push_back(vertex_down_left );
    vertices.push_back(vertex_up_right  );

    vertices.push_back(vertex_down_right);
    vertices.push_back(vertex_up_right);
    vertices.push_back(vertex_down_left);
```

이제 UV에 대해서입니다. 각 문자의 좌상단으로부터의 좌표는 다음과 같이 계산합니다.

``` cpp
    char character = text[i];
    float uv_x = (character%16)/16.0f;
    float uv_y = (character/16)/16.0f;
```

이 코드는 [A의 ASCII code](http://www.asciitable.com/)가 65이기 때문에 동작합니다.

65%16 = 1, 그러므로 A는 #1 열입니다. (0부터 시작!).

65/16 = 4, 그러므로 A는 #4 행입니다. ( 정수에 대한 나눗셈이기 때문에, 4.0625이 아닌 4가 됩니다.)

OpenGL 텍스처에 사용되는 [0.0 - 1.0] 범위에 맞추기 위해 두 값을 모두 16.0으로 나눕니다.

이제 정점만 빼고는 이전에 했던 것과 동일합니다.

``` cpp
    glm::vec2 uv_up_left    = glm::vec2( uv_x           , 1.0f - uv_y );
    glm::vec2 uv_up_right   = glm::vec2( uv_x+1.0f/16.0f, 1.0f - uv_y );
    glm::vec2 uv_down_right = glm::vec2( uv_x+1.0f/16.0f, 1.0f - (uv_y + 1.0f/16.0f) );
    glm::vec2 uv_down_left  = glm::vec2( uv_x           , 1.0f - (uv_y + 1.0f/16.0f) );

    UVs.push_back(uv_up_left   );
    UVs.push_back(uv_down_left );
    UVs.push_back(uv_up_right  );

    UVs.push_back(uv_down_right);
    UVs.push_back(uv_up_right);
    UVs.push_back(uv_down_left);
 }
```

남은 것은 늘 그랬듯이 버퍼를 바인드하고, 버퍼를 채우고, 셰이더 프로그램을 선택하고, 텍스처를 바인드하고, 버텍스 어트리뷰트들을 활성화/바인드/설정하고, 블렌딩을 활성화하고, glDrawArray를 호출합니다. 와우 ! 해냈습니다.

주의해야할 매우 중요한 것이 있는데 좌표를 [0,800][0,600] 범위에서 생성하였다는 것입니다. 다른 말로는 여기서는 행렬이 필요 없다는 것입니다. 버텍스 셰이더는 아주 간단한 계산으로이 좌표들을 [-1,1][-1,1] 범위로 변환합니다(이 계산은 C++에서 할 수도 있습니다).

``` glsl
void main(){

    // Output position of the vertex, in clip space
    // map [0..800][0..600] to [-1..1][-1..1]
    vec2 vertexPosition_homoneneousspace = vertexPosition_screenspace - vec2(400,300); // [0..800][0..600] -> [-400..400][-300..300]
    vertexPosition_homoneneousspace /= vec2(400,300);
    gl_Position =  vec4(vertexPosition_homoneneousspace,0,1);

    // UV of the vertex. No special space for this one.
    UV = vertexUV;
}
```
{: .highlightglslvs }

프래그먼트 셰이더는 적은 일만 합니다.

``` glsl
void main(){
    color = texture( myTextureSampler, UV );
}
```
{: .highlightglslfs }

그런데 이 코드를 실제 제품에 사용하지 마십시오. 이 코드는 라틴 문자 밖에 다루지 못하기 때문입니다. 또는 인도나 중국, 일본 (독일까지, 이미지에 &szlig 문자는 없습니다)에 제품을 판매하지 마십시오. 이 텍스처는 제 로케일에서 만들었기 때문에 프랑스에서 거의 잘 동작할 것입니다(&eacute;, &agrave;, &ccedil;, 등). 다른 튜토리얼이나 라이브러리로부터 코드를 적용할 때, 그것들 대부분은 호환되지 않는 OpenGL 2를 사용한다는 것에 주의하십시오. 불행이도 저는 UTF-8을 다루는 좋은 라이브러리를 알지 못합니다.

어쨌든 조엘 스폴스키의 [The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets (No Excuses!)](http://www.joelonsoftware.com/articles/Unicode.html)는 반드시 읽어봐야 합니다.

만약 큰 텍스트를 필요로 한다면 [밸브의 논문](http://www.valvesoftware.com/publications/2007/SIGGRAPH2007_AlphaTestedMagnification.pdf)도 읽어 보시기 바랍니다.
