---
layout: post
title: C++ 궁극의 pow()는 무엇일까
tags: C++ pow algorithm dp
feature-img: "assets/img/2019-01-14/c-introduction.jpg"
---

[백준 1463번](https://www.acmicpc.net/problem/1463) 1로 만들기. 이 문제에 관한 풀이가 아닌 이 문제의 풀이에서 생긴 의구점에 대한 포스팅이다.

>주의하세요, 이 글은 멍청이가 쓴 글입니다.

우선, 이 문제의 정해(?)는 Dynamic Programming 인 듯 한데, 난 아직 Dynamic Programming 을 입문조차 하지 못했다고 생각
한다. 실망했다면 죄송... 백트래킹을 이해할 때 매우 큰 도움을 받은 영상이 바로 아래에 보이는 것 같다.

<iframe width="640" height="360" src="https://www.youtube.com/embed/-xlSysSwG7w"  
 frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>  

```cpp
inline bool promising(int next, int nextCount) {
    if (next < 1)
        return false;

    if (mMin != numeric_limits<int>::max()) {
        int leastRemain = mMin - nextCount - 1;

        if (next > pow(3, leastRemain))
            return false;
    }

    return true;
}
```

내가 구현한 <code>promising()</code> 을 보면, 1 이상이고 찾은 최소값을 1회 이상 찾았을 때, <code>std::pow()</code>
를 부른다.

대회도 아니고, 뭐... 안되면 안되는 것이라는 생각에 적은 코드가 분명하기에, <code>pow()</code> 가 주는 께름칙한 느낌에 대하
생각해 볼 필요가 있을 것인데,

우선, <code>pow()</code> 는 [**상수시간**](https://www.acmicpc.net/board/view/27801)을 가진다고 한다. 하지만 동시에
실수연산에 기반하고 있기 때문에 **정수계산을 목적으로 하는 알고리즘에 최적화 되어있지 않다**.

정수의 거듭제곱을 어떻게 계산할지에 대한 가장 직관적인 대답은 요고겠지 아마.

```cpp
int ipow(int base, int exp)
{
    int result = 1;
    for (;;)
    {
        if (exp & 1)
            result *= base;
        exp >>= 1;
        if (!exp)
            break;
        base *= base;
    }

    return result;
}
```

$$O(exp)$$의 시간복잡도를 가지는데, 글쎄.. 최선은 아닐거같다는 생각이 강하게 든다.

우선적으로, $$5^{31}$$을 계산하는데 30번의 곱셈을 할 필요가 없다. [**Addition-Chain-Exponentiation**](https://en.wikipedia.org/wiki/Addition-chain_exponentiation)
을 이용하면, 다음과 같이 7번의 곱셈식으로 나타낼 수 있기 때문이다.

<p><script type="math/tex">5^2=5^1×5^1,\space 5^3=5^2×5^1,\space 5^6=5^3×5^3,\space 5^{12}=5^6×5^6,\space 5^{24}=5^{12}×5^{12},\space 5^{30}=5^{24}×5^{6},\space 5^{31}=5^{30}×5^1</script></p>

왠지, 저 최적화 된 식인 <code>30, 24, 12, 6, 3, 2</code> 의 값을 한번씩만 저장했다가, 끼워넣기 하면서 풀면 선형시간 내에 풀릴 것 같은 느낌이 든다.

하지만, 31을 30과 1, 30을 24와 6으로 나누어야 한다는 [결정론적 알고리즘](https://wkdtjsgur100.github.io/P-NP/)을 다항시간 내에 정의 할 수가 없다.

>On the other hand, the determination of a shortest addition chain is hard: no efficient optimal methods are currently known for arbitrary exponents, and the related problem of finding a shortest addition chain for a given set of exponents has been proven **NP-complete**.

>The problem of finding the shortest addition chain cannot be solved by dynamic programming, because it does **NOT** satisfy the assumption of **optimal substructure**.

임의의 지수에 대해 가장 짧은 Addition-Chain 을 구하는 **효과적인** 방법이 없고, 주어진 지수 집합으로부터 가장 짧은 체인을 찾아내는 것에 대한 **NP-완전성**이 이미 입증되었다.

결국, [**최적하위구조**](https://en.wikipedia.org/wiki/Optimal_substructure)를 가정 할 수 없기 때문에 *(하나의 하위구조를 결정하면,
보다 하위구조로부터 최적성을 종속받는다. 31제곱의 최적 Set 인 30이 결정되는 그 하부구조의 24와 6이 그 하부구조인 12와 3에서 3과 2의 많은 중복 노드를
가지기 때문..인가?)* , **동적 프로그래밍**에서 해당 문제를 최적화 시킬 수 없다.

즉, 처음부터 목적했던 "궁극의" <code>pow()</code> 는 없고, 상황에 따라 *Case-By-Case*로 접근 할 수 있겠다.

정수지수 연산에서는 **분할정복**을 통한 $$O(\log_2{exp})$$시간의 알고리즘을 생각할 수 있다.

$$X^n=\begin{cases}X^{n-1/2}X^{n-1/2}X &\text{X: odd} \\ X^{n/2}X^{n/2} &\text{X: even} \end{cases}$$

```cpp
long long int dividePow(int X, int e) {
    if (e == 0)
        return 1;

    if (e % 2)
        return dividePow(X, e - 1) * X;

    long long int half = dividePow(X, e / 2);

    return half * half;
}
```

좋아, 그럼 이제 테스트를 해보자!!

```cpp
clock_t begin, end;

begin = clock();
for (int i = 0; i < 100000; ++i) {
    int b = 2, e = 99;
    long long int ret = 1;
    while (e--)  {
        ret *= b;
    }
}
end = clock();

cout << "수행시간(while) : " << fixed << ((double)(end - begin) / CLOCKS_PER_SEC) << endl;

begin = clock();
for (int i = 0; i < 100000; ++i) {
    dividePow(2, 99);
}
end = clock();

cout << "수행시간(dividePow) : " << fixed << ((double)(end - begin) / CLOCKS_PER_SEC) << endl;

begin = clock();
for (int i = 0; i < 100000; ++i) {
    pow(2, 99);
}
end = clock();

cout << "수행시간(pow) : " << fixed << ((double)(end - begin) / CLOCKS_PER_SEC) << endl;
```

```$xslt
Apple LLVM version 10.0.0 (clang-1000.10.44.4)
수행시간(while) : 0.020929
수행시간(dividePow) : 0.008717
수행시간(pow) : 0.000174
```

??? 낚인건가? 저 정도 상수시간이라면 뭐가 문제 인 것이지?? 첫번째야 뭐 지수 값이 커지면 루프를 많이 돌테지, `std::pow()`의 선형
시간을 따라가는 놈이 아무도 없다. 그래서 실행 환경을 바꾸어 보았다.

```$xslt
gcc version 4.6.3
수행시간(while) : 0.025054
수행시간(dividePow) : 0.006691
수행시간(pow) : 0.009769

```

이번엔 `pow()` 에서 굉장한 시간을 보여주고 있다. 두 컴파일러가 어떤 차이를 보이는지에 대해선 안타깝게도 나는 어셈블리어를 읽을 줄 모른다..ㅎ

그런데 한가지 발견한 것은, `pow(2.0, 99.0)` 이런식으로, 실수형으로 적어주면 결과가 다음과 같이 나온다는 것이다.

```$xslt
gcc version 4.6.3
수행시간(while) : 0.025224
수행시간(dividePow) : 0.006218
수행시간(pow) : 0.000238
```

`pow()` 에서 이전과 40배 이상의 차이를 보여준다. 이 말은 결국, 구조적 최적화가 문제가 아니라 Type-Casting 비용에 의한 비용의 차이일 가능성이 높다는 말??
만약 그 비용을 최적화 할 수 있다면, 내가 보기에 `pow()` 함수의 선형시간은 매우 이상적이라고 본다. ([정수지수의 pow() 퍼포먼스 비교](https://baptiste-wicht.com/posts/2017/09/cpp11-performance-tip-when-to-use-std-pow.html))

`pow()` 를 정수연산에 사용할 때 주의 할 점은 속도 뿐 만 아니, [정밀도...](https://www.geeksforgeeks.org/power-function-cc/) 라고 하는것 같다.

또한, 실수지수 연산의 **유동적인 정밀도**에 기반한 <code>pow()</code> 함수의 [다양한 구현](https://sunhyeon.wordpress.com/2014/11/09/1584/)
이 있을 수 있다.