---
layout: page
status: publish
published: true
title: '튜토리얼 7 : 모델 불러오기'
date: '2011-05-08 17:48:12 +0200'
date_gmt: '2011-05-08 17:48:12 +0200'
categories: [tuto]
order: 70
tags: []
---

지금가지 우리는 정육면체에 대한 정보를 소스코드에 직접 코딩했습니다. 이 방법이 성가시고 수정하기 쉽지 않다는 것에 동의하실 겁니다.

이번 튜토리얼에서는 파일로부터 3D 메쉬를 로딩하는 방법을 배우겠습니다. 텍스처 로더를 만들었던 것처럼 아주 작고 제한적인 로더를 작성해보겠습니다. 그리고나서 이 작업을 더 잘할 수 있는 실제 라이브러리들을 몇 개 알려드리겠습니다.

튜토리얼을 가능한 단순하게 유지시키기 위해 매우 간단하고 널리 사용되는 OBJ 포맷을 사용하겠습니다. 그리고 정점 당 한 개의 UV 좌표와 한 개의 법선만 존재하는 OBJ 파일을 다루겠습니다(지금 당장은 법선이 무엇인지 몰라도 됩니다).

# OBJ 불러오기

common/objloader.cpp와 common/objloader.hpp에 위치한 이 함수의 원형은 다음과 같습니다.

``` cpp
bool loadOBJ(
    const char * path,
    std::vector < glm::vec3 > & out_vertices,
    std::vector < glm::vec2 > & out_uvs,
    std::vector < glm::vec3 > & out_normals
)
```

loadOBJ는 파일 "path"를 읽어들여 그 데이터를 out_vertices/out_uvs/out_normals에 채워 줍니다. 그리고 뭔가 잘못되었다면 false를 반환합니다. std::vector는 C++에서 크기를 원하는 대로 조절 수 있는 glm::vec3에 대한 배열을 선언하는 방법입니다. 수학적인 vector와는 아무런 상관이 없습니다. 그냥 배열일 뿐입니다. 진짜로요. 끝으로 &는 이 함수가 std::vector들을 수정할 수 있다는 것을 의미합니다.

## OBJ 파일 예제

OBJ 파일은 대부분 다음와 같이 생겼습니다.

```
# Blender3D v249 OBJ File: untitled.blend
# www.blender3d.org
mtllib cube.mtl
v 1.000000 -1.000000 -1.000000
v 1.000000 -1.000000 1.000000
v -1.000000 -1.000000 1.000000
v -1.000000 -1.000000 -1.000000
v 1.000000 1.000000 -1.000000
v 0.999999 1.000000 1.000001
v -1.000000 1.000000 1.000000
v -1.000000 1.000000 -1.000000
vt 0.748573 0.750412
vt 0.749279 0.501284
vt 0.999110 0.501077
vt 0.999455 0.750380
vt 0.250471 0.500702
vt 0.249682 0.749677
vt 0.001085 0.750380
vt 0.001517 0.499994
vt 0.499422 0.500239
vt 0.500149 0.750166
vt 0.748355 0.998230
vt 0.500193 0.998728
vt 0.498993 0.250415
vt 0.748953 0.250920
vn 0.000000 0.000000 -1.000000
vn -1.000000 -0.000000 -0.000000
vn -0.000000 -0.000000 1.000000
vn -0.000001 0.000000 1.000000
vn 1.000000 -0.000000 0.000000
vn 1.000000 0.000000 0.000001
vn 0.000000 1.000000 -0.000000
vn -0.000000 -1.000000 0.000000
usemtl Material_ray.png
s off
f 5/1/1 1/2/1 4/3/1
f 5/1/1 4/3/1 8/4/1
f 3/5/2 7/6/2 8/7/2
f 3/5/2 8/7/2 4/8/2
f 2/9/3 6/10/3 3/5/3
f 6/10/4 7/6/4 3/5/4
f 1/2/5 5/1/5 2/9/5
f 5/1/6 6/10/6 2/9/6
f 5/1/7 8/11/7 6/10/7
f 8/11/7 7/12/7 6/10/7
f 1/2/8 2/9/8 3/13/8
f 1/2/8 3/13/8 4/14/8
```

So :
* #은 주석입니다. C++에서 // 와 같습니다.
* usemtl과 mtllib는 모델의 모습을 묘사합니다. 이 튜토리얼에서는 사용하지 않습니다.
* v는 정점입니다.
* vt는 정점의 텍스처 좌표입니다.
* vn은 정점의 법선입니다.
* f는 면입니다.

