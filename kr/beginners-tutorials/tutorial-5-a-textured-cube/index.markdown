---
layout: page
status: publish
published: true
title: 'Tutorial 5 : A Textured Cube'
date: '2011-04-26 07:55:58 +0200'
date_gmt: '2011-04-26 07:55:58 +0200'
categories: [tuto]
order: 50
tags: []
---

이번 튜토리얼에서는 다음과 같은 것들 배울 것입니다.

* UV 좌표에 대하여.
* 직접 텍스처를 로딩하는 방법
* 이를 OpenGL에서 사용하는 방법
* 필터과 밉맵이 무엇이며 그리고 어떻게 사용하는지
* 알파 채널이 무엇인지

# UV 좌표에 대하여

메쉬에 텍스처를 입히려면 각 삼각형에 이미지의 어느 부분이 사용되어야 하는지를 OpenGL에 알려주어야 합니다. 이는 UV 좌표를 이용하여 이루어집니다.

각 정점은 자신의 위치에 U와 V 이렇게 한쌍의 실수(float)값을 가질 수 있습니다. 이 좌표는 다음과 같은 방식으로 텍스처에 접근하는데에 사용됩니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-5-textured-cube/UVintro.png)


텍스처가 삼각형위로 어떻게 변형되는지를 잘 보세요.

# 직접 .BMP 이미지를 불러오기

BMP 파일 포맷을 아는 것은 별로 중요하지 않습니다. BMP를 로딩할 수 있는 수 많은 라이브러리들이 있기 때문이죠. 하지만 매우 간단하고 또 내부에서 어떠한 일이 일어나는지를 이해하는데 도움이 될 수 있습니다. 그러므로 처음부터 BMP 파일 로더를 작성해보도록 하겠습니다. 어떻게 동작하는지 파악하는데만 사용하시고 <span style="text-decoration: underline;">다시는 사용하지 마세요</span>.

여기 로딩 함수의 선언이 있습니다.

``` cpp
GLuint loadBMP_custom(const char * imagepath);
```

다음과 같이 사용하죠.

``` cpp
GLuint image = loadBMP_custom("./my_texture.bmp");
```

어떻게 BMP 파일을 읽어들이는지 살펴보겠습니다.

먼저 약간의 데이터가 필요합니다. 이러한 변수들은 파일을 읽을 때 설정 될 것 입니다.

``` cpp
// Data read from the header of the BMP file
unsigned char header[54]; // Each BMP file begins by a 54-bytes header
unsigned int dataPos;     // Position in the file where the actual data begins
unsigned int width, height;
unsigned int imageSize;   // = width*height*3
// Actual RGB data
unsigned char * data;
```

이제 실제로 파일을 열어야겠죠.

``` cpp
// Open the file
FILE * file = fopen(imagepath,"rb");
if (!file){printf("Image could not be opened\n"); return 0;}
```

파일의 첫 번째 54바이트는 헤더입니다. 헤더는 "이 파일이 정말 BMP 파일인지?", 이미지의 크기는 얼마인지, 그리고 픽셀당 비트 수 등을 담고 있습니다. 헤더를 읽어 보겠습니다.

``` cpp
if ( fread(header, 1, 54, file)!=54 ){ // If not 54 bytes read : problem
    printf("Not a correct BMP file\n");
    return false;
}
```

헤더는 항상 BM으로 시작합니다. 사실 .BMP 파일을 헥사 에디터로 열어보면 다음과 같을겁니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-5-textured-cube/hexbmp.png)

그러므로 처음 두 바이트가 'B'와 'M'인지를 확인합니다.

``` cpp
if ( header[0]!='B' || header[1]!='M' ){
    printf("Not a correct BMP file\n");
    return 0;
}
```

이제 파일로부터 이미지의 크기와 실제 데이터의 위치 등을 읽을 수 있습니다.

``` cpp
// Read ints from the byte array
dataPos    = *(int*)&(header[0x0A]);
imageSize  = *(int*)&(header[0x22]);
width      = *(int*)&(header[0x12]);
height     = *(int*)&(header[0x16]);
```

누락된 정보가 있다면 직접 만들어야 합니다.

``` cpp
// Some BMP files are misformatted, guess missing information
if (imageSize==0)    imageSize=width*height*3; // 3 : one byte for each Red, Green and Blue component
if (dataPos==0)      dataPos=54; // The BMP header is done that way
```

이제 이미지의 크기를 알기 때문에 이미지를 저장할 버퍼를 할당하고 읽어들입니다.

