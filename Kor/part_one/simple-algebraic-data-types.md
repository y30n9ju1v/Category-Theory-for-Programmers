# Bartosz Milewski's Programming Cafe

Category Theory, Haskell, Concurrency, C++

January 13, 2015

## Simple Algebraic Data Types

Posted by Bartosz Milewski under , ,

Categories for Programmers. Previously Products and Coproducts. See the Table of Contents.

저희는 두 가지 기본적인 데이터 타입 결합 방법을 확인했습니다. 바로 제품(product)과 코프로덕트(coproduct)를 사용하는 것입니다. 실제로 많은 일상적인 프로그래밍에서 사용되는 데이터 구조는 이 두 가지 메커니즘만을 사용하여 구축할 수 있습니다. 이러한 사실은 중요한 실질적인 결과를 가져옵니다. 데이터 구조의 많은 속성은 구성 가능(composable)합니다. 예를 들어, 기본적인 타입의 값 비교 방법을 알고, 제품 및 코프로덕트 타입으로 이러한 비교를 일반화할 수 있다면, 복합 타입의 등호 연산자를 자동으로 유도할 수 있습니다. 하스켈(Haskell)에서는 복합 타입의 상당 부분에 대해 등호, 비교, 문자열로의 변환(to and from string) 등을 자동으로 유도할 수 있습니다.

프로그래밍에서 제품 타입과 합 타입이 어떻게 사용되는지 좀 더 자세히 살펴보겠습니다.

## Product Types

두 종류의 제품을 계산하는 데 있어 프로그래밍 언어에서 가장 기본적인 형태는 튜플입니다. 하스케일에서는 튜플이 원시 타입 생성자이며, C++에서는 표준 라이브러리에 정의된 비교적 복잡한 템플릿입니다.

짝은 엄밀히 말해 교환이 되지 않습니다. (Int, Bool) 튜플이 (Bool, Int) 튜플로 대체될 수 없는 이유는, 두 튜플이 동일한 정보를 담고 있더라도 그렇습니다. 하지만 이 튜플들은 동형(isomorphism)에 대해 교환됩니다. 여기서 동형은 swap 함수에 의해 주어지며, swap 함수는 그 자체의 역함수입니다.

```
swap :: (a, b) -> (b, a)
swap (x, y) = (y, x)
```

두 데이터 쌍을 단순히 다른 형식으로 동일한 데이터를 저장하는 것으로 생각하시면 됩니다. 빅 엔디언과 래틀 엔디언과 같은 경우와 마찬가지입니다.

여러 종류를 하나의 제품으로 결합할 때, 중첩된 괄호 쌍을 중첩하는 방식으로 여러 개를 묶을 수 있습니다. 하지만 더 간편한 방법이 있습니다. 중첩된 괄호 쌍은 튜플과 동일한 효과를 냅니다. 이는 서로 다른 중첩 방식이 비례적이기 때문입니다. 예를 들어, a, b, c 세 가지 종류를 순서대로 결합하고 싶다면 다음과 같이 두 가지 방법으로 할 수 있습니다.

```
((a, b), c)
```

or

```
(a, (b, c))
```

이러한 유형들은 서로 다릅니다. 한 유형을 다른 유형이 기대하는 함수로 전달할 수 없지만, 이들의 요소들은 일대일 대응 관계에 있습니다. 즉, 한 유형은 다른 유형으로 매핑되는 함수가 존재합니다.

```
alpha :: ((a, b), c) -> (a, (b, c))
alpha ((x, y), z) = (x, (y, z))
```

and this function is invertible:

```
alpha_inv :: (a, (b, c)) -> ((a, b), c)
alpha_inv  (x, (y, z)) = ((x, y), z)
```

그래서 이것은 일대일 대응 관계를 의미합니다. 단순히 동일한 데이터를 다른 방식으로 포장한 것과 같습니다.

제품 유형을 이진 연산으로 해석할 수 있습니다. 이러한 관점에서 보면, 위에 제시된 동형성은 모노이드에서 본 연합 법칙과 매우 유사하게 보입니다.

```
(a * b) * c = a * (b * c)
```

