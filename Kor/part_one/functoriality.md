# Bartosz Milewski's Programming Cafe

Category Theory, Haskell, Concurrency, C++

February 3, 2015

## Functoriality

Posted by Bartosz Milewski under , , , ,

This is part 8 of Categories for Programmers. Previously: Functors. See the Table of Contents.

이제 함수형(functor)이 무엇인지, 그리고 몇 가지 예제를 살펴보셨으니, 더 큰 함수형을 더 작은 함수형에서 어떻게 구성할 수 있는지 알아보겠습니다. 특히, 카테고리 내 객체 간의 매핑(mapping)에 해당하는 타입 생성자(type constructors)가 어떻게 함수형(functor)으로 확장될 수 있는지 살펴보는 것이 흥미롭습니다.

## Bifunctors

함석은 카테고리(Cat) 내의 모니즘(morphism)이므로, 모니즘 – 특히 함수 –에 대한 많은 직관들이 함석에도 적용됩니다. 예를 들어, 두 개의 인자를 받는 함수와 마찬가지로, 두 개의 인자를 받는 함석 또는 이인함석을 가질 수 있습니다. 객체에 대해서는 이인함석이 각 카테고리 C와 D에서 나온 모든 객체 쌍을 하나의 객체 E로 매핑한다는 것을 의미합니다. 이는 C×D의 카르테시안 곱에서 매핑을 한다는 것을 말하는 것과 같습니다.

이 부분은 비교적 간단하게 이해하실 수 있을 겁니다. 하지만 functoriality의 핵심은 bifunctor가 모피어를 매핑해야 한다는 점입니다. 이번 경우에는 C에서 온 모피어와 D에서 온 모피어를 하나로 묶어 E에서 온 모피어로 변환하는 작업을 수행해야 합니다.

다시 한번 말씀드리자면, C×D라는 곱셈 범주에서 쌍선형은 단일 선형으로 정의됩니다. 카테토리곱의 카테토리곱에서 선형을 정의할 때, 한 쌍의 객체에서 다른 쌍의 객체로 향하는 쌍의 선형으로 정의합니다. 이러한 쌍의 선형은 직관적인 방식으로 합성될 수 있습니다.

```
(f, g) ∘ (f', g') = (f ∘ f', g ∘ g')
```

저희는 이 구성이 연관성을 가지며, 항등 성질을 갖습니다. 즉, 항등 사상 쌍 (id, id)를 포함하고 있습니다. 따라서 카테고리 순서곱은 실제로 카테고리입니다.

좀 더 쉽게 이해하기 위해, bifunctors는 각 인수를 따로 functors로 생각하는 방식으로 이해할 수 있습니다. 즉, functors에서 associativity와 identity preservation과 같은 functorial 법칙을 bifunctors로 번역하는 대신, 각 인수에 대해 따로 검증하는 것으로 충분합니다. 하지만 일반적으로 각 인수에 대해 별개의 functoriality를 확인하는 것만으로는 joint functoriality를 증명하는 데는 충분하지 않습니다. joint functoriality가 실패하는 범주를 premonoidal 범주라고 합니다.

Let’s define a bifunctor in Haskell. In this case all three categories are the same: the category of Haskell types. A bifunctor is a type constructor that takes two type arguments. Here’s the definition of the Bifunctor typeclass taken directly from the library Control.Bifunctor:

자, 하스케일에서 bifunctor를 정의해 보겠습니다. 이 경우, 세 가지 범주 모두 동일합니다: 하스케일 타입의 범주입니다. bifunctor는 두 개의 타입 인수를 받는 타입 생성자입니다. 다음은 라이브러리 Control.Bifunctor에서 직접 가져온 Bifunctor 타입 클래스 정의입니다:

```
class Bifunctor f where
    bimap :: (a -> c) -> (b -> d) -> f a b -> f c d
    bimap g h = first g . second h
    first :: (a -> c) -> f a b -> f c b
    first g = bimap g id
    second :: (b -> d) -> f a b -> f a d
    second = bimap id
```

변수 f는 bifunctor를 나타냅니다. 모든 타입 시그니처에서 두 개의 타입 인수에 적용된다는 것을 확인할 수 있습니다. 첫 번째 타입 시그니처는 bimap를 정의하며, 이는 두 개의 함수를 동시에 매핑하는 것입니다. 결과는 bifunctor의 타입 생성자를 통해 생성된 타입에 작동하는 lifted 함수, (f a b -> f c d)입니다. bimap는 first와 second에 대한 기본 구현이 있으며, 이는 각 인수에 functoriality가 존재하면 충분하다는 것을 보여줍니다. (이전 언급했듯이, 두 개의 맵이 교환되지 않으면 항상 작동하는 것은 아니며, 즉 first g . second h가 second h . first g와 같지 않을 수 있습니다.)

