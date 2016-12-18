---
layout: page
status: publish
published: true
title: '튜토리얼 9 : VBO 인덱싱'
date: '2011-05-12 19:21:49 +0200'
date_gmt: '2011-05-12 19:21:49 +0200'
categories: [tuto]
order: 10
tags: []
---

# 인덱싱의 원리

지금까지 우리는 VBO를 만들 때, 두 개의 삼각형이 꼭지점을 공유할 경우 정점을 중복해서 정의했습니다.

이 튜토리얼에서는 동일한 정점을 여러번 재사용할 수 있는 인덱싱을 소개합니다. 이것은 *인덱스 버퍼*라는 것을 이용해서 수행할 수 있습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-9-vbo-indexing/indexing1.png)


인덱스 버퍼는 여러가지 *어트리뷰트 버퍼*(위치, 색, UV 좌표, 다른 UV 좌표, 법선 등...)를 참조하는 정수를 메쉬의 각 삼각형마다 세개씩 가지고 있습니다. OBJ 파일 포맷과 비슷한 것 같지만 한 가지 큰 차이가 있습니다. 바로 하나의 인덱스 버퍼만을 가진다는 것입니다. 이는 어떠한 정점이 두 개의 삼각형 사이에 공유되어야 한다면 모든 어트리뷰트가 같아야만 한다는 것을 의미합니다.

# 공유 vs 분리

법선의 예를 들어봅시다. 아래 그림과 같은 두개의 삼각형을 만든 아티스트는 아마도 완만한 표면을 표현하기를 원했을지도 모릅니다. 그러므로 이 경우 두 삼각형의 법선을 섞어서 하나의 정점 법선을 만들 수 있습니다. 더 잘 보여주기 위해 완만한 표면을 나타내는 붉은 선을 추가했습니다. 

![](http://www.opengl-tutorial.org/assets/images/tuto-9-vbo-indexing/goodsmooth.png)

하지만 두 번째 그림에서는 아트스트는 시각적으로 "뾰족한" 모서리를 원했습니다. 만약 두 삼각형의 법선을 합쳐버리면 셰이더에서는 이전과 비슷하게 일반적으로 보간(interplate)을 해서 완곡하게 만들어버릴 것입니다.
![](http://www.opengl-tutorial.org/assets/images/tuto-9-vbo-indexing/badmooth.png)

이 경우에는 사실 각 정점마다 다른 두개의 법선을 가지는 것이 낫습니다. OpenGL에서 이를 위한 방법은 법선을 제외한 정점의 모든 어트리뷰트들을 중복해서 가지는 것입니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-9-vbo-indexing/spiky.png)


# OpenGL에서 VBO 인덱싱

인덱싱을 사용하는 것은 매우 쉽습니다. 먼저 적절한 인덱스들을 채워 넣을 추가적인 버퍼를 생성해야 합니다. 코드는 이전과 같지만 ARRAY_BUFFER가 아닌 ELEMENT_ARRAY_BUFFER를 인자로 사용합니다.

``` cpp
std::vector<unsigned int> indices;

// fill "indices" as needed

// Generate a buffer for the indices
 GLuint elementbuffer;
 glGenBuffers(1, &elementbuffer);
 glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, elementbuffer);
 glBufferData(GL_ELEMENT_ARRAY_BUFFER, indices.size() * sizeof(unsigned int), &indices[0], GL_STATIC_DRAW);
```

메쉬를 그리기 위해 glDrawArrays 대신 다음과 같은 코드를 수행합니다.

``` cpp
// Index buffer
 glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, elementbuffer);

 // Draw the triangles !
 glDrawElements(
     GL_TRIANGLES,      // mode
     indices.size(),    // count
     GL_UNSIGNED_INT,   // type
     (void*)0           // element array buffer offset
 );
```

(빠른 노트, "unsigned int" 대신 "unsigned short"를 사용하는 것이 낫습니다. 그 이유는 적은 메모리를 사용하고 이로 인해 더 빠르기 때문입니다.)

# 인덱스 버퍼 채우기

이제 실제로 문제가 있습니다. 앞서 말했듯이 OpenGL은 하나의 인덱스 버퍼만을 사용할 수 있습니다. 반면 OBJ (Collada와 같이 다른 유명한 3D 포맷들도)는 "어트리뷰트"마다 하나의 인덱스 버퍼를 가집니다. 이 말은 우리가 직접 어떠한 방법으로 여러개의 인덱스 버퍼를 하나의 인덱스 버퍼로 변환해주어야 한다는 의미입니다.

이를 위한 알고리즘은 다음과 같습니다. :

```
각 입력된 정점마다
   모든 이미 출력한 정점들 중에 동일한(= 모든 어트리뷰트가 동일한) 정점이 존재하는지 찾습니다.
   존재한다면:
      이미 동일한 정점이 VBO안에 존재하므로 사용합니다.
   존재하지 않는다면:
      동일한 정점이 없으므로 VBO에 추가합니다.
```

실제 C++ 코드는 "common/vboindexer.cpp"에서 찾을 수 있습니다. 주석을 아주 열심히 달았기 때문에 위 알고리즘을 이해했다면 충분히 이해할 수 있습니다.

동일하다는 기준은 정점의 위치, UV, 법선이 **동일한지입니다. 더 많은 어트리뷰트가 필요하다면 적절히 응용하시길 바랍니다.

단순하게 하기위해 동일한 정점을 찾는 방법은 선형 검색을 이용했습니다. 실제 사용에는 std::map이 더 적절할 것입니다.

# 추가 : FPS 카운터

익덱싱과 직접적인 연관이 있지는 않지만 [the FPS counter](http://www.opengl-tutorial.org/miscellaneous/an-fps-counter/)를 살펴보기 적절한 시점인 것 같습니다. 결과적으로 인덱싱으로 인한 속도 향상을 살펴 볼 수 있기 때문입니다. 다른 성능 도구는 [Tools - Debuggers](http://www.opengl-tutorial.org/miscellaneous/useful-tools-links/#debugging-tools)에서 볼 수 있습니다.
