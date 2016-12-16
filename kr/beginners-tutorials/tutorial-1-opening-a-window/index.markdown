---
layout: page
status: publish
published: true
title: '튜토리얼 1 : 창 열기'
date: '2011-04-07 17:54:16 +0200'
date_gmt: '2011-04-07 17:54:16 +0200'
categories: [tuto]
order: 10
tags: []
---

# 소개

첫 번째 튜토리얼에 오신것을 환영합니다.

OpenGL로 뛰어들기 전에 각 튜토리얼과 함께 하는 코드를 빌드하는 방법을 먼저 배우겠습니다. 그리고 가장 중요한, 이 코드들을 여러분이 직접 가지고 노는 방법을 배우겠습니다.

# 준비물

이 튜토리얼을 따라하기 위해서 특별한 사전 준비가 필요하지는 않습니다. 어떠한 프로그래밍 언어(C, Java, Lisp, Javascript, 무엇이든)의 경험이 있다면 코드를 완전히 이해하는데 더 도움이 되겠지만 필수는 아닙니다. 그저 두가지를 동시에 배워야하기에 좀 더 복잡해질 뿐이지요.

모든 튜토리얼은 "C++로 쉽게" 쓰여져 있습니다. 가급적 코드를 단순하게 만들려고 많은 노력을 기울였습니다. 템플릿도 없고, 클래스도 없으며, 포인터도 없습니다. 그렇기 때문에 Java밖에 모르더라도 모든 것을 이해할 수 있을 것입니다.

# 모든 걸 잊어버리세요

아무것도 알 필요가 없습니다. 다만 OpenGL에 대해 이미 알고 있는 모든 것을 잊어 버리십시오. 만약 glBegin()과 같은 것들을 알고 있다면 잊어 버리십시오. 여러분은 여기서 최신 OpenGL(OpenGL 3와 4)를 배울 것입니다. 대부분의 온라인 튜토리얼은 "낡은" OpenGL (OpenGL 1과 2)를 가르치고 있습니다. 그러므로 여러분의 머리가 혼란스러워 녹아 버리기 전에 모든 것을 잊어 버리십시오.
 
# 튜토리얼 빌드하기

모든 튜토리얼은 윈도우와 리눅스, 그리고 맥에서 빌드가 가능합니다. 이 모든 플랫폼에서 그 순서는 대략 다음과 같습니다.

* **드라이버를 업데이트하세요**!! 반드시요. 경고합니다.
* 만약 컴파일러가 없다면 다운 받으세요.
* CMake를 설치하세요.
* 튜토리얼의 소스코드를 다운로드하세요.
* CMake를 이용해 프로젝트를 생성하세요.
* 프로젝트를 빌드하세요.
* 샘플을 가지고 노세요!

이제 각 플랫폼마다 자세한 절차를 제공할 것입니다. 어느 정도 수정이 필요할 수도 있습니다. 확실치 않다면 윈도우에서의 설명을 잘 읽어 보고 적용해보세요.

## 윈도우에서 빌드하기
 