<!-- image -->

bimap

첫 번째와 두 번째 타입 시그니처는 각각 f 함수가 첫 번째와 두 번째 인수에 대해 함수적 특성을 증명하는 두 가지 fmaps입니다.

저희는 **비맵(bimap)**을 활용하여 두 가지 유형 정의에 대한 기본 구현을 제공합니다.

비스펀ctor를 선언할 때, bimap을 구현하고 기본값을 사용하거나, first와 second 모두 구현하고 bimap의 기본값을 사용하는 선택지가 있습니다. (물론, 세 가지 모두 구현할 수도 있지만, 이와 같이 관련성을 유지하는 것을 담당하셔야 합니다.)

## Product and Coproduct Bifunctors

중요한 예시로, 카테고리적 제품(categorical product)이 있습니다. 이는 보편적 구성(universal construction)에 의해 정의되는 두 객체의 곱입니다. 만약 이 제품이 어떤 두 객체에 대해 존재한다면, 그 객체에서 제품으로의 매핑은 이중 함수식(bifunctorial)입니다. 이는 일반적으로, 특히 하스케일(Haskell)에서 사실입니다.

```
instance Bifunctor (,) where
    bimap f g (x, y) = (f x, g y)
```

선택의 여지가 많지 않습니다. `bimap`은 단순히 첫 번째 함수를 첫 번째 컴포넌트에 적용하고, 두 번째 함수를 두 번째 컴포넌트에 적용합니다. 타입 정보만 주어지면 코드가 거의 자동으로 작성됩니다.

```
bimap :: (a -> c) -> (b -> d) -> (a, b) -> (c, d)
```

여기서 bifunctor의 기능은 두 종류의 유형을 쌍으로 만드는 것입니다. 예를 들어: **pair**

```
(,) a b = (a, b)
```

양성성(duality)에 의해, 어떤 범주 내의 모든 쌍의 대상에 대해 정의된 곱생성(coproduct)은 또한 비기 함수식(bifunctor)이 됩니다. 하스케일(Haskell)에서는 Either 타입 생성자가 비기 함수식의 인스턴스라는 점이 이를 보여줍니다.

```
instance Bifunctor Either where
    bimap f _ (Left x)  = Left (f x)
    bimap _ g (Right y) = Right (g y)
```

This code also writes itself.

자, 잠시 지난번에 저희가 이야기했던 모나이달 카테고리에 대해 다시 한번 생각해 보겠습니다. 모나이달 카테고리는 객체에 작용하는 이진 연산자를 정의하며, 동시에 단위 객체도 함께 갖습니다. 집합(Set)은 카티산 곱(cartesian product)에 대해 모나이달 카테고리이며, 단일(singleton) 집합이 단위로 사용됩니다. 또한, 이산 합(disjoint union)에 대해서도 모나이달 카테고리를 형성하며, 빈 집합(empty set)이 단위로 사용됩니다. 제가 언급하지 않았던 중요한 점은 모나이달 카테고리의 한 가지 요구 사항은 이진 연산자가 바이펀ctor(bifunctor)라는 것입니다. 이는 매우 중요한 요구 사항입니다. 저희는 모나이달 제품(monoidal product)이 카테고리의 구조와 호환되어야 하는데, 이는 모리즘(morphism)에 의해 정의됩니다. 이제 모나이달 카테고리의 완전한 정의에 한 걸음 더 다가서게 됩니다(자연 변환(naturality)에 대해 더 배워야 합니다).

## Functorial Algebraic Data Types

저희는 여러 가지 파라미터화된 데이터 타입들이 함수형(functor)으로 나타나는 사례들을 확인해 보았습니다. 저희는 이 데이터 타입들에 대해 `fmap`을 정의할 수 있었습니다. 복잡한 데이터 타입들은 더 간단한 데이터 타입들로 구성됩니다. 특히, 대수적 데이터 타입(Algebraic Data Types, ADTs)은 합(sums)과 곱(products)를 사용하여 생성됩니다. 저희는 얼마 전에 합과 곱이 함수형이라는 것을 확인했습니다. 또한, 함수형은 합성(composition)될 수 있다는 것을 알고 있습니다. 따라서 ADT의 기본적인 구성 요소들이 함수형이라는 것을 보여준다면, 파라미터화된 ADT 역시 함수형이라는 것을 알 수 있을 것입니다.

