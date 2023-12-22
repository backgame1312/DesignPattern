# 프로토타입 예제 코드 분석
## 분석할 예제
```C++
#include <iostream>
#include <list>
#include <iterator>
#include <map>
using namespace std;

class Position {
  public : 
    Position() {}
    Position(int x, int y) { x_ = x; y_ = y; }
    int x_, y_;
};

class Graphic {
  public :
    virtual void Draw(Position& pos) = 0;
    virtual Graphic* Clone() = 0;
};

class Triangle : public Graphic {
  public :
    void Draw(Position& pos) {}
    Graphic* Clone() { return new Triangle(*this); }
};

class Rectangle : public Graphic {
  public : 
    void Draw(Position& pos) {}
    Graphic* Clone() { return new Rectangle(*this); }
};

class GraphicComposite : public Graphic {
  public : 
    void Draw(Position& pos) {}
    Graphic* Clone() { 
      GraphicComposite *pGraphicComposite = new GraphicComposite(*this);
      list<Graphic*>::iterator iter1;
      list<Graphic*>::iterator iter2;

      iter2 = pGraphicComposite->components_.begin();
      for (iter1 = components_.begin(); iter1 != components_.end(); iter1++) {
        Graphic* pNewGraphic = (*iter1)->Clone();
        *iter2 = pNewGraphic;
        iter2++;
      }

      return pGraphicComposite;
    }

  private : 
    list<Graphic*> components_;
};

class Document {
  public :
    void Add(Graphic* pGraphic) {}
};

class Mouse {
  public :
    bool IsLeftButtonPushed() { 
      static bool isPushed = false;
      // -- GUI 함수 활용 Left Button 상태 체크
      isPushed = ! isPushed;
	  return isPushed;
    }

    Position GetPosition() {
      Position pos;
      // -- GUI 함수 활용 현재 마우스 위치 파악
      return pos;
    }
};

Mouse _mouse; // -- Global Variable

class GraphicEditor {
  public : 
    void AddNewGraphics(Graphic* pSelected) {
      Graphic* pObj = pSelected->Clone();
      while (_mouse.IsLeftButtonPushed()) {
        Position pos = _mouse.GetPosition();
        pObj->Draw(pos);
      }

      curDoc_.Add(pObj);
    }

  private :
    Document curDoc_;
};

class Palette {
  public :
    Palette() {
      Graphic* pGraphic = new Triangle;
      items_[1] = pGraphic;

      pGraphic = new Rectangle;
      items_[2] = pGraphic;

      // -- 필요한 만큼 등록 
    }

    void RegisterNewGraphic(Graphic* pGraphic) {
      items_[items_.size()+1] = pGraphic;
    }

    Graphic* GetSelectedObj() {
      return items_[GetItemOrder()];
    }

    int GetItemOrder() {
      int i=1;
      Position curPos = _mouse.GetPosition();
      // -- 현재 마우스 위치가 몇 번째 항목을 지정하는 지 판별
      return i;
    }

  private :
    map<int, Graphic*> items_;
};

void
main()
{
  Palette palette;
  GraphicEditor ged;

  ged.AddNewGraphics(palette.GetSelectedObj());
}
```

### Position 클래스
```C++
class Position {
public:
    Position() {}
    Position(int x, int y) { x_ = x; y_ = y; }
    int x_, y_;
};
```
- 좌표를 나타내는 멤버 변수 x_, y_가 선언 됨
- 기본 생성자는 아무런 매개변수를 받지 않고 객체를 생성함
- > 아무런 초기화를 수행하지 않고, 기본적으로 x_, y_를 0으로 초기화됨
- 매개변수가 있는 생성자는 x와 y좌표를 받아 객체를 초기화함
- 이 클래스는 단순히 좌표를 표현하는데 사용되며, 좌표를 초기화하거나 가져오기 위한 간단한 인턴페이스를 제공

### Graphic 클래스
```C++
class Graphic {
public:
    virtual void Draw(Position& pos) = 0;
    virtual Graphic* Clone() = 0;
};
```
- ``Graphic`` 클래스는 추상화를 통해 여러 그래픽 객체를 효과적으로 다룰 수 있도록 하며, 파생 클래스들은 이를 상속하여 특정 그래픽 유형의 동작을 정의할 수 있다.
-  ``Draw`` 순수 가상 함수는 파생 클래스에서 구현되어야 하는 메서드로, 주어진 위치에 그래픽을 그리는 역할을 한다.
- ``Clone`` 메서드는 또 다른 순수 가상 함수로, 파생 클래스에서 구현되어야 하는 메서드이며, 그래픽 객체의 복제본을 만들어 반환한다.
  > 순수 가상 함수는 이 클래스가 추상 기본 클래스임을 나타내고, 파생 클래스에서 이를 반드시 구현해야한다.
- ``Graphic`` 클래스는 다양한 그래픽 객체를 표현하는 데 사용된다.
  > 파생 클래스에서 그래픽 객체의 특정 유형을 구현하고 ``Draw`` 및 ``Clone`` 함수를 각각 해당 객체의 렌더링 및 복제에 맞게 구현해야한다.
