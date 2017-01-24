---
layout: page
status: publish
published: true
title: 'Tutorial 14 : Render To Texture'
date: '2011-05-26 19:33:15 +0200'
date_gmt: '2011-05-26 19:33:15 +0200'
categories: [tuto]
order: 60
tags: []
---
Render-To-Texture는 다양한 효과를 만드는데 유용한 방법입니다. 기본 아이디어는 지금까지 해왔듯이 씬을 렌더링하지만 텍스처에 렌더링을 해 나중에 다시 사용하는 것입니다.

게임 내 카메라나, 포스트 프로세싱 등 여러분이 상상할 수 있는 다양한 GFX를 어플리케이션에 포함시킬 수 있습니다.

# Render To Texture

세 가지 작업을 수행 할 것입니다. 렌더링을 할 텍스처를 생성하고, 텍스처에 실제로 렌더링을 수행한 후, 생성된 텍스처를 사용합니다. 

## 렌더 타겟 생성

우리가 렌더링을 하는 곳을 프레임버퍼라고 부릅니다. 프레임버퍼는 텍스처들과 깊이버퍼(선택적으로)의 컨테이너입니다. OpenGL에서 다른 오브젝트들과 비슷한 방법으로 만들어집니다. 


``` cpp
// 0, 1 또는 더 많은 텍스처와 0나 1 깊이 버퍼를 다시 그룹핑 할 프레임 버퍼.
GLuint FramebufferName = 0;
glGenFramebuffers(1, &FramebufferName);
glBindFramebuffer(GL_FRAMEBUFFER, FramebufferName);
```

셰이더가 출력하는 RGB가 저장될 텍스처를 생성해야 합니다. 이 코드 역시 어렵지 않습니다.

``` cpp
// 렌더링 할 텍스처
GLuint renderedTexture;
glGenTextures(1, &renderedTexture);

// 새로 생성한 텍스처를 "Bind"합니다. 앞으로의 텍스처 함수들은 이 텍스처를 수정 할 것입니다.
glBindTexture(GL_TEXTURE_2D, renderedTexture);

// Give an empty image to OpenGL ( the last "0" )
glTexImage2D(GL_TEXTURE_2D, 0,GL_RGB, 1024, 768, 0,GL_RGB, GL_UNSIGNED_BYTE, 0);

// Poor filtering. Needed !
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
```
깊이 버퍼도 필요합니다. 실제로 텍스처에 무엇을 그릴지에 따라 필요 없을 수도 있지만, 우리는 수잔을 그릴 것이기 때문에 깊이 테스트가 필요합니다.

``` cpp
// 깊이 버퍼
GLuint depthrenderbuffer;
glGenRenderbuffers(1, &depthrenderbuffer);
glBindRenderbuffer(GL_RENDERBUFFER, depthrenderbuffer);
glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH_COMPONENT, 1024, 768);
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, GL_RENDERBUFFER, depthrenderbuffer);
```

끝으로 프레임버퍼를 설정합니다.

``` cpp
// "renderedTexture"를 색상 attachement #0으로 설정합니다.
glFramebufferTexture(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, renderedTexture, 0);

// 드로우 버퍼의 리스트를 설정합니다.
GLenum DrawBuffers[1] = {GL_COLOR_ATTACHMENT0};
glDrawBuffers(1, DrawBuffers); // "1" is the size of DrawBuffers
```

``` cpp
// 항상 프레임 버퍼의 상태를 확인합니다.
if(glCheckFramebufferStatus(GL_FRAMEBUFFER) != GL_FRAMEBUFFER_COMPLETE)
return false;
```

## 텍스처로 렌더링하기

텍스처로 렌더링하는 것은 몹시 직관적입니다. 단순히 프레임 버퍼를 바인딩하고 이전과 같이 씬을 그리면 됩니다.

``` cpp
// 프레임 버퍼로 렌더링
glBindFramebuffer(GL_FRAMEBUFFER, FramebufferName);
glViewport(0,0,1024,768); // Render on the whole framebuffer, complete from the lower left corner to the upper right
```