먼저, 파라미터화된 대수적 데이터 타입의 구성 요소는 무엇인가요? 첫째, 함수형의 타입 파라미터에 의존하지 않는 아이템들이 있습니다. 예를 들어, Maybe의 Nothing이나 List의 Nil과 같습니다. 이러한 아이템들은 Const 함수와 동일합니다. Const 함수는 타입 파라미터를 무시합니다. (실제로는 두 번째 타입 파라미터이며, 저희에게 중요한 첫 번째 파라미터는 상수 상태로 유지됩니다.)

그러면 Just in Maybe와 같이 타입 매개변수 자체를 단순히 캡슐화하는 요소들도 있습니다. 이는 정체성 공분류(identity functor)와 같습니다. 저는 이전에 정체성 모피(identity morphism)를 Cat에서 언급했지만, Haskell에서 그 정의를 제공하지 않았습니다. 여기 그 정의입니다:

```
data Identity a = Identity a
```

```
instance Functor Identity where
    fmap f (Identity x) = Identity (f x)
```

정체성은 항상 단 하나의 (불변하는) 값, 즉 type a의 값을 저장하는 가장 단순한 컨테이너로 생각할 수 있습니다.

이 모든 다른 대수적 데이터 구조는 제품과 합을 사용하여 이 두 기본 요소로부터 구성됩니다. 

새로운 정보를 바탕으로, Maybe 타입 생성자에 대해 다시 한번 자세히 살펴보겠습니다.

```
data Maybe a = Nothing | Just a
```

저희는 두 가지 유형의 합을 제공하며, 현재는 이 합이 함수형(functorial)임을 알고 있습니다. 첫 번째 부분은 어떤 것도 `Const()`를 통해 작용하면 표현될 수 없다는 것을 의미합니다. (여기서 `Const()`는 … 를 의미합니다.) 두 번째 부분은 정체성 함수(identity functor)의 다른 이름일 뿐입니다.  `Maybe`는 동형(isomorphism)을 통해 정의될 수도 있습니다.

```
type Maybe a = Either (Const () a) (Identity a)
```

물론입니다. 이 구성은 Either와 두 개의 함수형인 Const()와 Identity를 결합한 것입니다. Const()는 실제로 bifunctor이지만, 여기서는 항상 부분적으로 적용되어 사용됩니다.

저희는 이미 함수형(functor)의 합성이 또 다른 함수형(functor)이라는 것을 확인해 드렸습니다. 이제는 이중 함수형(bifunctor)의 경우에도 마찬가지라는 것을 쉽게 이해하실 수 있도록 설명드리겠습니다. 핵심은 두 개의 함수형(functor)와 이중 함수형(bifunctor)의 조합이 모피즘(morphism)에 어떻게 작용하는지를 파악하는 것입니다. 두 개의 모피즘이 주어졌을 때, 하나의 함수형(functor)로 하나를, 다른 함수형(functor)로 다른 하나를 각각 리프팅(lift)합니다. 그 결과로 생성된 리프팅된 모피즘 쌍을 이중 함수형(bifunctor)로 다시 리프팅합니다.

저희는 Haskell을 사용하여 이 구성 요소를 표현할 수 있습니다. 이를 위해, `bf` (두 개의 타입을 인자로 받는 타입 생성자), `fu` 및 `gu` (각각 하나의 타입 변수를 인자로 받는 타입 생성자)라는 데이터 타입을 정의하고, `a`와 `b`라는 두 개의 일반적인 타입을 사용합니다. 먼저 `fu`를 `a`에 적용하고, `gu`를 `b`에 적용한 다음, 그 결과로 나온 두 개의 타입을 `bf`에 적용합니다.

```
newtype BiComp bf fu gu a b = BiComp (bf (fu a) (gu b))
```

저희는 객체 또는 유형에 대한 설명을 드리고 있습니다. 하스케일에서는 함수를 인수에 적용하는 것과 마찬가지로, 유형 생성자에 유형을 적용합니다. 문법은 동일합니다.

길을 잃으셨다면, 먼저 BiComp을 Either, Const (), Identity, a, 그리고 b 순서대로 적용해 보시길 바랍니다. 그러면 저희의 기본적인 Maybe b 버전을 복구하실 수 있습니다 (a는 무시됩니다).

