# Composite 패턴

Composite 디자인 패턴은 객체를 트리 구조로 구성해 부분과 전체를 동일하게 다루게 해주는 패턴입니다. 이를 통해 복잡한 계층 구조를 단순화하고, 개별 객체와 복합 객체를 동일하게 처리할 수 있습니다.

**주요 개념:**

1. **Component**: 공통 인터페이스로, Leaf와 Composite에 공통된 작업을 정의합니다.
2. **Leaf**: 트리의 말단 요소로, 실제 작업을 수행합니다.
3. **Composite**: 복합 객체로, 다른 Leaf나 Composite를 자식으로 포함하고 관리합니다.

**왜 Composite 패턴을 사용할까?**
생각해보세요. 파일 시스템을 예로 들면, 파일과 폴더를 어떻게 관리할 수 있을까요? 파일은 Leaf, 폴더는 Composite로 볼 수 있습니다. 폴더는 파일과 다른 폴더를 포함할 수 있죠. Composite 패턴을 사용하면 파일과 폴더를 동일하게 다룰 수 있습니다. 이렇게 하면 클라이언트 코드는 개별 객체와 복합 객체를 구분할 필요 없이 동일한 방식으로 처리할 수 있어 훨씬 단순해집니다.

**간단한 코드 예시:**

```java
java코드 복사
// Component 인터페이스
interface Component {
    void operation();
}

// Leaf 클래스
class Leaf implements Component {
    public void operation() {
        // Leaf-specific operation
        System.out.println("Leaf operation");
    }
}

// Composite 클래스
class Composite implements Component {
    private List<Component> children = new ArrayList<>();

    public void add(Component component) {
        children.add(component);
    }

    public void remove(Component component) {
        children.remove(component);
    }

    public void operation() {
        for (Component child : children) {
            child.operation();
        }
    }
}

// 사용 예
public class CompositePatternDemo {
    public static void main(String[] args) {
        Component leaf1 = new Leaf();
        Component leaf2 = new Leaf();
        Composite composite = new Composite();
        composite.add(leaf1);
        composite.add(leaf2);

        composite.operation();
    }
}

```

이 예시에서 `Composite` 객체는 여러 `Component` 객체를 포함하고 있으며, 각각의 `Component`는 `operation()` 메서드를 호출합니다. 클라이언트 코드는 복잡한 트리 구조를 간단하게 처리할 수 있습니다. Composite 패턴을 사용하면 객체를 관리하고 확장하는 작업이 훨씬 쉬워집니다.
