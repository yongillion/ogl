---
layout: page
status: publish
published: true
title: FPS 카운터
date: '2012-01-28 16:24:41 +0100'
date_gmt: '2012-01-28 16:24:41 +0100'
categories: []
order: 40
tags: []
---

실시간 그래픽에서 성능에 주의를 기울이는 것은 중요합니다. 목표로 하는 FPS (일반적으로 60이나 30)를 선택하고 모든 것이 그 안에 들어올 수 있도록 하는 것은 좋은 실천입니다.

FPS 카운터는 다음과 같이 생겼습니다.

``` cpp
 double lastTime = glfwGetTime();
 int nbFrames = 0;

 do{

     // 속도 측정
     double currentTime = glfwGetTime();
     nbFrames++;
     if ( currentTime - lastTime >= 1.0 ){ // 마지막 printf()가 1초보다 이전이었다면
         // 출력하고 타이머 리셋
         printf("%f ms/frame\n", 1000.0/double(nbFrames));
         nbFrames = 0;
         lastTime += 1.0;
     }

     ... 나머지 메인 루프
```

이 코드에 특이한 점이 있는데, 1초에 몇 개의 프레임을 그려졌는지를 출력하는 것이 아니라  프레임을 하나 그리는데 필요한 시간(1초동안 평균)을 밀리초로 출력한다는 것입니다.

이것이 사실 **훨씬 좋습니다**. FPS에 절대 의존하지 마십시오. FramesPerSecond = 1/SecondsPerFrame, 이것은 역관계이며 우리 인간은 이러한 관계를 이해하는데 형편없습니다. 예를 들어보죠.

여러분은 1000 FPS (1ms/frame)의 엄청난 렌더링 함수를 만들었습니다. 하지만 셰이더에서 작은 계산을 실수 했으며 이로 인해 0.1ms의 비용이 추가되었습니다. 그리고 펑, 1/0.0011 = 900. 100FPS를 읽어버렸습니다. *절대 성능 분석용으로 FPS를 사용하지 마십시오*

60fps 게임을 만들려면 목표는 16.6666ms입니다. 30fps 게임을 만들고 싶다면 target은 33.333입니다. 이것이 당신이 알아야 할 전부입니다.

이 코드는 [튜토리얼 9 : VBO 인덱싱]({{site.baseurl}}/intermediate-tutorials/tutorial-9-vbo-indexing/)으로부터 시작하는 모든 코드에서 사용가능합니다. [tutorial09_vbo_indexing/tutorial09.cpp](https://github.com/opengl-tutorials/ogl/blob/master/tutorial09_vbo_indexing/tutorial09.cpp#L142) 를 참조하세요. 다른 퍼포먼스 도구는 [Tools - Debuggers]({{site.baseurl}}/miscellaneous/useful-tools-links/#debugging-tools)에서 사용 가능합니다.