새로운 데이터 타입인 BiComp은 a와 b 모두에 대해 bifunctor로 작동하지만, bf가 자체적으로 Bifunctor이고 fu와 gu가 Functor일 경우에만 그렇습니다. 컴파일러는 bf에 대해 bimap의 정의가 사용될 것이며, fu와 gu에 대한 fmap의 정의가 존재한다는 것을 알아야 합니다. Haskell에서는 인스턴스 선언에 조건부 형태로 표현됩니다: 클래스 제약 조건의 집합 다음에 화살표가 있습니다:

```
instance (Bifunctor bf, Functor fu, Functor gu) =>
  Bifunctor (BiComp bf fu gu) where
    bimap f1 f2 (BiComp x) = BiComp ((bimap (fmap f1) (fmap f2)) x)
```

비컴(BiComp)에 대해 비맵(bimap)을 적용하는 방식은 bf에 대한 비맵과 fu, gu에 대한 두 개의 f맵을 기준으로 설명됩니다. 컴파일러는 비컴이 사용될 때마다 모든 타입 정보를 자동으로 추론하고, 올바른 오버로딩된 함수를 선택합니다.

“bimap”의 정의에서 나타나는 ‘x’는 다음과 같은 유형을 가집니다.

```
bf (fu a) (gu b)
```

이것은 상당히 복잡한 내용입니다. 외부 비매핑(outer bimap)이 외부 BF 레이어를 통과하고, 두 개의 F맵(fmaps)이 각각 FU와 GU 아래로 파고듭니다. F1과 F2의 유형이 다음과 같을 경우:

```
f1 :: a -> a'
f2 :: b -> b'
```

그 결과는 최종적으로 bf (fu a') (gu b') 유형으로 나타납니다.

```
bimapbf :: (fu a -> fu a') -> (gu b -> gu b') 
  -> bf (fu a) (gu b) -> bf (fu a') (gu b')
```

자사고조립퍼즐을 좋아하시는 분들께서는 이러한 종류의 퍼즐 조작 활동으로 즐거운 시간을 보내실 수 있습니다.

저희는 ‘Maybe’가 함수형(functor)이라는 것을 증명할 필요가 없다는 사실을 알게 되었습니다. 이는 ‘Maybe’가 두 개의 함수형 기본 요소(functorial primitives)의 합으로 구성되었기 때문에 자연스럽게 발생하는 결과입니다.

신중한 독자분들은 다음과 같은 질문을 던지실 수 있습니다. 연산자 함수(Functor) 인스턴스, 특히 대수적 데이터 유형을 유도하는 과정이 워낙 정형화되어 있다면, 컴파일러가 이를 자동화하고 수행할 수 있는 것은 아닌지요? 물론 가능하며, 실제로 그렇게 하고 있습니다. 이를 위해서는 소스 파일 상단에 다음 줄을 포함하여 특정 Haskell 확장 기능을 활성화해야 합니다.

```
{-# LANGUAGE DeriveFunctor #-}
```

그렇다면, 데이터 구조에 **Functor**를 추가하는 단계를 진행해 드리겠습니다.

```
data Maybe a = Nothing | Just a
  deriving Functor
```

저희는 고객님께 맞는 **매칭(fmap)**을 즉시 구현해 드리겠습니다.

대수적 데이터 구조의 일관성 덕분에, Functor뿐만 아니라 제가 이전에 언급한 Eq 타입 클래스 등 여러 타입 클래스의 인스턴스도 유도할 수 있습니다. 컴파일러에게 여러분만의 타입 클래스 인스턴스를 유도하도록 가르치는 방법도 있지만, 이것은 다소 고급 기능입니다. 하지만 핵심 아이디어는 동일합니다. 기본적인 블록들과 합집합(sums) 및 곱(products)에 대한 동작을 제공하고, 컴파일러가 나머지 부분을 알아내도록 하는 것입니다.

## Functors in C++

C++ 프로그래머이신 경우, 함수형 프로그래밍의 핵심인 함수형 프로그래밍(functors)을 구현하는 것은 스스로 해결해야 할 문제일 수 있습니다. 하지만 C++에서는 몇 가지 형태의 대수적 데이터 구조(algebraic data structures)를 인식할 수 있어야 하며, 이러한 데이터 구조를 일반적인 템플릿(generic template)으로 만들어주시면, fmap을 빠르게 구현할 수 있습니다.

