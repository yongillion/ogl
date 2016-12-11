---
layout: page
status: publish
published: true
title: 'Tutorial 2 : The first triangle'
date: '2011-04-07 18:54:11 +0200'
date_gmt: '2011-04-07 18:54:11 +0200'
categories: [tuto]
order: 20
tags: []
---
{:TOC}

이번에도 꽤 긴 튜토리얼이 될 것입니다.

OpenGL 3를 사용하면 복잡한 작업을 쉽게 작성할 수 있긴 하지만, 사실은 간단한 삼각형을 그리는 것조차 꽤나 어렵습니다.

주기적으로 코드를 잘라내기/붙여넣기 하는 것을 잊지 마십시오.

**<span style="color: red">만약 시작시 프로그램이 중단된다면, 잘못된 디렉토리에서 실행하고 있을 확률이 높습니다. 첫 번째 튜토리얼을 주의 깊게 읽고 Visual Studio를 설정하세요.</span>**

## VAO

여기서 자세히 다루지는 않겠지만, Vertex Array Object 라는 것을 만들어서 현재 객체로 설정해야 합니다.

``` cpp
GLuint VertexArrayID;
glGenVertexArrays(1, &VertexArrayID);
glBindVertexArray(VertexArrayID);
```

윈도우를 만든 후 (= OpenGL 컨텍스트가 생성된 후) 다른 OpenGL 호출 이전에 이 작업을 한번 수행합니다. 

VAO에 대해 더 자세히 알고 싶다면 몇가지 다른 튜토리얼들이 있지만 별로 중요하지는 않습니다.

## 화면 좌표

삼각형은 세 점으로 정의됩니다. 3D 그래픽에서 "점(point)"에 관해 얘기할 때 우리는 보통 "정점(vertex)"이라는 단어를 사용하는데(복수형 "vertices"), 정점은 세개의 좌표로 이루어집니다. X, Y, Z.
이 세 좌표에 대해서 다음과 같은 방법으로 생각할 수 있습니다.

- X 는 오른쪽
- Y 는 위쪽
- Z 는 당신의 뒤쪽(맞아요, 뒤쪽. 앞쪽이 아니라)

이를 시각화하는 더 좋은 방법이 있습니다. 오른손 법칙을 사용하는 것입니다.

- X는 엄지.
- Y는 검지
- Z는 중지. 엄지를 오른쪽으로 향하게 하고, 검지를 하늘로 향하게 하면, 중지가 등 뒤쪽을 향하게 될 겁니다.

Z가 뒤쪽을 가리키는게 이상해 보이나요? 100년이 넘은 오른손 법칙 수학은 많은 유용한 도움을 줍니다. 단점은 직관적이지 않은 Z뿐입니다.

참고로 손을 자유롭게 움직일 수 있다는 것에 주목하시길 바랍니다. 손을 움직이면 여러분의 X, Y, Z 또한 움직입니다. 나중에 더 자세히 살펴보도록 하겠습니다.

우리는 삼각형을 만들기 위해 세개의 3D 좌표가 필요합니다.

``` cpp
// An array of 3 vectors which represents 3 vertices
static const GLfloat g_vertex_buffer_data[] = {
   -1.0f, -1.0f, 0.0f,
   1.0f, -1.0f, 0.0f,
   0.0f,  1.0f, 0.0f,
};
```

첫 번째 정점은 (-1,-1,0)입니다. 이는 우리가 어떠한 방법으로 변환하지 않는 한, 화면상의 (-1,-1)에 출력될 것이라는 것을 의미합니다. 화면 원점은 중심에 있으며, X는 오른쪽, Y는 위쪽입니다. 이것은 와이드 스크린에서 제공되는 것입니다.

![screenCoordinates]({{site.baseurl}}/assets/images/tuto-2-first-triangle/screenCoordinates.png){: height="165px" width="300px"}

이는 그래픽 카드에 내장된 것으로 변경할 수 없습니다. (-1,-1)은 화면의 왼쪽 아래를 나타내며, (0,1)은 화면의 위쪽 가운데를 나타냅니다. 우리가 준 삼각형 좌표는 화면의 대부분을 차지하게 될 것입니다.

## 삼각형 그리기

다음 단계는 이 삼각형을 OpenGL에게 제공하는 것입니다. 우리는 버퍼를 생성함으로써 이를 수행합니다. 

