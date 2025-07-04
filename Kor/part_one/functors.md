# Bartosz Milewski's Programming Cafe

Category Theory, Haskell, Concurrency, C++

January 20, 2015

## Functors

Posted by Bartosz Milewski under ,

This is part of Categories for Programmers. Previously: Simple Algebraic Data Types. See the Table of Contents.

반복해서 말씀드려 죄송하지만, 함수르(functor)에 대해 말씀드리는 내용은 다음과 같습니다. 함수르는 매우 단순하지만 강력한 개념입니다. 범주론 또한 이러한 단순하면서도 강력한 개념들로 가득 차 있습니다. 함수르는 두 개의 범주, C와 D 사이의 매핑(mapping)입니다. 즉, 범주 사이의 관계를 나타내는 함수라고 할 수 있습니다. 범주 C의 객체 a가 있다면, 그 이미지를 범주 D에서 F a (괄호 없이)로 표기합니다. 하지만 범주는 객체만 있는 것이 아니라, 객체와 그 연결을 나타내는 모피즘(morphism)도 포함합니다. 함수르 또한 모피즘을 매핑합니다. 즉, 모피즘을 무작정 매핑하는 것이 아니라, 연결을 보존합니다. 예를 들어, 범주 C에서 모피즘 f가 객체 a에서 객체 b로 연결된다면,

```
f :: a -> b
```

D, F f 이미지의 F 이미지가 A 이미지와 B 이미지의 연결을 형성할 것입니다.

```
F f :: F a -> F b
```

(This is a mixture of mathematical and Haskell notation that hopefully makes sense by now. I won’t use parentheses when applying functors to objects or morphisms.)  As you can see, a functor preserves the structure of a category: what’s connected in one category will be connected in the other category. But there’s something more to the structure of a category: there’s also the composition of morphisms. If h is a composition of f and g:

```
h = g . f
```

저희는 F 하위 이미지의 결과가 f와 g의 이미지들을 조합한 것으로 보이기를 바랍니다.

```
F h = F g . F f
```

마지막으로, C의 모든 항등 사상(identity morphism)은 D의 항등 사상으로 매핑됩니다.

```
F ida = idF a
```

여기, ida는 객체 a에서 정체성을 나타내고, idF a는 F a에서 정체성을 나타냅니다. 이러한 조건들은 함수보다 훨씬 더 엄격하게 제한합니다. 함수형(Functor)은 카테고리의 구조를 보존해야 합니다. 카테고리를 객체들이 모니즘(morphism)들의 네트워크로 연결된 집합으로 생각한다면, 함수형은 이 직물에 찢어진 부분을 만들 수 없습니다. 객체를 뭉치게 하거나 여러 모니즘을 하나로 붙일 수는 있지만, 절대로 분리할 수 없습니다. 이러한 찢어짐 금지 조건은 미적분학에서 알고 있을 수 있는 연속성 조건과 유사합니다. 이 점에서 함수형은 “연속적”이라고 할 수 있지만, 함수형에 대해 더 엄격한 연속성의 개념도 존재합니다. 함수와 마찬가지로, 함수형은 축소와 포함을 모두 수행할 수 있습니다. 특히 소스 카테고리가 대상 카테고리보다 훨씬 작을 때 포함 측면이 더 두드러집니다. 극단적인 경우 소스는 단일 객체와 단일 모니즘을 가진 단일 객체 카테고리일 수 있습니다. 단일 객체 카테체에서 다른 카테고리로의 함수형은 단순히 해당 카테고리 내의 객체 하나를 선택합니다. 이는 대상 집합에서 선택된 요소와 유사합니다. 가장 극단적인 축소 함수형은 상수 함수형 Δc라고 합니다. 이 함수형은 소스 카테고리의 모든 객체를 대상 카테고리 내의 선택된 객체 c로 매핑합니다. 또한 소스 카테고리의 모든 모니즘을 정체 모니즘 idc로 매핑합니다. 이는 마치 블랙홀처럼 모든 것을 하나의 특이점으로 압축합니다. 우리는 한계(limit)와 반한계(colimit)를 논의할 때 이 함수형을 더 많이 살펴볼 것입니다.

## Functors in Programming

자, 이제 프로그래밍에 대해 현실적인 논의를 시작해 보겠습니다. 저희는 유형(types)과 함수(functions)의 범주를 가지고 있습니다. 자기 범주 안으로 매핑하는 함수, 즉 엔도범주(endofunctor)에 대해 이야기할 수 있습니다. 유형의 범주에서 엔도범주는 무엇일까요? 우선, 유형을 유형으로 매핑합니다. 이러한 매핑의 예는 이미 본 적이 있을 것입니다. 다른 유형으로 매개변수화된 유형 정의를 말씀드리는 것입니다. 몇 가지 예를 살펴보겠습니다.

### The Maybe Functor

"Maybe"의 정의는 타입 a에서 타입 Maybe a로의 매핑입니다.

```
data Maybe a = Nothing | Just a
```

