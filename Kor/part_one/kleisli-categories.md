# Bartosz Milewski's Programming Cafe

Category Theory, Haskell, Concurrency, C++

December 23, 2014

## Kleisli Categories

Posted by Bartosz Milewski under , ,

이전 시리즈, “프로그래머를 위한 카테고리: 크고 작은”에서 몇 가지 간단한 예시를 보여드렸습니다. 이번 에피소드에서는 좀 더 심층적인 예제를 함께 살펴보겠습니다. 시리즈에 처음 오시는 분들을 위해 목차를 안내해 드립니다.

## Composition of Logs

저희가 유형 모델링과 순수 함수를 카테고리로서 정의하는 방법을 살펴보았습니다. 또한, 카테고리 이론에서 부작용이나 순수하지 않은 함수를 모델링하는 방법도 언급했습니다. 이제 그러한 예시를 하나 살펴보겠습니다. 바로 함수가 실행 과정을 기록하거나 추적하는 경우입니다. 명령형 언어에서는 아마도 전역 상태를 수정하여 구현될 가능성이 높습니다.

```
string logger;

bool negate(bool b) {
     logger += "Not so! ";
     return !b;
}
```

알고 계시는 바와 같이, 이 함수는 순수 함수가 아니며, 메모이제이션된 버전에서는 로그를 생성하지 못할 것입니다. 이 함수는 부수 효과를 가지고 있습니다.

현대적인 프로그래밍에서는 최대한 전역 변수를 변경하는 것을 피하려고 노력합니다. 특히 동시성으로 인한 복잡성을 줄이기 위해서입니다. 또한, 이와 같은 코드는 라이브러리에 절대 포함되지 않도록 해야 합니다.

다행히도, 이 함수를 순수 함수로 만들 수 있습니다. 단순히 로그를 명시적으로 전달하면 됩니다. 정규 출력과 함께 업데이트된 로그를 포함하는 문자열 인자를 추가하여 그렇게 할 수 있습니다.

```
pair<bool, string> negate(bool b, string logger) {
     return make_pair(!b, logger + "Not so! ");
}
```

이 함수는 순수 함수이며, 부작용이 없으며, 동일한 인자를 전달할 때마다 항상 동일한 튜플을 반환합니다. 필요하다면 메모이제이션할 수 있습니다. 하지만 누적되는 로그의 특성상, 특정 호출이 발생하기 위한 모든 가능한 이력을 메모이제이션해야 합니다. 각 이력에 대해 별도의 메모 항목을 생성해야 합니다.

```
negate(true, "It was the best of times. ");
```

and

```
negate(true, "It was the worst of times. ");
```

and so on.

저희는 이 기능의 인터페이스가 도서관 함수로서 최적화된 형태가 아니라고 판단했습니다. 호출자는 반환되는 문자열을 무시할 수 있으므로 큰 부담은 아니지만, 문자열을 입력으로 전달해야 하는 불편함이 있습니다.

더욱 간결하게 동일한 작업을 수행할 수 있는 방법이 있을까요? 관심사를 분리하는 방법이 있을까요? 이 간단한 예제에서 negate 함수의 주된 목적은 하나의 불리언 값을 다른 불리언 값으로 변환하는 것입니다. 로깅은 부가적인 기능이며, 허용되는 사항입니다. 하지만 함수에서 생성된 메시지를 하나의 연속적인 로그로 묶어 처리하는 것은 별도의 관심사입니다. 저희는 함수가 문자열을 생성하도록 하는 것을 원하지만, 로깅 기능에서 벗어나도록 덜 부담스럽게 만드는 것이 바람입니다. 이에 대한 타협안은 다음과 같습니다:

```
pair<bool, string> negate(bool b) {
     return make_pair(!b, "Not so! ");
}
```

이 아이디어는 함수 호출 사이에 로그가 집계된다는 것입니다.

이를 어떻게 구현할 수 있는지 살펴보려면, 조금 더 현실적인 예제를 살펴보겠습니다. 문자열에서 문자열로 변환하는 하나의 함수가 있습니다. (대문자를 소문자로 변환하는) 함수입니다.

