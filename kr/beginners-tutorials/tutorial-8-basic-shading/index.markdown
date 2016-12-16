---
layout: page
status: publish
published: true
title: '튜토리얼 8 : 셰이더 기초'
date: '2011-05-08 19:12:46 +0200'
date_gmt: '2011-05-08 19:12:46 +0200'
categories: [tuto]
order: 80
tags: []
---

여덟 번째 튜토리얼에서 우리는 기본적인 셰이더에 대해 배울 것입니다. 아래와 같은 것들을 배웁니다.

* 광원에서 가까울수록 더 밟아진다.
* 빛이 반사되었을 때 보이는 하이라이트(스펙큘러 라이팅, 정반사광).
* 빛이 모델에 직접 닿지 않을 경우 어두워집니다(디퓨즈 라이팅, 난반사광). 
* 기타 (Ambient Lighting, 환경광)

다음은 배우지 않습니다.

* 그림자. 그림자는 전용 튜토리얼을 만들어야 할만큼 넓은 주제입니다.
* 거울과 같은 반사(물도 포함됩니다.)
* 표면에 흩어진 무언가와의 (왁스같은) 상호작용에 의한 정교한 빛.
* 이방성(異方性) 머테리얼 (브러쉬 메탈 재질 같은).
* 물리 기반 셰이딩. 현실괴 유사하게 흉내내려는 시도.
* 앰비언트 오클루젼 (동굴안과 같은 어두움)
* 컬러 블리딩(붉은 카펫이 흰 천장을 약간 붉게 만드는 것)
* 반투명
* 글로벌 일루미네이션 같은 것들 (이는 위 모든 것들을 엮은 것입니다).

한 마디로 기본만 하겠습니다.

# 법선

앞선 몇개의 튜토리얼에서 법선을 사용허긴 했지만 정확한 의미를 알려드리지 않았습니다.

## 삼각형의 법선 Triangle normals

평면의 법선은 평면에 직각이고 길이가 1인 벡터입니다.

삼각형의 법선은 삼각형에 직각이고 길이가 1인 벡터입니다. 모서리중 두 개의 외적 계산한 후 정규화(길이를 1로 만드는 것)함으로써 쉽게 계산할 수 있습니다(a와 b를 외적하면 a와 b 모두에 수직인 벡터가 나옵니다, 기억하나요?).

의사 코드로는

```
triangle ( v1, v2, v3 )
edge1 = v2-v1
edge2 = v3-v1
triangle.normal = cross(edge1, edge2).normalize()
```

법선(normal)과 정규화(normalize())를 헷갈리면 안됩니다. Normalize()는 벡터(아무 벡터나, 꼭 법선이 아니라)를 자신의 길이로 나누어서 그 길이를 1로 만듭니다. 법선은 음... 법선을 나타냅니다.

## 정점의 법선

확장하여, 정점 주변의 모든 삼각형의 법선을 조합한 것을 정점의 법선이라고 부릅니다. 이는 정점 셰이더에서 유용하게 사용되기 때문에 삼각형이 아니라 정점을 다룰 것입니다. 그러므로 정점에 대한 정보를 가지는 것이 더 낫습니다. 그리고 OpenGL에서는 삼각형에 대한 정보를 가질 수 있는 방법이 없습니다. 의사 코드로는

```
vertex v1, v2, v3, ....
triangle tr1, tr2, tr3 // all share vertex v1
v1.normal = normalize( tr1.normal + tr2.normal + tr3.normal )
```

## OpenGL에서 정점의 법선 사용하기

OpenGL에서 법선을 사용하는 것은 매우 쉽습니다. 법선은 위치나 색, UV 좌표와 마찬가지로 정점의 어트리뷰트입니다. 그렇기 때문에 해왔던 것처럼 하면 됩니다. 튜토리얼 7에서 만들었던 loadOBJ 함수는 이미 이를 OBJ 파일로부터 읽어들입니다.

``` cpp
GLuint normalbuffer;
 glGenBuffers(1, &normalbuffer);
 glBindBuffer(GL_ARRAY_BUFFER, normalbuffer);
 glBufferData(GL_ARRAY_BUFFER, normals.size() * sizeof(glm::vec3), &normals[0], GL_STATIC_DRAW);
```