다음은 중요한 부분입니다. ‘Maybe’ 자체는 타입이 아니라 타입 생성자입니다. ‘Int’나 ‘Bool’과 같은 타입 인수를 제공해야 ‘Maybe’를 실제 타입으로 만들 수 있습니다. 아무런 인수가 없는 ‘Maybe’는 타입에 대한 함수를 나타냅니다. 하지만 ‘Maybe’를 함수형으로 변환할 수 있을까요? (이후 저는 프로그래밍 맥락에서 ‘functor’를 언급할 때, 거의 항상 ‘endofunctor’를 의미한다고 말씀드릴 수 있습니다.) ‘functor’는 단순히 객체(여기서는 타입)의 매핑뿐만 아니라 모피즘(여기서는 함수)의 매핑도 포함합니다. a에서 b로의 모든 함수에 대해:

```
f :: a -> b
```

저희는 Maybe a에서 Maybe b로 함수를 생성하고자 합니다. 이러한 함수를 정의하기 위해, Maybe의 두 생성자에 해당하는 두 가지 경우를 고려해야 합니다. Nothing 경우에는 간단하게 Nothing을 반환하면 됩니다. 그리고 인수가 Just인 경우에는 해당 내용에 함수 f를 적용합니다. 따라서 Maybe 아래에서 f의 이미지는 다음과 같습니다:

```
f’ :: Maybe a -> Maybe b
f’ Nothing = Nothing
f’ (Just x) = Just (f x)
```

덧붙여 말씀드리자면, 하스케일에서는 변수 이름에 작은 따옴표를 사용할 수 있는데, 이러한 경우에 매우 유용합니다. 하스케일에서는 모리즘 매핑 부분은 fmap이라는 고차 함수로 구현합니다. Maybe의 경우 다음과 같은 시그니처를 가집니다:

```
fmap :: (a -> b) -> (Maybe a -> Maybe b)
```

저희는 종종 fmap이 함수를 띄우는 역할을 한다는 말씀을 많이 합니다. 띄워진 함수는 Maybe 값에 작용합니다. 일반적으로, 커링 때문에 이 시그니처는 두 가지 방식으로 해석될 수 있습니다. 첫째, 하나의 인수를 받는 함수로 해석될 수 있습니다. 이 함수는 자기 자신(a->b)가 되는 함수이며, Maybe a -> Maybe b를 반환합니다. 둘째, 두 개의 인수를 받는 함수로 해석될 수 있으며, Maybe b를 반환합니다.

```
fmap :: (a -> b) -> Maybe a -> Maybe b
```

이전 논의 내용을 바탕으로, Maybe에 대한 fmap 구현은 다음과 같습니다.

```
fmap _ Nothing = Nothing
fmap f (Just x) = Just (f x)
```

마이비 타입 생성자와 fmap 함수가 퓨니ctor로서 작용한다는 것을 보여주기 위해서는, fmap이 항등함수와 합성함수를 보존한다는 것을 증명해야 합니다. 이러한 것들을 “퓨니ctor 법칙”이라고 부르지만, 이는 단순히 범주의 구조를 보존한다는 것을 보장하는 것입니다.

### Equational Reasoning

기능법의 공식을 증명하기 위해, 저는 등식 추론 방법을 사용하겠습니다. 이는 하스케일에서 흔히 사용되는 증명 기법입니다. 하스케일 함수가 등식으로 정의된다는 점을 활용하는 방법입니다. 즉, 좌변과 우변은 항상 같음을 이용할 수 있습니다. 변수 이름을 변경하여 충돌을 피하면서, 한쪽을 다른 쪽으로 대체할 수 있습니다. 이는 함수를 인라인하는 것과 마찬가지이거나, 반대로 표현식을 함수로 재구성하는 것과 같습니다. 예를 들어 항등 함수를 살펴보겠습니다.

```
id x = x
```

저희가 살펴보시면, 예를 들어 어떤 표현식 안에 `id y`가 나타나는 것을 보실 수 있습니다. 이 경우, `y`로 대체하는 방식으로 처리할 수 있습니다. (이를 ‘인라인’이라고 합니다). 또한, `id`가 표현식에 적용된 경우, 예를 들어 `id (y + 2)`와 같이 처리할 수 있으며, 이 경우 해당 표현식 자체인 `y + 2`로 대체할 수 있습니다. 이러한 대체는 양방향으로 적용 가능합니다. 즉, 어떤 표현식 `e`를 `id e`로 대체할 수도 있고, 반대로 `id e`를 `e`로 대체할 수도 있습니다. (이를 ‘리팩토링’이라고 합니다). 함수가 패턴 매칭으로 정의된 경우, 각 하위 정의를 독립적으로 사용할 수 있습니다. 예를 들어, 위에서 설명드린 `fmap`의 정의를 살펴보시면, `fmap f Nothing`을 `Nothing`으로 대체하거나, 반대로 `Nothing`을 `fmap f Nothing`으로 대체할 수 있습니다. 실제로 어떻게 작동하는지 살펴보겠습니다. 먼저, ‘항등성 보존’이라는 개념을 살펴보겠습니다.

