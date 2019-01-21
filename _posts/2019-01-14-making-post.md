---
layout: post
title: Jekyll 블로그를 만드러 보았당
tags: jekyll Type-Theme css MarkDown
feature-img: "assets/img/2019-01-14/1_kFaSuc5rVdJ_fM2c_bHszQ.png"
---
<code>HTML</code>과 <code>CSS</code> 등은 다시는 보기 싫지만, 나는 내가 다시 볼 것 같은 다른 위대하신 분들의 포스팅이나 Documents를
Chrome탭에서 쉽게 놓아주지 못한다.

{% include image.html url='/assets/img/2019-01-14/screenshot2019-01-14am12.18.49.png' description='요따만한게 3~4개씩 있으면10분 간격으로 화가 난다' alt='크롬캡쳐' %}

그래서 블로그를 만들어서 그걸 정리해보려고 했다~~ 라는게 정계의 학설...

개발자라고 하시는 분들은 <code>username.github.io</code> 같은 도메인의 웹페이지에 블로깅을 하시던데, 나에게 Github는 그냥 과제물을 저따 올려두면
교수님 칭찬서비스를 받을 수도 있는 정도였다.

아니 뭐, 어찌 되던 블로그를 만들어야지! 라는 다짐과 이제 나도 블로그하는 개발자 형님들 비스므리한 벌래가 될 수 있을지 모른다는 기대감에 신나게
시작하였지만...

{% include image.html url='/assets/img/2019-01-14/screenshot2019-01-14am1.21.59.png' description='요까지 오는데 3일이 걸렸다...' alt='IDE캡쳐' %}

써본적도 없는 <code>CSS</code> 는 생각보다(?) 과학적이었는데, 주력이 자바와 같은 C++스타일의 객체지향인 내가 보기에 정적인 View의 스타일링을 위한
구조라기엔 확실히 객체지향적 설계가 눈에 보였다.

{% include image.html url='/assets/img/2019-01-14/screenshot2019-01-14am1.35.24.png' description='@extends와 도트 연산자 등은 HTML 구조와 연계되어 직관적이고, 리팩토링을 위한 설계가 용이해질것 같다' alt='IDE캡쳐' %}

일단, 이 문서에 쓰인 [Type-Theme]는 반응형 웹디자인을 기반으로 소셜 네트워크 프로필 아이콘, 스타일 수식 지원 등 다양한 기능을 가지고 있다. 예를 들면, 
$$S_n = a \times \frac{1-r^n}{1-r}$$ 요로코롬.

하지만, 너무 심플하다. 진짜.. <code>Footer</code> 는 죄송하지만 만들다가 만거같아요.. ㅠㅠ

그래서 테마색상을 바꿨다. 그냥 처음엔 아무것도 모르고 <code>페이지 소스 보기</code>에서 바꾸고 싶은 태그를 찾아서 전부
<code>type-theme.scss</code>에다가 적어주다가,

{% include image.html url='/assets/img/2019-01-14/569091304.76.png' description='읽어보란건-제발-처-읽어보자.png' alt='깃허브 캡' %}

그러고 Footer 를 만들어야 하는데, 정-신-병 걸릴 뻔 했다. 3일만에 당장 이쁘게 반응형으로 설계 하려고 해서 그런가..

크게 Profile 영역과 Description 영역으로 나누고, 오른쪽과 왼쪽 끝에 걸쳐지면서, 화면 크기가 작으면 오른쪽 파트를 아래로 내려가면서 View 가 조정되도록
하고 싶었다.

```html
<div class="footer-col-half first">
    <a href="/">
        <img class="avatar" src="/assets/img/avatar.png" alt=""/>
    </a>
    <h1 class="name">Lee-Su-Gyun</h1>
    <a href="mailto:random_lee@naver.com">random_lee@naver.com</a>
</div>
<!--위: 왼쪽, 아래: 오른쪽-->
<div class="footer-col-half second">
    <p>HTML, CSS 등 1도 관심없지만 웹개발자, 디자이너 분들 존경합니다. 이 블로그는 내가 알고싶은것 내가 알아냈지만 언제든지 보면서 오지게 뽕 땡길 수 있는 것만 올립니다.</p>
    <p>Powered by <a href="https://jekyllrb.com/">Jekyll</a> Based on <a href="https://github.com/rohanchandra/type-theme">Type Theme</a></p>
</div>
```

처음엔 <code>inline-block</code> 두 개를 <code>width: 50%</code> 속성을 줘서 해결하려 했으나, 오른쪽 inline-block 은 항상 [아래로] 내려갔다.

inline-block 이 기본적으로 가지는 특성을 <code>width: 49%</code> 같은 꼼수로 해결 할 수 있었으나, 그러면 완벽히 오른쪽으로 안가자나 이런~~ㅅㅂ~~...
이딴건 1% 라도 용납할 수 없다.

그렇게 수 많은 시도 끝에 이 테마의 <code>_header.scss</code>를 인용했는데, 각 <code><div></code> 를 <code>width: 50%</code> 로 항상
일렬로 정렬되게 만들고, 오른쪽 자식의 <code><p></code> 를 <code>float: right</code> 로 항상 오른쪽으로 정렬되게 하는 것이다. 그렇게 하면,
Description 이 항상 50% 크기의 왼쪽에 밀리는 관계로 오른쪽에 붙어 있으면서, 화면이 줄어들어 오른쪽의 <code>min-width</code>를 유지시킬 수 없으면
바로 아래로 내려가 왼쪽에 정렬되게 된다.

그렇게 마무리 되는가 싶었지만...

[포스트에 사진을 넣는 트릭]을 사용하다가, <code>"/assets/img/logo.png"| relative_url</code> 이 Jekyll 에 의해 생성된 이 포스트의 절대경로를
찾아와서 <code>http://localhost:4000/2019/01/14/assets/img/logo.png</code> 이딴식으로 접근하는데, 아무리 헤메어 봐도 이해할 수 없었다.

하지만 기본적으로 URL 을 <code>root</code> 기반으로 찾아준다는 사실을 알고 있었기 때문에 그냥,
<code>url='/assets/img/2019-01-14/logo.png'</code> 이런식으로 적어주니 이미지도 잘 찾아오게 되었다.

[마지막으로 MD 작성 팁]

~~이제 진짜 끝~~

{% include image.html url='/assets/img/2019-01-14/screenshot2019-01-14am2.42.50.png' description='머 어쩌라고 난 그렇겐 못한다' alt='깃허브 캡' %}


[Type-Theme]: https://github.com/rohanchandra/type-theme
[아래로]: https://stackoverflow.com/questions/6871996/two-inline-block-width-50-elements-wrap-to-second-line
[포스트에 사진을 넣는 트릭]: https://blog.jaeyoon.io/2017/12/jekyll-image.html
[마지막으로 MD 작성 팁]: https://hashcode.co.kr/questions/1772/%EB%A7%88%ED%81%AC%EB%8B%A4%EC%9A%B4-%EB%AC%B8%EB%B2%95-%EC%9E%91%EC%84%B1-%ED%8C%81