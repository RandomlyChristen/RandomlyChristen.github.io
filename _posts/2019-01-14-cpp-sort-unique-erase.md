---
layout: post
title: C++ sort, unique, erase 로 뭘 할 수 있었지??
tags: C++ sort unique erase stl
feature-img: "assets/img/2019-01-14/c-introduction.jpg"
---

[백준 2750번] 수 정렬하기. 입력받은 정수를 정렬, 중복제거 하여 출력.

간단한 구현 문제인데, 과거에 <code>unique()</code> 와 <code>erase()</code> 를 함께 사용했던 기억이 나서, 이 문제를 요걸로 풀어보기로 했다.

일단 [unique]의 구현부터 살펴보니,

{% highlight cpp %}
template<class ForwardIt>
ForwardIt unique(ForwardIt first, ForwardIt last)
{
    if (first == last)
        return last;
 
    ForwardIt result = first;
    while (++first != last) {
        if (!(*result == *first) && ++result != first) {
            *result = std::move(*first);
        }
    }
    return ++result;
}
{% endhighlight %}

요놈 <code>if (!(*result == *first) && ++result != first)</code> 이 이해가 안되는 부분이었다... 똑같이 <code>first</code>
와 <code>result</code> 를 증가시키면, 도대체 언제 저 조건이 참이라는 것이지??

AND 양쪽을 바꿔 보니 이해가 되었다!!

AND 양쪽을 바꾸면 문서에 정의된대로 함수가 작동하지 않았다.. 이래서 증감연산자 10극혐.

함수는 다음 변형으로 이해할 수 있다.

{% highlight cpp %}
template<class ForwardIt>
ForwardIt myUnique(ForwardIt first, ForwardIt last)
{
    if (first == last)
        return last;

    ForwardIt result = first;
    while (++first != last) {
        if (!(*result == *first)) {
            if (++result != first) {
                *result = std::move(*first);
            }
        }
        cout << (result == first) << endl;
    }
    return ++result;
}
{% endhighlight %}

AND 연산자의 첫번째 인수가 거짓이면 두번째 인수는 실행하지 않는다. 여기서 "실행" 이라는 애매뽕짝한 표현을 사용한 것은
증감연산자가 [시퀸스 포인트]에 종속적인 Behavior 를 띄기 때문.

따라서 <code>first</code> 기준, 기존 동일함을 유지했던 값과 다르고, <code>++result</code> 와 다른 포인트의 Iterator 일 때,
<code>move()</code> 를 통해 그 값을 ["이동"] 시킨다. (해당 함수는 상황에 따라 [무브 시맨틱]에 기반한 구현이 되어 있다.)

즉, <code>first</code> 가 기존과 "unique" 한 지를 검사하고, result 에 unique 한 값을 새로 써나가는 꼬라지다.

하지만 <code>result</code> 의 위치가 바뀔 때 마다, 검사의 기준이 바뀐다 ㅋ (대신 선형 시간). 따라서, 인접하지 않으면서 동일한 두 값에 대해서
이 녀석은 unique 하다고 판별, 다음 result 위치에 기록해버린다. 이래서, 구현 예시에는 <code>sort</code> 와 함께 쓰고 있다.
정렬된 컨테이너에선 항상 중복되는 값은 서로 인접한다는게 보장 되기 때문.

또한, 표준 문서는 이 함수의 return 값 뒤로 이어지는 Iterator 가 Container 에서 구체적으로 어떤 값을 Pointing 해야하는지
명시하고 있지 않는 것 같다.

{% highlight cpp %}
#include <iostream>
#include <algorithm>
#include <vector>
#include <string>
#include <cctype>
 
int main() 
{
    // remove duplicate elements
    std::vector<int> v{1,2,3,1,2,3,3,4,5,4,5,6,7};
    std::sort(v.begin(), v.end()); // 1 1 2 2 3 3 3 4 4 5 5 6 7 
    auto last = std::unique(v.begin(), v.end());
    // v now holds {1 2 3 4 5 6 7 x x x x x x}, where 'x' is indeterminate
    v.erase(last, v.end()); 
    for (int i : v)
      std::cout << i << " ";
    std::cout << "\n";
}
{% endhighlight %}

```$xslt
1 2 3 4 5 6 7
```

어째되었든, 정렬된 컨테이너의 해당 범위는 완벽하게 unique 하며, 함수는 마지막으로 기록한 지점의 다음 Iterator 를 리턴하는데,
이 뒤로 <code>end()</code> 가 리턴하는 Iterator 와 함께 <code>erase()</code> 를 부른다.

배열 기반 컨테이너의 <code>erase()</code> 같은 놈들이 어떤 짓을 하는지는 당해보면 알지.

{% highlight cpp %}
#include <iostream>
#include <vector>

using namespace std;

int main() {
    vector<int> vec = {1, 2, 3, 4, 5, 6, 7};
    vec.erase(vec.begin() + 2, vec.end() - 2);

    for (int i = 0; i < vec.size(); ++i)
        cout << vec[i] << " ";

    cout << '\t';

    for (int i = 0; i < vec.capacity(); ++i)
        cout << vec[i] << " ";

    return 0;
}
{% endhighlight %}

```$xslt
1 2 6 7     1 2 6 7 5 6 7
```

삭제는 논리적으로만 작동한다. 삭제되는 만큼 뒤에서 값을 복사해 오기때문에 반복적으로 값을 수정해야하는 구현에서 배열기반 컨테이너는 쓸 생각은
무조건 2순위.

참고로, ~~내가 당해봐서 아는데~~ 모든 원소를 삭제하고 <code>size()</code> 가 <code>0</code> 을 반환한다고 해도,
<code>front()</code> 나 <code>end()</code> 는 확신할 수 없는 값을 가르키고 있을지 모른다.



[백준 2750번]: https://www.acmicpc.net/problem/2750
[unique]: https://en.cppreference.com/w/cpp/algorithm/unique
["이동"]: https://en.cppreference.com/w/cpp/utility/move
[시퀸스 포인트]: https://dojang.io/mod/page/view.php?id=757
[무브 시맨틱]: http://blog.naver.com/PostView.nhn?blogId=poiusky5&logNo=220546171485&parentCategoryNo=&categoryNo=&viewDate=&isShowPopularPosts=false&from=postView