v.와 vt와 vn은 이해하기 쉽습니다. f는 좀 어려운데요 . f가 8/11/7 7/12/7 6/10/7 이라면,

* 8/11/7은 삼각형의 첫 번째 정점을 나타냅니다.
* 7/12/7은 삼각형의 두 번째 정점을 나타냅니다.
* 6/10/7은 삼각형의 세 번째 정점을 나타냅니다(하.).
* 첫 번째 정점의 경우 8은 어떠한 정점을 사용할지를 나타냅니다. 이 경우 -1.000000 1.000000 -1.000000(인덱스는 1부터 시작합니다. C++처럼 0이 아닙니다)이 사용됩니다.
* 11은 어떠한 텍스처 좌표를 사 나타냅니다. 이 경우 0.748355 0.998230입니다.
* 7은 사용할 법선을 나타냅니다. 이 경우 0.000000 1.000000 -0.000000입니다.

이 숫자들을 인덱스라고 부릅니다. 이는 몇몇 정점들이 같은 위치를 공유할 경우 파일에 "v"를 한번만 써 놓고, 여러번 사용할 수 있기 때문에 유용합니다. 그리고 메모리를 절약할 수 있습니다.

좋지 않은 소식은 OpenGL은 한 인덱스는 위치, 또 다른 인덱스는 텍스처, 또 다른 인덱스는 법선으로 지정하는 것이 불가능합니다. 그래서 이 튜토리얼에서는 인덱스를 사용하지 않는 메시를 만들고, 나중에 튜토리얼 9에서 이를 피하기 위해 인덱스를 다루는 방법을 따로 설명하겠습니다.

## 블렌더에서 OBJ 파일 생성하기

우리의 장남감 로더는 몹시 제한적이기 때문에 파일을 내보낼 때 올바른 옵션으로 설정되도록 주의를 기울여야 합니다. 블렌더에서 아래와 같이 설정합니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-7-model-loading/Blender.png)


## 파일 읽기

좋습니다. 실제 코드로 가보죠. .obj의 컨텐츠를 담을 임시 변수들이 필요합니다.

``` cpp
std::vector< unsigned int > vertexIndices, uvIndices, normalIndices;
std::vector< glm::vec3 > temp_vertices;
std::vector< glm::vec2 > temp_uvs;
std::vector< glm::vec3 > temp_normals;
```

"튜토리얼 5: 정육면체에 텍스트 입히기" 에서 파일을 여는 방법을 배웠습니다.

``` cpp
FILE * file = fopen(path, "r");
if( file == NULL ){
    printf("Impossible to open the file !\n");
    return false;
}
```

Let's read this file until the end :
파일을 끝까지 읽어드립니다.

``` cpp
while( 1 ){

    char lineHeader[128];
    // read the first word of the line
    int res = fscanf(file, "%s", lineHeader);
    if (res == EOF)
        break; // EOF = End Of File. Quit the loop.

    // else : parse lineHeader
```

(각 행의 첫 번째 단어가 128이하라고 가정한 것에 유의하세요. 매우 멍청한 가정이지만 이 장난감 파서에서는 문제 없습니다.)

정점부터 다루어 보죠.

``` cpp
if ( strcmp( lineHeader, "v" ) == 0 ){
    glm::vec3 vertex;
    fscanf(file, "%f %f %f\n", &vertex.x, &vertex.y, &vertex.z );
    temp_vertices.push_back(vertex);
```
즉, 행의 첫 번째 단어가 "v"라면 나머지는 3개의 실수입니다. 그러므로 glm::vec3를 이 값들로 만들고 벡터에 집어 넣습니다.

``` cpp
}else if ( strcmp( lineHeader, "vt" ) == 0 ){
    glm::vec2 uv;
    fscanf(file, "%f %f\n", &uv.x, &uv.y );
    temp_uvs.push_back(uv);
```

"v"가 아니라 "vt"라면 나머지는 2개의 실수입니다. 그러므로 glm::vec2를 만들어 벡터에 집어 넣습니다.

법선도 동일합니다.

``` cpp
}else if ( strcmp( lineHeader, "vn" ) == 0 ){
    glm::vec3 normal;
    fscanf(file, "%f %f %f\n", &normal.x, &normal.y, &normal.z );
    temp_normals.push_back(normal);
```

