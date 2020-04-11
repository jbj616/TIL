## SOLID

- SRP : 단일 책임 원칙
- OCP : 개방 - 패쇄 원칙
- LSP : 리스코프 치환 원칙
- ISP : 인터페이스 분리 원칙
- DIP : 의존 관계 역전 원칙



### 단일 책임 원칙 (SRP)

**한 클래스는 변경에 대한 이유를 하나만 가져야 한다.**

- 클래스를 변경할 때 클래스를 변경하기 위한 다른 이유를 생각할 수 있다면 해당 클래스는 SRP를 위반한 것이다
- 책임은 '변경의 축'이다
  - 사람들이 모여서 운동, 독서 등 취미에 대한 이야기를 하고 있다 그러던 중 누군가가 영화에 대해 이야기를 하게 되었고 대화의 주제가 영화로 집중되기 시작한다. 영화의 내용, 영화 장면 등 영화에 이야기 하는 것이 강해져 이야기 주제를 영화와 취미로 나누는 것이 더 낫다. => 클래스 간에 코드가 이동하거나 결합체가 파괴됨.



### 개방-폐쇄 원칙(OCP)

**소프트웨어 아티팩트(artifact)는 확장에 대해 열려 있어야 하고, 수정에 대해서는 닫혀 있어야 한다.**

```javascript
function drawAllShapes(shapes){
  shapes.forEach((shape)=>{
    if(shape.type === 'circle'){
      drawCircle(shape);
    }else if (shape.type === 'square'){
      drawSquare(shape)
    }
  })
}
```

- 만약 Shape를 확장하는 도형이 생겼을 때 `drawAllShapes()` 함수를 수정해야한다. => **OCP 원칙 위반**

- 그렇다면 어떻게 해결할까?

  ```javascript
  class Circle {
    draw() {}
  }
  class Square {
    draw() {}
  }
  
  function drawAllShapes(shapes){
    shapes.forEach((shape)=>{
      shape.draw()
    })
  }
  ```

  - 위 소스 코드는 새로운 도형이 나와도 `drawAllshapes()` 코드가 변경되지 않는다. => **OCP 원칙 준수**



### 리스코프 치환 원칙(LSP)

> 여기서 원하는 것은 다음의 대체 속성과 같다. 타입 S의 각 객체 o1과 타입 T의 각 객체 o2가 있을 때, T로 프로그램 P를 정의했음에도 불구하고 o1이 o2로 치환될 때 P의 행위가 변하지 않으면 S는 P의 서브 타입이다. (바버라 리스코프)

**서브 타입은 그것의 기반 유형으로 치환 가능해야 한다.**

```java
public class Square extends Rectangle {
  public void setWidth(int width){
    this.width = width;
    this.height = height;
  }
  public void setHeight(int height){
    this.width = height;
    this.height = height;
  }
}
public class App {
  public void resize(Rectangle rectangle){
    rectangle.setWidth(10);
    rectangle.setHeight(5);
    int area = rectangle.getArea();
  }
}
```

- 위 코드의 문제점이 무엇일까?
  - Square 객체(정사각형)는 사각형에 포함된다. 하지만 App 클래스에서 실행한 메소드에서 `resize()`에 따르면 width과 height가 각각 다르게 설정된다. 
  - App 객체 관점에서 바라본 Rectangle은 모든 서브 타입이 동일하게 동작해야한다. 즉, Rectangle을 상속하고 있는 Square이 들어와도 대체 가능해야한다는 말이다. 왜냐하면 Sqaure과 Rectangle은 서로 서브 타입관계이기에 행위가 변하지 않아야하기 때문이다

- 어떻게 해결해야 할까?

  ```java
  abstract class Shape {
      public abstract int getArea();
  }
  ```

  상위 클래스가 공통적인 특징을 가지며 하위 클래스가 독자적?인 특성을 가지도록 코드를 작성해야한다.



### 인터페이스 분리 원칙(ISP)

**클라이언트는 자신이 사용하지 않는 메소드에 의존하도록 강제되어서는 안 된다.**



기존에 fly기능과 drive기능이 있는 드론을 예로 들어보자

```java
interface Drone {
  fly();
  drive();
}
```

만약에 새로운 기능에 jump라는 기능이 추가가된다면 

```java
interface Drone {
  fly();
  drive();
  jump();
}
```

기존에 있던 드론에게 영향을 끼칠 수 밖에없다!!

ISP 원칙에 따라 리팩토링 해보자

```java
interface FlyController {
  fly();
}
interface DriveController {
  drive();
}
interface DroneController implements FlyController, DriveController{
  fly();
  drive();
}
```

각 기능에 대해 작은 인터페이스를 추가하여 DroneController이 필요한 기능만 사용할 수 있고 향후 변경 사항에도 영향을 받지 않는다. 하지만 이 것은 SRP 원칙을 위반한다.

- DroneController를 구현하는 구현체는 하나 이상의 변경 사유를 포함하게 된다.

  - SRP 원칙에 따르면 어떻게 될까?

    ```java
    interface FlyController {
      fly();
    }
    class FlyManager implements FlyController {
      public void fly(){};
    }
    
    interface DriveController {
      drive();
    }
    class DriveManager implements DriveController {
      public void drive(){};
    }
    ```

    좀 더 작은 컨트롤러 API의 구현체로 만들어야한다



### 의존 관계 역전 원칙(DIP)

- **상위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안 된다. 둘 모두 추상화에 의존해야 한다.**
- **추상화는 구체적인 사항에 의존해서는 안 된다. 구체적인 사항이 추상화에 의존해야 한다.**



```java
public class Exporter {
  public Exporter(JdbcRepository repository, ZipCompressor compressor, CsvGenerator generator){
    ..
  }
  public void exporrt(){
    List<Item> items = repository.find();
    
    String csvFilePath = generator.generate(items);
    String compressed = compressor.zip(csvFilePath);
  }
}
```

- 위 코드의 문제점이 무엇일까?

  - Exporter 객체가 JdbcRepository, ZipCompressor 등 구체적인 클래스들에 직접 의존한다.

- 어떻게 해결해야 할까?

  ```java
  public Exporter(Repository repository, Compressor compressor, Generator generator){
      ..
    }
  ```

  - Exporter 객체를 Repository, Compressor 등 인터페이스에 의존하고 이는 구체적인 클래스에의해 구현되도록 한다. => Repository의 새로운 구현체를 추가해도 코드는 변경되지 않는다.