저희는 트리 데이터 구조를 살펴보고 싶습니다. 이를 하스케일(Haskell) 언어로 정의할 때, 재귀적 합체 유형(recursive sum type)으로 정의하게 됩니다.

```
data Tree a = Leaf a | Node (Tree a) (Tree a)
    deriving Functor
```

이전에 말씀드린 것처럼, C++에서 합집합 유형(sum types)을 구현하는 한 가지 방법은 클래스 계층 구조를 사용하는 것입니다. 객체 지향 언어에서는 기본 클래스인 `Functor`의 가상 함수로 `fmap`을 구현하고 모든 하위 클래스에서 이를 재정의하는 것이 자연스럽습니다. 하지만 `fmap`이 템플릿이기 때문에, 작동하는 객체의 유형(this 포인터)뿐만 아니라 적용된 함수의 반환 유형으로도 템플릿화될 수 없기 때문에 불가능합니다. 가상 함수는 C++에서 템플릿화될 수 없습니다. 따라서 `fmap`은 일반적인 함수(free function)로 구현하고, 패턴 매칭 대신 `dynamic_cast`를 사용하겠습니다.

기본 클래스는 동적 캐스팅을 지원하기 위해 최소한 하나의 가상 함수를 정의해야 합니다. 따라서, 이는 좋은 방법이므로, 생성자를 가상 함수로 만들어 드리겠습니다. (virtual)

```c++
template<class T>
struct Tree {
    virtual ~Tree() {};
};
```

리프는 단순한 아이덴티티 함수어(Identity functor)의 위장된 표현이라고 할 수 있습니다.

```c++
template<class T>
struct Leaf : public Tree<T> {
    T _label;
    Leaf(T l) : _label(l) {}
};
```

The Node is a product type:

```c++
template<class T>
struct Node : public Tree<T> {
    Tree<T> * _left;
    Tree<T> * _right;
    Node(Tree<T> * l, Tree<T> * r) : _left(l), _right(r) {}
};
```

저희는 fmap을 구현할 때, Tree의 유형에 따른 동적 디스패칭을 활용합니다. Leaf 경우에는 Identity 버전의 fmap을 적용하고, Node 경우에는 두 개의 Tree 퓨니ctor 복사본으로 구성된 bifunctor처럼 취급합니다. C++ 프로그래머이시라면, 이러한 용어로 코드를 분석하는 것에 익숙하지 않으실 수 있지만, 이는 범주론적 사고의 좋은 훈련이 될 것입니다.

```c++
template<class A, class B>
Tree<B> * fmap(std::function<B(A)> f, Tree<A> * t)
{
    Leaf<A> * pl = dynamic_cast <Leaf<A>*>(t);
    if (pl)
        return new Leaf<B>(f (pl->_label));
    Node<A> * pn = dynamic_cast<Node<A>*>(t);
    if (pn)
        return new Node<B>( fmap<A>(f, pn->_left)
                          , fmap<A>(f, pn->_right));
    return nullptr;
}
```

단순화를 위해 메모리 및 자원 관리 문제를 간소화하기로 결정했습니다. 하지만 실제 코드에서는 스마트 포인터(unique 또는 shared, 귀사의 정책에 따라)를 사용하는 것이 일반적일 것입니다.

하셀케일(Haskell) 구현의 `fmap` 함수와 비교해 보시는 것을 추천드립니다.

```
instance Functor Tree where
    fmap f (Leaf a) = Leaf (f a)
    fmap f (Node t t') = Node (fmap f t) (fmap f t')
```

이 구현은 컴파일러에 의해 자동으로 유도될 수도 있습니다.

## The Writer Functor

이전에 제가 설명드린 Kleisli 카테고리에는 반드시 다시 방문할 것을 약속드렸습니다. 그 카테고리 내의 Morphisms는 Writer 데이터 구조를 반환하는 “장식된” 함수로 표현되었습니다.

```
type Writer a = (a, String)
```

저희는 말씀드린 대로, 장식(embellishment)이 somehow endofunctors와 관련이 있다는 점을 확인했습니다. 실제로, Writer 타입 생성자(type constructor)는 a에 functorial하게 작동하며, 저희는 fmap을 별도로 구현할 필요가 없습니다. 왜냐하면 이것은 단순한 제품 타입(product type)이기 때문입니다.