프래그먼트 셰이더에 약간의 변형이 필요합니다. 

``` cpp
layout(location = 0) out vec3 color;
```

이는 변수 "color"에 기록할 때, 실제로 우리가 만든 텍스처인 Render Target 0에 기록되는 것을 의미합니다. 그 이유는 DrawBuffer[0]가 GL_COLOR_ATTACHMENT*i*이이며 이 경우 *renderedTexture*이기 때문입니다.

다시 말하면

* *color*은 layour(location=0)이기 때문에 첫 번째 버퍼로 기록됩니다.
* DrawBuffers[1] = {GL_COLOR_ATTACHMENT0} 이므로 첫 번째 버퍼는 GL_COLOR_ATTACHMENT0입니다.
* GL_COLOR_ATTACHMENT0에 "renderedTexture*를 부착하였으므로, 여기에 색상이 기록됩니다.
* 달리 말하면 GL_COLOR_ATTACHMENT0를 GL_COLOR_ATTACHMENT로 변경해도 여전히 동작합니다.

주의 : OpenGL < 3.3에서는 layout(location=i)이 없습니다. 대신 glFragData[i] = mvvalue 를 사용할 수 있씁니다.

<div><span style="font-size: medium;"><span style="line-height: 24px;">
</span></span></div>

## 렌더링 된 텍스처 사용하기

화면을 가득채우는 간단한 사각형(quad)를 그릴 것입니다. 이제껏 그래왔듯이 버퍼, 쉐이더, ID 등이 필요합니다.

``` cpp
// 풀스크린 사각형의 FBO
GLuint quad_VertexArrayID;
glGenVertexArrays(1, &quad_VertexArrayID);
glBindVertexArray(quad_VertexArrayID);

static const GLfloat g_quad_vertex_buffer_data[] = {
    -1.0f, -1.0f, 0.0f,
    1.0f, -1.0f, 0.0f,
    -1.0f,  1.0f, 0.0f,
    -1.0f,  1.0f, 0.0f,
    1.0f, -1.0f, 0.0f,
    1.0f,  1.0f, 0.0f,
};

GLuint quad_vertexbuffer;
glGenBuffers(1, &quad_vertexbuffer);
glBindBuffer(GL_ARRAY_BUFFER, quad_vertexbuffer);
glBufferData(GL_ARRAY_BUFFER, sizeof(g_quad_vertex_buffer_data), g_quad_vertex_buffer_data, GL_STATIC_DRAW);

// 쉐이더의 GLSL 프로그램을 생성하고 컴파일합니다.
GLuint quad_programID = LoadShaders( "Passthrough.vertexshader", "SimpleTexture.fragmentshader" );
GLuint texID = glGetUniformLocation(quad_programID, "renderedTexture");
GLuint timeID = glGetUniformLocation(quad_programID, "time");
```
이제 화면에 렌더링하기를 원합니다. glBindFramebuffer의 두 번째 파라미터에 0을 넘기면 됩니다.

``` cpp
// 화면으로 렌더링
glBindFramebuffer(GL_FRAMEBUFFER, 0);
glViewport(0,0,1024,768); // Render on the whole framebuffer, complete from the lower left corner to the upper right
```

풀 스크린 사각형을 아래와 같은 쉐이더로 그릴 수 있습니다.

``` glsl
#version 330 core

in vec2 UV;

out vec3 color;

uniform sampler2D renderedTexture;
uniform float time;

void main(){
    color = texture( renderedTexture, UV + 0.005*vec2( sin(time+1024.0*UV.x),cos(time+768.0*UV.y)) ).xyz;
}
```
{: .highlightglslfs }

이 코드는 단순히 텍스처를 샘플링하지만 시간에 따라 약간의 오프셋을 추가합니다.  

# 결과

 

