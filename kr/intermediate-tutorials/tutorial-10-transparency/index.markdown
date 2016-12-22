---
layout: page
status: publish
published: true
title: '튜토리얼 10 : 반투명'
date: '2011-05-13 23:00:42 +0200'
date_gmt: '2011-05-13 23:00:42 +0200'
categories: [tuto]
order: 20
tags: []
---

# 알파 채널

알파 채널의 개념은 아주 간단합니다. RGB 대신 RGBA를 씁니다.


``` glsl
// 출력 데이터 : 이제 vec4입니다.
out vec4 color;
```
{: .highlightglslfs }

첫 번째 3 요소는 여전히 .xyz swizzle 오퍼레이터로 접근하는 반면 마지막은 .a로 접근합니다.
``` glsl
color.a = 0.3;
```
{: .highlightglslfs }

조금 비직관적으로 alpha = 불투명도를 나타냅니다. 그러므로 alpha = 1은 완전히 불투명하고, alpha = 0은 완전히 투명합니다.

여기 간단히 알파 채널을 0.3으로 하드코딩 하였습니다. 대신 유니폼을 사용하거나 RGBA 텍스처(TGA는 알파 채널을 지원하며 GLFW는 TGA를 지원합니다)로 읽어 들이고 싶을 것입니다.

여기 그 결과입니다. 메쉬를 통과해서 봄으로 후면이 보이지 않는 일이 없도록 후면 제거를 끄는 것을 잊지 마십시오(glDisable(GL_CULL_FACE)).

![](http://www.opengl-tutorial.org/assets/images/tuto-10-transparency/transparencyok.png)


# 순서 문제 !

이전 스크린샷에서 괜찮아 보이지만 그건 우리가 운이 좋기 때문입니다. 최종 색은 우리의 눈이 적절한 깊이를 인식할 수 있도록 하기 때문에 순서가 매우 중요합니다.

## The problem
50%의 알파로 정사각형을 두 개 그렸습니다. 하나는 초록색, 하나는 빨간색으로. 순서가 중요한 것을 알 수 있습니다.

![](http://www.opengl-tutorial.org/assets/images/tuto-10-transparency/transparencyorder.png)

이러한 현상을 우리 씬에서도 일어납니다. 뷰 포인트를 조금 변경해보죠.

![](http://www.opengl-tutorial.org/assets/images/tuto-10-transparency/transparencybad.png)

이는 매우 어려운 문제입니다. 아마 게임에서 반투명한 것을 많이 보지는 못했을 겁니다. 그렇지 않나요?

## 일반적인 해결책

일반적인 해결책은 모든 반투명한 삼각형을 정렬하는 것입니다. 맞습니다. 모든 반투명한 삼각형이요.

* 월드의 모든 불투명한 부분을 그려서 깊이 버퍼가 이미 숨겨진 반투명 삼각형을 그리지 않도록 합니다.
* 반투명 삼각형들을 멀리서 가까운 순으로 정렬합니다.
* 반투명 삼각형들을 그립니다.

qsort(C에서)나 std::sort(C++에서)를 이용해서 원하는 것은 무엇이든 정렬할 수 있습니다. 여기서 싶이 다루지는 않겠습니다. 그 이유는...

## 경고

위에서 언급한 방법은 확실히 동작합니다(다음 섹션에서 이에 대해 더). 하지만,

* 필레이트 제약을 받을 것입니다. 모든 프래그먼트가 열 번, 스무 번 혹은 그 이상 그려질 수도 있기 때문입니다. 이는 부족한 메모리 버스에게 너무 가혹합니다. 일반적으로 깊이 버퍼는 "먼" 프래그먼트들이 그려지지 않게 합니다. 하지만 여기서 우리는 명시적으로 이들을 정렬했기 때문에 사실상 깊이 버퍼는 쓸모 없어집니다.
* 더 똑똑한 최적화를 하지 않는 이상 이 과정은 픽셀당 네 번(4xMSAA를 사용하므로) 수행 될 것입니다.
* 모든 반투명 삼각형을 정렬하는 것은 시간이 걸립니다.
* 삼각형 삼각형마다 텍스처를 변경해야 하거나, 또는 더 최악으로 셰이더를 변경해야 한다면 정말 심각한 성능 문제에 빠지게 됩니다. 절대 이러한 것은 하지 마십시오.

좋은 충분한 해결책은 보통 다음과 같습니다. 

* 반투명한 폴리곤의 최대 개수를 제한하세요.
* 그 모든 것들에 동일한 셰이더와 동일한 텍스처를 사용하세요.
* 만약 그것들이 많이 다르게 보이는 것 같다면 다른 텍스처를 사용하세요!
* 정렬을 하지 않고도 별로 나빠보이지 않는다면, 운이 좋다고 생각해야 합니다.

## 순서에 무관한 반투명

다수의 기법들이 조사해 볼만한 가치가 있기 때문에, 여러분의 엔진이 정말 정말 최신 반투명 기법을 필요로 한다면 다음을 참조하세요.

* [The original 2001 Depth Peeling paper](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.18.9286&rep=rep1&type=pdf): 픽셀-퍼펙트한 결과이지만 별로 빠르진 않습니다.
* [Dual Depth Peeling](http://developer.download.nvidia.com/SDK/10/opengl/src/dual_depth_peeling/doc/DualDepthPeeling.pdf) : 개선이 미미합니다.
* 버킷 정렬에 대한 몇가지 논문. 프래그먼트의 배열을 사용해 셰이더에서 깊이 순으로 정렬합니다.
* [ATI's Mecha Demo](http://fr.slideshare.net/hgruen/oit-and-indirect-illumination-using-dx11-linked-lists) : 빠르고 좋습니다. 하지만 구현하기 힘들고 최신 하드웨어가 필요합니다. 프래그먼트들의 링크드 리스트를 사용합니다.
* [Cyril Crassin's variation on the ATI's  technique](http://blog.icare3d.org/2010/07/opengl-40-abuffer-v20-linked-lists-of.html) : 구현하기 몹시 힘듭니다.

리틀 빅 플래닛 같은 강력한 콘솔에서 구동되는 최신 게임도 반투명 레이어를 한 개만 가지고 있다는 것을 주목하세요.

# 블렌드 함수

앞의 코드들이 동작하도록 하기 위해서는 블렌드 함수를 셋업해야 합니다.

``` cpp
// 블렌딩을 켭니다.
glEnable(GL_BLEND);
glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
```

이것이 의미하는 바는,
```

New color in framebuffer =
           current alpha in framebuffer * current color in framebuffer +
           (1 - current alpha in framebuffer) * shader's output color
```

위 이미지에서 붉은 색이 위에 있는 경우에 대한 예제입니다.

``` cpp
new color = 0.5*(0,1,0) + (1-0.5)*(1,0.5,0.5); // (붉은 색은 이미 흰 배경과 블렌딩 되었습니다.)
new color = (1, 0.75, 0.25) = 오렌지 색
```

 