하지만 Kleisli 범주와 함수치의 관계는 일반적으로 어떻게 될까요? Kleisli 범주는 범주로서, 합성(composition)과 항등(identity)을 정의합니다. 다시 한번 말씀드리자면, 합성 연산은 ‘fish operator’로 표현됩니다. 

```
(>=>) :: (a -> Writer b) -> (b -> Writer c) -> (a -> Writer c)
m1 >=> m2 = \x -> 
    let (y, s1) = m1 x
        (z, s2) = m2 y
    in (z, s1 ++ s2)
```

그리고 정체성 모스함수를 반환(return)이라는 함수로 표현합니다.

```
return :: a -> Writer a
return x = (x, "")
```

이것을 살펴보시면, 이 두 함수의 종류를 충분히 오래(정말 오래, 말이죠) 관찰하시면, fmap으로 사용하기에 적절한 타입 시그니처를 가진 함수를 만드는 방법을 찾을 수 있습니다. 예를 들어, 다음과 같습니다.

```
fmap f = id >=> (\x -> return (f x))
```

여기서는 물고기 운영자가 두 가지 함수를 결합합니다. 그 중 하나는 익숙한 `id` 함수이고, 다른 하나는 람다 함수로, 이 람다 함수의 인자에 `return`을 적용하여 결과를 얻는 것입니다. 가장 어렵게 느껴지는 부분은 아마도 `id` 함수의 사용일 것입니다. 람다 운영자에 전달되는 인자는 “정상적인” 타입이 되어서 “장식된” 타입으로 반환되어야 하는 것 아닌가요? 음, 사실이 아닙니다. `a -> Writer b`와 같은 타입 변수가 “정상적인” 타입이 아니라는 점을 말씀드리고 싶습니다. 그것은 타입 변수이므로, 무엇이든 될 수 있습니다. 특히, 장식된 타입인 `Writer b`일 수도 있습니다.

저희는 Writer a를 가져와서 Writer a로 변환합니다. 어 Fisheries 운영자는 a의 값을 추출하여 x로 전달합니다. 그 후, f는 Writer a로 변환하고, 반환 값은 Writer b로 장식합니다. 모든 것을 합치면 Writer a를 입력받아 Writer b를 반환하는 함수가 생성되며, 이것이 fmap이 만들어내야 하는 정확한 결과입니다.

이 논거주는 매우 일반적입니다. Writer를 어떤 타입 생성자로든 대체할 수 있습니다. fish operator와 return을 지원한다면 fmap도 정의할 수 있습니다. 따라서 Kleisli 카테고리에 적용되는 장식은 항상 함수형(functor)입니다. (물론 모든 함수형이 Kleisli 카테고리를 만들어내는 것은 아닙니다.)

저희가 정의한 fmap이 컴파일러가 Functor를 통해 저희에게 도출했을 fmap과 동일할 수 있다는 점에 놀라움을 금할 수 없을 것입니다. 실제로 그렇습니다. 이는 하스켈이 다형적 함수를 구현하는 방식 덕분입니다. 이를 파라메트릭 다형성이라고 하며, 이는 “무료 정리”라고 불리는 정리의 원인이 됩니다. 그 중 하나는 주어진 타입 생성자에 대한 fmap 구현이 항등성을 보존하는 경우 반드시 고유하다는 것을 말합니다.

## Covariant and Contravariant Functors

이제 작성자 폰터(writer functor)를 살펴보았습니다. 이제 독자 폰터(reader functor)로 돌아가 보겠습니다. 이는 부분적으로 적용된 함수-화살표 타입 생성자(function-arrow type constructor)를 기반으로 했습니다.

```
(->) r
```

저희는 이를 유형 동의어로 재정의할 수 있습니다.

```
type Reader r a = r -> a
```

이전에 저희가 살펴본 것처럼, 이 **Functor** 인스턴스는 다음과 같이 읽습니다.

```
instance Functor (Reader r) where
    fmap f g = f . g
```

하지만 튜플 타입 생성자나 Either 타입 생성자와 마찬가지로, 함수 타입 생성자 역시 두 개의 타입 인자를 받습니다. 튜플과 Either는 모두 두 인자를 모두 받아 처리하는 bifunctor의 특징을 가지고 있었습니다. 함수 생성자 역시 bifunctor의 특징을 갖추고 있을까요?

첫 번째 인자를 통해 함수형 방식으로 만들어 보겠습니다. 독자(Reader)와 유사하지만, 인자만 뒤집은 형태를 사용합니다. 

```
type Op r a = a -> r
```

