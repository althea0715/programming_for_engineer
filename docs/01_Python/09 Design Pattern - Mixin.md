# Mixin

다중 상속의 일종인데, `Base Class`를 상속하고 `Base Class`를 지원하는 추가 `Class`를 작성해서 다중상속하는 형태를 말합니다.

몇 가지 나름대로의 규칙이 있는데

1. 생성자가 없습니다. 파이썬의 경우 `__init__` 및 `attribute`를 선언 및 정의하지 않습니다.
2. 가급적이면 `Mixin` 이라는 키워드를 클래스명에 작성하는 것이 추천됩니다.
3. 파이썬의 경우 상속할 때 가급적이면 BaseClass, Mixin1, Mixin2 ~ 순으로 상속합니다. 왜냐면 __mro__에 의거하여 호출되는 순서가 달라질 수도 있기 때문입니다. 아래의 예시에선 CardGame, Card, CardMixin 순으로 우선순위가 설정됩니다.

```python
import random

# 필요한 부분
class Card:

    card = [i for i in range(1, 11)]

    def draw(self):
        return random.choices(self.card)

# 추가하고 싶은 부분
class CardMixin:
    def even(self):

        return [c for c in self.card if c % 2 == 0]


class CardGame(Card, CardMixin):
    pass

# 확인
cg = CardGame()
cg.even()
```

위에서 볼 수 있듯이, 본래 클래스에서 무언가 추가 기능을 구축하고 싶을 때 사용하면 됩니다. 만약에 단일 상속으로 위와 같이 구현하려면 필연적으로 `Card`를 `CardMixin`이 상속을 받아서 `CardGame`이 다시 상속을 받는 형태를 띄게 될 겁니다. 

개인적으로 PyQt 등의 UI를 구현할 때 Widget에 추가 기능이 필요할 때 사용하면 좋겠습니다.