```
string toUpper(string s) {
    string result;
    int (*toupperp)(int) = &toupper; // toupper is overloaded
    transform(begin(s), end(s), back_inserter(result), toupperp);
    return result;
}
```

그리고 또, 문자열을 문자열 벡터로 나누는 기능으로, 공백을 기준으로 분리하는 기능입니다.

```
vector<string> toWords(string s) {
    return words(s);
}
```

실제 작업은 보조 기능어에서 이루어집니다.

```
vector<string> words(string s) {
    vector<string> result{""};
    for (auto i = begin(s); i != end(s); ++i)
    {
        if (isspace(*i))
            result.push_back("");
        else
            result.back() += *i;
    }
    return result;
}
```

저희는 toUpper 및 toWords 함수를 수정하여, 기존 반환 값 위에 메시지 문자열을 추가하도록 하겠습니다. 

이러한 함수들의 반환 값을 “장식”하여 처리할 것입니다. 이를 일반적인 방식으로 처리하기 위해, 첫 번째 요소는 임의의 타입 A의 값이고 두 번째 요소는 문자열인 튜플을 캡슐화하는 Writer 템플릿을 정의하도록 하겠습니다.

```
template<class A>
using Writer = pair<A, string>;
```

다음은 장식된 기능들입니다.

```
Writer<string> toUpper(string s) {
    string result;
    int (*toupperp)(int) = &toupper;
    transform(begin(s), end(s), back_inserter(result), toupperp);
    return make_pair(result, "toUpper ");
}

Writer<vector<string>> toWords(string s) {
    return make_pair(words(s), "toWords ");
}
```

저희는 이 두 함수를 하나의 더욱 화려한 함수로 조합하여 문자열을 대문자로 변환하고 단어로 나누는 동시에, 이러한 작업들을 기록하는 기능을 추가하고 싶습니다. 다음은 이를 수행하는 방법입니다.

```
Writer<vector<string>> process(string s) {
    auto p1 = toUpper(s);
    auto p2 = toWords(p1.first);
    return make_pair(p2.first, p1.second + p2.second);
}
```

저희는 목표를 달성했습니다. 로그의 집계가 개별 함수들의 문제로 더 이상 이어지지 않습니다. 각 함수들은 자체적으로 메시지를 생성하며, 이러한 메시지들은 외부적으로 합쳐져 하나의 큰 로그를 형성합니다.

이제 이 스타일로 작성된 전체 프로그램을 상상해 보십시오. 반복적이고 오류 발생 가능성이 높은 코드로 인해 혼란스러울 수 있습니다. 하지만 저희는 프로그래머입니다. 저희는 반복적인 코드를 해결하는 방법을 알고 있습니다. 바로 추상화를 통해 해결하는 것입니다. 

하지만 이것은 일반적인 추상화가 아닙니다. 저희는 함수 합성 자체를 추상화해야 합니다. 함수 합성 자체가 바로 범주론의 핵심이기 때문입니다. 따라서 더 많은 코드를 작성하기 전에, 범주론적인 관점에서 문제를 분석해 보겠습니다.

## The Writer Category

여러분의 기능을 더욱 풍성하게 만들어 추가적인 기능들을 활용하는 아이디어가 매우 효과적인 것으로 나타났습니다. 앞으로 더 많은 사례를 살펴보게 될 것입니다. 시작점은 저희의 기존 타입과 함수 카테고리입니다. 타입은 객체로 유지하되, 함수를 장식된 형태로 정의하여 모듈로 사용하게 됩니다.

예를 들어, 정수형(int)에서 불리언(bool)로 변환하는 ‘isEven’ 함수를 장식된 함수로 변환하여 모듈로 사용하는 경우를 생각해 볼 수 있습니다. 중요한 점은 이 모듈이 여전히 정수형(int)과 불리언(bool) 사이의 연결(arrow)으로 간주된다는 것입니다.

```
pair<bool, string> isEven(int n) {
     return make_pair(n % 2 == 0, "isEven ");
}
```

카테고리 이론의 법칙에 따라, 이 모리즘을 어떤 객체에서든 유도되는 다른 모리즘과 결합할 수 있어야 합니다. 특히, 저희가 이전에 정의한 부정 모리즘과 결합할 수 있어야 합니다.