```cpp
// 이 변수는 버택스 버퍼를 식별하는데 사용됩니다.
GLuint vertexbuffer;
// 한개의 버퍼를 생성합니다. vertexbuffer에 식별자가 반환됩니다.
glGenBuffers(1, &vertexbuffer);
// 아래 명령은 'vertexbuffer'를 buffer로 사용하도록 합니다.
glBindBuffer(GL_ARRAY_BUFFER, vertexbuffer);
// OpenGL에 정점들을 제공합니다.
glBufferData(GL_ARRAY_BUFFER, sizeof(g_vertex_buffer_data), g_vertex_buffer_data, GL_STATIC_DRAW);
```


이 작업은 한번만 수행합면 됩니다.

이제 아무것도 그리지 않았었던 메인 루프에서 우리는 웅장한 삼각형을 그릴 수 있습니다. 

``` cpp
// 1rst attribute buffer : vertices
glEnableVertexAttribArray(0);
glBindBuffer(GL_ARRAY_BUFFER, vertexbuffer);
glVertexAttribPointer(
   0,                  // attribute 0. No particular reason for 0, but must match the layout in the shader.
   3,                  // size
   GL_FLOAT,           // type
   GL_FALSE,           // normalized?
   0,                  // stride
   (void*)0            // array buffer offset
);
// Draw the triangle !
glDrawArrays(GL_TRIANGLES, 0, 3); // Starting from vertex 0; 3 vertices total -> 1 triangle
glDisableVertexAttribArray(0);
```

운이 좋다면 다음과 같은 결과를 볼 수 있을 것입니다. (<span style="color: red">**보지 못하더라도 당황하지 마세요**</span>) :

![triangle_no_shader]({{site.baseurl}}/assets/images/tuto-2-first-triangle/triangle_no_shader1.png){: height="232px" width="300px"}

좀 지루한 흰색입니다. 붉은 색으로 칠해 보도록 하겠습니다. 이를 위해 '쉐이더'라는 것을 사용할 것입니다.

## 쉐이더

# 쉐이더 컴파일

가장 간단한 구성에서는 두 개의 쉐이더가 필요합니다. 각 정점에 대해 수행될 버텍스 쉐이더(Vertex Shader)와 각 샘플에 대해 수행될 프래그먼트 쉐이더입니다. 4x 안티앨리어생을 사용하기 때문에 각 픽셀당 4개의 샘플이 존재합니다.

셰이더는 OpenGL의 일부인 GLSL: GL Shader Language라고 부르는 언어로 프로그래밍 됩니다. C나 Java와 달리 GLSL은 런타임에 컴파일되어야 합니다. 즉 어플리케이션이 수행될 때마다 모든 쉐이더가 다시 컴파일된다는 의미입니다.

두 개의 쉐이더는 대개 별개의 파일에 있습니다. 이 예제에서 우리는 SimpleFragmentShader.fragmentshader와 SimpleVertexShader.vertexshader를 사용합니다. 확장자는 무의미하며, .txt나 .glsl이 될 수도 있습니다.

여기 코드가 있습니다. 프로그램에서 딱 한번만 이 작업을 수행하므로, 코드를 완전히 이해할 필요는 없으며 주석만으로 충분합니다. 이 함수는 다른 모든 튜토리얼에서도 사용되므로 별개의 파일 "common/loadShader.cpp"에 두었습니다. 버퍼와 마찬가지로 쉐이더 역시 직접 접근할 수 없으며, 단지 ID를 통해 접근합니다. 실제 구현은 드라이버 내부에 숨겨져 있습니다.

