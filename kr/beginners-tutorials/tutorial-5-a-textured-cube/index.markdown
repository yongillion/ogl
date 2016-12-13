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

![]({{site.baseurl}}/assets/images/tuto-5-textured-cube/UVintro.png)


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

![]({{site.baseurl}}/assets/images/tuto-5-textured-cube/hexbmp.png)

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

We'll have a look at the fragment shader first. Most of it is straightforward :

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

* The fragment shader needs UV coordinates. Seems fair.
* It also needs a "sampler2D" in order to know which texture to access (you can access several texture in the same shader)
* Finally, accessing a texture is done with texture(), which gives back a (R,G,B,A) vec4. We'll see about the A shortly.

The vertex shader is simple too, you just have to pass the UVs to the fragment shader :

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

Remember "layout(location = 1) in vec2 vertexUV" from Tutorial 4 ? Well, we'll have to do the exact same thing here, but instead of giving a buffer (R,G,B) triplets, we'll give a buffer of (U,V) pairs.

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

The UV coordinates above correspond to the following model :

![]({{site.baseurl}}/assets/images/tuto-5-textured-cube/uv_mapping_blender.png)

The rest is obvious. Generate the buffer, bind it, fill it, configure it, and draw the Vertex Buffer as usual. Just be careful to use 2 as the second parameter (size) of glVertexAttribPointer instead of 3.

This is the result :

![]({{site.baseurl}}/assets/images/tuto-5-textured-cube/nearfiltering.png)

and a zoomed-in version :

![]({{site.baseurl}}/assets/images/tuto-5-textured-cube/nearfiltering_zoom.png)

# What is filtering and mipmapping, and how to use them

As you can see in the screenshot above, the texture quality is not that great. This is because in loadBMP_custom, we wrote :

``` cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
```

This means that in our fragment shader, texture() takes the texel that is at the (U,V) coordinates, and continues happily.

![]({{site.baseurl}}/assets/images/tuto-5-textured-cube/nearest.png)

There are several things we can do to improve this.

## Linear filtering

With linear filtering, texture() also looks at the other texels around, and mixes the colours according to the distance to each center. This avoids the hard edges seen above.

![]({{site.baseurl}}/assets/images/tuto-5-textured-cube/linear1.png)

This is much better, and this is used a lot, but if you want very high quality you can also use anisotropic filtering, which is a bit slower.

## Anisotropic filtering

This one approximates the  part of the image that is really seen through the fragment. For instance, if the following texture is seen from the side, and a little bit rotated, anisotropic filtering will compute the colour contained in the blue rectangle by taking a fixed number of samples (the "anisotropic level") along its main direction.

![]({{site.baseurl}}/assets/images/tuto-5-textured-cube/aniso.png)

## Mipmaps

Both linear and anisotropic filtering have a problem. If the texture is seen from far away, mixing only 4 texels won't be enough. Actually, if your 3D model is so far away than it takes only 1 fragment on screen, ALL the texels of the image should be averaged to produce the final color. This is obviously not done for performance reasons. Instead, we introduce MipMaps :

![](http://upload.wikimedia.org/wikipedia/commons/5/5c/MipMap_Example_STS101.jpg)

* At initialisation time, you scale down your image by 2, successively, until you only have a 1x1 image (which effectively is the average of all the texels in the image)
* When you draw a mesh, you select which mipmap is the more appropriate to use given how big the texel should be.
* You sample this mipmap with either nearest, linear or anisotropic filtering
* For additional quality, you can also sample two mipmaps and blend the results.

Luckily, all this is very simple to do, OpenGL does everything for us provided that you ask him nicely :

``` cpp
// When MAGnifying the image (no bigger mipmap available), use LINEAR filtering
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
// When MINifying the image, use a LINEAR blend of two mipmaps, each filtered LINEARLY too
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
// Generate mipmaps, by the way.
glGenerateMipmap(GL_TEXTURE_2D);
```

# How to load texture with GLFW

Our loadBMP_custom function is great because we made it ourselves, but using a dedicated library is better. GLFW2 can do that too (but only for TGA files, and this feature has been removed in GLFW3, that we now use) :

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

# Compressed Textures

At this point, you're probably wondering how to load JPEG files instead of TGA.

Short answer : don't. GPUs can't understand JPEG. So you'll compress your original image in JPEG, and decompress it so that the GPU can understand it. You're back to raw images, but you lost image quality while compressing to JPEG.

There's a better option.

## Creating compressed textures


* Download [The Compressonator](http://developer.amd.com/Resources/archive/ArchivedTools/gpu/compressonator/Pages/default.aspx), an AMD tool
* Load a Power-Of-Two texture in it
* Generate mipmaps so that you won't have to do it on runtime
* Compress it in DXT1, DXT3 or in DXT5 (more about the differences between the various formats on [Wikipedia](http://en.wikipedia.org/wiki/S3_Texture_Compression)) :

![]({{site.baseurl}}/assets/images/tuto-5-textured-cube/TheCompressonator.png)

* Export it as a .DDS file.

At this point, your image is compressed in a format that is directly compatible with the GPU. Whenever calling texture() in a shader, it will uncompress it on-the-fly. This can seem slow, but since it takes a LOT less memory, less data needs to be transferred. But memory transfers are expensive; and texture decompression is free (there is dedicated hardware for that). Typically, using texture compression yields a 20% increase in performance. So you save on performance and memory, at the expense of reduced quality.

## Using the compressed texture

Let's see how to load the image. It's very similar to the BMP code, except that the header is organized differently :

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

After the header is the actual data : all the mipmap levels, successively. We can read them all in one batch :

 

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

Here we'll deal with 3 different formats : DXT1, DXT3 and DXT5. We need to convert the "fourCC" flag into a value that OpenGL understands.

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

Creating the texture is done as usual :

``` cpp
    // Create one OpenGL texture
    GLuint textureID;
    glGenTextures(1, &textureID);

    // "Bind" the newly created texture : all future texture functions will modify this texture
    glBindTexture(GL_TEXTURE_2D, textureID);
```

And now, we just have to fill each mipmap one after another :

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

## Inversing the UVs

DXT compression comes from the DirectX world, where the V texture coordinate is inversed compared to OpenGL. So if you use compressed textures, you'll have to use ( coord.u, 1.0-coord.v) to fetch the correct texel. You can do this whenever you want : in your export script, in your loader, in your shader...

# Conclusion

You just learnt to create, load and use textures in OpenGL.

In general, you should only use compressed textures, since they are smaller to store, almost instantaneous to load, and faster to use; the main drawback it that you have to convert your images through The Compressonator (or any similar tool)

# Exercices


* The DDS loader is implemented in the source code, but not the texture coordinate modification. Change the code at the appropriate place to display the cube correctly.
* Experiment with the various DDS formats. Do they give different result ? Different compression ratios ?
* Try not to generate mipmaps in The Compressonator. What is the result ? Give 3 different ways to fix this.


# References


* [Using texture compression in OpenGL](http://www.oldunreal.com/editing/s3tc/ARB_texture_compression.pdf) , S&eacute;bastien Domine, NVIDIA