```
pair<bool, string> negate(bool b) {
     return make_pair(!b, "Not so! ");
}
```

물론입니다. 두 개의 모필드를 일반 함수처럼 동일한 방식으로 구성할 수는 없습니다. 입력과 출력의 불일치로 인해 구성 방식은 다음과 같이 보일 것입니다.

```
pair<bool, string> isOdd(int n) {
    pair<bool, string> p1 = isEven(n);
    pair<bool, string> p2 = negate(p1.first);
    return make_pair(p2.first, p1.second + p2.second);
}
```

다음과 같이 구성하는 방법이 안내되어 드립니다.

먼저, 새로 구성 중인 이 범주에서 두 개의 모리즘을 결합하기 위해 다음 단계를 따르십시오.

- 첫 번째 모리즘에 해당하는 장식된 함수를 실행합니다.
- 결과 쌍의 첫 번째 구성 요소를 추출하여 두 번째 모리즘에 해당하는 장식된 함수에 전달합니다.
- 첫 번째 결과의 두 번째 구성 요소(문자열)와 두 번째 결과의 두 번째 구성 요소(문자열)를 연결합니다.
- 최종 결과의 첫 번째 구성 요소를 연결된 문자열과 함께 묶어 새로운 쌍을 반환합니다.

저희가 이 컴포지션을 C++에서 더 높은 수준의 함수로 추상화하기 위해서는 세 가지 객체에 해당하는 세 가지 타입으로 템플릿을 사용해야 합니다. 이 템플릿은 저희 규칙에 따라 구성 가능한 장식된 두 개의 함수를 받아 세 번째 장식된 함수를 반환해야 합니다.

```
template<class A, class B, class C>
function<Writer<C>(A)> compose(function<Writer<B>(A)> m1, 
                               function<Writer<C>(B)> m2)
{
    return [m1, m2](A x) {
        auto p1 = m1(x);
        auto p2 = m2(p1.first);
        return make_pair(p2.first, p1.second + p2.second);
    };
}
```

저희는 이제 이 새로운 템플릿을 사용하여 toUpper와 toWords 함수를 결합한 이전 예제를 다시 살펴보고 구현할 수 있습니다.

```
Writer<vector<string>> process(string s) {
   return compose<string, string, vector<string>>(toUpper, toWords)(s);
}
```

현재 컴포즈 템플릿으로 타입 변환 과정에서 여전히 많은 소음이 발생하고 있습니다. C++14 호환 컴파일러를 사용하고, 일반화된 람다 함수와 반환 타입 추론을 지원하는 경우 이러한 문제를 방지할 수 있습니다. (이 코드에 대한 공적은 에릭 니블러에게 있습니다.)

```
auto const compose = [](auto m1, auto m2) {
    return [m1, m2](auto x) {
        auto p1 = m1(x);
        auto p2 = m2(p1.first);
        return make_pair(p2.first, p1.second + p2.second);
    };
};
```

새로운 정의에 따라, 프로세스 구현은 다음과 같이 단순화됩니다.

```
Writer<vector<string>> process(string s){
   return compose(toUpper, toWords)(s);
}
```

아직 끝내지 않았습니다. 저희는 새로운 카테고리에서 구성(composition)을 정의했지만, 정체성 모필(identity morphisms)은 무엇일까요? 이는 저희의 일반적인 정체성 함수가 아닙니다! 타입 A에서 타입 A로 돌아가는 모필이어야 하며, 다음과 같은 장식된 함수 형태를 갖추고 있습니다:

```
Writer<A> identity(A);
```

구성 요소로서 그들은 반드시 특정 방식으로 행동해야 합니다. 저희의 구성 정의를 살펴보시면, 정체성 모듈은 인자를 변경하지 않고 단순히 빈 문자열만 로그에 추가해야 한다는 것을 확인하실 수 있습니다.

```
template<class A>
Writer<A> identity(A x) {
    return make_pair(x, "");
}
```