이번에는 반환 타입, r, 과 인자 타입, a 를 변경합니다. fmap 을 구현할 수 있는지 확인해 보겠습니다. fmap 의 타입 시그니처는 다음과 같습니다:

```
fmap :: (a -> b) -> (a -> r) -> (b -> r)
```

단순히 두 개의 함수, 즉 ‘a’를 받아 각각 ‘b’와 ‘r’을 반환하는 함수만으로는 ‘b’를 받아 ‘r’을 반환하는 함수를 만드는 것은 불가능합니다. 만약 어떤 식으로든 첫 번째 함수를 반전시켜 ‘b’를 받아 ‘a’를 반환하도록 만들 수 있다면 가능했을 것입니다. 하지만 임의의 함수를 반전시키는 것은 불가능하지만, 반대되는 범주로 나아갈 수는 있습니다.

간단히 요약해 드리겠습니다. 각 범주 C에 대해 쌍대 범주 Cop이 존재합니다. 이는 C와 동일한 객체들을 가지고 있지만, 모든 화살표 방향이 반대로 설정된 범주입니다.

저희는 Cop와 다른 범주 D 사이를 오가는 함수자를 고려해 보겠습니다.

F :: Cop → D

저희는 이러한 함수형(functor)을 통해 Cop 내의 morphism fop :: a → b를 D 내의 morphism F fop :: F a → F b로 매핑합니다. 하지만 morphism fop은 실제로는 C라는 원래 카테고리 내의 morphism f :: b → a에 대응합니다. 이 점에 주목해 주십시오. 

저희는 이제 F가 일반적인 함수형(functor)이지만, F를 기반으로 또 다른 매핑(mapping)을 정의할 수 있다는 점을 말씀드리겠습니다. 이 매핑은 함수형이 아니므로, G라고 부르겠습니다. 이는 C에서 D로의 매핑입니다. 객체는 F와 동일한 방식으로 매핑하지만, 모리즘(morphism)을 매핑할 때 반대 모리즘(reverse morphism)으로 반전시킵니다. 즉, C에서 f :: b → a 라는 모리즘을 가져와서 먼저 반대 모리즘 fop :: a → b 로 매핑한 다음, 함수형 F를 적용하여 F fop :: F a → F b 를 얻게 됩니다.

F a 와 G a 는 동일하며, F b 와 G b 도 동일하다는 점을 고려해 주시면 감사하겠습니다.

G f :: (b → a) → (G a → G b)

이것은 ‘반전된 기능자’입니다. 모리즘의 방향을 이와 같이 반전시키는 카테고리 매핑은 반전형 기능자라고 합니다. 반전형 기능자는 단순히 반대 카테고리에서 나오는 일반적인 기능자일 뿐입니다. 참고로, 저희가 지금까지 연구해 온 일반적인 기능자들은 공변형 기능자라고 합니다.

저희는 하스케일에서 반류(contravariant) 함수형을 정의하는 타입 클래스를 소개합니다. 이 타입 클래스는 특히 반류(contravariant) 엔도 함수형(endofunctor)을 나타냅니다.

```
class Contravariant f where
    contramap :: (b -> a) -> (f a -> f b)
```

저희 Op 타입 생성자는 이에 해당합니다.

```
instance Contravariant (Op r) where
    -- (b -> a) -> Op r a -> Op r b
    contramap f g = g . f
```

저희 서비스에서 제공하는 기능 f는 Op – 즉, 함수 g의 내용을 삽입하는 방식으로 작동합니다.

옵(Op)의 `contramap` 정의를 더 명확하게 설명드리겠습니다. 혹시, 단순히 함수 합성 연산자에서 인자를 뒤집는 것과 동일하다는 점을 주목하신다면, 더욱 간결하게 표현될 수 있습니다. 인자를 뒤집는 데 특화된 함수가 존재하는데, 이 함수는 `flip:`이라고 합니다.

```
flip :: (a -> b -> c) -> (b -> a -> c)
flip f y x = f x y
```

With it, we get:

```
contramap = flip (.)
```

## Profunctors

저희가 확인한 결과, 함수-화살표 연산자는 첫 번째 인자 기준으로 역변환(contravariant)이고 두 번째 인자 기준으로 직변환(covariant)입니다. 이러한 연산자를 부르는 명칭이 있을까요? 만약 대상 범주(target category)가 Set이라면, 이러한 연산자를 프로 Funktions터(profunctor)라고 부릅니다. 역변환 함수형(contravariant functor)은 반대 범주(opposite category)로부터의 직변환 함수형과 동치(equivalent)이기 때문입니다.

