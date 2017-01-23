---
layout: page
status: publish
published: true
title: '튜토리얼 13 : 노말 매핑 (Normal Mapping)'
date: '2011-05-26 06:07:04 +0200'
date_gmt: '2011-05-26 06:07:04 +0200'
categories: [tuto]
order: 50
tags: []
---

13번째 튜토리얼에 오신 것을 환영합니다! 오늘은 노말 매핑(Normal Mapping)에 대해서 이야기 해 보겠습니다. 


[튜토리얼 8 : 셰이더 기본](http://www.opengl-tutorial.org/beginners-tutorials/tutorial-8-basic-shading/)에서 배웠으므로 삼각형의 법선(Normal, 노말)을 이용해 명암을 주는 방법을 알고 있습니다. 지금까지는 정점(Vertex)당 하나의 법선을 가졌습니다. 이 법선은 텍스처(Texture)에서 샘플링한 색상과 무관하게, 삼각형 내에서 부드럽게 변경되었습니다. 노말 매핑의 기본적인 아이디어는 법선도 색상과 유사한 변화를 주는 것입니다.

# 노말 텍스처 (Normal Texture)

"노말 텍스처(Normal Texture, 또는 노말 맵(Normal Map)이라고 부릅니다)"는 다음과 같이 보입니다. 

![](http://www.opengl-tutorial.org/assets/images/tuto-13-normal-mapping/normal.jpg)


각 XYZ 벡터는 RGB 텍셀(Texel)로 인코딩됩니다. 각 색상 요소는 0에서 1사이이며, 벡터 컴포넌트는 -1에서 1사이입니다. 그렇기 때문에 이 간단한 매핑은 텍셀에서 법선으로 변경됩니다.

``` c
normal = (2*color)-1 // 각 컴포넌트에
```

텍스처는 보통 푸른 톤인데 전반적으로, 법선은 "표면의 바깥쪽"으로 향하기 때문입니다. X는 텍스처의 평면에서 오른쪽이며, Y는 (텍스처의 평면에서) 위쪽입니다. 그러므로 오른손 법칙에서 Z 좌표는 텍스처의 평면에서 "바깥쪽"을 의미합니다.

이 텍스처는 난반사(Diffuse)처럼 매핑됩니다. 큰 문제가 있는데, 각 삼각형의 개별 공간(접선 공간(Tangent Space), 또는 이미지 공간(Image Space)라고 부릅니다)으로 표현된 법선들을 어떻게 (명암 공식에서 사용되는)모델 공간으로 변환하는지에 대한 것입니다.

# 접선(Tangent)과 종법선(Bitangent Normal)

여러분은 어떠한 공간(이번엔 접선 공간)을 정의하는데 사용되는 행렬에 이제 익숙합니다. 우리는 세 개의 벡터가 필요합니다. 우리는 이미 UP 벡터를 알고 있습니다. 바로 법선입니다. 법선은 블렌더로부터 제공받거나 또는 간단히 외적을 통해 삼각형으로부터 계산할 수 있습니다. 그림에서 법선은 노말 맵의 전반적인 색처럼 푸른색으로 표현되었습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-13-normal-mapping/NormalVector.png)

이제 우리는 표면에 직각인 벡터, 접선 T가 필요합니다. 하지만 그러한 벡터는 무수히 많습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-13-normal-mapping/TangentVectors.png)

어떠한 것을 선택해야 할까요? 이론적으로는 아무 것이나 사용해도 되지만, 이상한 모서리를 피하려면 이웃한 삼각형들과 모두 일관되게 사용해야 합니다. 일반적인 방법은 접선의 방향을 텍스처 좌표의 방향과 동일하게 맞추는 것입니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-13-normal-mapping/TangentVectorFromUVs.png)


우리는 세개의 기저 벡터가 필요하므로 종법선 B 역시 계산합니다(아무 접선 벡터도 괜찮지만 수학적 계산을 쉽게 하기 위해서는 모든 벡터가 직각이어야 합니다.).