그리고

``` cpp
 // 3rd attribute buffer : normals
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
```

시작하기에 충분합니다.

# 디퓨즈 라이팅

## 표면 법선의 중요성

빛이 오브젝트에 닿을 때 중요한 부분 모든 방향으로 반사된다는 것입니다. 이것을 "디퓨트 컴포넌트"라고 부릅니다(다른 일부는 어떠한 일이 일어나는지 곧 살펴보겠습니다.).
When light hits an object, an important fraction of it is reflected in all directions. This is the "diffuse component". (We'll see what happens with the other fraction soon)

![](http://www.opengl-tutorial.org/assets/images/tuto-8-basic-shading/diffuseWhite1.png)

어떠한 빛이 표면에 닿으면 표면은 빛이 닿은 각도에 따라 다르게 비춰집니다.

및이 표면에 직각으로 닿으면 적은 면적에 집중됩니다. 만얄 기울어진 각도로 닿으면 동일한 광량이 더 넓은 면적으로 퍼지게 됩니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-8-basic-shading/diffuseAngle.png)

이는 표면의 각 점 중 빛이 기울어져 닿은 점이 더 어두워 보인다는 것을 의미합니다(하지만 더 많은 점이 빛날 것이므로 빛의 총량은 같다는 것을 기억하시길 바랍니다).

이는 픽셀의 색을 계산할 때, 표면에 닿는 빛과 표면 법선의 각도가 중요하다는 것을 의미합니다. 그러므로

``` glsl
// 법선과 빛의 방향 사이의 각도에 대한 코사인을 구합니다.
// clamped above 0
//  - 빛이 삼각형에 수직이면 -> 1
//  - 및이 삼각형에 직각이면 -> 0
float cosTheta = dot( n,l );

color = LightColor * cosTheta;
```
{: .highlightglslfs }

코드에서 n은 표면의 법선이고, l은 표면에서 빛으로 향하는 유닛 벡터입니다(반대가 아닙니다. 직관적이지 않지만 수학 계산을 쉽게 해줍니다).

## 부호에 주의

cosTheta를 구하는 공식에 무언가가 빠졌습니다. 만약 빛이 삼각형의 뒤에 있다면 n과 l은 반대 방향이고, n.l은 음수가 됩니다. 이는 color = 어떠한 음수가 된다는 것을 의미합니다. 그러므로 우리는 cosTheta가 0이하일 경우 0으로 고정시킵니다. 

``` glsl
// 법선과 빛의 방향 사이의 각도에 대한 코사인을 구합니다.
// 그리고 음수일 겨웅 0으로 고정합니다.
//  - 빛이 삼각형에 수직이면 -> 1
//  - 빛이 삼각형에 직각이면 -> 0
//  - 및이 삼각형의 뒤에 있으면 -> 0
float cosTheta = clamp( dot( n,l ), 0,1 );

color = LightColor * cosTheta;
```
{: .highlightglslfs }

## 머테리얼 색

당연히 출력되는 색상은 머테리얼의 색에 따라 달라집니다. 이 그림에서 흰색 빛은 초록색과 빨간색, 녹색으로 만들어집니다. 빛이 빨간색 머테리얼에 닿으면 초록색과 파란색은 흡수되고 빨간색만이 남습니다. 

![](http://www.opengl-tutorial.org/assets/images/tuto-8-basic-shading/diffuseRed.png)


우리는 이것을 간단히 곱해서 모델링할 수 있습니다.

``` glsl
color = MaterialDiffuseColor * LightColor * cosTheta;
```
{: .highlightglslfs }

## 빛 모델링하기

양초와 같이 특정 점으로 부터 공간의 모든 방향으로 방출되는 빛이 있다고 가정합니다.

이 경우 표면에 전달되는 빛의 양은 광원과의 거리에 따라 달라집니다. 멀수록 더 적은 빛을 받습니다. 사실 빛의 양은 거리의 제곱에 따라 줄어듭니다.

``` glsl
color = MaterialDiffuseColor * LightColor * cosTheta / (distance*distance);
```
{: .highlightglslfs }

끝으로 빛의 세기를 조절하기 위한 파라미터가 필요합니다. 이걸 LightColor로 인코딩 될 수 있지만(그리고 나중에 나오는 튜토리얼에서 해볼 것입니다), 지금은 색(예, 흰색)과 세기 (예 60와트)만 가지도록 합니다.

``` glsl
color = MaterialDiffuseColor * LightColor * LightPower * cosTheta / (distance*distance);
```
{: .highlightglslfs }

## 모든 것을 합치기

이 코드가 동작하게 하기 위해서 우리는 몇 개의 파라미터들(여러가지 색과 세기들)과 코드가 조금 더 필요합니다.

MaterialDiffuseColor는 텍스처로부터 간단히 가져옵니다.

LightColor와 LightPower는 GLSL 유니폼을 통해 셰이더로 넘겨집니다.

cosTheta는 n과 l에 의해 결정됩니다. 어떠한 공간에서 해도 상관없지만 둘 모두 같은 공간이어야 합니다. 카메라 공간해서 하도록 하겠습니다. 그 이유는 공간에서 빛의 위치를 계산하기 쉽기 때문입니다.

``` glsl
// Normal of the computed fragment, in camera space
 vec3 n = normalize( Normal_cameraspace );
 // Direction of the light (from the fragment to the light)
 vec3 l = normalize( LightDirection_cameraspace );
```
{: .highlightglslfs }

Normal_cameraspace와 LightDirection_cameraspace가 정점 셰이더에서 계산되어 프래그먼트 셰이더로 전달됩니다.

``` glsl
// 절단 공간에서 정점의 위치를 출력합니다: MVP * position
gl_Position =  MVP * vec4(vertexPosition_modelspace,1);

// 월드 공간에서 정점의 위치: M * position
Position_worldspace = (M * vec4(vertexPosition_modelspace,1)).xyz;

// 카메라 공간에서 정점에서 카메라로 향하는 벡터
// 카메라 공간에서 카메라는 원점(0,0,0)에 위치합니다.
vec3 vertexPosition_cameraspace = ( V * M * vec4(vertexPosition_modelspace,1)).xyz;
EyeDirection_cameraspace = vec3(0,0,0) - vertexPosition_cameraspace;

// 카메라 공간에서 정점으로부터 빛으로 향하는 벡터. M은 단위 행렬이기 때문에 누락되었다. 
vec3 LightPosition_cameraspace = ( V * vec4(LightPosition_worldspace,1)).xyz;
LightDirection_cameraspace = LightPosition_cameraspace + EyeDirection_cameraspace;

// 카메라 공간에서 정점의 법선
Normal_cameraspace = ( V * M * vec4(vertexNormal_modelspace,0)).xyz; // Only correct if ModelMatrix does not scale the model ! Use its inverse transpose if not.
```
{: .highlightglslvs }

이 코드는 강한 인상을 주지만 사실 튜토리얼 3 : 행렬에서 배우지 않은 것은 없습니다. 무슨 일이 일어나는지 이해하기 쉽도록 각 벡터의 이름에 공간의 이름을 넣도록 주의를 기울였습니다. **여러분도 이렇게 하시길 바랍니다.**

M과 V는 MVP와 동일한 방법으로 셰이더에 전달되는 모델과 뷰 행렬입니다.

## 작업하기

빛은 산란시키기 위한 모든 준비가 되었습니다. 어서 해보시고, 쉽지 않을겁니다 :)

## 결과

디퓨즈 컴포넌트만으로 다음과 같은 결과를 얻을 수 있습니다(또한번 서투른 텍스처에 사과드립니다).
With only the Diffuse component, we have the following result (sorry for the lame texture again) :

![](http://www.opengl-tutorial.org/assets/images/tuto-8-basic-shading/diffuse_only.png)

이전보다 낫지만 여전히 뭔가 부족합니다. clamp()를 사용했기 때문에 특히 수잔의 등쪽에 완전히 어둡습니다.

# The Ambient component

앰비언트 컴포넌트는 가장 큰 사기입니다.

실제로는 수잔의 등쪽이 지금보다 빛을 조금 더 받을 것입니다. 그 이유는 램프가 수잔의 뒤에 있는 벽을 비추므로 오브젝트의 뒤로 (아주 약간) 반사되어 올 것이기 때문입니다.

이를 계산하는 비용은 끔찍하게 많이 듭니다.

그래서 일반적으로는 단순히 가짜 빛을 사용합니다. 사실은 단순히 3D 모델이 빛을 *방출*하도록 해서 완전히 검은색으로 보이지 않게 합니다.

다음과 같은 방법으로 할 수 있습니다.

``` glsl
vec3 MaterialAmbientColor = vec3(0.1,0.1,0.1) * MaterialDiffuseColor;
```
{: .highlightglslfs }

``` glsl
color =
 // 앰비언트 : 간접광을 흉내냅니다.
 MaterialAmbientColor +
 // 디퓨즈 : 오브젝트의 "색"
 MaterialDiffuseColor * LightColor * LightPower * cosTheta / (distance*distance) ;
```
{: .highlightglslfs }

어떻게 되는지 봅시다.

## 결과

좀 더 낫습니다. (0.1, 0.1, 0.1)은 조절하면 더 나은 결과를 얻을 수 있습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-8-basic-shading/diffuse_ambiant.png)


# 스펙큘러 컴포넌트

반사되는 빛의 또 다른 일부는 대부분 표면과 대칭되는 방향으로 반사됩니다. 이것은 스펙큘러 컴포넌트입니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-8-basic-shading/specular.png)