```
fmap id = id
```

다음 두 가지 경우를 고려해 보십시오. “아무것도 하지 않음”과 “단순히 진행” 두 가지입니다. 먼저 첫 번째 경우를 설명드리겠습니다. (Haskell 유사 코드를 사용하여 왼쪽 변수를 오른쪽 변수로 변환하는 방법을 보여드리겠습니다.)

```
fmap id Nothing 
= { definition of fmap }
  Nothing 
= { definition of id }
  id Nothing
```

마지막 단계에서 저는 id의 정의를 거꾸로 사용했습니다. “Nothing”이라는 표현을 id Nothing으로 대체했죠. 실제로는 같은 표현이 중간에 나타날 때까지 “양 끝에서 촛불을 태우는 것”과 같은 방식으로 증명을 진행합니다. 여기서도 “Nothing”이 바로 그런 표현이었습니다. 두 번째 경우 또한 간단합니다.

```
fmap id (Just x) 
= { definition of fmap }
  Just (id x) 
= { definition of id }
  Just x
= { definition of id }
  id (Just x)
```

자, 이제 fmap이 합성(composition)을 보존한다는 것을 보여드리겠습니다.

```
fmap (g . f) = fmap g . fmap f
```

First the Nothing case:

```
fmap (g . f) Nothing 
= { definition of fmap }
  Nothing 
= { definition of fmap }
  fmap g Nothing
= { definition of fmap }
  fmap g (fmap f Nothing)
```

And then the Just case:

```
fmap (g . f) (Just x)
= { definition of fmap }
  Just ((g . f) x)
= { definition of composition }
  Just (g (f x))
= { definition of fmap }
  fmap g (Just (f x))
= { definition of fmap }
  fmap g (fmap f (Just x))
= { definition of composition }
  (fmap g . fmap f) (Just x)
```

다음 코드를 살펴보시면 이해가 더 쉬울 것입니다. C++ 스타일의 “함수” 중 부작용이 있는 경우, 균형 추론은 작동하지 않는다는 점을 강조 드리고 싶습니다.

```c++
int square(int x) {
    return x * x;
}

int counter() {
    static int c = 0;
    return c++;
}

double y = square(counter());
```

등식 추론을 사용하시면 제곱 함수를 즉시 계산하여 다음과 같은 결과를 얻으실 수 있습니다.

```c++
double y = counter() * counter();
```

이 변환은 유효하지 않으며, 결과가 동일하지 않을 것입니다. 그럼에도 불구하고, C++ 컴파일러가 매크로로 제곱 함수를 구현할 경우, 매우 좋지 않은 결과를 초래할 수 있습니다.

### Optional

함수형터(Functors)는 하스케일에서 쉽게 표현될 수 있지만, 제네릭 프로그래밍과 고차 함수를 지원하는 모든 언어에서 정의될 수 있습니다. C++의 Maybe에 해당하는 템플릿 타입인 `optional`을 살펴보겠습니다. 여기에는 실제 구현의 개요가 포함되어 있으며, 실제 구현은 다양한 방식으로 전달되는 방식, 복사 의미론, 그리고 C++의 특징적인 자원 관리 문제 등을 다루는 훨씬 복잡한 내용입니다.

```c++
template<class T>
class optional {
    bool _isValid; // the tag
    T    _v;
public:
    optional()    : _isValid(false) {}         // Nothing
    optional(T x) : _isValid(true) , _v(x) {}  // Just
    bool isValid() const { return _isValid; }
    T val() const { return _v; }
};
```

이 템플릿은 함수형의 정의 중 하나인 타입 매핑을 제공합니다. 어떤 타입 T를 새로운 타입 optional<\T>로 매핑합니다. 함수의 경우, 이 템플릿의 동작 방식을 정의해 드리겠습니다.

```c++
template<class A, class B>
std::function<optional<B>(optional<A>)> 
fmap(std::function<B(A)> f) 
{
    return [f](optional<A> opt) {
        if (!opt.isValid())
            return optional<B>{};
        else
            return optional<B>{ f(opt.val()) };
    };
}
```

이것은 고차 함수로, 다른 함수를 인자로 받아 새로운 함수를 반환하는 형태입니다. 다음은 이 함수의 비고차 함수 버전입니다.

```c++
template<class A, class B>
optional<B> fmap(std::function<B(A)> f, optional<A> opt) {
    if (!opt.isValid())
        return optional<B>{};
    else
        return optional<B>{ f(opt.val()) };
}
```

선택의 어려움 때문에 C++에서 람다 패턴을 추상화하는 것이 문제가 될 수 있습니다. 템플릿 메서드 방식으로 fmap을 구성하는 것도 그중 하나입니다. 템플릿 가상 함수를 사용할 수 없다는 점이 특히 어려움을 더합니다. 람다 함수를 인터페이스로 상속할 것인지, 아니면 커리드 또는 언커리드 프리 템플릿 함수로 사용할 것인지 고려해야 합니다. C++ 컴파일러가 누락된 타입을 정확하게 추론할 수 있을지, 아니면 명시적으로 지정해야 할지 생각해 보십시오. 예를 들어, 입력 함수 f가 int를 bool로 취하는 경우, 컴파일러는 g의 타입을 어떻게 파악할까요?