- 해당 클래스는 추상 클래스로써, ``Graphic`` 클래스를 상속하는 다양한 구현 클래스들은 동일한 인터페이스를 제공하지만, 각자의 특성에 맞게 행동한다.
  > 다형성을 통해 여러 그래픽 유형을 일관된 방식으로 다룰 수 있다.
- ``Clone`` 함수가 ``Graphic*``를 반환하므로, 파생 클래스에서 이를 오버라이드할 때에도 적절한 파생 클래스의 포인터를 반환해야한다.

### Triangle 클래스와 Rectangle 클래
```C++
class Triangle : public Graphic {
public:
    void Draw(Position& pos) {}
    Graphic* Clone() { return new Triangle(*this); }
};
```
```C++
class Rectangle : public Graphic {
public:
    void Draw(Position& pos) {}
    Graphic* Clone() { return new Rectangle(*this); }
};
```
- ```Trangle`` 클래스와 ``Rectangle`` 클래스는 ``Graphic`` 클래스를 public으로 상속하여 메서드들을 오버라이드 한다.
  > 이들은 각각 삼각형과 사각형을 그리는 구현이 필요하다.

### GraphicComposite 클래스
```C++
class GraphicComposite : public Graphic {
public:
    void Draw(Position& pos) {}
    Graphic* Clone() {
        GraphicComposite* pGraphicComposite = new GraphicComposite(*this);
        list<Graphic*>::iterator iter1;
        list<Graphic*>::iterator iter2;

        iter2 = pGraphicComposite->components_.begin();
        for (iter1 = components_.begin(); iter1 != components_.end(); iter1++) {
            Graphic* pNewGraphic = (*iter1)->Clone();
            *iter2 = pNewGraphic;
            iter2++;
        }

        return pGraphicComposite;
    }

private:
    list<Graphic*> components_;
};
```
- ``GraphicComposite`` 클래스는 복합 그래픽 객체를 나타내기 위한 클래스로, 여러 개의 그래픽 객체를 리스트로 관리하고, ``Draw`` 및 ``Clone`` 메서드를 통해 이를 조작할 수 있도록 한다.
- ``GraphicComposite`` 클래스는 ``Graphic`` 클래스를 상속하여 메서드들을 오버라이드한다.
- > 현재 비어 있는 상태로, 실제로 복합 그래픽을 그리는 구현이 필요하다.
- 멤버 변수는 ``Graphic`` 포인터를 저장하는 리스트로, 복함 그래픽을 구성하는 각각의 그래픽 객체를 나타낸다.
- ``GraphicComposite`` 클래스는 여러 개의 그래픽 객체를 조합하여 하나의 그래픽으로 다루기 위한 클래스이다.

### Document 클래스
```C++
class Document {
public:
    void Add(Graphic* pGraphic) {}
};
```
- ``Document`` 클래스는 주로 여러 그래픽 객체를 관리하고 추가하는 데 사용될 것으로 예상된다.
  > 현재 구현은 그래픽 객체를 추가하는 메서드만을 가지고 있으며, 이를 실제로 어떻게 처리할지에 대한 구체적인 로직은 나중에 추가되어야 한다.
- ``Add`` 메서드는 문서에 그래픽 객체를 추가하는 메서드이다.
  > 그래픽 객체의 포인터를 받아 문서에 추가한다.
- ``Document`` 클래스는 여러 그래픽 객체들을 담는 그래픽 문서를 나타낸다.
- 다른 클래스, 예를 들면 ``GraphicEditor`` 클래스에서 ``Add`` 메서드를 활용하여 그래픽을 문서에 추가할 수 있다.

### Mouse 클래스
```C++
class Mouse {
public:
    bool IsLeftButtonPushed() {
        static bool isPushed = false;
        isPushed = !isPushed;
        return isPushed;
    }