다만, 모노이드의 경우에는 두 가지 곱셈 방법을 사용하는 것이 서로 동일했지만, 여기서는 ‘위상동형’까지는 동일하다고 말씀드릴 수 있습니다.

만약 우리가 동형사상(isomorphisms)을 수용하고 엄격한 평등성을 요구하지 않는다면, 훨씬 더 나아가 단위 타입( (), )이 곱셈의 단위 역할을 하는 방식과 마찬가지로 단위가 될 수 있음을 보여드릴 수 있습니다. 특히, 어떤 타입의 값과 단위를 결합하는 것은 추가적인 정보를 제공하지 않습니다. The type:

```
(a, ())
```

is isomorphic to a. Here’s the isomorphism:

```
rho :: (a, ()) -> a
rho (x, ()) = x
```

```
rho_inv :: a -> (a, ())
rho_inv x = (x, ())
```

이러한 관찰 사항들을 정리하면, 집합(집합들의 범주)은 단단층 범주(monoidal category)라고 할 수 있습니다. 즉, 객체들을 곱셈하는 방식으로, 여기서 곱셈은 카르테시안 곱을 취하는 것을 의미합니다. 앞으로 단단층 범주에 대해 더 자세히 설명드리겠습니다.

제품 유형을 정의하는 더 일반적인 방법이 있습니다. 특히, 곧 살펴보실 내용이지만, 합본 유형과 함께 사용될 때 더욱 그렇습니다. 이 방법은 여러 인수를 가지는 명명된 생성자를 사용합니다. 예를 들어, 튜플은 다음과 같이 대안적으로 정의할 수 있습니다.

```
data Pair a b = P a b
```

여기, Pair a b는 두 개의 다른 타입, a와 b로 타입화된 Pair 타입의 이름입니다. P는 데이터 생성자의 이름입니다. Pair 타입은 두 개의 타입을 전달하여 정의합니다. Pair 타입 생성자에 두 개의 적절한 타입 값을 전달하여 Pair 값을 생성합니다. 예를 들어, String과 Bool의 쌍으로 value stmt를 정의할 수 있습니다.

```
stmt :: Pair String Bool
stmt = P "This statements is" False
```

첫 번째 줄은 타입 선언입니다. Pair 생성자를 사용하여 a와 b를 String과 Bool로 대체합니다. 이는 Pair의 일반적인 정의에서 사용되는 방식입니다. 두 번째 줄에서는 구체적인 문자열과 불리언 값을 P 데이터 생성자에 전달하여 실제 값을 정의합니다. 타입 생성자는 타입을 생성하는 데 사용되고, 데이터 생성자는 값을 생성하는 데 사용됩니다.

타입 및 데이터 생성자에 대한 네임스페이스가 Haskell에서 분리되어 있기 때문에, 동일한 이름을 둘 다 사용하게 되는 경우가 많습니다. 예를 들어, 다음과 같습니다.

```
data Pair a b = Pair a b
```

만약 충분히 자세히 살펴보신다면, 내장된 Pair 타입은 이와 유사한 선언의 변형으로도 볼 수 있습니다. 즉, Pair라는 이름 대신 괄호(,) 연산자를 사용하는 것입니다. 실제로 괄호(,)를 다른 이름 지정 생성자와 마찬가지로 사용하여 괄호 표기법을 통해 튜플을 생성할 수 있습니다.

```
stmt = (,) "This statement is" False
```

비슷하게, (,,)를 사용하여 트리플을 만들 수도 있고, 그 외에도 다양한 방법이 있습니다.

일반적인 튜플이나 코플러를 사용하는 대신, 다음과 같이 특정 제품 유형을 명명하여 정의할 수도 있습니다.

```
data Stmt = Stmt String Bool
```

String과 Bool을 기반으로 만들어졌지만, 자체적인 이름과 생성자를 제공합니다. 이러한 선언 방식의 장점은 동일한 내용이지만 서로 다른 의미와 기능을 가진 여러 유형을 정의할 수 있다는 점입니다. 이러한 유형들은 서로 대체될 수 없습니다.