```c++
auto g = fmap(f);
```

앞으로 여러 개의 함수형 변환(functor)이 등장할 가능성이 있으니, 특히 그렇다면 다음과 같은 사항을 염두에 두시기 바랍니다. (더 자세한 내용은 곧 설명드리겠습니다.)

### Typeclasses

자, 하스케일이 퓨니ctor를 추상화하는 방법을 알아보겠습니다. 타입 클래스 메커니즘을 사용합니다. 타입 클래스는 공통 인터페이스를 지원하는 타입들의 패밀리를 정의합니다. 예를 들어, 동등성을 지원하는 객체들의 클래스는 다음과 같이 정의됩니다:

```
class Eq a where
    (==) :: a -> a -> Bool
```

이 정의는 type a가 Eq 클래스를 구현하는 경우, 두 개의 type a 인수를 받는 (\==) 연산자를 지원한다는 것을 명시합니다. Haskell에게 특정 타입이 Eq임을 알려주려면, 해당 타입을 Eq 클래스의 인스턴스로 선언하고 (\==)의 구현을 제공해야 합니다. 예를 들어, 두 개의 Float 타입으로 구성된 2차원 점(product type)의 정의를 고려해 보면 다음과 같습니다.

```
data Point = Pt Float Float
```

점의 균등성을 정의할 수 있습니다.

```
instance Eq Point where
    (Pt x y) == (Pt x' y') = x == x' && y == y'
```

여기서 저는 (\==) 연산자를 infix 위치, 즉 두 패턴 (Pt x y)와 (Pt x' y') 사이에 사용했습니다. 함수의 몸체는 단일 등호 (=)를 따릅니다. Point가 Eq 인스턴스로 선언되면, Point를 직접 등호로 비교할 수 있습니다. C++나 Java와 달리, Point를 정의할 때 Eq 클래스(또는 인터페이스)를 명시할 필요는 없습니다. client 코드에서 나중에 정의할 수 있습니다. Typeclasses는 Haskell에서 함수(및 연산자)를 오버로딩하는 유일한 메커니즘입니다. 우리는 fmap을 다양한 functors에 대해 오버로딩하는 데 필요할 것입니다. 하지만 한 가지 복잡한 점이 있습니다. functor는 유형이 아닌 유형의 매핑으로 정의되며, 유형 생성자입니다. Eq와 같이 유형의 가족이 아닌 유형 생성자의 가족이 필요합니다. 다행히 Haskell 유형 클래스는 유형과 함께 유형 생성자도 작동합니다. 따라서 Functor 클래스의 정의는 다음과 같습니다:

```
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

Functor로 정의되려면, 특정 타입 서명(type signature)을 가진 `fmap` 함수가 존재해야 합니다. 작은따옴표 `f`는 `a`와 `b`와 같은 타입 변수이며, 이는 타입 생성자(type constructor)를 나타낸다는 것을 컴파일러가 사용법을 통해 추론하기 때문입니다. 예를 들어 `f a`와 `f b`와 같이 다른 타입에 적용하는 것을 보면 알 수 있습니다. 따라서 Functor 인스턴스를 선언할 때는 `Maybe`와 마찬가지로 타입 생성자를 지정해야 합니다.

```
instance Functor Maybe where
    fmap _ Nothing = Nothing
    fmap f (Just x) = Just (f x)