![](http://www.opengl-tutorial.org/assets/images/tuto-14-render-to-texture/wavvy.png)


# 추가 작업


## 깊이 사용하기

경우에 따라 렌더링 텍스처를 사용할 때 깊이를 필요로 할 수도 있습니다. 이 경우 단순히 아래와 같은 방법으로 생성한 텍스처에 렌더링하면 됩니다.

``` cpp
glTexImage2D(GL_TEXTURE_2D, 0,GL_DEPTH_COMPONENT24, 1024, 768, 0,GL_DEPTH_COMPONENT, GL_FLOAT, 0);
```

("24"는 비트 단위의 정밀도입니다. 필요에 따라 16, 24, 32 중에 선택할 수 있습니다. 일반적으로 24가 좋습니다.)

시작하기에 이 정도면 충분하지만 제공된 소스 코드 역시 이것을 구현합니다.

이 방법은 드라이버가 [Hi-Z](http://fr.slideshare.net/pjcozzi/z-buffer-optimizations)와 같은 최적화를 이용할 수 없기 때문에 다소 느릴 수 있다는 것에 주의하십시오. 

이 스크린샷에서 깊이 레벨은 인위적으로 "과장"하였습니다. 일반적으로 깊이 텍스처에서 무엇인가를 보기는 매우 어렵습니다. Near = Z near 0 = black, far = Z near 1 = white. 

![](http://www.opengl-tutorial.org/assets/images/tuto-14-render-to-texture/wavvydepth.png)


## 멀티 샘플링 

"기본" 텍스처 대신 멀티 샘플링된 텍스처로 기록할 수도 있습니다. C++ 코드에서 glTexImage2D 대신 [glTexImage2DMultisample](http://www.opengl.org/sdk/docs/man3/xhtml/glTexImage2DMultisample.xml)를 사용하고, 프래그먼트 쉐이더에서 smapler2D/texture 대신 sampler2DMS/texelFetch를 사용하면 됩니다.

큰 주의사항이 하나 있는데,  texelFetch는 가져올 샘플의 수를 나타내는 인자를 필요로 합니다. 다른 말로는 자동 "필터링"이 없다는 말입니다(멀티 샘플링에 대해서 이야기 할 때 정확한 용어는 "resolution"입니다).

So you may have to resolve the MS texture yourself, in another, non-MS texture, thanks to yet another shader.

어렵진 않지만 덩치가 큽니다.

## Multiple Render Targets

동시에 여러 텍스처에 기록할 수도 있습니다.

그저 여러 개의 텍스처를 만들고(모두 같은 사이즈여야 합니다!), 각각에 대해 다른 색상 어태치먼트로 glFramebufferTexture를 호출하고, glDrawBuffer를 업데이트 된 파라미터((2,{GL_COLOR_ATTACHMENT0,GL_COLOR_ATTACHMENT1}})와 같은)로 호출한 후, 프래그먼트 쉐이더에 다른 출력 변수를 추가합니다. 

``` glsl
layout(location = 1) out vec3 normal_tangentspace; // or whatever
```
{: .highlightglslfs }

힌트: 만약 효과적으로 벡터를 텍스처로 출력하고 싶다면 8비트 대신 16이나 32비트 정밀도를 가지는 플로팅-포인트 텍스처가 존재합니다. [glTexImage2D](http://www.opengl.org/sdk/docs/man/xhtml/glTexImage2D.xml)의 레퍼런스를 참조합니다(GL_FLOAT을 검색하세요).

힌트2 : 이전 버전의 OpenGL에서는 glFragData[1] = myvalue를 대신 사용합니다.

# 연습

* glViewport(0,0,1024,768) 대신 glViewport(0,0,512,768)를 사용해 보세요. 프레임버퍼와 화면 모두에 시도해 봅니다.
* 마지막 프래그먼트 쉐이더에서 다른 UV 좌표를 사용해 보세요.
* 쿼드를 실제 변환 행렬로 변환해 보세요. 처음에는 하드 코딩 해본 후 controls.hpp의 함수들로 시도해 봅니다. 무엇을 알게 되었나요?