튜플과 다중 인자 생성자를 사용하여 프로그래밍을 진행하면 복잡하고 오류가 발생하기 쉬울 수 있습니다. 각 구성 요소가 무엇을 나타내는지 추적하는 것이 어려울 수 있기 때문입니다. 따라서 구성 요소에 이름을 부여하는 것이 훨씬 더 권장됩니다. 하스케일에서는 이름이 지정된 필드를 가진 데이터 유형을 레코드로, C에서는 구조체로 부릅니다.

### Records

간단한 예제를 살펴볼까요? 화학 원소를 설명하기 위해 이름(name)과 기호(symbol)라는 두 개의 문자열, 그리고 원자 번호(atomic number)라는 정수를 하나의 데이터 구조로 결합하고자 합니다. 튜플(String, String, Int)을 사용하고 각 구성 요소가 무엇을 나타내는지 기억할 수 있습니다. 예를 들어, He가 Helium의 접두사라는 점과 같이 패턴 매칭을 통해 구성 요소를 추출할 수 있습니다.

```
startsWithSymbol :: (String, String, Int) -> Bool
startsWithSymbol (name, symbol, _) = isPrefixOf symbol name
```

이 코드는 오류가 발생하기 쉽고, 읽고 유지 관리하기에도 어려움이 많습니다. 대신 레코드를 정의하는 것이 훨씬 더 좋습니다.

```
data Element = Element { name         :: String
                       , symbol       :: String
                       , atomicNumber :: Int }
```

이 두 변환 함수를 통해 확인할 수 있듯이, 이 두 표현은 서로 대응 관계에 있습니다.

```
tupleToElem :: (String, String, Int) -> Element
tupleToElem (n, s, a) = Element { name = n
                                , symbol = s
                                , atomicNumber = a }
```

```
elemToTuple :: Element -> (String, String, Int)
elemToTuple e = (name e, symbol e, atomicNumber e)
```

저희는 기록 필드의 이름이 해당 필드에 접근하는 데 사용되는 함수로도 활용된다는 점을 주목해 주시기 바랍니다. 예를 들어, atomicNumber라는 이름의 필드는 atomicNumber라는 필드에 접근하는 데 사용됩니다. 저희는 atomicNumber를 함수 형태로 사용하는 것을 권장합니다.

```
atomicNumber :: Element -> Int
```

Element의 최고 수준의 구문으로 인해, 저희 함수인 startWithSymbol이 더욱 읽기 쉬워졌습니다.

```
startsWithSymbol :: Element -> Bool
startsWithSymbol e = isPrefixOf (symbol e) (name e)
```

저희는 Haskell의 기술을 활용하여 `isPrefixOf` 함수를 infix 연산자로 변환할 수도 있습니다. 이를 위해 backquote를 사용하여 함수를 감싸면 거의 문장처럼 읽히도록 만들 수 있습니다.

```
startsWithSymbol e = symbol e `isPrefixOf` name e
```

이 경우에는 괄호를 생략하셔도 괜찮습니다. 괄호는 함수 호출보다 우선순위가 낮은 연산자 때문에 사용되기 때문입니다.

## Sum Types

세트(sets) 카테고리 제품이 제품 유형을 낳듯이, 코프로덕트(coproduct)는 합(sum) 유형을 낳습니다. 하스케일(Haskell)에서 합 유형의 표준 구현은 다음과 같습니다:

```
data Either a b = Left a | Right b
```

물론 쌍과 마찬가지로, Eithers는 동형사상까지 교환 가능하며, 중첩될 수 있으며, 중첩 순서는 교환 가능합니다. 즉, 트리플의 등가 합을 정의할 수 있습니다.

```
data OneOfThree a b c = Sinistral a | Medial b | Dextral c
```

and so on.

결과적으로, Set는 코프로덕트를 기준으로 볼 때 대칭 단단치 범주이기도 합니다. 이중 연산의 역할은 분리 합이 담당하고, 단위 원소의 역할은 초기 객체가 담당합니다. 유형의 관점에서 보면, Either를 단단치 연산자로, 비상위 객체인 Void를 중성 원소로 사용합니다. Either는 ‘+’와 유사하게 생각할 수 있으며, Void는 ‘0’과 같습니다. 실제로 Void를 합체 유형에 더하면 내용이 변하지 않습니다. 예를 들어:

```
Either a Void
```

이것은 a와 동형입니다. 왜냐하면 이 유형의 ‘Right’ 버전은 만들 수 없기 때문입니다. Void 유형의 값 자체가 존재하지 않기 때문입니다. Either 유형의 Void 인접한 값들은 Left 생성자를 사용하여 구성되며, 단순히 a 유형의 값을 캡슐화하는 것입니다. 따라서 상징적으로는 a + 0 = a 입니다.

합본 타입은 하스케일에서 꽤 흔하게 사용되지만, C++에서의 합집합이나 변형체는 그만큼 흔하게 사용되지는 않습니다. 그 이유는 여러 가지가 있습니다.

가장 먼저, 가장 기본적인 수형(sum types)은 단순히 열거(enumerations)이며, C++에서 enum을 사용하여 구현됩니다. 하스케일 수형(sum type)에 해당하는 내용은 다음과 같습니다:

```
data Color = Red | Green | Blue
```

is the C++:

```
enum { Red, Green, Blue };
```

An even simpler sum type:

```
data Bool = True | False
```

is the primitive bool in C++.

단순한 합집합 타입으로 값의 존재 여부를 나타내는 것은 C++에서 특수한 트릭과 “불가능한” 값들, 예를 들어 빈 문자열, 음수, NULL 포인터 등을 사용하여 다양하게 구현됩니다. 이러한 선택적 기능은 의도적인 경우, 하스케일에서는 Maybe 타입으로 표현됩니다.

```
data Maybe a = Nothing | Just a
```

마이비 타입은 두 개의 타입의 합으로 구성되어 있습니다. 두 생성자를 개별 타입으로 분리하시면 이 점을 확인하실 수 있습니다. 첫 번째 타입은 다음과 같이 구성됩니다:

```
data NothingType = Nothing
```

무엇인가라는 값 하나만을 가진 열거(enumeration)입니다. 다시 말해, 단일(singleton) 유형이며, 이는 ()와 같은 유닛 타입과 같습니다. 다음 부분은:

```
data JustType a = Just a
```

단순히 유형 a의 축약된 표현일 뿐입니다. ‘Maybe’를 다음과 같이 인코딩할 수도 있었습니다:

```
type Maybe a = Either () a
```

더 복잡한 타입 연산(sum types)은 C++에서 포인터를 사용하여 종종 모방됩니다. 포인터는 null 상태일 수도 있고, 특정 타입의 값을 가리킬 수도 있습니다. 예를 들어, 재귀적으로 정의될 수 있는 Haskell 리스트 타입과 같이 말이죠.

```
List a = Nil | Cons a (List a)
```

널 포인터 트릭을 사용하여 빈 리스트를 구현할 수 있습니다.

```
template<class A> 
class List {
    Node<A> * _head;
public:
    List() : _head(nullptr) {}  // Nil
    List(A a, List<A> l)        // Cons
      : _head(new Node<A>(a, l))
    {}
};
```

Nil과 Cons라는 두 가지 Haskell 생성자를 List 클래스의 두 가지 오버로딩된 생성자로 번역한 것을 보실 수 있습니다. 이러한 생성자는 각각 (값 없음, Nil의 경우) 그리고 값과 리스트 (Cons의 경우)와 같은 유사한 인수를 사용합니다. List 클래스는 합산형의 두 구성 요소를 구별하기 위해 태그를 사용할 필요가 없습니다. 대신, \_head라는 특별한 nullptr 값을 사용하여 Nil을 인코딩합니다.

하지만 해스켈(Haskell)과 C++의 타입(type)을 비교할 때 가장 큰 차이점은 해스켈의 데이터 구조(data structure)가 불변(immutable)하다는 것입니다. 특정 생성자(constructor)를 사용하여 객체를 생성하면, 그 객체는 어떤 생성자가 사용되었고 어떤 인자(argument)가 전달되었는지 영원히 기억하게 됩니다. 따라서 “energy”로 생성된 Just 객체는 절대 Nothing으로 변하지 않습니다. 마찬가지로, 빈 리스트는 영원히 비어 있으며, 세 개의 요소로 이루어진 리스트는 항상 동일한 세 개의 요소를 갖게 됩니다.