``` cpp
GLuint LoadShaders(const char * vertex_file_path,const char * fragment_file_path){

	// Create the shaders
	GLuint VertexShaderID = glCreateShader(GL_VERTEX_SHADER);
	GLuint FragmentShaderID = glCreateShader(GL_FRAGMENT_SHADER);

	// Read the Vertex Shader code from the file
	std::string VertexShaderCode;
	std::ifstream VertexShaderStream(vertex_file_path, std::ios::in);
	if(VertexShaderStream.is_open()){
		std::string Line = "";
		while(getline(VertexShaderStream, Line))
			VertexShaderCode += "\n" + Line;
		VertexShaderStream.close();
	}else{
		printf("Impossible to open %s. Are you in the right directory ? Don't forget to read the FAQ !\n", vertex_file_path);
		getchar();
		return 0;
	}

	// Read the Fragment Shader code from the file
	std::string FragmentShaderCode;
	std::ifstream FragmentShaderStream(fragment_file_path, std::ios::in);
	if(FragmentShaderStream.is_open()){
		std::string Line = "";
		while(getline(FragmentShaderStream, Line))
			FragmentShaderCode += "\n" + Line;
		FragmentShaderStream.close();
	}

	GLint Result = GL_FALSE;
	int InfoLogLength;


	// Compile Vertex Shader
	printf("Compiling shader : %s\n", vertex_file_path);
	char const * VertexSourcePointer = VertexShaderCode.c_str();
	glShaderSource(VertexShaderID, 1, &VertexSourcePointer , NULL);
	glCompileShader(VertexShaderID);

	// Check Vertex Shader
	glGetShaderiv(VertexShaderID, GL_COMPILE_STATUS, &Result);
	glGetShaderiv(VertexShaderID, GL_INFO_LOG_LENGTH, &InfoLogLength);
	if ( InfoLogLength > 0 ){
		std::vector<char> VertexShaderErrorMessage(InfoLogLength+1);
		glGetShaderInfoLog(VertexShaderID, InfoLogLength, NULL, &VertexShaderErrorMessage[0]);
		printf("%s\n", &VertexShaderErrorMessage[0]);
	}



	// Compile Fragment Shader
	printf("Compiling shader : %s\n", fragment_file_path);
	char const * FragmentSourcePointer = FragmentShaderCode.c_str();
	glShaderSource(FragmentShaderID, 1, &FragmentSourcePointer , NULL);
	glCompileShader(FragmentShaderID);

	// Check Fragment Shader
	glGetShaderiv(FragmentShaderID, GL_COMPILE_STATUS, &Result);
	glGetShaderiv(FragmentShaderID, GL_INFO_LOG_LENGTH, &InfoLogLength);
	if ( InfoLogLength > 0 ){
		std::vector<char> FragmentShaderErrorMessage(InfoLogLength+1);
		glGetShaderInfoLog(FragmentShaderID, InfoLogLength, NULL, &FragmentShaderErrorMessage[0]);
		printf("%s\n", &FragmentShaderErrorMessage[0]);
	}



	// Link the program
	printf("Linking program\n");
	GLuint ProgramID = glCreateProgram();
	glAttachShader(ProgramID, VertexShaderID);
	glAttachShader(ProgramID, FragmentShaderID);
	glLinkProgram(ProgramID);

	// Check the program
	glGetProgramiv(ProgramID, GL_LINK_STATUS, &Result);
	glGetProgramiv(ProgramID, GL_INFO_LOG_LENGTH, &InfoLogLength);
	if ( InfoLogLength > 0 ){
		std::vector<char> ProgramErrorMessage(InfoLogLength+1);
		glGetProgramInfoLog(ProgramID, InfoLogLength, NULL, &ProgramErrorMessage[0]);
		printf("%s\n", &ProgramErrorMessage[0]);
	}

	
	glDetachShader(ProgramID, VertexShaderID);
	glDetachShader(ProgramID, FragmentShaderID);
	
	glDeleteShader(VertexShaderID);
	glDeleteShader(FragmentShaderID);

	return ProgramID;
}
```

# 버텍스 쉐이더

먼저 버텍스 쉐이더를 작성해 봅시다. 첫 번재 행은 컴파일러에게 OpenGL 3 문법을 사용할 것임을 알려줍니다.

``` glsl
#version 330 core
```

두 번째 행은 입력 데이터를 선언합니다.

``` glsl
layout(location = 0) in vec3 vertexPosition_modelspace;
```

이 행에 대해 더 자세히 설명하겠습니다.

