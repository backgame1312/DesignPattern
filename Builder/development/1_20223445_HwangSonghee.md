```c++
#include <iostream>
#include <string>
using namespace std;

// 캐릭터 클래스
class Character {
    string name; //직업
    int level = 1; //레벨
    string weapon; //무기
    int weaponAttack = 1; //무기의 공격력
    string armor; //방어구
public:
    Character() = default;

    Character(const Character& other) {
        name = other.name;
        level = other.level;
        weapon = other.weapon;
        weaponAttack = other.weaponAttack;
        armor = other.armor;
    }
    // 멤버 함수
    void setName(const string newName) {
        name = newName;
    }
    void setLevel(int newLevel) {
        level = newLevel;
    }
    void setWeapon(const string& newWeapon, int newWeaponAttack) {
        weapon = newWeapon;
        weaponAttack = newWeaponAttack;
    }
    void setArmor(const string& newArmor) {
        armor = newArmor;
    }

    // 캐릭터 정보 출력 함수
    void showInfo() const {
        cout << "[캐릭터 정보]" << endl;
        cout << " *직업: " << name << endl;
        cout << " *레벨: " << level << endl;
        cout << " *무기: " << weapon << "(공격력:" << weaponAttack << ')' << endl;
        cout << " *방어구: " << armor << endl;
    }
```
*캐릭터 클래스*
* 해당 코드에서 ```character 클래스```는 게임 캐릭터의 속성을 나타내며, 각각의 멤버변수를 가지고 있다.
* 멤버 변수는 ```name```, ```level```, ```weapon```, ```weaponAttack```, ```armor```로 각각 직업, 레벨, 무기, 무기의 공격력, 방어구 변수이다.
* 또한 ```showInfo 함수```는 해당 캐릭터의 정보를 표시하는 함수이다.
* 해당 함수는 ```const``` 키워드를 사용하여 멤버 변수들을 읽기 전용으로 사용한다.
```c++
};
// 빌더 인터페이스
class CharacterBuilder {
public:
    virtual ~CharacterBuilder() {}
    virtual void setName(const string& name) = 0;
    virtual void setLevel(int level) = 0;
    virtual void setWeapon(const string& weapon, int weaponAttack) = 0;
    virtual void setArmor(const string& armor) = 0;
    virtual Character* getResult() = 0;
};
```
*빌더 인터페이스*
* 해당 코드에서 ```builder 클래스```는 순수 가상 함수들로 이루어진 추상 클래스이다.
* 이 인터페이스는 캐릭터 클래스에서 정의한 멤버 변수를 빌더에서 설정할 수 있도록 하고 있다.
* 각각 캐릭터의 직업을 설정하는 함수, 캐릭터의 레벨을 설정하는 함수, 캐릭터의 무기와 무기의 공격력을 설정하는 함수, 캐릭터의 방어구를 설정하는 함수이다.
* 이 빌더 클래스를 활용하여 다양한 종류의 캐릭터를 생성할 수 있다. 이를 위해서는 빌더 클래스를 상속받아 구체적인 빌더 클래스를 정의하여야 한다.