![](http://www.opengl-tutorial.org/assets/images/tuto-13-normal-mapping/NTBFromUVs.png)


여기 알고리즘이 있습니다. if we note deltaPos1 and deltaPos2 two edges of our triangle, and deltaUV1 and deltaUV2 the corresponding differences in UVs, we can express our problem with the following equation

``` c
deltaPos1 = deltaUV1.x * T + deltaUV1.y * B
deltaPos2 = deltaUV2.x * T + deltaUV2.y * B
```

T와 B에 대한 이 시스템을 풀면 벡터들을 가지게 됩니다 ! (아래 코드를 보세요)

T, B, N 벡터를 가지게 되면, 접선 공간을 모델 공간으로 변환할 수 잇는 행렬 역시 가지게 됩니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-13-normal-mapping/TBN.png)

이 TBN 행렬로 (텍스처로부터 추출한) 법선을 모델 공간으로 변환할 수 있습니다. 하지만 이것은 일반적으로 다른 방법으로 수행됩니다. 모든 것을 모델 공간에서 접선 공간으로 변환한 뒤 추출한 법선을 그대로 유지합니다. 모든 계산은 접선 공간에서 수행되며 아무것도 변하지 않습니다.

이 역변환은 단순히 역행렬을 가지기만 하면 됩니다. 이 경우 전치행렬(transposed matrix)(직교행렬(orthogonal matrix), 즉 모든 벡터는 서로 직각입니다. 아래 "더 많이"를 참조하세요)이기도 하므로 계산하기 매우 쉽습니다.

``` c
invTBN = transpose(TBN)
```