```

덧붙여 말씀드리자면, Functor 클래스와 함께 Maybe를 포함한 여러 간단한 데이터 타입에 대한 인스턴스 정의는 표준 Prelude 라이브러리의 일부입니다.

### Functor in C++

C++에서도 동일한 접근 방식을 시도해 보시겠습니까? 타입 생성자는 템플릿 클래스와 대응하므로, 유사하게 fmap을 템플릿 템플릿 매개변수 F를 사용하여 매개변수화할 수 있습니다. 이 방식의 문법은 다음과 같습니다:

```c++
template<template<class> F, class A, class B>
F<B> fmap(std::function<B(A)>, F<A>);
```

다양한 함수형에 맞게 이 템플릿을 특화하는 데 저희가 기회를 갖고 싶습니다. 하지만 C++에서는 템플릿 함수 부분 특화를 금지하고 있습니다. 즉, 다음과 같은 코드는 사용할 수 없습니다:

```c++
template<class A, class B>
optional<B> fmap<optional>(std::function<B(A)> f, optional<A> opt)
```

대신, 우리는 uncurried fmap의 원래 정의로 돌아가 function overloading를 사용해야 합니다.

```c++
template<class A, class B>
optional<B> fmap(std::function<B(A)> f, optional<A> opt) 
{
    if (!opt.isValid())
        return optional<B>{};
    else
        return optional<B>{ f(opt.val()) };
}
```

이 정의는 작동하지만, 두 번째 인수가 오버로드를 선택하기 때문에만 그렇습니다. 이 정의는 fmap의 보다 일반적인 정의를 완전히 무시합니다.

### The List Functor

함수형 프로그래밍에서 함수자의 역할을 좀 더 직관적으로 이해하기 위해서는 더 많은 예제를 살펴봐야 합니다. 또 다른 타입으로 매개변수화되는 모든 타입은 함수 후보가 될 수 있습니다. 일반적인 컨테이너는 저장하는 요소의 타입을 통해 매개변수화되므로, 매우 간단한 컨테이너인 리스트를 살펴보겠습니다.

```
data List a = Nil | Cons a (List a)
```

저희는 List라는 타입 생성자를 가지고 있습니다. 이는 어떤 타입 a를 List a 타입으로 매핑하는 역할을 합니다. List가 퓨니ctor임을 증명하기 위해서는 함수의 업라이팅을 정의해야 합니다. 즉, a->b 형태의 함수가 주어졌을 때, List a -> List b 형태의 함수를 정의하는 것입니다.

```
fmap :: (a -> b) -> (List a -> List b)
```

리스트를 처리하는 함수를 설계할 때, 두 가지 경우를 고려해야 합니다. 첫째, Nil 리스트의 경우 단순하게 Nil을 반환하면 됩니다. 빈 리스트는 처리할 내용이 거의 없기 때문입니다. 둘째, Cons 리스트의 경우 조금 까다로운데, 재귀 호출을 포함하기 때문입니다. 잠시 한 발짝 물러서서 우리가 무엇을 하려고 하는지 다시 한번 생각해 보겠습니다. 리스트의 요소 a, 함수 f (a를 b로 변환하는 함수), 그리고 b로 이루어진 리스트를 생성하는 것이 목표입니다. 명백한 방법은 함수 f를 사용하여 리스트의 각 요소를 a에서 b로 변환하는 것입니다. 실제로 어떻게 해야 할까요? (비어 있지 않은) 리스트는 머리와 꼬리의 Cons로 정의되었으므로, 머리에 함수 f를 적용하고 꼬리에 (업된) 함수 f를 적용합니다. 이는 재귀적 정의입니다. 왜냐하면 우리는 함수 f를 함수 f의 업된 버전으로 정의하기 때문입니다.

```
fmap f (Cons x t) = Cons (f x) (fmap f t)
```

오른쪽 손쪽에서 fmap f는 우리가 정의하는 리스트보다 짧은 리스트에 적용됩니다. 즉, 리스트의 꼬리 부분에 적용되는 것입니다. 따라서 리스트는 점점 짧아지는 리스트로 재귀적으로 진행되며, 결국 빈 리스트(Nil)에 도달하게 됩니다. 앞서 결정했듯이, Nil에 fmap f가 적용될 경우에도 Nil이 반환되므로 재귀가 종료됩니다. 최종 결과를 얻기 위해 새로운 헤드(f x)와 새로운 꼬리(fmap f t)를 Cons 생성자를 사용하여 결합합니다. 모든 것을 종합하면, 리스트 퓨니ctor의 인스턴스 선언은 다음과 같습니다.

```
instance Functor List where
    fmap _ Nil = Nil
    fmap f (Cons x t) = Cons (f x) (fmap f t)
```

C++에 더 익숙하시다면, std::vector를 고려해 보시는 것을 추천드립니다. std::vector는 가장 일반적인 C++ 컨테이너로 간주될 수 있으며, fmap의 구현은 std::transform의 얇은 캡슐화에 불과합니다.

```c++
template<class A, class B>
std::vector<B> fmap(std::function<B(A)> f, std::vector<A> v)
{
    std::vector<B> w;
    std::transform( std::begin(v)
                  , std::end(v)
                  , std::back_inserter(w)
                  , f);
    return w;
}
```

다음과 같이 사용하실 수 있습니다. 예를 들어, 숫자 시퀀스의 요소들을 제곱하는 데 활용하실 수 있습니다.

```c++
std::vector<int> v{ 1, 2, 3, 4 };
auto w = fmap([](int i) { return i*i; }, v);
std::copy( std::begin(w)
         , std::end(w)
         , std::ostream_iterator(std::cout, ", "));