이 불변성은 건설을 되돌릴 수 있게 합니다. 주어진 객체는 항상 그 구성에 사용된 부품으로 분해할 수 있습니다. 이 분해 과정은 패턴 매칭을 통해 이루어지며, 생성자를 패턴으로 재사용합니다. 생성자 인수가 있는 경우, 해당 인수는 변수(또는 다른 패턴)로 대체됩니다.

리스트 데이터 타입은 두 개의 생성자를 가지고 있습니다. 따라서 임의의 리스트를 해체하는 과정은 해당 생성자에 대응하는 두 가지 패턴을 사용합니다. 하나는 빈 Nil 리스트와 다른 하나는 Cons 방식으로 구성된 리스트에 해당합니다. 예를 들어, 리스트에 대한 간단한 함수 정의는 다음과 같습니다.

```
maybeTail :: List a -> Maybe (List a)
maybeTail Nil = Nothing
maybeTail (Cons _ t) = Just t
```

첫 번째 부분에서는 maybeTail의 정의에서 Nil 생성자를 패턴으로 사용하여 Nothing을 반환합니다. 두 번째 부분에서는 Cons 생성자를 패턴으로 사용합니다. 첫 번째 생성자 인수를 와일드카드(wildcard)로 대체합니다. 저희는 이 값에 관심이 없기 때문입니다. Cons의 두 번째 인수는 t라는 변수에 바인딩됩니다. (엄밀히 말하면, 변수는 절대 변하지 않지만, 여기서는 이러한 것들을 변수라고 부릅니다. 변수에 값이 바인딩되면, 변수는 절대 변하지 않기 때문입니다.) 반환되는 값은 Just t입니다. 이제 목록이 어떻게 생성되었는지에 따라, 해당 절(clause)에 매치될 것입니다. 만약 목록이 Cons를 사용하여 생성되었다면, 생성될 때 전달된 두 인수가 가져와집니다 (그리고 첫 번째 인수는 버려집니다).

더욱 복잡한 서브타입(sum types)은 C++에서 다형적 클래스 계층 구조를 사용하여 구현됩니다. 공통 조상(ancestor)을 가진 클래스들의 집합은 하나의 변종 타입으로 이해될 수 있으며, vtable은 숨겨진 태그 역할을 합니다. Haskell에서는 생성자에 대한 패턴 매칭과 특화된 코드를 호출하여 수행되는 작업을 C++에서는 vtable 포인터를 기반으로 가상 함수 호출을 디스패치하여 수행합니다.

C++에서 유니언을 덧셈 타입으로 사용하는 경우는 매우 드뭅니다. 유니언 안에 들어갈 수 있는 것이 상당히 제한되기 때문입니다. 심지어 std::string을 유니언에 넣는 것조차 복사 생성자가 있기 때문에 불가능합니다.

## Algebra of Types

별도로 제품 타입과 합집합 타입은 다양한 유용한 데이터 구조를 정의하는 데 활용될 수 있습니다. 하지만 그 진정한 강점은 이 두 가지를 결합할 때 나타납니다. 다시 한번, 구성의 힘을 활용하는 것입니다.

지금까지 저희가 밝혀낸 내용을 요약해 드리겠습니다. 저희는 타입 시스템의 기반이 되는 두 개의 교환 환형 구조를 확인했습니다. 첫째, Void를 중립 원소로 갖는 Sum 타입, 둘째, Unit 타입, (),를 중립 원소로 갖는 Product 타입입니다. 이와 같은 비유를 통해 Void는 0, Unit와 ()는 1에 해당한다고 생각할 수 있습니다.

이 분석을 얼마나 더 확장할 수 있을지 살펴보겠습니다. 예를 들어, 0으로 곱하면 0이 되는 걸 보시죠. 즉, 하나의 구성 요소가 Void와 동형인 제품 유형이 존재하는 걸까요? 예를 들어, Int와 Void와 같은 쌍을 만드는 것이 가능한 걸까요?