``` cpp
// Create a buffer
data = new unsigned char [imageSize];

// Read the actual data from the file into the buffer
fread(data,1,imageSize,file);

//Everything is in memory now, the file can be closed
fclose(file);
```

이제 실제 OpenGL 부분입니다. 텍스처를 만드는 것은 정점 버퍼를 생성하는 것과 몹시 비슷합니다. 텍스처를 만들고, 바인딩하고, 채우고, 설정합니다.

glTexImage2D에서 GL_RGB는 3-요소 색상을 사용한다는 것을 가리키며, GL_BGR은 RAM에 저장된 방법을 보여줍니다. 사실 BMP는 Red->Green->Blue 순서로 저장하지 않고 Blue->Green->Read 순서로 저장합니다. 그러므로 이를 OpenGL에게 알려줘야합니다.

``` cpp
// Create one OpenGL texture
GLuint textureID;
glGenTextures(1, &textureID);

// "Bind" the newly created texture : all future texture functions will modify this texture
glBindTexture(GL_TEXTURE_2D, textureID);

// Give the image to OpenGL
glTexImage2D(GL_TEXTURE_2D, 0,GL_RGB, width, height, 0, GL_BGR, GL_UNSIGNED_BYTE, data);

glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
```

마지막 두 줄은 나중에 설명하겠습니다. 이제 C++ 쪽에서 텍스처를 로딩하기 위해 이 함수를 사용할 수 있습니다.

``` cpp
GLuint Texture = loadBMP_custom("uvtemplate.bmp");
```

> 매우 중요한 부분 :** 2의 제곱인 텍스처를 사용해야 합니다!**
> 
> * 좋음 : 128\*128, 256\*256, 1024\*1024, 2\*2...
> * 나쁨 : 127\*128, 3\*5, ...
> * 괜찮지만 이상함 : 128\*256

# OpenGL에서 텍스처 사용하기

프래그먼트 셰이더를 먼저 살펴보겠습니다. 대부분은 어렵지 않습니다.

``` glsl
#version 330 core

// Interpolated values from the vertex shaders
in vec2 UV;

// Ouput data
out vec3 color;

// Values that stay constant for the whole mesh.
uniform sampler2D myTextureSampler;

void main(){

    // Output color = color of the texture at the specified UV
    color = texture( myTextureSampler, UV ).rgb;
}
```

{: .highlightglslfs }

Three things :
세 가지:

* 프래그먼트 셰이더는 UV 좌표가 필요합니다. 당연하게도요.
* 어떠한 텍스처에 접근해야 하는지를 알기 위해 "sampler2D" 또한 필요합니다(같은 셰이더에서 여러개의 텍스처에 접근할 수 있습니다).
* 끝으로 텍스처에 접근하는 것은 (R,G,B,A)를 반환하는 texture()를 통해 수행할 수 있습니다. 곧 보게 될 겁니다.

정점 셰이더도 간단합니다. UV를 프래그먼트 셰이더로 전달합니다.

``` glsl
#version 330 core

// Input vertex data, different for all executions of this shader.
layout(location = 0) in vec3 vertexPosition_modelspace;
layout(location = 1) in vec2 vertexUV;

// Output data ; will be interpolated for each fragment.
out vec2 UV;

// Values that stay constant for the whole mesh.
uniform mat4 MVP;

void main(){

    // Output position of the vertex, in clip space : MVP * position
    gl_Position =  MVP * vec4(vertexPosition_modelspace,1);

    // UV of the vertex. No special space for this one.
    UV = vertexUV;
}
```

{: .highlightglslvs }

튜토리얼 4에서 "layout(location = 1) in vec2 vertexUV"을 기억하시나요? 자, 완전히 동일한 작업을 여기서 할건데요, (R,G,B) 버퍼를 제공하는  대신 (U,V) 버퍼를 제공할 것입니다. 