```

C++ 컨테이너의 대부분은 `std::transform`와 같은 더 기본적인 기능인 `fmap`의 친척과 유사하게, 반복자를 구현하기 때문에 함수형 프로그래밍의 특징을 잘 보여주는 경우가 많습니다. 하지만 반복자와 임시 객체로 인해 함수의 단순성이 잃어버리는 경우가 종종 있습니다. (위의 `fmap` 구현 참조). 다행히 새로운 C++ 범위 라이브러리는 범위의 함수형적 특성을 더욱 뚜렷하게 보여줍니다.

### The Reader Functor

이제 혹시 어떤 직관을 얻으셨을 수도 있겠네요. 예를 들어, 퓨니ctor가 어떤 종류의 컨테이너와 유사하다는 것을요. 좀 더 자세히 설명드리자면, 함수 타입에 대한 심도 있는 논의는 앞으로 진행될 예정이고, 카테고리적 관점에서의 완전한 설명은 추후에 다루겠지만, 프로그래머로서 우리는 이미 함수 타입에 대해 어느 정도 이해하고 있습니다. 하스케일(Haskell)에서 함수 타입은 화살표 타입 생성자(->)를 사용하여 구성하는데, 이 생성자는 두 가지 타입을 받습니다. 바로 함수의 인자 타입과 반환 타입입니다. 이미 infix 형태로 보셨을 수도 있지만, 괄호를 사용하여 prefix 형태로도 사용할 수 있습니다.

```
(->) a b
```

정규 함수와 마찬가지로, 여러 개의 인수를 받는 타입 함수도 부분 적용이 가능합니다. 화살표에 단 하나의 타입 인수를 제공하더라도, 여전히 다른 하나를 요구합니다. 그래서 다음과 같습니다:

```
(->) a
```

이것은 타입 생성자(type constructor)입니다. 완전한 타입 a->b를 만들기 위해서는 또 다른 타입 b가 필요합니다. 현재 상태에서는 a를 매개변수로 하여 전체 타입 생성자 패밀리를 정의합니다. 이것이 또한 함수르(functor) 패밀리인지 확인해 보겠습니다. 두 개의 타입 매개변수를 다루는 것은 다소 혼란스러울 수 있으므로 이름을 바꾸어 봅시다. 이전 함수르 정의와 일관되게, 인자 타입 r과 결과 타입 a라고 부르겠습니다. 따라서 이 타입 생성자는 어떤 타입 a를 받아 r->a로 매핑합니다. 이것이 함수르임을 보여주기 위해, a->b 함수를 r->a를 취하고 r->b를 반환하는 함수로 들어 올리려고 합니다. 이것은 타입 생성자 (->) r이 각각 a와 b에 작용하여 형성된 타입입니다. 다음은 fmap이 이 경우에 적용된 타입 시그니처입니다.

```
fmap :: (a -> b) -> (r -> a) -> (r -> b)
```

다음 퍼즐을 풀어야 합니다: 함수 f(a→b)와 함수 g(r→a)가 주어졌을 때, r→b 형태의 함수를 만들어야 합니다. 두 함수를 합성하는 유일한 방법이 있으며, 그 결과가 우리가 필요로 하는 정확한 형태입니다. 그래서 저희의 fmap 구현을 소개합니다.

```
instance Functor ((->) r) where
    fmap f g = f . g