저희가 정의한 범주가 분명히 유효한 범주임을 쉽게 이해하실 수 있을 겁니다. 특히, 저희의 구성은 당연히 교환 법칙을 따릅니다. 각 쌍의 첫 번째 구성 요소가 어떻게 진행되는지 살펴보시면, 이는 일반적인 함수 구성이며, 교환 법칙을 따르는 것입니다. 두 번째 구성 요소는 연결되고, 연결 연산 역시 교환 법칙을 따릅니다.

눈썰미 있는 독자님께서 보시는 바와 같이, 이 구조를 어떤 모나이드에도 쉽게 일반화할 수 있습니다. compose 안에는 mappend를, identity 안에는 mempty를 사용하면 됩니다 (덧셈(+)과 빈 문자열("") 대신). 저희는 문자열만 기록하는 데 국한할 이유가 없습니다. 좋은 라이브러리 작성자는 라이브러리가 작동하기 위한 최소한의 제약 조건을 파악해야 합니다. 여기서 기록 라이브러리의 유일한 요구 사항은 로그가 모나이드적 속성을 갖는 것뿐입니다.

## Writer in Haskell

마찬가지로 하스케일에서는 좀 더 간결하게 표현할 수 있으며, 컴파일러로부터 훨씬 더 많은 도움을 받을 수 있습니다. 먼저 Writer 타입을 정의해 보겠습니다.

```
type Writer a = (a, String)
```

여기서는 간단히 타입 별칭을 정의하고 있습니다. C++의 `typedef` (또는 `using` 키워드)와 유사하며, `Writer` 타입은 `a`라는 타입 변수로 파라미터화됩니다. 이 타입은 `a`와 `String`의 쌍과 동일합니다. 튜플(pairs)의 구문은 매우 간단합니다. 괄호 안에 두 개의 항목을 쉼표로 구분하여 작성합니다.

저희의 모피즘(morphisms)은 임의의 타입에서 `Writer` 타입으로의 함수입니다.

```
a -> Writer b
```

저희는 이 연산을 재미있는 ‘물고기’ 연산이라고 선언하게 됩니다.

```
(>=>) :: (a -> Writer b) -> (b -> Writer c) -> (a -> Writer c)
```

이 함수는 두 개의 인수를 받으며, 각각 독립적인 함수입니다. 첫 번째 인수는 (a->Writer b) 형태이고, 두 번째 인수는 (b->Writer c) 형태이며, 결과는 (a->Writer c) 형태입니다. 고객님께 안내해 드리는 말씀이니, 편하게 참고해 주시기 바랍니다.

이 infix 연산자의 정의입니다. 두 개의 인자, 즉 ‘fishy symbol’의 양쪽에 나타나는 m1과 m2를 참조해 주십시오.

```
m1 >=> m2 = \x -> 
    let (y, s1) = m1 x
        (z, s2) = m2 y
    in (z, s1 ++ s2)
```

결과는 하나의 인자 x를 받는 람다 함수입니다. 람다는 백슬래시로 표현되며, 그리스 문자 람다(λ)의 다리가 잘린 모습으로 생각하시면 됩니다.

또한, let 표현을 사용하면 보조 변수를 선언할 수 있습니다. 여기서는 m1 함수의 호출 결과가 (y, s1)이라는 두 개의 변수로 패턴 매칭되고, m2 함수의 호출 결과는 첫 번째 패턴의 인자 y를 사용하여 (z, s2)로 패턴 매칭됩니다.

"하스케일에서는 C++에서 사용했던 접근자 대신 튜플을 패턴 매칭하는 것이 일반적입니다. 그 외에는 두 구현 간에 비교적 간단한 대응 관계가 있습니다."

전체적인 `let` 표현의 값은 해당 `in` 절에서 지정됩니다. 여기서는 첫 번째 요소는 `z`이고, 두 번째 요소는 문자열 `s1`과 `s2`의 연결 결과입니다.

또한, 저희 카테고리에 대한 항등 사상(identity morphism)을 정의하겠습니다. 다만, 나중에 명확해질 이유 때문에 이를 ‘return’이라고 부르겠습니다.

```
return :: a -> Writer a
return x = (x, "")
```

완성도를 위해, 장식된 함수 upCase와 toWords의 하스켈 버전들을 살펴보겠습니다. 고객님께 안내하는 것처럼 정중한 존댓말로 표현하겠습니다.