* 드라이버를 업데이트하는 것은 아주 쉽습니다. NVIDIA나 AMD의 웹사이트로 가서 드라이버를 다운 받으세요. GPU 모델을 모른다면 "제어판 -> 시스템 -> 장치 관리자 -> 디스플레이 어댑터"를 확인하세요. 통합 인텔 GPU를 사용하고 있다면 드라이버는 보통 OEM 회사들(Dell, HP, ...)에 의해 제공될 것입니다.
* Visual Studio 2015 Express를 컴파일러로 사용하기를 권장합니다. [여기서](https://www.visualstudio.com/en-US/products/visual-studio-express-vs) 무료로 다운받을 수 있습니다. 
* MinGW를 더 선호한다면 [Qt Creator](http://qt-project.org/)를 사용하기를 추천합니다. 원하는 것을 설치하세요. 뒤의 단계에서는 Visual Studio를 기준으로 설명하지만 다른 IDE에서도 비슷합니다.
* [CMake](http://www.cmake.org/cmake/resources/software.html)를 다운로드하여 설치하세요.
* [소스코드](http://www.opengl-tutorial.org/download/)를 다운로드하고 압축을 해제하세요. 예를 들면 "C:\Users\XYZ\Projects\OpenGLTutorials\"에 해제합니다.
* CMake를 실행합니다. 첫번째 행에 압축을 해제한 폴더를 지정합니다. 자신이 없다면 CMakeLists.txt 파일을 포함하고 있는 폴더를 선택하세요. 두 번째 행에는 컴파일러에 필요한 모든 것들이 저장될 폴더를 지정합니다. 예를 들면 "C:\Users\XYZ\Projects\OpenGLTutorials-build-Visual2015-64bits\" 혹은 "C:\Users\XYZ\Projects\OpenGLTutorials\build\Visual2015-36bits\". 어디든지 될 수 있으며 꼭 같은 폴더일 필요는 없습니다.
![](http://www.opengl-tutorial.org/assets/images/tuto-1-window/CMake.png)

* Configure 버튼을 클릭합니다. 프로젝트를 처음 설정하는 것이기 때문에 CMake는 어떤 컴파일러를 사용할 것인지 물을겁니다. 첫 단계에 설치한 컴파일러에 따라 적절히 선택합니다. 64Bit 윈도우를 사용하고 있다면 64Bit를 선택할 수 있습니다. 알지 못한다면 32Bit를 선택합니다.
* 붉은 줄이 나타나지 않을 때까지 Configure를 클릭합니다. 그리고 Generate를 클릭합니다. Visual Studio 프로젝트가 이제 생성됩니다. 이제 CMake는 잊어버려도 좋습니다.
* "C:\Users\XYZ\Projects\OpenGLTutorials-build-Visual2010-32bits\" 폴더를 엽니다. "Tutorials.sln" 파일을 볼수 있는데 이것을 Visual Studio로 여십시오.
![](http://www.opengl-tutorial.org/assets/images/tuto-1-window/directories.png)

*빌드* 메뉴에서 *모두 빌드하기*를 클릭합니다. 모든 튜토리얼과 의존성들이 컴파일 될 것입니다. 각 실행파일들 또한 "C:\Users\XYZ\Projects\OpenGLTutorials\"로 복사될 것입니다. 에러가 없기를 바랍니다.
![](http://www.opengl-tutorial.org/assets/images/tuto-1-window/visual_2010.png)

* "C:\Users\XYZ\Projects\OpenGLTutorials\playground"를 열고 "playground.exe"를 실행합니다. 검은 창이 나타나야 합니다.
![](http://www.opengl-tutorial.org/assets/images/tuto-1-window/empty_window.png)

Visual Studio 내에서 튜토리얼을 실행시킬 수도 있습니다. "playground"를 오른쪽 클릭한 후 "시작 프로젝트로 설정"을 선택합니다. 이제 'F5'를 눌러 코드를 디버깅할 수 있습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-1-window/StartupProject.png)

## 리눅스에서 빌드하기

많은 리눅스 배포판이 존재하기 때문에 모든 플랫폼에 대해 나열하는 것은 불가능합니다. 경우에 따라 수정이 필요할 것이며 주저없이 배포판의 문서를 읽도록 하십시오. 

* 최신 드라이버를 설치합니다. 소스 비공개 바이너리 드라이버를 설치하기를 강력히 권장합니다. GNU나 그 비슷한 것은 아니지만 동작합니다. 만약 배포판이 자동 설치를 지원하지 않을 경우 [Ubuntu's guide](http://help.ubuntu.com/community/BinaryDriverHowto)를 시도합니다.
* 필요한 모든 컴파일러와 도구들, 라이브러리들을 설치합니다. 필요한 리스트는 *"cmake make g++ libx11-dev libxi-dev libgl1-mesa-dev libglu1-mesa-dev libxrandr-dev libxext-dev libxi-dev"*입니다. `sudo apt-get install *****` 또는 `su && yum install ******`를 이용하세요.
* [소스코드](http://www.opengl-tutorial.org/download/)를 다운로드하고 압축을 해제하세요. 예를 들면 "~/Projects/OpenGLTutorials/"에 해제합니다.
* [Download the source code](http://www.opengl-tutorial.org/download/) and unzip it, for instance in ~/Projects/OpenGLTutorials/
* "cd in ~/Projects/OpenGLTutorials/" 후 다음 명령을 입력하세요.

 * mkdir build
 * cd build
 * cmake ..


* makefile이 build/ 디렉토리 안에 생성됩니다.
* "~/Projects/OpenGLTutorials/playground"를 열고 "playground.exe"를 실행합니다. 검은 창이 나타나야 합니다.
* 
[Qt Creator](http://qt-project.org/)와 같은 IDE를 꼭 사용하도록 합니다. 특히 Qt Creator는 CMake에 대한 지원이 내장되어 있기 때문에 디버깅시 훨씬 좋은 경험을 제공할 것입니다. Qt Creator에 대한 설명을 드립니다.

* QtCreator에서 "File->Tools->Options->Compile&Execute->CMake"로 갑니다.
* CMake의 경로를 설정합니다. 아마도 "/usr/bin/make"일 겁니다.
* "File->OpenProject"에서 "tutorials/CMakeLists.txt"를 선택합니다.
* 빌드 디렉토리를 선택합니다. 튜토리얼 폴더 바깥이 좋을 것입니다.
* 선택적으로 "-DCMAKE_BUILD_TYPE=Debug"를 매개변수에 설정합니다. 검증합니다.
* 아래쪽의 망치를 클릭합니다. 튜토리얼이 이제 tutorials/ 폴더로부터 실행됩니다.
* QtCreator에서 튜토리얼을 실행하기 위해서는 "Projects->Execution parameters->Working Directory"를 클릭하고 셰이더와 텍스쳐, 그리고 모델이 위치한 디렉토리를 선택합니다.예를 들어 튜토리얼 2의 경우 "~/opengl-tutorial/tutorial02_red_triangle/".


## 맥에서 빌드하기

맥에서의 절차는 윈도우와 아주 비슷합니다(Makefile 역시 지원되지만 여기서 설명하지 않겠습니다).

* 맥 앱 스토어에서 Xcode를 설치합니다.
* Install XCode from the Mac App Store
* [CMake](http://www.cmake.org/cmake/resources/software.html)를 다운로드하여 .dmg를 설치합니다. 커맨드 라인 도구를 설치할 필요는 없습니다.
* [소스코드](http://www.opengl-tutorial.org/download/)를 다운로드하고 압축을 해제하세요. 예를 들면 ~/Projects/OpenGLTutorials/"에 해제합니다.
* CMake를 실행합니다. 첫번째 행에 압축을 해제한 폴더를 지정합니다. 자신이 없다면 CMakeLists.txt 파일을 포함하고 있는 폴더를 선택하세요. 두 번째 행에는 컴파일러에 필요한 모든 것들이 저장될 폴더를 지정합니다. 예를 들면 "~/Projects/OpenGLTutorials_bin_XCode/". 어디든지 될 수 있으며 꼭 같은 폴더일 필요는 없습니다.
* Configure 버튼을 클릭합니다. 프로젝트를 처음 설정하는 것이기 때문에 CMake는 어떤 컴파일러를 사용할 것인지 물을겁니다. Xcode를 선택합니다.
* 붉은 줄이 나타나지 않을 때까지 Configure를 클릭합니다. 그리고 Generate를 클릭합니다. XCode 프로젝트가 이제 생성됩니다. 이제 CMake는 잊어버려도 좋습니다.
* "~/Projects/OpenGLTutorials_bin_XCode/" 폴더를 엽니다. "Tutorials.xcodeproj" 파일을 볼수 있는데 이 파일을 여십시오.
* Xcode의 Scheme 패널에서 원하는 튜토리얼을 선택한 후 Run 버튼을 눌러 컴파일과 실행을 합니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-1-window/Xcode-projectselection.png)


## Code::Blocks 주의

2개의 버그 때문에 (하나는 C::B, 다른 하나는 CMake) 때문에 "Project->Build Options->Make commands" 에서 명령줄을 다음과 같이 편집해야 합니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-1-window/CodeBlocksFix.png)


작업 디렉토리도 직접 설정해야 합니다. "Project->Properties -> Build targets -> tutorial N -> execution working dir ( src_dir/tutorial_N/ 입니다)".

# 튜토리얼 실행

올바른 디렉토리에서 튜토리얼을 직접 실행합니다. 실행파일을 간단히 더블 클릭합니다. 만약 명령줄을 좋아한다면 적절한 디렉토리로 이동(cd)합니다.

IDE에서 튜토리얼을 실행시키고 싶다면 적절한 작업 디렉토리를 설정하기 위한 위의 지시사항을 반드시 읽으십시오.

# 튜토리얼을 따라하는 방법

각 튜토리얼은 소스코드와 데이터가 함께 하며 "tutorialXX/"에서 찾을 수 있습니다. 하지만 이 프로젝트를 절대 수정하지 않기를 바랍니다. 이것들은 참고용입니다. 대신 "playground/playground.cpp"를 열고 이 파일을 수정합니다. 무엇이든 원하는 걸 해 봅니다. 어떻게 해야 할지 모르겠을 땐 아무 튜토리얼에 있는 것들을 복사/붙여넣기 하십시오. 그러면 모든 것들이 정상으로 돌아갈 것입니다. 

모든 튜토리얼마다 코드 조각들을 제공할 것입니다. 읽는 동안 그것들을 playground에 복사/붙여넣기하는 것을 주저하지 마세요. 실험해보는 것이 좋습니다. 완성된 코드를 단순히 읽기만 하는 것은 피하세요. 그렇게 해서는 많은 것을 배우지는 못합니다. 단순히 복사/붙여넣기만 하더라도 산더미 같은 문제들을 경험할 수 있을 것입니다.

# 창 열기

드디어 OpenGL 코드입니다!

사실 바로는 아닙니다. 모든 튜토리얼은 사실 특별하지 않은 무언가를 하길 위해 "저수준"의 방법들을 필요로 합니다. 이 부분은 사실 정말 지루하고 쓸모없습니다. 그러므로 우리는 이러한 작업들을 대신해 줄 외부 라이브러리인 GLFW를 사용할 것입니다. 이러한 작업을 직접 해보고 싶다면 윈도우에서는 Win32 API를, 리눅스에서는 X11 API, 맥에서는 Cocoa API를 사용할 수 있습니다. 또는 다른 고수준 라이브러리인 SFML, FreeGLUT, SDL, ... 등등을 사용할 수도 있습니다. [Links](http://www.opengl-tutorial.org/miscellaneous/useful-tools-links/) 페이지를 참조합니다.
 
자 이제 출발해 보겠습니다. 먼저 의존성들을 다루겠습니다. 콘솔에 메시지를 표시하기 위해 기본적으로 필요한 것들이 있습니다.

``` cpp
// 표준 헤더 파일들을 포함합니다.
#include <stdio.h>
#include <stdlib.h>
```

먼저 GLEW입니다. GLEW는 사실 정말 엄청난 마법입니다. 그렇지만 차후에 설명하도록 하겠습니다.

``` cpp
// GLEW를 포함합니다. 항상 gl.h 와 glfw.h 앞에 포함합니다. 왜냐면 엄청난 마법이기 때문이죠.
#include <GL/glew.h>
```

우리는 GLFW가 창과 키보드를 다루도록 할 것이기 때문에 역시 포함시킵니다.

``` cpp
// GLFW를 포함합니다.
#include <GL/glfw3.h>
```

이것은 3D 수학 라이브러리인데 지금 당장 필요하지는 않습니다. 하지만 곧 매우 유용하다는 것을 알게 될 것입니다. GLM에 특별한 마법은 없습니다. 원한다면 직접 작성할 수 있습니다만 GLM을 사용하면 매우 편리합니다. "using namespace"는 "glm::vec3" 대신 "vec3"로 작성할 수 있게 해 줍니다.

``` cpp
// GLM을 포함합니다.
#include <glm/glm.hpp>
using namespace glm;
```

이 모든 #include를 playground.cpp에 복사/붙여 넣은 후에 컴파일러는 main() 함수가 없다고 불평을 할 것입니다. 그러므로 한번 만들어 보겠습니다.
``` cpp
int main(){
```

가장 먼저 GLFW를 초기화합니다.

``` cpp
// GLFW를 초기화합니다.
if( !glfwInit() )
{
    fprintf( stderr, "Failed to initialize GLFW\n" );
    return -1;
}
```

이제 첫 번째 OpenGL 창을 만들 수 있습니다!

``` cpp
glfwWindowHint(GLFW_SAMPLES, 4); // 4x antialiasing
glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3); // OpenGL 3.3을 원합니다
glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE); // MacOS에서는 하지 않는 것이 좋습니다.
glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE); //We don't want the old OpenGL 

// 창을 열고 OpenGL 컨텍스트를 생성합니다.
GLFWwindow* window; // (In the accompanying source code, this variable is global)
window = glfwCreateWindow( 1024, 768, "Tutorial 01", NULL, NULL);
if( window == NULL ){
    fprintf( stderr, "Failed to open GLFW window. If you have an Intel GPU, they are not 3.3 compatible. Try the 2.1 version of the tutorials.\n" );
    glfwTerminate();
    return -1;
}
glfwMakeContextCurrent(window); // Initialize GLEW
glewExperimental=true; // Needed in core profile
if (glewInit() != GLEW_OK) {
    fprintf(stderr, "Failed to initialize GLEW\n");
    return -1;
}
```

빌드하고 실행합니다. 창이 나타났다가 바로 사라질 것입니다. 당연합니다. 사용자가 ESC 키를 누를 때까지 기다리도록 하겠습니다.

``` cpp
// 아래에서 ESC 키를 누르는 것을 감지할 수 있도록 합니다.
glfwSetInputMode(window, GLFW_STICKY_KEYS, GL_TRUE);

do{
    // 아직 아무 것도 그리지 않습니다. 튜토리얼 2에서 만나보죠.

    // 버퍼를 스왑.
    glfwSwapBuffers(window);
    glfwPollEvents();

} // ESC 키가 눌렸거나 창이 닫혔는지를 검사합니다.
while( glfwGetKey(window, GLFW_KEY_ESCAPE ) != GLFW_PRESS &&
glfwWindowShouldClose(window) == 0 );
```

이게 첫 번째 튜토리얼의 끝입니다! 튜토리얼에서는 실제로 삼각형을 그리는 방법을 배워 보겠습니다.