    Position GetPosition() {
        Position pos;
        return pos;
    }
};
```
- ``Mouse`` 클래스는 실제 하드웨어 마우스 동작을 시뮬레이션하기 위한 간단한 클래스이다.
  > 현재의 구현은 버튼의 토굴 및 마우스 위치 반환을 처리하고 있고, 상황에 따라 이를 활용하는 부분이 다른 클래스에서 구현되어 있을 것이다.
- ``IsLeftButtonPushed`` 메서드는 왼쪽 마우스 버튼의 상태를 토글하는 메서드로, 버튼이 눌린 상태에서 호출되면 버튼이 해제되고, 버튼이 해제된 상태에서 호출되면 버튼이 눌린다.
- ``GetPosition`` 메서드는 현재 마우스의 위치를 반환하는 메서드로, 현재는 빈 ``Position`` 객체를 반환하고 있다
- 멤버 변수는 왼쪽 마우스 버튼의 상태를 나타내는 정적 Bool형 변수로, ``IsLeftButtonPushed`` 메서드에서 토글된다.
- ``Mouse`` 클래스는 간단한 GUI 상호 작용을 시뮬레이션하기 위한 클래스로 보이고, 현재는 버튼의 상태 및 마우스 위치를 제공한다.

### GraphicEditor 클래스
```C++
class GraphicEditor {
public:
    void AddNewGraphics(Graphic* pSelected) {
        Graphic* pObj = pSelected->Clone();
        while (_mouse.IsLeftButtonPushed()) {
            Position pos = _mouse.GetPosition();
            pObj->Draw(pos);
        }

        curDoc_.Add(pObj);
    }

private:
    Document curDoc_;
};
```
- ``GraphicEditor`` 클래스는 그래픽 편집과 관련된 주요 작업을 처리하는 클래스이다.
  > 현재 구현에서 마우스 상태를 감지하고, 선택된 그래픽을 클론하여 계속해서 그리며, 이를 문서에 추가하는 작업이 이루어지고 있다.
- ``AddNewGraphics`` 메서드는 새로운 그래픽을 추가하는 메서드로, 선택된 그래픽을 기반으로 새로운 객체를 만들고, 마우스 버튼이 눌린 동안 해당 객체를 계속해서 그리며 문서에 추가한다.
- 멤버 변수는 현재 편집 중인 그래픽을 담고 있는 문서 객체이다.
- ``AddNewGraphics`` 메서드에서는 전역 변수 ``_mouse``를 사용하여 마우스 버튼 상태 및 위치를 확인하는데, 선택된 그래픽을 클론하고, 해당 그래픽을 마우스 버튼이 눌린 동안 계속해서 그리며 문서에 추가한다.

### Palette 클래스
```C++
class Palette {
public:
    Palette() {
        Graphic* pGraphic = new Triangle;
        items_[1] = pGraphic;

        pGraphic = new Rectangle;
        items_[2] = pGraphic;

        // -- 필요한 만큼 등록 
    }

    void RegisterNewGraphic(Graphic* pGraphic) {
        items_[items_.size() + 1] = pGraphic;
    }

    Graphic* GetSelectedObj() {
        return items_[GetItemOrder()];
    }

    int GetItemOrder() {
        int i = 1;
        Position curPos = _mouse.GetPosition();
        // -- 현재 마우스 위치가 몇 번째 항목을 지정하는 지 판별
        return i;
    }

private:
    map<int, Graphic*> items_;
};
```
- ``Palette`` 클래스는 그래픽 객체들을 등록하고 선택하는 데 사용되는 클래스이다.
  > 현재 구현에서는 초기에 삼각형과 직사각형을 등록하고, 마우스 위치에 따라 항목을 선택하는 동작을 수행한다.
- 생성자에서는 초기에 등록된 삼각형과 직사각형을 팔래트에 추가한다.
  > 이를 위해 그래픽 객체를 생성하고, ``items_`` 맵에 등록한다.
- ``RegisterNewGraphic`` 메서드는 새로운 그래픽 객체를 팔레트에 등록하는 메서드이며, 등록된 순서는 현재 ``items_`` 맵의 크기에 1을 더한 값으로 결정된다.
- ``GetSelectedObj`` 메서드는 현재 선택된 그래픽 객체를 반환하며, 이는 현재 마우스 위치에 따라 팔레트에서 선택된 항목을 가져오는데 사용된다.
- ``GetItemOrder`` 메서드는 현재 마우스 위치를 기반으로 선택된 팔레트 항목의 순서를 결정하며, 항상 1을 반환하고 있다.
- 멤버 변수는 팔레트에 등록된 그래픽 객체들을 젖아하는 맵이며, 각 항목은 정수 키를 가지고, 해당 키로 그래픽 객체에 접근할 수 있다.
- ``_mouse`` 전영 변수를 사용하여 현재 마우스의 위치를 얻어오고, 이를 기반으로 팔레트 항목을 선택하는 데 활용한다.

### main 함수
```C++
void main() {
    Palette palette;
    GraphicEditor ged;

    ged.AddNewGraphics(palette.GetSelectedObj());
}
```
- ``Palette`` 클래스의 객체를 생성하며, 이 객체는 프로그램에서 그래픽 객체들의 팔레트를 관리한다.
- ``GraphicEditor`` 클래스의 객체를 생성하며, 이 객체는 그래픽을 편집하고 문서에 추가하는 기능을 제공한다.
- ``AddNewGraphics`` 메서드를 호출하여 현재 선택된 그래픽 객체를 기반으로 새로운 그래픽을 추가하는 역할을 한다.
- ``AddNewGraphics`` 메서드를 호출하면서 선택된 그래픽 객체를 전달하며, 이 메서드 내에서는 마우스 버튼이 눌린 동안 해당 객체를 계속해서 그리며 문서에 추가한다.
- ``GetSelectedObj`` 메서드를 호출하여 현재 선택된 그래픽 객체를 얻어온다.
- ``main`` 함수의 끝에 도달하면 프로그램이 종료되며 이 때, 반환 타입이 ``void``이므로 별도의 반환 값은 주어지지 않는다.