두 값을 사용하여 쌍을 생성하려면 두 개의 값이 필요합니다. 정수를 쉽게 생성할 수 있지만 Void 타입의 값은 존재하지 않습니다. 따라서 어떤 타입 a에 대해, 타입 (a, Void)는 비어있습니다 – 즉, 값이 없으며, 따라서 Void와 같습니다. 다시 말해, a*0 = 0 입니다.

덧셈과 곱셈을 연결하는 또 다른 점은 분배 법칙입니다.

```
a * (b + c) = a * b + a * c
```

제품과 합집합 타입에도 적용되는지 궁금하시군요. 네, 그렇습니다. 일반적인 경우, 동형사상까지는 적용됩니다. 좌변은 다음과 같은 타입에 해당합니다.

```
(a, Either b c)
```

and the right hand side corresponds to the type:

```
Either (a, b) (a, c)
```

Here’s the function that converts them one way:

```
prodToSum :: (a, Either b c) -> Either (a, b) (a, c)
prodToSum (x, e) = 
    case e of
      Left  y -> Left  (x, y)
      Right z -> Right (x, z)
```

and here’s one that goes the other way:

```
sumToProd :: Either (a, b) (a, c) -> (a, Either b c)
sumToProd e = 
    case e of
      Left  (x, y) -> (x, Left  y)
      Right (x, z) -> (x, Right z)
```

함수 내부에서 패턴 매칭을 수행할 때, case of 문은 사용됩니다. 각 패턴 뒤에는 화살표와 패턴이 일치할 때 평가될 표현이 이어집니다. 예를 들어, prodToSum 함수를 호출할 때 다음과 같은 값을 전달하면:

```
prod1 :: (Int, Either String Float)
prod1 = (2, Left "Hi!")
```

경우에 따라 ‘Hi!’가 Left로 대체될 것입니다. 이는 ‘Left y’ 패턴과 일치하며, ‘y’ 자리에 ‘Hi!’가 들어가게 됩니다. 이미 ‘x’가 2로 매칭되었으므로, ‘case of’ 절과 전체 함수 결과는 예상대로 Left (2, “Hi!”)가 됩니다.

이 두 함수가 서로 역함수라는 것을 증명해 드릴 수는 없지만, 한번 생각해 보시면 분명히 그렇다는 것을 알 수 있을 겁니다. 이 함수들은 단순히 두 데이터 구조의 내용을 다시 포맷팅하는 것일 뿐입니다. 동일한 데이터를 다른 형식으로 표현한 것과 같습니다.

수학자들은 이러한 얽힌 모노이드를 지칭하기 위해 특정 용어를 사용하는데, 바로 ‘셈링(semiring)’입니다. 이는 완전한 원환(ring)이 아니며, 타입의 뺄셈을 정의할 수 없기 때문입니다. 그래서 셈링은 종종 “n이 없는 원환(ring)”이라는 말장난으로 ‘리그(rig)’라고도 불립니다(부정적인 의미). 하지만 이러한 점을 제외하고는, 자연수와 같은 표현을 통해 셈링을 설명하는 것을 타입에 대한 설명으로 번역하는 데 많은 활용 가치를 얻을 수 있습니다. 다음은 몇 가지 흥미로운 항목을 포함한 번역 테이블입니다.

| Numbers   | Types                           |
| --------- | ------------------------------- |
| 0         | Void                            |
| 1         | ()                              |
| a + b     | Either a b = Left a \| Right b  |
| a * b     | (a, b)  or  Pair a b = Pair a b |
| 2 = 1 + 1 | data Bool = True \| False       |
| 1 + a     | data Maybe = Nothing \| Just a  |

리스트 타입은 매우 흥미로운 개념입니다. 왜냐하면 이 리스트 타입은 방정식의 해를 나타내는 것으로 정의되기 때문입니다. 저희가 정의하는 이 타입은 방정식의 양변에 모두 나타납니다.

```
List a = Nil | Cons a (List a)
```