```

정말 간편하게 작동합니다! 괄호 표기법을 선호하신다면, 구성이 접두사 형태로 다시 작성될 수 있다는 점을 인지하여 이 정의를 더욱 간결하게 줄일 수 있습니다.

```
fmap f g = (.) f g
```

and the arguments can be omitted to yield a direct equality of two functions:

```
fmap = (.)
```

이 조합은 타입 생성자(->) r와 위에 제시된 fmap 구현을 합친 것으로, 이를 리더 함수터라고 합니다.

## Functors as Containers

저희는 프로그래밍 언어에서 일반적인 용도로 사용되는 컨테이너, 또는 특정 타입에 따라 값을 담는 객체 등의 예시를 살펴본 바 있습니다. 그런데 독자 함수(Reader functor)는 다소 이외의 경우로 보입니다. 왜냐하면 저희는 함수를 데이터를 생각하지 않기 때문입니다. 하지만 순수 함수는 메모이제이션될 수 있으며, 함수 실행을 테이블 조회로 바꿀 수 있다는 것을 확인했습니다. 테이블은 데이터를 구성하는 요소입니다. 반대로, 하스켈의 늦은 평가(lazy evaluation) 특성상, 리스트와 같은 전통적인 컨테이너가 실제로는 함수로 구현될 수도 있습니다. 예를 들어, 무한히 계속 생성되는 자연수 리스트를 생각해 보십시오. 이 리스트는 다음과 같이 간결하게 정의될 수 있습니다:

```
nats :: [Integer]
nats = [1..]
```

첫 번째 줄에서는 괄호 안에 2개의 사각형을 사용하여 Haskell의 리스트 내장 타입 생성자를 나타냅니다. 두 번째 줄에서는 괄호 안에 리스트 리터럴을 생성하는 방법을 보여줍니다. 무한한 리스트는 메모리에 저장될 수 없다는 점을 명심해야 합니다. 컴파일러는 필요할 때마다 정수를 생성하는 함수로 이를 구현합니다. Haskell은 데이터와 코스의 구분을 흐릿하게 만듭니다. 리스트는 함수로 간주될 수 있고, 함수는 인자를 결과로 매핑하는 테이블로 간주될 수 있습니다. 특히 함수의 도메인이 유한하고 너무 크지 않다면 이러한 접근 방식이 실용적일 수 있습니다. 하지만 `strlen`을 테이블 조회로 구현하는 것은 무한히 많은 문자열이 있기 때문에 실용적이지 않습니다. 프로그래머로서 우리는 무한대를 좋아하지 않지만, 범주론에서는 아침 식사로 무한대를 먹는 법을 배웁니다. 모든 문자열의 집합이든, 과거, 현재, 미래의 우주의 모든 가능한 상태의 모음이든, 우리는 그것을 처리할 수 있습니다! 따라서 저는 내장 타입 생성자(endofunctor에 의해 생성된 타입에 대한 값 또는 값들을 포함하는 객체)를 값 또는 값들을 포함하는 것으로 생각하는 것을 좋아합니다. 이러한 값들이 물리적으로 존재하지 않더라도 말이죠. C++ std::future와 같은 내장 타입 생성자의 예가 있습니다. 이 생성자는 특정 시점에 값을 포함할 수 있지만 보장되는 것은 아닙니다. 그리고 값을 액세스하려면 다른 스레드가 실행을 완료할 때까지 기다리는 것이 필요할 수 있습니다. Haskell IO 객체와 같은 또 다른 예는 사용자 입력 또는 모니터에 "Hello World!"가 표시된 우주의 미래 버전과 같은 값을 포함할 수 있습니다. 이러한 해석에 따르면, 내장 타입 생성자는 파라미터화된 타입의 값 또는 값들을 포함할 수 있는 것입니다. 또는 이러한 값들을 생성하는 레시피를 포함할 수도 있습니다. 우리는 값에 액세스할 수 있는지 여부에 대해 전혀 걱정하지 않습니다. 이는 완전히 선택 사항이며, 내장 타입 생성자의 범위를 벗어납니다. 우리는 단지 이러한 값들을 함수를 사용하여 조작할 수 있는지에만 관심이 있습니다. 값이 액세스될 수 있다면, 이러한 조작의 결과를 볼 수 있어야 합니다. 그렇지 않다면, 우리가 중요하게 생각하는 것은 조작이 올바르게 결합되고, 항등 함수와 함께 조작이 수행되지 않는다는 것입니다. 내장 타입 생성자 내부의 값에 액세스할 수 있다는 점을 얼마나 신경 쓰지 않는 것을 보여 드리기 위해, 이 생성자는 인수를 완전히 무시하는 타입 생성자를 보여 드리겠습니다.
```
data Const c a = Const c
```

Const 타입 생성자는 c와 a 두 가지 타입을 받습니다. 화살표 생성자와 마찬가지로, 이 생성자를 부분 적용하여 퓨니ctor를 만드는 방식으로 진행하겠습니다. (이) 데이터 생성자(또는 Const)는 c 타입의 단 하나의 값만 받습니다. a에 의존하지 않습니다. 이 타입 생성자에 대한 fmap의 타입은 다음과 같습니다.

```
fmap :: (a -> b) -> Const c a -> Const c b
```

함수형이 타입 인자를 무시하기 때문에, fmap의 구현은 함수 인자를 무시해도 자유롭습니다. 즉, 함수에 적용할 대상이 없기 때문입니다.

```
instance Functor (Const c) where
    fmap _ (Const v) = Const v
```

이 부분은 C++을 사용하면 조금 더 명확하게 설명될 수 있습니다. 컴파일 시간에 결정되는 타입 인자와 런타임에 결정되는 값과의 차이를 더욱 명확하게 이해할 수 있을 것입니다.

```c++
template<class C, class A>
struct Const {
    Const(C v) : _v(v) {}
    C _v;
};
```

C++ 구현체인 fmap 또한 함수 인자를 무시하고, Const 인자를 재형질환하는 것과 유사하게 작동합니다. 값 자체는 변경하지 않습니다.

```c++
template<class C, class A, class B>
Const<C, B> fmap(std::function<B(A)> f, Const<C, A> c) {
    return Const<C, B>{c._v};
}
```

이 구성 요소의 기묘함에도 불구하고, ‘Const’ 함수자는 많은 경우에 중요한 역할을 합니다. 범주론적으로는 제가 이전에 언급한 Δc 함수자의 특별한 경우로, 블랙홀의 엔도-함수체 케이스입니다. 앞으로 이 구성 요소에 대해 더 자세히 살펴보게 되실 겁니다.

## Functor Composition

카테고리 사이의 함수형(functor)이 합성된다는 것을 믿기 어렵지 않습니다. 이는 집합 사이의 함수 합성과 마찬가지입니다. 두 함수형이 객체에 작용할 때 합성하는 것은 각 함수의 객체 매핑의 합성과 같습니다. 또한 모리즘(morphism)에 작용할 때도 마찬가지입니다. 두 함수형을 거쳐 지나가면 항등 함수(identity morphism)는 항등 함수로, 모리즘의 합성은 모리즘의 합성과 같습니다. 크게 복잡한 것은 없습니다. 특히, 내부 함수(endofunctor)를 합성하는 것은 매우 쉽습니다. 혹시 maybeTail 함수를 기억하시나요? 하스켈(Haskell)의 리스트 내장 구현을 사용하여 다시 작성하겠습니다.

```
maybeTail :: [a] -> Maybe [a]
maybeTail [] = Nothing
maybeTail (x:xs) = Just xs
```

비어있는 리스트 생성자를 Nil이라고 불렀던 것을 이제 빈 괄호 배열 [ ]로 대체했습니다. Cons 생성자는 infix 연산자 : (콜론)으로 대체되었습니다. maybeTail의 결과는 Maybe와 []라는 두 개의 함수형을 합성한 타입입니다. 이 각 함수형은 자체적인 fmap 버전을 갖추고 있지만, 합성된 내용에 함수 f를 적용하고 싶을 때가 있습니다. 즉, Maybe 리스트의 내용을 대상으로 함수 f를 적용하고 싶다는 것입니다. 우리는 두 개의 함수형 층을 통과해야 합니다. 우리는 외부 Maybe를 통과하기 위해 fmap을 사용할 수 있지만, f는 리스트를 대상으로 작동하지 않으므로 Maybe 안에 f를 직접 전달할 수 없습니다. 따라서 내부 리스트에 (fmap f)를 전달하여 작동하도록 해야 합니다. 예를 들어, 정수 Maybe 리스트의 요소들을 제곱하는 방법을 살펴보겠습니다.

```
square x = x * x

