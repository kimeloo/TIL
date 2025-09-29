

## 개념
Decorator 패턴은 객체에 추가적인 기능을 동적으로 첨가하는 구조 패턴입니다. 기존 객체를 수정하지 않고도 새로운 기능을 추가할 수 있어 상속보다 유연한 대안을 제공합니다.

## 구조
```
Component (인터페이스)
├── ConcreteComponent (기본 구현)
└── Decorator (추상 데코레이터)
    └── ConcreteDecorator (구체적인 데코레이터)
```

## 주요 구성 요소
1. **Component**: 기본 기능을 정의하는 인터페이스
2. **ConcreteComponent**: Component의 기본 구현
3. **Decorator**: Component를 감싸는 추상 데코레이터
4. **ConcreteDecorator**: 실제 기능을 추가하는 구체적인 데코레이터

## Java 예제
```java
// Component 인터페이스
interface Coffee {
    String getDescription();
    double getCost();
}

// ConcreteComponent
class SimpleCoffee implements Coffee {
    @Override
    public String getDescription() {
        return "Simple Coffee";
    }

    @Override
    public double getCost() {
        return 2.0;
    }
}

// Decorator 추상 클래스
abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;

    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
}

// ConcreteDecorator
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 0.5;
    }
}

class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Sugar";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 0.2;
    }
}

// 사용 예제
public class DecoratorExample {
    public static void main(String[] args) {
        Coffee coffee = new SimpleCoffee();
        coffee = new MilkDecorator(coffee);
        coffee = new SugarDecorator(coffee);

        System.out.println(coffee.getDescription()); // Simple Coffee, Milk, Sugar
        System.out.println("Cost: $" + coffee.getCost()); // Cost: $2.7
    }
}
```

## Python 예제
```python
from abc import ABC, abstractmethod

class Component(ABC):
    @abstractmethod
    def operation(self):
        pass

class ConcreteComponent(Component):
    def operation(self):
        return "ConcreteComponent"

class Decorator(Component):
    def __init__(self, component):
        self._component = component

    def operation(self):
        return self._component.operation()

class ConcreteDecoratorA(Decorator):
    def operation(self):
        return f"ConcreteDecoratorA({self._component.operation()})"

class ConcreteDecoratorB(Decorator):
    def operation(self):
        return f"ConcreteDecoratorB({self._component.operation()})"

# 사용 예제
component = ConcreteComponent()
decorator1 = ConcreteDecoratorA(component)
decorator2 = ConcreteDecoratorB(decorator1)
print(decorator2.operation())  # ConcreteDecoratorB(ConcreteDecoratorA(ConcreteComponent))
```

## 장점
1. **런타임 기능 확장**: 실행 중에 객체에 기능을 동적으로 추가 가능
2. **단일 책임 원칙**: 각 데코레이터가 하나의 기능에만 집중
3. **조합 가능성**: 여러 데코레이터를 자유롭게 조합하여 다양한 기능 구현
4. **상속의 대안**: 상속보다 유연한 기능 확장 방법 제공

## 단점
1. **복잡성 증가**: 많은 작은 클래스들로 인한 복잡도 상승
2. **디버깅 어려움**: 여러 래퍼로 인한 스택 트레이스 복잡성
3. **순서 의존성**: 데코레이터 적용 순서에 따라 결과가 달라질 수 있음

## 실제 사용 예시
1. **Java I/O Stream**: BufferedReader, FileReader 등
2. **Web Framework**: 미들웨어, 인터셉터
3. **GUI 컴포넌트**: 스크롤바, 테두리 등의 UI 장식
4. **Cache 시스템**: 다양한 캐싱 전략 적용

## 언제 사용하는가?
- 객체의 기본 기능을 변경하지 않고 새로운 기능을 추가하고 싶을 때
- 상속으로 기능을 확장하기 어려운 경우
- 런타임에 기능을 동적으로 추가/제거해야 할 때
- 여러 기능을 조합하여 사용해야 할 때