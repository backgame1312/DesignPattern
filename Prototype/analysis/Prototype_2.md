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