Cop × D → Set

먼저, 하스케일 타입은 대략적으로 집합과 유사하다고 가정할 수 있습니다. 따라서, 두 개의 인수를 받는 타입 생성자 p에 "Profunctor"라는 이름을 적용합니다. 이 이름은 첫 번째 인수에 대해 반대 기능(contra-functorial)이고 두 번째 인수에 대해서는 기능(functorial)입니다. 다음은 Data.Profunctor 라이브러리에서 가져온 적절한 타입 클래스입니다.

```
class Profunctor p where
  dimap :: (a -> b) -> (c -> d) -> p b c -> p a d
  dimap f g = lmap f . rmap g
  lmap :: (a -> b) -> p b c -> p a c
  lmap f = dimap f id
  rmap :: (b -> c) -> p a b -> p a c
  rmap = dimap id
```

세 가지 기능 모두 기본 구현을 제공합니다. Bifunctor와 마찬가지로 Profunctor 인스턴스를 선언할 때, dimap를 구현하고 lmap 및 rmap의 기본값을 수락하거나, lmap 및 rmap을 모두 구현하고 dimap의 기본값을 수락하는 선택 사항이 있습니다.

<!-- image -->

dimap

저희는 이제 함수-화살표 연산자가 **프로펑터(Profunctor)**의 인스턴스임을 확인할 수 있습니다.

```
instance Profunctor (->) where
  dimap ab cd bc = cd . bc . ab
  lmap = flip (.)
  rmap = (.)
```

프로 Funktions(Profunctors)는 하스케일 렌즈 라이브러리에서 활용됩니다. 저희는 이들을 엔드(ends)와 코엔드(coends)에 대해 이야기할 때 다시 살펴보겠습니다.

## The Hom-Functor

위에 제시된 예시는, 두 객체 a와 b를 입력받아 그들 사이의 모필(morphism) 집합인 모필 집합 C(a, b)를 할당하는 맵핑 자체가 함수(functor)임을 보여주는 것입니다. 이는 Cop × C라는 곱 카테고리에서 Set(집합들의 카테고리)로의 함수입니다.

우선, 이의 작용 방식을 모스피즘에 정의하겠습니다. Cop×C 안에서 모스피즘은 C에서 유래된 모스피즘 쌍으로 정의됩니다.

```
f :: a'→ a
g :: b → b'
```

이 쌍을 해제하는 것은 C(a, b) 집합에서 C(a', b') 집합으로의 모피즘(함수)입니다. C(a, b)의 임의의 요소 h(a에서 b로의 모피즘)를 선택하여 다음 값으로 할당하십시오:

```
g ∘ h ∘ f
```

C(a', b')는 다음과 같은 요소에 해당합니다.

저희는 말씀드리고 말씀드리는 바는, **홈-퍼누ctor(hom-functor)**는 **프로펑터(profunctor)**의 특별한 경우라는 점을 확인하실 수 있습니다.

## Challenges

- Show that the data type:
data Pair a b = Pair a b
is a bifunctor. For additional credit implement all three methods of Bifunctor and use equational reasoning to show that these definitions are compatible with the default implementations whenever they can be applied.
- Show the isomorphism between the standard definition of Maybe and this desugaring:
type Maybe' a = Either (Const () a) (Identity a)
Hint: Define two mappings between the two implementations. For additional credit, show that they are the inverse of each other using equational reasoning.
- Let’s try another data structure. I call it a PreList because it’s a precursor to a List. It replaces recursion with a type parameter b.
data PreList a b = Nil | Cons a b
You could recover our earlier definition of a List by recursively applying PreList to itself (we’ll see how it’s done when we talk about fixed points).
Show that PreList is an instance of Bifunctor.
- Show that the following data types define bifunctors in a and b:
data K2 c a b = K2 c
data Fst a b = Fst a
data Snd a b = Snd b
For additional credit, check your solutions agains Conor McBride’s paper Clowns to the Left of me, Jokers to the Right.
- Define a bifunctor in a language other than Haskell. Implement bimap for a generic pair in that language.
- Should std::map be considered a bifunctor or a profunctor in the two template arguments Key and T? How would you redesign this data type to make it so?

Next: Function Types.

## Acknowledgment

As usual, big thanks go to Gershom Bazerman for reviewing this article.

Follow @BartoszMilewski