그림에서 볼 수 있듯이 로브(lobe)의 모양을 하고 있습니다. 극단적인 경우에는 디퓨즈 컴포넌트가 없어 로브(lobe)이 매우매우매우 좁아져서 (모든 빛이 한방향으로 반사되어서) 거울처럼 될 수도 있습니다.

(*사실 우리는 거울처럼 되도록 매개변수들을 변경할 수 있지만, 이 경우 거울이 취급하는 것은 빛 뿐이므로 이상한 거울을 얻게 될것입니다.)*

``` glsl
// Eye 벡터 (camera를 향하는)
vec3 E = normalize(EyeDirection_cameraspace);
// 삼각형이 빛을 반사하는 방향
vec3 R = reflect(-l,n);
// Eye 벡터와 Reflect 벡터사이의 각도 코사인
// clamped to 0
//  - 반사되는 쪽을 바라본다. -> 1
//  - 다른 곳을 바라본다. -> < 1
float cosAlpha = clamp( dot( E,R ), 0,1 );

color =
    // 앰비언트: 간접광을 흉내
    MaterialAmbientColor +
    // 디퓨트 : 오브젝트의 "색"
    MaterialDiffuseColor * LightColor * LightPower * cosTheta / (distance*distance) ;
    // 스펙큘라 : 거울처럼 반사되는 하이라이트
    MaterialSpecularColor * LightColor * LightPower * pow(cosAlpha,5) / (distance*distance);
```
{: .highlightglslfs }

R은 빛이 반사되는 방향입니다. E는 "eye"와 역방향입니다(l에 한것과 같이). 만약 이 둘 사이의 각이 작다면  반사하는 방향을 바로 보고 있는 것을 의미합니다.

pow(cosAlpha,5)는 스펙큘러 로브의 너비를 조절합니다. 더 얇은 로브를 얻기 위해서는 5만큼 증가시키면 됩니다.

## 최종 결과

![](http://www.opengl-tutorial.org/assets/images/tuto-8-basic-shading/diffuse_ambiant_specular.png)


스펙큘러가 코와 눈썹을 빛나게 하는 것에 주목하십시오.

이 명암 모델은 간단하기 때문에 수년간 사용되었습니다. 하지만 많은 문제를 가지고 있기  때문에 microfacet BRDF와 같은 물리 기반 모델로 변경되고 있습니다. 나중에 살펴보겠습니다.

다음 튜토리얼에서는 VBO의 성능을 개선하는 방법에 대해 배워보겠습니다. 첫번째 중급자용 튜토리얼이 될 것입니다!