``` cpp
// Two UV coordinatesfor each vertex. They were created with Blender. You'll learn shortly how to do this yourself.
static const GLfloat g_uv_buffer_data[] = {
    0.000059f, 1.0f-0.000004f,
    0.000103f, 1.0f-0.336048f,
    0.335973f, 1.0f-0.335903f,
    1.000023f, 1.0f-0.000013f,
    0.667979f, 1.0f-0.335851f,
    0.999958f, 1.0f-0.336064f,
    0.667979f, 1.0f-0.335851f,
    0.336024f, 1.0f-0.671877f,
    0.667969f, 1.0f-0.671889f,
    1.000023f, 1.0f-0.000013f,
    0.668104f, 1.0f-0.000013f,
    0.667979f, 1.0f-0.335851f,
    0.000059f, 1.0f-0.000004f,
    0.335973f, 1.0f-0.335903f,
    0.336098f, 1.0f-0.000071f,
    0.667979f, 1.0f-0.335851f,
    0.335973f, 1.0f-0.335903f,
    0.336024f, 1.0f-0.671877f,
    1.000004f, 1.0f-0.671847f,
    0.999958f, 1.0f-0.336064f,
    0.667979f, 1.0f-0.335851f,
    0.668104f, 1.0f-0.000013f,
    0.335973f, 1.0f-0.335903f,
    0.667979f, 1.0f-0.335851f,
    0.335973f, 1.0f-0.335903f,
    0.668104f, 1.0f-0.000013f,
    0.336098f, 1.0f-0.000071f,
    0.000103f, 1.0f-0.336048f,
    0.000004f, 1.0f-0.671870f,
    0.336024f, 1.0f-0.671877f,
    0.000103f, 1.0f-0.336048f,
    0.336024f, 1.0f-0.671877f,
    0.335973f, 1.0f-0.335903f,
    0.667969f, 1.0f-0.671889f,
    1.000004f, 1.0f-0.671847f,
    0.667979f, 1.0f-0.335851f
};
```

위 UV 좌표는 다음 모델에 대응됩니다. 

![](http://www.opengl-tutorial.org/assets/images/tuto-5-textured-cube/uv_mapping_blender.png)

남은 것은 명백합니다. 버퍼를 만들고, 바인드 하고, 채운 후, 설정하고, 정점 버퍼를 그립니다. glVertexAttribPointer의 두 번째 파라미터(size)에 3 대신 2를 사용하는 것을 주의합니다.

결과입니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-5-textured-cube/nearfiltering.png)

줌인 버전이고요.

![](http://www.opengl-tutorial.org/assets/images/tuto-5-textured-cube/nearfiltering_zoom.png)

# 필터링과 밉맵 사용법

위 스크린샷에서 보듯이 텍스처 품질이 썩 좋지 않습니다. 이유는 loadBMP_custom에서 다음과 같이 작성했기 때문입니다.

``` cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
```

이는 프래그먼트 셰이더에서 texture()가 (U,V)좌표에 있는 텍셀을 취한다는 것을 의미합니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-5-textured-cube/nearest.png)

이를 개선하기 위한 여러가지 방법이 있습니다.

## 선형 필터링 (Linear filtering)

선형 필터링에서는 texture()가 주변 텍셀들도 같이 고려합니다. 그리고 각 센터와의 거리에 따라 색상이 섞이게 됩니다. 이를 통해 위에서 본 것과 같은 하드-엣지를 피할 수 있습니다.  