```
upCase :: String -> Writer String
upCase s = (map toUpper s, "upCase ")
```

```
toWords :: String -> Writer [String]
toWords s = (words s, "toWords ")
```

이 함수 `map`은 C++의 `transform` 함수와 동일하게 작동합니다. 이 함수는 문자 변환 함수 `toUpper`를 문자열 `s`에 적용합니다. 보조 함수 `words`는 표준 Prelude 라이브러리에 정의되어 있습니다.

마지막으로, 두 함수의 조합은 fish 연산자를 통해 이루어집니다.

```
process :: String -> Writer [String]
process = upCase >=> toWords
```

## Kleisli Categories

혹시 추측하셨을 수도 있겠지만, 제가 이 범주를 갑자기 만들어낸 것은 아닙니다. 이는 ‘클라이슬리 범주’라는 용어에 해당하며, 이는 모나드(monad)를 기반으로 한 범주입니다. 아직 모나드에 대해 자세히 논의할 준비가 되어 있지는 않지만, 이들이 할 수 있는 가능성을 조금이나마 보여드리고 싶었습니다. 저희의 제한적인 목적을 위해, 클라이슬리 범주는 기본적으로 프로그래밍 언어의 타입을 객체로 사용합니다. 타입 A에서 타입 B로 이어지는 함수는 특정 장식(embellishment)을 사용하여 유도된 타입으로 이동합니다. 각 클라이슬리 범주는 이러한 모리즘(morphisms)을 구성하는 자체적인 방법을 정의하며, 그 구성에 대한 항등 모리즘(identity morphisms)도 정의합니다. (나중에 이 부정확한 용어 ‘장식(embellishment)’가 ‘엔도폰터(endofunctor)’라는 개념과 일치한다는 것을 보게 될 것입니다.)

이 글에서 제가 사용한 특정 모나드는 ‘라이터 모나드’라고 불리며, 함수 실행을 기록하거나 추적하는 데 사용됩니다. 또한 순수 계산에 효과를 포함하는 보다 일반적인 메커니즘의 예시이기도 합니다. 이전에 집합의 범주에서 프로그래밍 언어의 타입과 함수를 모델링할 수 있다는 것을 보셨을 겁니다. 여기서는 단순히 함수의 출력을 다른 함수의 입력으로 전달하는 것 이상의 기능을 수행하는 모니즘을 표현하는 약간 다른 범주를 사용하여 이 모델을 확장했습니다. 이제 추가적인 자유도를 활용할 수 있습니다. 바로 모니즘 자체의 구성입니다. 실제로 이것이 명령형 언어에서 전통적으로 측면 효과를 사용하여 구현되는 프로그램에 간단한 의미론적 의미를 부여할 수 있게 해주는 자유도입니다.

## Challenge

A function that is not defined for all possible values of its argument is called a partial function. It’s not really a function in the mathematical sense, so it doesn’t fit the standard categorical mold. It can, however, be represented by a function that returns an embellished type optional:

```
template<class A> class optional {
    bool _isValid;
    A    _value;
public:
    optional()    : _isValid(false) {}
    optional(A v) : _isValid(true), _value(v) {}
    bool isValid() const { return _isValid; }
    A value() const { return _value; }
};
```

As an example, here’s the implementation of the embellished function safe\_root:

```
optional<double> safe_root(double x) {
    if (x >= 0) return optional<double>{sqrt(x)};
    else return optional<double>{};
}
```

Here’s the challenge:

- Construct the Kleisli category for partial functions (define composition and identity).
- Implement the embellished function safe\_reciprocal that returns a valid reciprocal of its argument, if it’s different from zero.
- Compose safe\_root and safe\_reciprocal to implement safe\_root\_reciprocal that calculates sqrt(1/x) whenever possible.

## Acknowledgments

I’m grateful to Eric Niebler for reading the draft and providing the clever implementation of compose that uses advanced features of C++14 to drive type inference. I was able to cut the whole section of old fashioned template magic that did the same thing using type traits. Good riddance! I’m also grateful to Gershom Bazerman for useful comments that helped me clarify some important points.

Next: Products and Coproducts.

