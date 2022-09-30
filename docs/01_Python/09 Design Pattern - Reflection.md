# Reflection & Introspection

자바에선 Reflection, Objective-C에선 Introspection이라는 기법인데 런타입중에 동작을 검사하거나 수정을 하는 방법입니다. 이런 기법을 요령껏 사용하면 

## 1. Meta Global Method 경우

텍스트를 활용하여 함수를 불러오고 싶을 때 아래와 같은 방식으로 함수를 사용할 수 있습니다.

```python
def sum_(num1, num2):
    return num1 + num2

def minus_(num1, num2):
    return num1 - num2

def test(name, num1, num2):
    return globals()[f"{name}_"](num1, num2)

test("sum", 1, 2)
test("minus", 1, 2)
```

## 2. Meta Class Method 경우

텍스트를 활용하여 클래스 내부의 함수를 불러오고 싶을 때 아래와 같은 방식으로 함수를 사용할 수 있습니다.

```python
class Calculate:
    def sum_(self, num1, num2):
        return num1 + num2

    def minus_(self, num1, num2):
        return num1 - num2

    def test(self, name, num1, num2):
        return getattr(self, f"{name}_")(num1, num2)

Calculate().test("sum", 1, 2)
Calculate().test("minus", 1, 2)
```



