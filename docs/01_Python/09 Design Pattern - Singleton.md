# Singleton

`class`의 모든 내용을 동일하게 하기 위한 방법입니다. 만약 DB에서 어떤 데이터를 읽어오는 `class`를 만들었다고 가정하겠습니다. A라는 함수에서 사용하고나서 B라는 함수에서도 사용하고자 할 때는 *argument*로 전달하는 수 밖에 없습니다.

Singleton을 만들어서 사용하면 그냥 B라는 함수에서 새로운 `class`를 선언 및 정의하여 사용해도 A에서 수정한 *property*가 그대로 남아있어서 동일하게 사용할 수 있습니다.

단, python에서는 meta programming이 가능하므로 새로운 property를 강제로 할당하게 되는데 이럴 때까지 동일한 property가 유지되진 않습니다. 어디에서 수정하거나 만들어도 완전히 동일한 `class`를 사용하고자 한다면 **Borg**(보그) Pattern을 사용하시길 바랍니다. (개인적으로 만들어 놓은 `class`에 변수를 강제할당하는 행위는 지양하기 때문에 제 입장에선 동일하게 사용합니다.)

## 1. type class의 이해

python은 meta programming이 가능한 프로그래밍 언어로써 흔히 알고 있는 형태와 다른 방식으로 class를 만들 수 있습니다.

```python
# 클래스명, 상속받을 클래스(list 타입), 클래스 함수(dict 타입)
Card = type("Card", (object,), {"choice" : lambda self, x: x + 1})
Card.__init__ = lambda self, x: print(x * 10)
```

위와 같이 만든다면 실제로 아래와 같이 만든 것과 동일한 역할을 수행할 수 있습니다.

```python
class Card(object):
    def __init__(self, x):
        print(x * 10)
    def choice(self, x):
        return x + 1
```

이 지식을 토대로 Singleton Pattern 를 구현하겠습니다. 구현할 수 있는 방법이 몇가지 있는 것으로 파악되는데, 어차피 해당 Pattern을 쓸 것이라면 여러가지 방법보단 한 가지만 잘 기억하고 있는 것이 낫다고 생각하니 한가지만 구현토록 하겠습니다.

## 2. Singleton Pattern

**Singleton Pattern**은 생성자가 실행될 때를 포착해서 기존에 있는 클래스를 다시 return 해주는 형태입니다. 작성된 예제에선 meta class를 작성하고 이를 metaclass 상속을 통하여 **Singleton**을 구현합니다.

사전에 알아두면 좋은 정보는 

1. 모든 `class`는 type에서 만들어집니다.
2. __new__는 `class`가 처음 선언될 때 실행되는 함수입니다.
3. type을 상속 받은 class에서 __call__은 meta class를 상속받은 `class`의 __init__가 실행될 때를 말합니다.
4. metaclass=SingletonMeta로 상속받는 이유는 다중 상속시 mro에 의해 꼬일 수도 있기 때문입니다.

아래 코드를 확인해보겠습니다.

```python
class SingletonMeta(type):
    def __init__(cls, *args):
        # type에서 상속받으므로 관련된것 전부 받기 위해 실행합니다.
        super(SingletonMeta, cls).__init__(*args)
        
        # 처음 실행되는 것인지 확인하기 위한 변수입니다.
        cls._instance = None

    def __call__(cls, *args, **kwargs):
        if not cls._instance:
            # 처음 실행되었다면 class를 새로 만들어서 return 합니다.
            cls._instance = super(SingletonMeta, cls).__call__(*args, **kwargs)
        return cls._instance

class Card(metaclass=SingletonMeta):

    has_cards = []

    def __init__(self, new_number):
        self.has_cards.append(1)
        self.has_cards.append(new_number)


my_card = Card(2)
your_card = Card(3) # 이 class의 __init__는 무시됩니다.
your_card.has_cards.append(4)

print(my_card.has_cards, your_card.has_cards)
```

```
([1, 2, 4], [1, 2, 4])
```

SingletonMeta class에서 __init__의 *args는 아래와 같습니다.

```
('Card', (), {'__module__': '__main__', '__qualname__': 'Card', 'has_cards': [], '__init__': <function Card.__init__ at 0x7714fe2290>})
```

처음에 **type**로 **class**를 구현하는 것을 기억해보면 동일한 것을 알 수 있습니다. Card class의 원형을 읽어온다고 생각하면 되겠습니다.

SingletonMeta **class**에서 __call__의 *args, *kwargs는 아래와 같습니다.

```python
# my_card = Card(2)의 경우
args : (2,)
kwargs : {}
```

Card의 __init__의 argument인 것을 알 수 있습니다. 최종적으로 나중에 사용하게 된다면 그냥 SingletonMeta class를 복사하여 metaclass로 받아 사용하시면 됩니다. 