, i.e. :
![](http://www.opengl-tutorial.org/assets/images/tuto-13-normal-mapping/transposeTBN.png)


# VBO 준비


## 접선과 종법선 계산

법선 뿐아니라 접선와 종법선 역시 필요하므로 이것들을 전체 메쉬에 대해 계산해야 합니다. 이것을 별도의 함수에서 수행하겠습니다.

``` cpp
void computeTangentBasis(
    // inputs
    std::vector<glm::vec3> & vertices,
    std::vector<glm::vec2> & uvs,
    std::vector<glm::vec3> & normals,
    // outputs
    std::vector<glm::vec3> & tangents,
    std::vector<glm::vec3> & bitangents
){
```

각 삼각형에 대해 모서리(deltaPos)와 deltaUV를 계산합니다.

``` cpp
    for ( int i=0; i<vertices.size(); i+=3){

        // Shortcuts for vertices
        glm::vec3 & v0 = vertices[i+0];
        glm::vec3 & v1 = vertices[i+1];
        glm::vec3 & v2 = vertices[i+2];

        // Shortcuts for UVs
        glm::vec2 & uv0 = uvs[i+0];
        glm::vec2 & uv1 = uvs[i+1];
        glm::vec2 & uv2 = uvs[i+2];

        // Edges of the triangle : postion delta
        glm::vec3 deltaPos1 = v1-v0;
        glm::vec3 deltaPos2 = v2-v0;

        // UV delta
        glm::vec2 deltaUV1 = uv1-uv0;
        glm::vec2 deltaUV2 = uv2-uv0;
```
이제 우리는 접선과 종법선을 계산하는데 아래 공식을 이용할 수 있습니다.

``` cpp
        float r = 1.0f / (deltaUV1.x * deltaUV2.y - deltaUV1.y * deltaUV2.x);
        glm::vec3 tangent = (deltaPos1 * deltaUV2.y   - deltaPos2 * deltaUV1.y)*r;
        glm::vec3 bitangent = (deltaPos2 * deltaUV1.x   - deltaPos1 * deltaUV2.x)*r;
```
끝으로 *접선*과 *종법선* 버퍼를 채웁니다. 이 버퍼들은 아직 인덱싱되지 않았기 때문에 각 정점들은 각자의 것을 가지고 있습니다.

``` cpp
        // Set the same tangent for all three vertices of the triangle.
        // They will be merged later, in vboindexer.cpp
        tangents.push_back(tangent);
        tangents.push_back(tangent);
        tangents.push_back(tangent);

        // Same thing for binormals
        bitangents.push_back(bitangent);
        bitangents.push_back(bitangent);
        bitangents.push_back(bitangent);

    }
```

## 인덱싱

VBO를 인덱싱하는 것은 이전과 거의 같지만 미묘한 차이가 있습니다.

유사한 정점을 찾더라도(동일한 위치, 법선, 동일한 텍스처 좌표), 그 정점의 접선과 종법선을 그대로 쓰고 싶지는 않습니다. 그것들의 평균을 사용하고 싶습니다. 그러므로 코드를 조금 수정하겠습니다.

``` cpp
        // Try to find a similar vertex in out_XXXX
        unsigned int index;
        bool found = getSimilarVertexIndex(in_vertices[i], in_uvs[i], in_normals[i],     out_vertices, out_uvs, out_normals, index);

        if ( found ){ // A similar vertex is already in the VBO, use it instead !
            out_indices.push_back( index );

            // Average the tangents and the bitangents
            out_tangents[index] += in_tangents[i];
            out_bitangents[index] += in_bitangents[i];
        }else{ // If not, it needs to be added in the output data.
            // Do as usual
            [...]
        }
```

아무런 정규화도 여기에서는 하지 않았다는 것에 주목하십시오. 이는 정말 유용한데 이 방법을 사용하면 작은 삼각형은 더 작은 접선과 종법선 벡터를 가지며, 큰 삼각형(최종 모형에 더 많은 기여를 하는)에 비해 최종 벡터에 더 적은 영향을 미칠 것이기 때문입니다.

# 셰이더


## 추가 버퍼 & 유니폼

접선과 종법선을 위한 두 개의 새로운 버퍼가 필요합니다.

``` cpp
    GLuint tangentbuffer;
    glGenBuffers(1, &tangentbuffer);
    glBindBuffer(GL_ARRAY_BUFFER, tangentbuffer);
    glBufferData(GL_ARRAY_BUFFER, indexed_tangents.size() * sizeof(glm::vec3), &indexed_tangents[0], GL_STATIC_DRAW);

    GLuint bitangentbuffer;
    glGenBuffers(1, &bitangentbuffer);
    glBindBuffer(GL_ARRAY_BUFFER, bitangentbuffer);
    glBufferData(GL_ARRAY_BUFFER, indexed_bitangents.size() * sizeof(glm::vec3), &indexed_bitangents[0], GL_STATIC_DRAW);
```
새로운 법선 텍스처를 위한 유니폼 변수도 필요합니다.

``` cpp
    [...]
    GLuint NormalTexture = loadTGA_glfw("normal.tga");
    [...]
    GLuint NormalTextureID  = glGetUniformLocation(programID, "NormalTextureSampler");
```
그리고 3x3 모델 뷰 행렬도 필요합니다. 사실 엄밀히 말하면 이것이 꼭 필요하지만 작업을 더 쉽게 해줍니다. 나중에 다시 설명 드리겟습니다. 좌-상단 3x3 부분만 있으면 되는데 그 이유는 방향을 곱할 것이기 때문에 평행 이동 부분은 빠져도 되기 때문입니다.

``` cpp
    GLuint ModelView3x3MatrixID = glGetUniformLocation(programID, "MV3x3");
```

모든 드로잉 코드는 다음과 같습니다.

``` cpp
        // 화면을 클리어합니다.
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

        // 셰이더를 사용합니다.
        glUseProgram(programID);

        // 키보드와 마우스 입력으로부터 MVP 행렬을 계산합니다.
        computeMatricesFromInputs();
        glm::mat4 ProjectionMatrix = getProjectionMatrix();
        glm::mat4 ViewMatrix = getViewMatrix();
        glm::mat4 ModelMatrix = glm::mat4(1.0);
        glm::mat4 ModelViewMatrix = ViewMatrix * ModelMatrix;
        glm::mat3 ModelView3x3Matrix = glm::mat3(ModelViewMatrix); // Take the upper-left part of ModelViewMatrix
        glm::mat4 MVP = ProjectionMatrix * ViewMatrix * ModelMatrix;

		// 행렬을 현재 셰이더의 "MPV" 유니폼으로 보냅니다.
        glUniformMatrix4fv(MatrixID, 1, GL_FALSE, &MVP[0][0]);
        glUniformMatrix4fv(ModelMatrixID, 1, GL_FALSE, &ModelMatrix[0][0]);
        glUniformMatrix4fv(ViewMatrixID, 1, GL_FALSE, &ViewMatrix[0][0]);
        glUniformMatrix4fv(ViewMatrixID, 1, GL_FALSE, &ViewMatrix[0][0]);
        glUniformMatrix3fv(ModelView3x3MatrixID, 1, GL_FALSE, &ModelView3x3Matrix[0][0]);

        glm::vec3 lightPos = glm::vec3(0,0,4);
        glUniform3f(LightID, lightPos.x, lightPos.y, lightPos.z);

        // 디퓨즈 텍스처를 텍스처 유닛 0에 바인드합니다.
        glActiveTexture(GL_TEXTURE0);
        glBindTexture(GL_TEXTURE_2D, DiffuseTexture);
        // "DiffuseTextureSampler" 샘플러를 유저 텍스처 유닛 0에 설정합니다.
        glUniform1i(DiffuseTextureID, 0);

        // 노말 텍스처를 텍스처 유닛 1에 바인드합니다.
        glActiveTexture(GL_TEXTURE1);
        glBindTexture(GL_TEXTURE_2D, NormalTexture);
        // "NormalTextureSampler" 샘플러를 유저 텍스처 유닛 1에 설정합니다.
        glUniform1i(NormalTextureID, 1);

        // 첫 번째 어트리뷰트 버퍼: 정점들
        glEnableVertexAttribArray(0);
        glBindBuffer(GL_ARRAY_BUFFER, vertexbuffer);
        glVertexAttribPointer(
            0,                  // attribute
            3,                  // size
            GL_FLOAT,           // type
            GL_FALSE,           // normalized?
            0,                  // stride
            (void*)0            // array buffer offset
        );

        // 두 번째 어트리뷰트 버퍼: UV들
        glEnableVertexAttribArray(1);
        glBindBuffer(GL_ARRAY_BUFFER, uvbuffer);
        glVertexAttribPointer(
            1,                                // attribute
            2,                                // size
            GL_FLOAT,                         // type
            GL_FALSE,                         // normalized?
            0,                                // stride
            (void*)0                          // array buffer offset
        );

        // 세 번재 어트리뷰트 버퍼: 법선들
        glEnableVertexAttribArray(2);
        glBindBuffer(GL_ARRAY_BUFFER, normalbuffer);
        glVertexAttribPointer(
            2,                                // attribute
            3,                                // size
            GL_FLOAT,                         // type
            GL_FALSE,                         // normalized?
            0,                                // stride
            (void*)0                          // array buffer offset
        );

        // 네 번째 어트리뷰트 버퍼: 접선들
        glEnableVertexAttribArray(3);
        glBindBuffer(GL_ARRAY_BUFFER, tangentbuffer);
        glVertexAttribPointer(
            3,                                // attribute
            3,                                // size
            GL_FLOAT,                         // type
            GL_FALSE,                         // normalized?
            0,                                // stride
            (void*)0                          // array buffer offset
        );

        // 다섯 번째 어트리뷰트 버퍼들: 종법선들
        glEnableVertexAttribArray(4);
        glBindBuffer(GL_ARRAY_BUFFER, bitangentbuffer);
        glVertexAttribPointer(
            4,                                // attribute
            3,                                // size
            GL_FLOAT,                         // type
            GL_FALSE,                         // normalized?
            0,                                // stride
            (void*)0                          // array buffer offset
        );

        // 인덱스 버퍼
        glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, elementbuffer);

        // 삼각형을 그립니다!
        glDrawElements(
            GL_TRIANGLES,      // mode
            indices.size(),    // count
            GL_UNSIGNED_INT,   // type
            (void*)0           // element array buffer offset
        );

        glDisableVertexAttribArray(0);
        glDisableVertexAttribArray(1);
        glDisableVertexAttribArray(2);
        glDisableVertexAttribArray(3);
        glDisableVertexAttribArray(4);

        // Swap buffers
        glfwSwapBuffers();
```

## 정점 셰이더

앞서 말했듯이 모든 것을 카메라 공간에서 수행할 것입니다. 그 이유는 카메라 공간에서는 프래그먼트의 위치를 얻는 것이 간단하기 때문입니다. 이는 우리가 T, B, N 벡터들을 ModelView 행렬과 곱한 이유입니다.

``` glsl
    vertexNormal_cameraspace = MV3x3 * normalize(vertexNormal_modelspace);
    vertexTangent_cameraspace = MV3x3 * normalize(vertexTangent_modelspace);
    vertexBitangent_cameraspace = MV3x3 * normalize(vertexBitangent_modelspace);
```
{: .highlightglslfs }

이 세 벡터들은 다음과 같은 방법으로 TBN 행렬을 정의합니다.

```

    mat3 TBN = transpose(mat3(
        vertexTangent_cameraspace,
        vertexBitangent_cameraspace,
        vertexNormal_cameraspace
    )); // You can use dot products instead of building this matrix and transposing it. See References for details.
```

이 행렬은 카메라 공간에서 탄젠트 공간으로 변환합니다. 우리는 이것을 접선 공간에서 빛의 방향과 눈의 방향을 계산하는데 사용할 수 있습니다.

```

    LightDirection_tangentspace = TBN * LightDirection_cameraspace;
    EyeDirection_tangentspace =  TBN * EyeDirection_cameraspace;
```

## 프래그먼트 셰이더

탄젠트 공간에서 법선은 정말 직관적으로 얻을 수 있습니다. 법선 텍스처로부터 얻습니다.

``` glsl
    // Local normal, in tangent space
    vec3 TextureNormal_tangentspace = normalize(texture( NormalTextureSampler, UV ).rgb*2.0 - 1.0);
```
{: .highlightglslfs }

이제 필요한 모든 것을 갖췄습니다. 디퓨즈 라이트는 n과 l이 접선 공간에서 표현되는 *clamp( dot( n,l ), 0,1 )*를 이용합니다(어떠한 공간에서 내적과 외적을 구하든 상관없지만, 반드시 l과 n이 같은 공간이어야 합니다.). 스펙큘러 라이트는 *clamp( dot( E,R ), 0,1 )*를 사용하며 E와 R은 탄젠트 공간에서 표현되어야 합니다. 와우!

# 결과

지금까지의 결과입니다.

* 법선이 많이 면하기 때문에 블록들이 울퉁불퉁해 보입니다.
* 법선 텍스처가 균일하게 파란색이기 때문에 시멘트는 평평해 보입니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-13-normal-mapping/normalmapping.png)


