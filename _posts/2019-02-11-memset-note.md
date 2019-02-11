---
layout: post
title: memset, 이런 실수 하지말자
tags: C++ std memset fill
feature-img: "assets/img/2019-01-14/c-introduction.jpg"
---

`string.h`의 `memset()` 함수를 통한 배열 원소의 초기화는 **매우 제한적**이다.

우선, `memset()`은 [다음](http://www.cplusplus.com/reference/cstring/memset/)과 같이 정의된다.

>`void * memset ( void * ptr, int value, size_t num );`  
 Fill block of memory
 Sets the first num bytes of the block of memory pointed by ptr to the specified value (interpreted as an unsigned char).

* 첫 번째 인자로 받은 주소로부터 두 번째 인자로 받은 값을 **바이트 단위로** 반복작성한다.
* 두 번째 인자는 `int`형으로 받지만, 작성되는 데이터는 `unsigned char`로 해석된다.

---

즉, 각 바이트 블럭 당 할당되는 값은 **0 ~ 255(`0xFF`)** 사이의 값만 유의미하게 적용된다는 말이다.

```cpp
#include <iostream>
#include <cstring>

using namespace std;

int main() {
    int result = 0;

    memset(&result, 300, 1);

    cout << result;

    return 0;
}
```
```
44
```

위의 프로그램에서 `memset`의 두 번째 인자는 `unsigned char`를 넘어서는 값이다.

이 때, 300 즉, `0x12C`의 [Little-endian에 의거한 바이트 오더](https://genesis8.tistory.com/37)의 첫번째 블럭의 값은 44(`0x2C`)인데,
이 값이 `result`의 메모리 위치에 1 바이트만큼 작성되면서, 출력이 44가 되는 것이다.

그렇다면, 코드를 다음과 같이 수정하면 어떤 결과가 나올까?

```cpp
memset(&result, 300, sizeof(int));
```
```
741092396
```

`0x2C2C2C2C`. 눈치가 빠르다면 이 결과에 쉽게 수긍할 것이다.

---

`memset`이라는 함수는 절대로 입력된 `int`값을 작성해주는 값이 아니다. 절대 주의 할 것. 

초기화하고자 하는 것이 배열이거나, 데이터가 정수형이 아니라면 더욱이 잘 살펴보아야 할 것이다.

나같이 많은 사람들이 `memset`의 레퍼런스를 잘못이해하고 있었을 경우가 많을 것이라고 생각되는 Behavior 는 다음과 같은 것일 것이다.
* `memset(arr, 0, sizeof(int) * arrSize);`
* `memset(arr, -1, sizeof(int) * arrSize);`

이 흔한 코드를 보고, *"아, 모든 배열 원소를 해당 값으로 초기화해주는 함수이구나!"*라고 말이다.

하지만, 0은 첫번째 바이트 블록의 값이 당연히 `0x0`이기 때문에 몇 번을 반복하든, 어느 블록을 참조하든 결과가 당연히 0일 뿐이다.

-1의 경우는 값이 `unsigned char`로 해석되면, 1의 2의 보수를 나타내는 비트 패턴이 나타내는 값이 할당되는데, 그 값의 바이트 블록은
255(`0xFF`)이다. 해당 패턴이 반복되는 메모리 역시, 어느 블록을 참조하든 결과를 `int`로 나타내면 -1(`0xFFFFFFFF`)이 나오는 것이다.

---

결론, 배열을 초기화 어떤 값으로 초기화 할 때는, C++스러운 Template 함수인 `std::fill`을 사용하자.

[참고](http://www.cplusplus.com/reference/algorithm/fill/?kw=fill)