- "vec3"는 GLSL에서 3개의 구성요소를 가지는 벡터입니다. 우리가 삼각형을 선언할 때 사용한 glm::vec3와 유사하지만 조금 다릅니다. 중요한 것은 우리가 C++에서 3개의 구성요소를 사용한다면, GLSL에서도 3개의 구성요소를 사용한다는 것입니다.
- "layout(location = 0)"는 *vertexPosition_modelspace*를 제공하는데 사용되는 버퍼를 나타냅니다. 각 정점은 다수의 어트리뷰트(속성)을 가질 수 있습니다. 위치, 하나 이상의 색상, 하나 이상의 텍스처 좌표, 많은 다양한 것들. OpenGL은 색상이 무엇인지 모릅니다. 그저 vec3만 볼 뿐입니다. 그러므로 우리는 어떠한 버퍼가 어떠한 입력에 대응되는지를 말해줘야 합니다. glVertexAttribPointer의 첫 번째 매개 변수와 같은 값으로 레이아웃을 설정하면 됩니다. 값이 "0"인지는 중요하지 않습니다. 12가 될 수도 있습니다(하지만 glGetIntegerv(GL_MAX_VERTEX_ATTRIBS, &v)보다 클 순 없습니다). 중요한 것은 양쪽이 같은 숫자여야 한다는 것입니다.
- "vertexPosition_modelspace"는 다른 이름이어도 됩니다. 이 값은 버텍스 쉐이더가 실행될 때마다 버텍스의 좌표를 포함합니다. 
- "in"은 어떠한 입력 데이터임을 의미합니다. 곧 "out" 키워드도 보게 될 것입니다. 

각 정점에 대해 호출되는 함수는 C와 동일하게 main이라고 부릅니다. 

``` glsl
void main(){
```

이 main 함수는 그저 정점의 위치를 버퍼에 있던 값으로 설정합니다. 그러므로 만약 우리가 (1,1)을 주면 삼각형은 화면 오른쪽 상단의 정점을 하나 가지게 될 것입니다. 다음 튜토리얼에서 입력된 위치에 대해 보다 흥미로운 계산을 하는 방법을 살펴 보겠습니다.

``` glsl
  gl_Position.xyz = vertexPosition_modelspace;
  gl_Position.w = 1.0;
}
```

gl_Position은 내장 변수 중에 하나입니다. *반드시* 이 변수에 어떠한 값을 할당해야 합니다. 나머지 변수들은 선택적으로 할당하면 됩니다. 튜토리얼 4에서 나머지 변수들에 대해 알아보도록 하겠습니다.

# 프래그먼트 쉐이더

For our first fragment shader, we will do something really simple : set the color of each fragment to red. (Remember, there are 4 fragment in a pixel because we use 4x AA)

첫 번째 프래그먼트 쉐이더는 정말 간단한 작업을 수행합니다. 각 프래그먼트의 색상을 빨간색으로 설정합니다(4x AA를 사용하기 때문에 픽셀당 4개의 프래그먼트가 존재한다는 것을 기억하십시오).

``` glsl
#version 330 core
out vec3 color;
void main(){
  color = vec3(1,0,0);
}
```

vec3(1,0,0)은 빨간색을 의미합니다. 이는 컴퓨터 화면의 색상은  빨강, 초록, 파랑, 이 세가지가 순서대로 표현되기 때문입니다. 그러므로 (1,0,0)은 초록과 파랑이 없는 완전한 빨간색을 의미합니다.

## 모든 것을 하나로

메인 루프 이전에 LoadShaders 함수를 호출합니다.

```cpp
// Create and compile our GLSL program from the shaders
GLuint programID = LoadShaders( "SimpleVertexShader.vertexshader", "SimpleFragmentShader.fragmentshader" );
```

메인 루프 안에서 먼저 화면을 지웁니다. 이전에 glClearColor(0.0f, 0.0f, 0.4f, 0.0f)를 호출했기 때문에 배경 색이 진한 파란색으로 변경될 것입니다.

``` cpp
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```

그리고 OpenGL에게 쉐이더를 사용하고 싶다고 알려주면 됩니다.

``` cpp
// Use our shader
glUseProgram(programID);
// Draw triangle...
```

... 드디어 여기 빨간 삼각형이 있습니다!

![red_triangle]({{site.baseurl}}/assets/images/tuto-2-first-triangle/red_triangle.png){: height="231px" width="300px"}

In the next tutorial we'll learn transformations : How to setup your camera, move your objects, etc.
다음 튜토리얼에서는 변환에 대해 배우게 됩니다. 카메라를 셋업하는 방법과, 오브젝트를 이동시키는 방법 등을 배우겠습니다.  