mis :: Maybe [Int]
mis = Just [1, 2, 3]

mis2 = fmap (fmap square) mis
```

컴파일러는 타입 분석을 통해, 외부 fmap의 경우 Maybe 인스턴스 구현을 사용하고, 내부의 경우에는 리스트 함수형 구현을 사용하도록 판단할 것입니다. 위 코드가 다음과 같이 다시 작성될 수 있다는 점이 즉각적으로 명확하게 드러나지 않을 수 있습니다.

```
mis2 = (fmap . fmap) square mis
```

하지만 fmap은 단순히 하나의 인수를 처리하는 함수로 간주될 수 있다는 점을 기억해 주시기 바랍니다.

```
fmap :: (a -> b) -> (f a -> f b)
```

저희 경우에는 (fmap . fmap)에서 두 번째 fmap이 다음과 같은 인자를 받습니다:

```
square :: Int -> Int
```

and returns a function of the type:

```
[Int] -> [Int]
```

The first fmap then takes that function and returns a function:

```
Maybe [Int] -> Maybe [Int]
```

마지막으로, 그 함수는 mis에 적용됩니다. 따라서 두 개의 함수자를 합성하면, 해당 함수자들의 fmap을 합성한 함수자가 됩니다. 범주론으로 돌아가 보면, 함수자 합성은 분명히 결합 법칙이 성립합니다(객체의 매핑은 결합 법칙이 성립하고, 모니즘의 매핑도 결합 법칙이 성립합니다). 또한 모든 범주에는 단순한 항등 함수자가 존재합니다. 이 함수자는 모든 객체를 자기 자신으로 매핑하고, 모든 모니즘을 자기 자신으로 매핑합니다. 따라서 함수자는 어떤 범주에서 모니즘과 동일한 모든 속성을 갖습니다. 그렇다면 그 범주는 무엇일까요? 객체가 범주이고 모니즘이 함수자인 범주여야 합니다. 즉, 범주의 범주입니다. 하지만 모든 범주의 범주는 자기 자신을 포함해야 하며, 우리는 모든 집합의 집합이 불가능했던 동일한 종류의 역설에 빠지게 될 것입니다. 그러나 작은 범주들의 집합인 Cat(이 집합은 매우 크므로 구성원일 수 없습니다)이라는 작은 범주들의 집합이 존재합니다. 작은 범주란 집합보다 더 큰 것이 아닌 집합을 이루는 객체를 갖는 범주를 의미합니다. 참고로 범주론에서는 무한하고 셀 수 없이 많은 집합도 “작은” 것으로 간주됩니다. 저는 이러한 점들을 언급하고 싶었습니다. 왜냐하면 우리는 추상화 수준이 여러 단계에서 동일한 구조가 반복적으로 나타나는 것을 인식할 수 있다는 사실이 매우 놀랍기 때문입니다. 우리는 나중에 함수자가 범주를 형성한다는 것을 알게 될 것입니다.

## Challenges

- Can we turn the Maybe type constructor into a functor by defining:
fmap \_ \_ = Nothing
which ignores both of its arguments? (Hint: Check the functor laws.)
- Prove functor laws for the reader functor. Hint: it’s really simple.
- Implement the reader functor in your second favorite language (the first being Haskell, of course).
- Prove the functor laws for the list functor. Assume that the laws are true for the tail part of the list you’re applying it to (in other words, use induction).

## Acknowledgments

Gershom Bazerman is kind enough to keep reviewing these posts. I’m grateful for his patience and insight.

Next: Functoriality