저희가 평소와 같이 대체 작업을 진행하고, List a를 x로 대체하시면 다음과 같은 방정식이 됩니다:

```
x = 1 + a * x
```

전통적인 대수학적 방법으로는 해결하기 어렵습니다. 왜냐하면 덧셈이나 나눗셈을 수행할 수 없기 때문입니다. 하지만, x를 오른쪽 변에 (1 + a\*x)로 계속 대체하면서 분배 법칙을 활용하면 다음과 같은 연쇄적인 과정을 시도해 볼 수 있습니다.

```
x = 1 + a*x
x = 1 + a*(1 + a*x) = 1 + a + a*a*x
x = 1 + a + a*a*(1 + a*x) = 1 + a + a*a + a*a*a*x
...
x = 1 + a + a*a + a*a*a + a*a*a*a...
```

결과적으로, 우리는 무한한 제품(튜플)의 합으로 귀결되며, 이는 다음과 같이 해석될 수 있습니다. 리스트는 비어있는 상태이거나, 1, 단일 요소 리스트인 'a', 두 개의 요소로 이루어진 리스트인 'a\*a', 세 개의 요소로 이루어진 리스트인 'a\*a\*a' 등으로 표현될 수 있습니다. 즉, 리스트는 바로 'a'의 연속이라고 할 수 있습니다.

그것 외에도 목록에는 훨씬 더 많은 내용이 있으며, 함수형(functors)과 고정점(fixed points)에 대해 배우는 후에 다시 다루게 될 것입니다.

수식을 풀면서 기호 변수를 다루는 것은 바로 대수학입니다. 이것이 이러한 유형의 이름을 부여하는 것이죠. 알기 쉽게 설명해 드리겠습니다.

마지막으로, 저는 유형 연산의 매우 중요한 한 가지 해석을 언급하고 싶습니다. 두 유형 a와 b의 곱셈은 a 유형의 값과 b 유형의 값을 모두 포함해야 한다는 점에 주목하십시오. 즉, 두 유형 모두 유효한 값(populated)을 가지고 있어야 합니다. 반면에 두 유형의 합은 a 유형의 값 또는 b 유형의 값 중 하나를 포함합니다. 따라서 하나만 유효한 값(populated)을 가지고 있으면 됩니다. 논리 연산인 and와 or 또한 semiring을 형성하며, 이 또한 유형 이론으로 매핑될 수 있습니다.

| Logic    | Types                          |
| -------- | ------------------------------ |
| false    | Void                           |
| true     | ()                             |
| a \|\| b | Either a b = Left a \| Right b |
| a && b   | (a, b)                         |

이 비유는 더욱 깊이 파고들며, 논리와 타입 이론 사이의 커리-행위스모사이크즘의 기초가 됩니다. 함수 타입에 대해 이야기할 때 다시 돌아와서 살펴보겠습니다.

## Challenges

- Show the isomorphism between Maybe a and Either () a.
- Here’s a sum type defined in Haskell:
data Shape = Circle Float
           | Rect Float Float
When we want to define a function like area that acts on a Shape, we do it by pattern matching on the two constructors:
area :: Shape -> Float
area (Circle r) = pi * r * r
area (Rect d h) = d * h
Implement Shape in C++ or Java as an interface and create two classes: Circle and Rect. Implement area as a virtual function.
- Continuing with the previous example: We can easily add a new function circ that calculates the circumference of a Shape. We can do it without touching the definition of Shape:
circ :: Shape -> Float
circ (Circle r) = 2.0 * pi * r
circ (Rect d h) = 2.0 * (d + h)
Add circ to your C++ or Java implementation. What parts of the original code did you have to touch?
- Continuing further: Add a new shape, Square, to Shape and make all the necessary updates. What code did you have to touch in Haskell vs. C++ or Java? (Even if you’re not a Haskell programmer, the modifications should be pretty obvious.)
- Show that a + a = 2 * a holds for types (up to isomorphism). Remember that 2 corresponds to Bool, according to our translation table.

Next: Functors.

### Acknowledments

Thanks go to Gershom Bazerman for reviewing this post and helpful comments.