# 더 많이


## 직교화 (Orthogonalization)

정점 셰이더에서 역행렬 대신 전치행렬을 사용했습니다. 그 이유는 더 빠르기 때문입니다. 하지만 이것은 행렬이 표현하는 공간이 직교일 때만 동작하는데 아직은 그렇지 못합니다. 다행이도 직교화하는 방법은 쉽습니다. 그냥 computeTangentBasis() 마지막에서 접선을 법선에 직각이 되게 만들어주면 됩니다.

``` glsl
t = glm::normalize(t - n * glm::dot(n, t));
```
{: .highlightglslvs }

이 공식이 이해하기 어려울 수 있으므로 조금이나마 도움이 될만한 그림을 준비했습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-13-normal-mapping/gramshmidt.png)

n and t are almost perpendicular, so we "push" t in the direction of -n by a factor of dot(n,t)

[여기](http://www.cse.illinois.edu/iem/least_squares/gram_schmidt/)에 이 것를 설명하는 애플릿이 있습니다(2개의 벡터만을 사용합니다).

## 왼손-오른손 법칙

일반적으로는 걱정 할 필요가 없지만 몇몇 경우에 대칭인 모델을 사용할 경우 UV는 잘못된 방향을 가리킬 수 있고, T 역시 잘못된 방향을 가지게 됩니다.

뒤집어야 할지 말아야할지를 검사하는 것은 쉽습니다. TBN은 반드시 오른손 좌표계로 이루어져야 하므로, 즉 cross(n,t)은 반드시 b의 방향과 같아야 합니다.

수학에서 "벡터 A는 Vector B와 같은 방향을 가진다"라는 것은 dot(A, B) > 0을 의미합니다. 그러므로 dot( cross(n,t) , b ) 인지를 검사하면 됩니다.

만약 false라면 t를 뒤집습니다.

``` c
if (glm::dot(glm::cross(n, t), b) < 0.0f){
     t = t * -1.0f;
 }
```

이 작업을 computeTangentBasis()의 마지막에서 각 정점에 대해 수행합니다.

## 스펙큘러 텍스처

그저 재미로 스펙큘러 텍스처를 코드에 추가하였습니다. 텍스처는 다음과 같이 생겼습니다. 

![](http://www.opengl-tutorial.org/assets/images/tuto-13-normal-mapping/specular.jpg)

스펙큘로 색상으로 사용했던 단순히 "vec3(0.3,0.3,0.3)" 회색 대신 이 텍스처를 사용합니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-13-normal-mapping/normalmappingwithspeculartexture.png)


이제 시멘트가 항상 검게 보입니다. 텍스처에서 시멘트가 스펙큘러 요소가 없다고 표현하기 때문입니다.

## Immediate 모드에서 디버깅

이 웹사이트의 진짜 목적은 여러분들이 이제는 폐기된 느리고 여러 측면에서 문제가 많은 Immediate 모드를 사용하지 않도록 하는 것입니다.

그렇긴 하지만 Immediate 모드는 디버깅에 정말 유용하기도 합니다. 

![](http://www.opengl-tutorial.org/assets/images/tuto-13-normal-mapping/immediatemodedebugging.png)

즉시 모드에서 선을 그려서 접선공간을 시각화했습니다.

이랄 위해서는 3.3 코어 프로파일을 포기해야 합니다.

``` cpp
glfwOpenWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_COMPAT_PROFILE);
```
그리고 행렬을 OpenGL의 구식 파이프라인에 넣어줍니다(또 다른 셰이더를 만들어도 되지만 이 방법이 더 쉽습니다).

``` cpp
glMatrixMode(GL_PROJECTION);
glLoadMatrixf((const GLfloat*)&ProjectionMatrix[0]);
glMatrixMode(GL_MODELVIEW);
glm::mat4 MV = ViewMatrix * ModelMatrix;
glLoadMatrixf((const GLfloat*)&MV[0]);
```

셰이더를 끕니다.

``` cpp
glUseProgram(0);
```
선들을 그립니다(이 경우 법선들은 정규화되고 0.1이 곱해져 적절한 정점에 적용되었습니다).

``` cpp
glColor3f(0,0,1);
glBegin(GL_LINES);
for (int i=0; i<indices.size(); i++){
    glm::vec3 p = indexed_vertices[indices[i]];
    glVertex3fv(&p.x);
    glm::vec3 o = glm::normalize(indexed_normals[indices[i]]);
    p+=o*0.1f;
    glVertex3fv(&p.x);
}
glEnd();
```
실제 게임에는 절대 Immediate 모드를 사용하지 마십기오. 디버깅 용도로만 사용하십시오. 그리고 반드시 코어 프로필을 다시 활성화시켜 Immediate 모드를 사용하디 않는다는 것을 명시하십시오.

## 색상으로 디버깅

디버깅시에 벡터의 값을 시각화하면 유용합니다. 가장 쉬운 방법은 프레임버퍼에 실제 색상 대신 이 값을 쓰는 것입니다. 예를 들어 LightDirection_tangentspace 를 시각화 해 보겠습니다.

``` glsl
color.xyz = LightDirection_tangentspace;
```
{: .highlightglslfs }

![](http://www.opengl-tutorial.org/assets/images/tuto-13-normal-mapping/colordebugging.png)

의미는 다음과 같습니다.

* 원통의 오른쪽 부분은 빛(작은 흰선으로 표현되었습니다)으로의 방향이 UP(접선공간에서)입니다. 달리 말하면 빛이 삼각형의 법선의 방향에 있다는 것입니다.
* 원통의 가운데 부분은 빛이 접선 방향(+X 방향으로)에 존재합니다.

약간의 팁 :

* 무엇을 시각화하고 싶은지에 따라 정규화를 해야 할 수도 있습니다.
* 보고 있는 것에 대해 무엇인지 도통 알 수가 없을 때는 green과 blue를 강제로 0으로 설정해서 모든 컴포넌트들을 따로 시각화하십시오.
* 알파로 엉망이 되는 것을 피하십시오. 알파는 너무 복잡합니다 :)
* 음수에 대해서 시각화하고 싶다면 노말 텍스처와 같은 방법을 사용하면 됩니다. (v+1.0)/2.0 로 시각화를 합니다. 즉 검은 색은 -1로, 흰 색은 +1을 나타내게 됩니다. 그럼에도 불구하고 보는 것을 이해하는 것은 어렵습니다.

 

## 변수명으로 디버깅

이미 언급했듯이 벡터가 존재하는 공간을 정확하게 아는 것은 매우 중요합니다. 절대 카메라 공간의 벡터와 모델 공간의 벡터간의 내적을 구하지 마십시오.

변수명에 공간을 붙이면 ("..._modelspace") 산식 버그를 고치는데 엄청난 도움이 됩니다.

## 
## 노말 맵을 만드는 방법

James O'Hare에 의해 만들어 졌습니다. 클릭하면 크게 볼 수 있습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-13-normal-mapping/normalMapMiniTut.jpg)


# 연습

* indexVBO_TBN의 벡터들을 추가하기 전에 정규화한 후 무슨일이 일어나는지 확인해 보십시오.
* 다른 벡터들을 컬러 모드를 이용해 시각해 보세요(예를 들면 EyeDirection_tangentspace). 그리고 보이는 것을 이해하려고 해보세요.  


# 도구 & 링크

* [Crazybump](http://www.crazybump.com/) , 노말 맵을 만드는 뛰어난 툴이지만 유료입니다.
* [Nvidia의 포토샵 플러그인](http://developer.nvidia.com/nvidia-texture-tools-adobe-photoshop). 무료이지만 포토샵은 그렇지 않죠...
* [여러 사진으로 노말 맵 만들기](http://www.zarria.net/nrmphoto/nrmphoto.html)
* [한개의 사진으로 노말 맵 만들기](http://www.katsbits.com/tutorials/textures/making-normal-maps-from-photographs.php)
* 좀 더 많은 정보 [matrix transpose](http://www.katjaas.nl/transpose/transpose.html)


# References


* [Lengyel, Eric. "Computing Tangent Space Basis Vectors for an Arbitrary Mesh". Terathon Software 3D Graphics Library, 2001.](http://www.terathon.com/code/tangent.html)
* [Real Time Rendering, third edition](http://www.amazon.com/dp/1568814240)
* [ShaderX4](http://www.amazon.com/dp/1584504250)




