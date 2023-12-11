# Player 클래스
```C++
class Player {
    string playerName;
    int playerDamage;

public:
    Player(string PN, int PD) : playerName(PN), playerDamage(PD) {}

    void attack(Spawner* spawner) {
        cout << playerName << "이(가) " << spawner->getName() << "에게 " << playerDamage << "의 데미지를 가했습니다!" << endl;
        spawner->receiveDamage(playerDamage);

        int remainingHealth = spawner->getHealth();

        if (remainingHealth <= 0) {
            cout << spawner->getName() << "이(가) 소멸되었습니다." << endl;
        }
        else {
            int remainingHealth = spawner->getHealth();
            cout << "공격력이 낮아 " << spawner->getName() << "이(가) 소멸되지 않았습니다." << endl;
            cout << "남은 체력: " << remainingHealth << endl;
        }
        delete spawner;
    }
};
```
- 플레이어 이름과 플레이어의 공격 데미지를 멤버 변수로 가지고 있다.
- 생성자로 플레이어의 이름과 공격 데미지를 초기화한다.
- ``attack`` 메서드는 공격 메서드로, 스포너에게 데미지를 입히고 상태를 출력한다.
  > 스포너의 채력이 0 이하이면 소멸 메세지를 출력하고, 그렇지 않으면 남은 체력을 표시한다.
- 마지막으로 생성된 스포너를 삭제한다.

# SpawnerPrototype 템플릿 클래스
찾아본 결과, 받은 코드가 프로토타입과 전략 패턴이 섞여있는 것을 확인했다. 그래서 템플릿 클래스인 ``SpawnerPrototype``을 새로 작성을 하고, 기존의 자식 클래스를 수정했다.
```C++
template <class T>
class SpawnerPrototype : public Spawner {
public:
    SpawnerPrototype(string name, int health) : Spawner(name, health) {}

    Spawner* clone() const override {
        return new T(*static_cast<const T*>(this));
    }

    void spawn() const override {
        cout << T::getSpawnMessage() << endl;
    }
};
```
- ``Spanwer`` 클래스를 상속는 클래스이다.
- ``T`` : 템플릿 매개변수로, 스포너의 구체적인 유형을 나타낸다.
- > ``SpawnerPrototype`` 클래스가 어떤 유형의 스포너를 생성할지를 나타낸다.
- 생성자는 스포너의 이름과 체력을 초기화한다.
- ``clone`` 메서드는 복제 메서드로, 현제 객체를 복제하여 반환한다.
  - override는 ``Spawner`` 클래스에 있는 ``clone`` 메서드를 오버라이드하겠다는 것을 명시적으로 나타낸다.
  - 현재 객체를 ``T`` 타입으로 캐스팅한 후 복제해서 반환한다.
- ``spawn`` 함수는 스포너의 스폰 메세지를 출력한다.
- - override는 ``Spawner`` 클래스에 있는 ``spawn`` 메서드를 오버라이드하겠다는 것을 명시적으로 나타낸다.
 
# 자식 클래스
### 수정 전 클래스
```C++
class ZombieSpawner : public Spawner {
    string nameZombie;
    double attackZombie;
public:
    ZombieSpawner(int count, string name, double attack)
        : Spawner(count), nameZombie(name), attackZombie(attack) {}

    Spawner* clone() const override {
        return new ZombieSpawner(*this);
    }
    void spawn() const override {
        cout << "좀비가 생성되었습니다!" << endl;
    }
};

class SpiderSpawner : public Spawner {
    string nameSpider;
    double attackSpider;
public:
    SpiderSpawner(int count, string name, double attack)
        : Spawner(count), nameSpider(name), attackSpider(attack) {}

    Spawner* clone() const override {
        return new SpiderSpawner(*this);
    }
    void spawn() const override {
        cout << "거미가 생성되었습니다!" << endl;
    }
};
```
### 수정 후 클래스 
```C++
class ZombieSpawner : public SpawnerPrototype<ZombieSpawner> {
public:
    ZombieSpawner(string name, int health = 15) : SpawnerPrototype(name, health) {}

    static string getSpawnMessage() {
        return "좀비가 스폰되었습니다!";
    }
};

class SkeletonSpawner : public SpawnerPrototype<SkeletonSpawner> {
public:
    SkeletonSpawner(string name, int health = 7) : SpawnerPrototype(name, health) {}

    static string getSpawnMessage() {
        return "스켈레톤이 스폰되었습니다!";
    }
};

class CreeperSpawner : public SpawnerPrototype<CreeperSpawner> {
public:
    CreeperSpawner(string name, int health = 12) : SpawnerPrototype(name, health) {}

    static string getSpawnMessage() {
        return "크리퍼가 스폰되었습니다!";
    }
};
```
- 거미 스포너를 삭제하고 스켈레톤 스포너로 변경했고, 크리퍼 스포너를 추가했다.
- 각각의 클래스는 ``SpawnerPrototype`` 템플릿 클래스를 상속받는다.
- 각각의 클래스는 템플릿 매개변수로 자기 자신을 전달한다.
- 각 클래스는 자체적인 스폰 메세지를 반환하는 정적 메서드 ``getSpawnMessage``를 가지고 있다.