![](http://www.opengl-tutorial.org/assets/images/tuto-5-textured-cube/linear1.png)

훨씬 낫군요. 이것은 매우 많이 사용됩니다. 만약 매우 높은 품질을 얻고 싶다면 이방성 필터링을 사용할 수도 있습니다. 매우 느리지만요.

## 이방성 필터링 (Anisotropic filtering)

이방성 필터링은 프래그먼트를 통해 실제로 보여지는 이미지 부분의 근사치를 구합니다. 예를 들어 아래와 같은 텍스처가 약간 돌아가고 옆에서 보여지게 된다면, 이방성 필터링은 주 방향을 따라서 파란 직사각형 안에 포함된 색상을 고정된 샘플의 수(이방성 필터링 레벨)만큼 취해서 계산 할 것입니다.  

![](http://www.opengl-tutorial.org/assets/images/tuto-5-textured-cube/aniso.png)

## 밉맵 (Mipmaps)

선형과 이방성 필터링은 모두 문제가 있습니다. 텍스처가 매우 멀리서 보여지게 되면 4개의 텍셀만으로는 충분치 않습니다. 실제로 3D 모델이 아주 멀리 있어 화면상의 한 프래그먼트만 차지하게 된다면, 최종 색상을 구하기 위해 이미지의 모든 텍셀들의 평균을 계산해야 합니다. 이는 성능상의 이유로 수행할 수 없음이 분명합니다. 대신 밉맵을 소개합니다. 

![](http://upload.wikimedia.org/wikipedia/commons/5/5c/MipMap_Example_STS101.jpg)

* At initialisation time, you scale down your image by 2, successively, until you only have a 1x1 image (which effectively is the average of all the texels in the image)
* 초기화 시간에 이미지를 1/2로 줄입니다. 잇따라서 1x1 크기의 이미지(사실상 이미지내 모든 텍셀들의 평균)를 얻을 때까지 반복합니다.
* 메시를 그릴 때 텍셀이 얼마나 커야하는지를 고려하여 적절한 밉맵을 선택합니다.
* Nearest, 선형, 이방성 필터링 중에 하나를 사용해 밉맵을 샘플링합니다.
* 추가적인 품질을 위해 두 개의 밉맵을 샘플링해서 그 결과를 섞을 수도 있습니다.

다행이도 이 모든 작업을 매우 간단하게 수행할 수 있습니다. OpenGL에게 친절히 요청하면 모든 것들을 수행해 줄 것입니다.  

``` cpp
// 이미지를 확대할 때(더 큰 밉맵이 없을 때), 선형 필터링을 사용합니다.
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
// When MINifying the image, use a LINEAR blend of two mipmaps, each filtered LINEARLY too
// 이미지를 축소할 때 두개의 밉맵을 선형으로 섞으며, 각각은 선형 필터링을 사용합니다.
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
// 밉맵을 생성합니다.
glGenerateMipmap(GL_TEXTURE_2D);
```

# GLFW로 텍스처 로딩하는 방법

loadBMP_custom 함수는 우리가 직접 만들었다는 점에서 대단하긴하지만 전용 라이브러리를 사용하는 것이 더 좋습니다. GLFW2 역시 이 작업을 수행할 수 있습니다(하지만 TGA 파일만 가능하며, 지금 우리가 사용하는 GLFW3에서 이 기능은 제거되었습니다).

``` cpp
GLuint loadTGA_glfw(const char * imagepath){

    // Create one OpenGL texture
    GLuint textureID;
    glGenTextures(1, &textureID);

    // "Bind" the newly created texture : all future texture functions will modify this texture
    glBindTexture(GL_TEXTURE_2D, textureID);

    // Read the file, call glTexImage2D with the right parameters
    glfwLoadTexture2D(imagepath, 0);

    // Nice trilinear filtering.
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
    glGenerateMipmap(GL_TEXTURE_2D);

    // Return the ID of the texture we just created
    return textureID;
}
```

# 압축된 텍스처

이 시점에서 TGA 대신 JPEG를 로딩하는 방법이 궁금할 수도 있습니다.

간단히 말하면 하지 마십시오. GPU는 JPEG을 이해하지 못합니다. 그러므로 원래 이미지를 JPEG로 압축했다면, GPU가 이해할 수 있도록 압축을 해제해야 합니다. RAW 이미지를 얻겠지만 JPEG로 압축함으로 인한 이미지 품질 손실이 나타날 것입니다.

여기 더 좋은 옵션이 있습니다.

## Creating compressed textures


* AMD 툴인 [The Compressonator](http://developer.amd.com/Resources/archive/ArchivedTools/gpu/compressonator/Pages/default.aspx), 를 다운로드 받습니다.
* 2의 거듭제곱인 텍스처를 불러옵니다.
* 밉맵을 생성합니다. 그러면 런타임에 이를 수행할 필요가 없습니다.
* DXT1이나 DXT3 또는 DXT5로 압축합니다(각종 포맷들의 차이점을 알고 싶다면 [Wikipedia](http://en.wikipedia.org/wiki/S3_Texture_Compression)).

![](http://www.opengl-tutorial.org/assets/images/tuto-5-textured-cube/TheCompressonator.png)

* .DDS 파일로 추출.

이 시점에서 이미지는 GPU와 직접 호환되는 포맷으로 압축되었습니다. 셰이더에서 texture()가 호출될 때마다 실시간으로 압축이 해제 될 것입니다. 느려 보이겠지만 적은 양의 메모리를 사용하기 때문에 적은 양의 데이터만 전송하면 됩니다. 메모리 전송은 비용은 매우 큽니다. 그리고 텍스처 압축해제는 공짜입니다(이를 위한 전용 하드웨어가 있으니까요). 일반적으로 텍스처를 압축하면 성능이 20% 정도 증가합니다. 그러므로 품질은 낮추는 것으로 성능과 메모리를 절약할 수 있습니다.

## 압축된 텍스처 사용하기

이미지를 로딩하는 방법을 살펴보겠습니다. 헤더가 다르게 구성되었다는 것을 제외하고는 BMP 코드와 매우 유사합니다.

``` cpp
GLuint loadDDS(const char * imagepath){

    unsigned char header[124];

    FILE *fp;

    /* try to open the file */
    fp = fopen(imagepath, "rb");
    if (fp == NULL)
        return 0;

    /* verify the type of file */
    char filecode[4];
    fread(filecode, 1, 4, fp);
    if (strncmp(filecode, "DDS ", 4) != 0) {
        fclose(fp);
        return 0;
    }

    /* get the surface desc */
    fread(&header, 124, 1, fp); 

    unsigned int height      = *(unsigned int*)&(header[8 ]);
    unsigned int width         = *(unsigned int*)&(header[12]);
    unsigned int linearSize     = *(unsigned int*)&(header[16]);
    unsigned int mipMapCount = *(unsigned int*)&(header[24]);
    unsigned int fourCC      = *(unsigned int*)&(header[80]);
```

헤더 이후에는 실제 데이터인 모든 밉맵 레벨들이 연속적으로 위치합니다. 모두 한번에 읽을 수 있습니다. 
 

``` cpp
    unsigned char * buffer;
    unsigned int bufsize;
    /* how big is it going to be including all mipmaps? */
    bufsize = mipMapCount > 1 ? linearSize * 2 : linearSize;
    buffer = (unsigned char*)malloc(bufsize * sizeof(unsigned char));
    fread(buffer, 1, bufsize, fp);
    /* close the file pointer */
    fclose(fp);
```

세 가지 포맷 중에 하나를 다룹니다. DXT1, DXT3, DXT5. "fourCC" 플래그를 OpenGL이 이해할 수 있는 값으로 변경 할 필요가 있습니다.  

``` cpp
    unsigned int components  = (fourCC == FOURCC_DXT1) ? 3 : 4;
    unsigned int format;
    switch(fourCC)
    {
    case FOURCC_DXT1:
        format = GL_COMPRESSED_RGBA_S3TC_DXT1_EXT;
        break;
    case FOURCC_DXT3:
        format = GL_COMPRESSED_RGBA_S3TC_DXT3_EXT;
        break;
    case FOURCC_DXT5:
        format = GL_COMPRESSED_RGBA_S3TC_DXT5_EXT;
        break;
    default:
        free(buffer);
        return 0;
    }
```

지금까지 해온 것 처럼 텍스처를 생성합니다.

``` cpp
    // Create one OpenGL texture
    GLuint textureID;
    glGenTextures(1, &textureID);

    // "Bind" the newly created texture : all future texture functions will modify this texture
    glBindTexture(GL_TEXTURE_2D, textureID);
```

이제 밉맵을 잇따라서 채웁니다.

``` cpp
    unsigned int blockSize = (format == GL_COMPRESSED_RGBA_S3TC_DXT1_EXT) ? 8 : 16;
    unsigned int offset = 0;

    /* load the mipmaps */
    for (unsigned int level = 0; level < mipMapCount && (width || height); ++level)
    {
        unsigned int size = ((width+3)/4)*((height+3)/4)*blockSize;
        glCompressedTexImage2D(GL_TEXTURE_2D, level, format, width, height, 
            0, size, buffer + offset);

        offset += size;
        width  /= 2;
        height /= 2;
    }
    free(buffer); 

    return textureID;
```

## UV 뒤집기 (Inversing the UVs)

DXT 압축은 V 텍스처 좌표가 뒤집어져잇는 DirectX 세계에서 왔습니다. 그러므로 텍스처를 압축하면 정확한 텍셀을 얻기 위해서 (coord.u, 1.0-coord.v)를 사용해야 합니다. 이 작업은 원하는 곳 어디서나 할 수 있습니다. 스크립트나 로더, 셰이더...


# 결론

이제 OpenGL에서 텍스처를 생성하고 로드하고 사용하는 방법을 배웠습니다.

일반적으로는 압축된 텍스처만을 사용합니다. 더 작고, 거의 즉시 로드할 수 있으며, 사용하기 빠르기 때문이죠. 단점은 이미지를 Compressonator(또는 유사한 툴)을 통해 압축해야 하는 것이죠.

# 연습

* 소스코드에 DDS 로더는 구현되었지만 텍스처 좌표 수정은 없습니다. 정육면체를 올바르게 출력하기 위해 적합한 위치의 코드를 수정하세요.
* 다양한 DDS 포맷들을 사용해 보세요. 다른 결과가 나오나요? 압축률은 어떤가요?
* Compressonator에서 밉맵을 생성하지 말아 보세요. 결과가 어떤가요? 이를 수정하기 위한 세가지 방법을 제시하세요.


# 참조


* [Using texture compression in OpenGL](http://www.oldunreal.com/editing/s3tc/ARB_texture_compression.pdf) , S&eacute;bastien Domine, NVIDIA