이제 "f"는 좀 더 복잡합니다.

``` cpp
}else if ( strcmp( lineHeader, "f" ) == 0 ){
    std::string vertex1, vertex2, vertex3;
    unsigned int vertexIndex[3], uvIndex[3], normalIndex[3];
    int matches = fscanf(file, "%d/%d/%d %d/%d/%d %d/%d/%d\n", &vertexIndex[0], &uvIndex[0], &normalIndex[0], &vertexIndex[1], &uvIndex[1], &normalIndex[1], &vertexIndex[2], &uvIndex[2], &normalIndex[2] );
    if (matches != 9){
        printf("File can't be read by our simple parser : ( Try exporting with other options\n");
        return false;
    }
    vertexIndices.push_back(vertexIndex[0]);
    vertexIndices.push_back(vertexIndex[1]);
    vertexIndices.push_back(vertexIndex[2]);
    uvIndices    .push_back(uvIndex[0]);
    uvIndices    .push_back(uvIndex[1]);
    uvIndices    .push_back(uvIndex[2]);
    normalIndices.push_back(normalIndex[0]);
    normalIndices.push_back(normalIndex[1]);
    normalIndices.push_back(normalIndex[2]);
```

사실 좀 더 많은 데이터를 읽어들인다는 점을 제외하면 앞의 코드들과 비슷합니다.

## 데이터 처리

지금까지 한 작업은 단순히 데이터의 "형태"를 변경한 것입니다. 문자열에서 std::vector들로 변환했습니다. 이 것만으로 충분하지 않습니다. OpenGL이 원하는 형식으로 만들어야 합니다. 즉 인덱스들을 제거하고 평범한 glm::vec3들로 만들어야 합니다. 이 작업을 인덱싱이라고 부릅니다.

각 삼각형("f"로 시작하는 행)들의 각 정점(v/vt/vn)을 훑도록 하겠습니다.

``` cpp
    // For each vertex of each triangle
    for( unsigned int i=0; i<vertexIndices.size(); i++ ){
```

정점 위치에 대한 인덱스는 vertexIndices[i]입니다.

``` cpp
unsigned int vertexIndex = vertexIndices[i];
```

위치 값은 temp_vertices[ vertexIndex-1 ]입니다(C++의 인덱스는 0부터 시작핮지만 OBJ의 인덱스는 1부터 시작하므로 -1을 해줍니다.).

``` cpp
glm::vec3 vertex = temp_vertices[ vertexIndex-1 ];
```

새로운 정점의 위치를 저장합니다.

``` cpp
out_vertices.push_back(vertex);
```

UV들과 법선들에도 동일하게 적용하면 끝입니다!

# 로딩한 데이터를 사용하기

여기까지 진행하면 변경할 부분은 거의 없습니다. 지금까지 해오던 static const GLfloat g_vertex_buffer_data[] = {...} 선언 대신 std::vector vertices를 선언합니다(UV와 법선도 마찬가지입니다). 그리고 적절한 매개 변수로 loadOBJ를 호출합니다.

``` cpp
// Read our .obj file
std::vector< glm::vec3 > vertices;
std::vector< glm::vec2 > uvs;
std::vector< glm::vec3 > normals; // Won't be used at the moment.
bool res = loadOBJ("cube.obj", vertices, uvs, normals);
```

그리고 OpenGL에 배열 대신 벡터를 전달합니다.

``` cpp
glBufferData(GL_ARRAY_BUFFER, vertices.size() * sizeof(glm::vec3), &vertices[0], GL_STATIC_DRAW);
```

끝입니다!

# 결과

텍스처가 빈약해서 죄송합니다. 제가 좋은 아티스트는 아니라서요. :( 기증 해주시면 감사합니다!

![](http://www.opengl-tutorial.org/assets/images/tuto-7-model-loading/ModelLoading.png)


# 다른 포맷/로더

이 작은 로더는 시작하기에는 충분할지 모르지만 실제로는 사용하기는 좋지 않습니다. 사용 할만한 도구들을 [Useful Links & Tools](http://www.opengl-tutorial.org/miscellaneous/useful-tools-links/) 페이지를 통해 살펴보세요. 하지만 *실제로* 이것들을 사용하기 전에 튜토리얼 9를 학습할 때까지 기다리는 것을 추천드립니다.
