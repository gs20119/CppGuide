# C에서 ++된 기능들

### 아직 C++이 어색한 사람들을 위하여

자세한 내용은 [https://modoocode.com/](https://modoocode.com/) 참고

---

## 네임스페이스

```cpp
#include <iostream>
using namespace std;
namespace hello{ int foo(){ return 3; } }

int main(){
  cout << hello::foo();
}
```

네임스페이스를 사용하는 이유는 불러운 모듈과 작성 중인 프로그램과 메소드명이 충돌하지 않기 위함이다. 두번째 줄을 사용하면 std 네임스페이스를 생략할 수 있다.

---

## 레퍼런스

기존의 포인터를 이용한 변수 전달

```cpp
#include <iostream>
using namespace std;
void change(int* p){ *p = 3; return; }

int main(){
  int num = 5; cout << num << endl; // num = 5
  change(&num); cout << num << endl; // num = 3
  return 0;
}
```

레퍼런스를 이용한 변수 전달

```cpp
#include <iostream>
using namespace std;
void change(int& p){ p = 3; return; }
void _say_(const int& p){ cout << p+2 << endl; return; }
void say(int p){ p += 2; cout << p << endl; return ; }

int main(){
  int num = 5; cout << num << endl; // num = 5
  change(num); cout << num << endl; // num = 3
  _say_(2); say(num); // num = 3
  return 0;
}
```

레퍼런스를 사용하면 두 변수가 하나의 객체(메모리)를 가리키도록 만들 수 있다. 메소드의 매개변수에서 사용하기 용이하지만, 대신 상수를 대입할 수 없다.

const 레퍼런스를 사용하면 매개변수에 상수를 넣을 수 있고, 매개변수는 읽기 전용이 되어 실수로라도 바뀌지 않는다. 차라리 레퍼런스를 사용하지 않으면 되지 않나? 그럼에도 const 레퍼런스가 자주 사용되는 이유는 메소드 호출 시 거대한 객체를 새로 복사하면서 메모리를 낭비할 일이 없기 때문이다. 객체의 원본을 실시간으로 보여주면서 불필요한 변경이나 복사를 원하지 않을 때 사용하면 좋다.

---

## 메모리 동적할당 new와 delete

```cpp
#include <iostream>
using namespace std;

int main(){
  int* p = new int; 
  *p = 3; cout << *p << endl; 
  delete p; 
  p = nullptr; // optional
  
  int* list = new int[5]; // list = array of int
  for(int i=0; i<5; i++) list[i] = 2*i+3;
  delete[] list;
  
  int* plist[5]; // plist = array of int*
  for(int i=0; i<5; i++){ plist[i] = new int; *plist[i] = 2*i+3; }
  for(int* p : plist){ cout << *p << " "; delete p; } // = for every p in plist
  return 0;
}
```

변수 초기화시 스택에 자동으로 정적 할당된 메모리는 운영체제가 알아서 변수 사용이 끝나면 회수해가지만, 힙에 프로그래머가 new로 직접 동적 할당한 메모리는 delete로 해제해주지 않으면 프로그램이 끝나기 전까지 회수되지 않는다.

```cpp
char *p;
for(int i=0; i<1000000; i++)
  p = new char[1024];
```

예를 들어, 다음 코드는 짧은 문자열을 반복해서 초기화시킬 뿐이지만 무려 1GB의 메모리 누수를 발생시킨다. new를 사용할 때에는 메모리 관리에 신경써야 한다.

---

## 객체지향 기본

```cpp
#include <iostream>
using namespace std;
class Animal{
  private: 
    static int total; // static variable
    int food; // cannot access this.food
    int weight;
  public:
    Animal(int f = 0, int w = 50) 
      : food(f), weight(w) { total++; }
    static void count(){ cout << total << endl; }
    void set(int f, int w){ food = f; weight = w; }
    void feed(int df); // defined outside
    void view() const{ // this method doesn't change anything
      cout << food << " " << weight << endl; 
    }
};
int Animal::total = 0;
void Animal::feed(int df){ food += df; weight += (df/3); }

int main(){
  Animal anim(30,40); // implicit
  Animal anim_ = Animal(30,40); // explicit
  anim.set(100,50);
  anim.feed(30);
  anim.view();
  Animal::count(); // call static method
  return 0;
}
```

몇가지 유용한 기능이 있다. 우선, 생성자에 한해서 코드와 같이 클래스의 필드 변수에 간편하게 초기값을 대입할 수 있다. 이를 초기화 리스트라고 하며, 이 방법으로 대입이 불가능한 상수(const) 필드 변수도 생성자를 통해 초기화시킬 수 있다.

또한, 클래스 내부 메소드를 대강 선언하고 외부에서 정의할 수 있다. 클래스의 선언부를 깔끔하게 유지하고 싶다면 사용하는 것이 좋다.

---

## 함수의 오버로딩

```cpp
#include <iostream>
using namespace std;
int add(int a, int b){ return a+b; } // doesn't make error
int add(int a, int b, int c){ return a+b+c; }
float add(float a, float b){ return a+b; }

int main(){
  cout << add(1,2) << endl;
  cout << add(2.0f, 3.0f) << endl;
  cout << add(1,3,9) << endl;
  return 0;
}
```

매개변수 칸이 달라야 한다는 것이 오버로딩의 핵심이다.

---

## 객체지향과 동적 할당: 복사 생성자와 소멸자

```cpp
#include <iostream>
class Tank{
  private:
    int hp, x, y;
    const int damage; 
    char* name;
  public:
    Tank(int x_, int y_, const char* name_);
    Tank(const Tank& T); // copy constructor
    void move(int dx, int dy);
    void attack(Tank* opponent);
    ~Tank(); // destructor
};
int Tank::total = 0; 
Tank::Tank(int x, int y, const char* name_)
  : x(x), y(y), hp(50), damage(5){ 
  name = new char[strlen(name_)+1];
  strcpy(name, name_);
}
Tank::Tank(const Tank& T)
  : x(T.x), y(T.y), hp(T.hp), damage(T.damage){
  name = new char[strlen(T.name)+1];
  strcpy(name, T.name);
}
void Tank::move(int dx, int dy){ x += dx; y += dy; }
void Tank::attack(Tank* opponent){ opponent->hp -= damage; }
Tank::~Tank(){ if(name != NULL) delete[] name; }
```

일반적인 객체에서도 동적 할당을 사용할 수 있다. 객체를 생성할 때 생성자가 호출되며, 동적 할당을 해제할 때는 소멸자가 호출된다. 객체 내에서 수거해야 하는 동적 할당된 메모리를 직접 처리하면 된다. 또, 다른 객체를 레퍼런스로 받아오는 생성자를 복사 생성자라고 하며, 단순히 등호 대입 연산을 쓰면 호출할 수 있다.

```cpp
int main(){
  Tank* tanks[2]; // array of tank pointers
  tanks[0] = new Tank(2,3,"Tanky");
  tanks[1] = new Tank(3,5,"Tonky");
  Tank copyTank = *(tanks[1]); // activate copy constructor
  tanks[0] -> move(3,2); // equal to (*tanks[0]).move(3,2)
  tanks[1] -> attack(tanks[0]);
  for(int i=0; i<2; i++) delete tanks[i];
}
```

실제로 사용할 때는 객체의 포인터에 new와 생성자로 객체를 동적 할당해주고, 사용이 완료되면 delete로 메모리를 해제해주면 된다. 복사 생성자와 소멸자는 필수는 아니지만 필요할 때가 생길 수 있다. 특히 객체 내부에 동적 할당되는 구성요소나 객체가 있을 때 잘 정의해두지 않으면 사용에 문제가 발생하게 된다.

---

## 레퍼런스를 리턴하는 메소드

```cpp
#include <iostream>
using namespace std;
class A{
  private:
    int x;
  public:
    A(int c) : x(c) {}
    A& object(){ return *this; }
    int& access(){ return x; }
    int read(){ return x; }
    void show(){ cout << x << endl; }
};

int main(){
  A a(5); a.show();
  int& c = a.access(); 
  c = 4; a.show(); // success a.x = 4
  // int& d = a.get(); // error
  int d = a.access();
  d = 3; a.show(); // fail a.x = 3
}
```

클래스를 사용하는 경우 객체 내부의 필드 변수를 밖에 내보내서 편집해야 할 때가 생긴다. 그럴 때 메소드의 반환형을 레퍼런스로 설정해두면 객체에서 내보낸 필드 변수를 외부에서 직접 접근할 수 있게 된다.

자기 자신을 반환하는 메소드에도 레퍼런스는 유용하게 사용된다. 여기에서 this는 객체 자신을 가르키는 포인터를 나타내는 키워드이다.

---

## 연산자 오버로딩

```cpp
#include <iostream>
using namespace std;
class MyPair{
  private:
    int a, b;
  public:
    MyPair(int a, int b): a(a), b(b) {}
    MyPair& operator=(const MyPair& pr);
    MyPair operator+(const MyPair& A, const MyPair& B) ;
    bool operator==(const MyPair& A, const MyPair& B);
    int& operator[](const int index);
};
MyPair& MyPair::operator=(const MyPair& pr){
  a = pr.a; b = pr.b;
  return *this;
}
int& MyPair::operator[](const int index){
  if(index < 0 or index > 1) return 0;
  return (index == 0) ? a : b;
}
MyPair MyPair::operator+(const MyPair& A, const MyPair& B){
  MyPair temp(A.a+B.a, A.b+B.b);
  return temp;
}
bool MyPair::operator==(const MyPair& A, const MyPair& B){
  return (A.a==B.a)&&(A.b==B.b);
}
```

오버로딩은 사칙연산 연산자들에게도 적용된다. 매개변수로 상수 레퍼런스를 사용하며, 대입이나 원소 접근 연산의 경우 레퍼런스를 리턴하도록 한다.

```cpp
int main(){
  MyPair A(1,2), B(2,3);
  cout << (A+B)==(B+A) << endl;
  MyPair C = A+B+A;
  C[1] = 4; cout << C << endl;
  return 0;
}
```

---

## 객체지향: 상속과 다형성

객체지향의 꽃은 상속이다. 다음은 상속과 오버라이딩의 예시를 보여준다.

```cpp
#include <bits/stdc++.h>
using namespace std;
class Base{
  private:
    int x, y;
  protected:
    int z;
    void tell(){ cout << x << " " << y << " " << z << endl; }
  public:
    Base(int x) : x(x), y(x+2), z(2*x) {}
    void call(){ cout << x << endl; }
    
};
class Child : public Base{ 
  private:
    int x, y;
  public:
    Child(int x, int y) : Base(x+y), x(x), y(y) {} // construct parent first
    void call(){ // overriding
      cout << x << " " << y << " " << z << endl; 
      tell(); // access protected parent method
    }
};
```

자식 클래스가 부모 클래스를 물려받을 때 공개 범위를 설정할 수 있다. 크게 세 가지 옵션이 있다. private으로 물려받으면 자식을 통해 부모의 멤버에 접근할 수 없게 되며, protected로 물려받으면 자식의 멤버 메소드만이 부모의 protected와 public 멤버에 접근 가능하며, public으로 물려받으면 외부에서도 자식 객체를 통해 부모의 public 멤버에 접근할 수 있게 된다.

```cpp
int main(){
  Base b(1); b.call(); // 1
  Child c(2,3); 
  c.call(); // 2 3 10 then 5 7 10 (overriding)
  c.tell(); // error (protected)
  Base* p_c = new Child(2,3); // polymorphism (up casting)
  Base& cc = c; // polymorphism with reference
  p_c -> call(); // 5 (not overriding)
  delete p_c;
  return 0;
}
```

다형성도 상속과 함께 객체지향 프로그래밍의 필연적인 요소 중 하나이다. 다형성은 public으로 부모를 상속했을 때만 사용할 수 있다. 다형성의 장점은, 부모 클래스 포인터가 임의의 자식 클래스 객체를 가리킬 수 있다는 점이다.

```cpp
#include <bits/stdc++.h>
using namespace std;
class Base{
  private:
    int x, y;
  public:
    Base(int x) : x(x), y(x+2) {}
    virtual void call(){ cout << "Base " << x << endl; }
    void tell(){ cout << "Base " << x << " " << y << endl; }
};
class Child : public Base{ 
  private:
    int x, y;
  public:
    Child(int x, int y) : Base(x+y), x(x), y(y) {} // construct parent first
    void call() override{ cout << x << " " << y << endl; }
    void tell(){ cout << x << " " << y << endl; }
};

int main(){
  Base* p_b = new Base(3);
  p_b -> call(); // Base 3 
  p_b -> tell(); // Base 3 5
  Base* p_c = new Child(4,5);
  p_c -> call(); // 4 5 
  p_c -> tell(); // Base 9 11
  delete p_b, p_c;
  return 0;
}
```

다형성을 통해 정의된 자식 객체는 부모 클래스 멤버에 자유롭게 접근이 가능한 대신, 오버라이드가 일어나지 않는다. 오버라이드를 원한다면 부모 멤버 메소드에 virtural 키워드를 앞에 붙여주면 된다. 버추얼 메소드를 오버라이드하는 자식 멤버 메소드에 override 키워드를 붙여서 명시해줄 수 있다.

다형성을 사용하는데 각 클래스의 소멸자가 필요한 경우 반드시 부모 클래스의 소멸자를 virtual로 선언해주어야 메모리 누수가 발생하지 않는다.

---

## 매력적인 템플릿과 함수객체, 함수포인터

다음 코드는 템플릿의 기능을 이용하여 임의 타입의 벡터를 구현한 것이다.

```cpp
#include <bits/stdc++.h>
using namespace std;
template <typename T>
class Vector{
  private:
    T* data;
    int cap, len;
  public:
    Vector(int n=1) : data(new T[n]), cap(n), len(0) {}
    Vector(const Vector& v){
      len = cap = v.size();
      data = new T[cap];
      for(int i=0; i<len; i++) data[i] = v[i];
    }
    T& operator[](int i){ return data[i]; }
    int size(){ return len; }
    void swap(int i, int j){
      T temp = data[i]; 
      data[i] = data[j];
      data[j] = temp;
    }
    void push_back(T s){
      if(cap == len){
        T* bag = new T[cap*2];
        for(int i=0; i<len; i++) bag[i] = data[i];
        delete[] data;
        data = bag;
      }data[len++] = s;
    }
    ~Vector(){ if(data) delete[] data; }
};
```

이렇게 클래스나 구조체, 함수의 선언부 앞에 템플릿을 지정해주면 형태를 간편하게 일반화시킬 수 있다. 구현하고자 하는 것이 추상적일 때 사용하면 좋다. 이때 키워드 typename은 '임의의 타입 T'를 의미한다.

```cpp
int main(){
  Vector<int> v(2);
  v[0] = 2; v[1] = 3;
  v.push_back(4);
  v.push_back(-1);
  for(int i=0; i<v.len(); i++) 
    cout << v[i] << " ";
  Vector<string> d();
  d.push_back("qwer");
  d[0] = "asdf";
  return 0;
}
```

함수, 클래스는 기본적으로 하나의 틀이다. 이 틀의 형태를 프로그래머가 원하는 범위 안에서 일반화시키고 싶을 때 생성자와 변수를 추가해도 해결할 수 있지만, 템플릿을 사용하면 그 의미를 간결하고 명확하게 표현할 수 있다.

```cpp
template <int N, typename T> 
T say(const T& a){ 
  cout << N << " " << a << endl;
  return a;
}

template <int N> // specification (T = int)
int say(const int& a){
  cout << N << "." << a << endl;
  return a;
}

int main(){
  say<20119>(3.91); // double : 20119 3.91
  say<20118>(18); // int : 20118.18
}
```

템플릿을 선언할 때 꼭 타입만 들어가지는 않고, (객체가 아닌) 변수도 속성으로 넣을 수도 있고, 템플릿 안에 여러 속성들을 연달아 넣을 수도 있다.

함수의 매개변수에 템플릿을 적용한 경우 함수 사용 시 타입명을 언급하지 않아도 알아서 인식해준다. 또한, 템플릿에서 가상으로 정의한 타입명 대신에 특정 타입을 집어넣어 다시 정의하면 ‘템플릿 특수화’가 적용되어 오버로딩처럼 작용한다.

```cpp
template <typename T> // Functor : use object as function
struct Comp_Struct{ 
  bool operator()(const T& a, const T& b){ return a>b; }
};

template <typename T> // just common function
bool Comp_Func(const T& a, const T& b){ return a<b; }

template <typename Cont, typename Comp>
void bubble_sort(Cont& cont, Comp& comp){
  for(int i=0; i<cont.size(); i++)
    for(int j=i+1; j<cont.size(); j++)
      if(!comp(cont[i],cont[j])) cont.swap(i,j);
}

int main(){
  Vector<int> v(5);
  v[0]=2; v[1]=3; v[2]=1; v[4]=5; v[5]=4;
  
  Comp_Struct<int> comp; // using functor
  bubble_sort(v,comp); 
  for(int i=0; i<v.size(); i++) cout << v[i];

  bool (*Cmp)(const int&, const int&) = Comp_Func<int>; 
  cout << Comp_Func<int>(23,57) << Comp_Func<int>(8,2) << endl;
  cout << Cmp(23,57) << Cmp(8,2) << endl; // function pointer
  bubble_sort(v,Cmp); // = bubble_sort(v,Comp_Func<int>)
  for(int i=0; i<v.size(); i++) cout << v[i];
}
```

위 코드는 클래스에 () 연산자를 정의하여 함수처럼 사용하는 잔기술인 ‘함수객체‘와, 함수를 정의한 뒤 ’함수포인터‘를 사용하는 방식을 단적으로 보여준다. 함수를 선언했을 때 함수의 이름은 함수포인터가 된다. 함수 포인터에 소괄호와 매개변수를 붙여주면 함수가 작동하는 방식이다. 이런 개념이 있기 때문에 코드처럼 함수 자체를 매개변수로 넘겨주거나, 함수를 복사하는 등의 행위가 가능하다.

---

## 가변 길이 템플릿

매개변수 개수가 자유로운 함수를 템플릿으로 구현할 수 있다.

```cpp
#include <iostream>
using namespace std;

template <typename T> // base
void print(T arg){ cout << arg << endl; }

template <typename T, typename... Types> 
void print(T arg, Types... args){
  cout << arg << ", ";
  print(args...); // recursive 
}

int main(){
  print(1, 3.1, "abc");
  print(1, 2, 3, 4, 5, 6, 7);
}
```

이 코드에서 세 점은 함수 파라미터 팩이라고 부르며, 0개 이상의 매개변수를 나타낸다. 템플릿에 파라미터 팩을 사용하는 경우 재귀를 기본으로 하기 때문에 맨 앞의 변수를 단일 매개변수로 따로 꺼내주고, args가 비어있을 때 호출할 기본함수를 앞에 함께 선언해주어야 함수가 정상적으로 종료될 수 있다.

---

## 모르면 손해 보는 C++ STL

표준으로 제공하는 라이브러리인 STL에서는 크게 다음 세가지를 제공한다.

객체를 보관하고 편집할 수 있는 **자료구조**

   문자열 string / 순서쌍 pair, tuple / 선형 자료구조 vector, list, stack, queue, deque

   셋과 맵 set, map, unordered_set, unordered_map 등

자료구조와 반복자로 작업을 수행하는 **알고리즘**

   정렬 sort, stable_sort, partial_sort

   원소 탐색 find, find_all,

   원소 제거 remove, remove_if 등

자료구조의 원소에 접근할 수 있는 **반복자**는 타입 iterator으로 정의되며 STL 자료구조에서 포인터처럼 사용된다.

```cpp
#include <bits/stdc++.h>
using namespace std;
template <typename T>
void print(vector<T>& vec){ // reference (without copy)
  typename vector<T>::iterator itr; // iterator
  for(T x : vec) cout << x << " "; 
  cout << endl;
  for(itr = vec.begin(); itr != vec.end(); ++itr) 
    cout << *itr << " "; // *itr = element of vector
}
```

위 코드에서 사용한 '범위기반 반복문'은 컨테이너 v의 모든 원소 x에 대해 반복하라는 의미가 담겨있다. STL 자료구조 뿐만 아니라 포인터로 선언하지 않은 배열에서도 사용할 수 있다.

반복자로 STL 자료구조의 위치에 접근하려면 해당 라이브러리에서 제공하는 메소드를 사용하면 된다. 특정 조건을 만족하는 위치를 찾아 반복자를 리턴하는 메소드가 있다. 가장 대표적인 건 처음과 끝을 의미하는 begin()과 end(). 범위를 벗어나지 않을만큼 반복자에 정수를 더하거나 빼면 인덱스가 몇 칸 옮겨진 위치로 이동한다.

여담으로 반복자 타입명 앞에 typename을 적어준 이유는 A::B 형태에서 A(=vector<T>)가 정확하게 뭔지 알 수 없으므로 타입명이 아니라 클래스 A의 평범한 멤버일 가능성이 있기 때문이다. typename은 템플릿 선언에도 사용되지만 뒤에 나오는 단어가 타입명임을 명시하는 기능도 한다.

```cpp
bool even(int a){ return (a%2)==0; }
int main(){
  vector<int> v({1, 2, 3, 4, 5, 6, 7, 8, 9, 10});
  v.insert(v.begin()+5, 11); // insert(pos,value)
  v.insert(v.end()-2, 23);
  v.push_back(32);
  v.erase(remove_if(v.begin(),v.end(),even),v.end()); // erase all even numbers
  for(int x : v) cout << x << " "; 
}
```

삽입과 삭제가 빈번하게 일어나는 코드에서 반복자를 사용할 때는 주의를 기울여야 한다. 가급적 반복자를 직접 조작하기보다는 STL 알고리즘을 찾아 적극적으로 활용하는 것이 좋다. remove_if는 true로 판정되는 원소들을 무시하고 모두 앞으로 당기며, erase는 주어진 구간의 원소를 지운다.

---

## 이름이 없는 일회용 함수 람다

함수를 따로 만들어 이름(함수포인터)을 넘겨주는 대신 람다를 사용하면 편리할 때가 있다.

```cpp
#include <bits/stdc++.h>
using namespace std;
int main(){
  vector<int> v({1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12});
  v.erase(remove_if(v.begin(),v.end(),
      [](int a) -> bool{ return (a%2)==0; } // lambda function
    ),v.end());
  for(int x : v) cout << x << " "; 
}
```

특히 람다 함수에서 대괄호로 표기되는 캡쳐리스트는 매우 강력한 기능이다. 람다 함수는 보통 다른 메소드에서 정해진 형식을 가진 함수를 넘겨주기를 원할 때 사용하므로 매개변수를 아무렇게나 넣어줄 수 없다. 하지만 캡쳐리스트로 람다에 몰래 내가 원하는 다른 변수들을 집어넣을 수 있다.

```cpp
#include <bits/stdc++.h>
using namespace std;
int main(){
  vector<int> collect;
  vector<int> v({1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12});
  v.erase(remove_if(v.begin(), v.end(), 
      [&collect](int a) -> bool{ // & means not pointer, but reference
        if((a%2)==0 && a<10){ 
          collect.push_back(a); 
          return true; 
        }else return false;
      }
    ),v.end());
  for(int x : v) cout << x << " "; cout << endl;
  for(int x : collect) cout << x << " ";
}
```

참고로 람다 함수 없이 이 문제를 해결할 다른 방법은 함수객체를 사용하는 것이다.

```cpp
class func{ 
  public:
    vector<int>& c;
    func(vector<int>& vec) : c(vec) {}// constructor
    bool operator()(int a);
};

bool func::operator()(int a){
  if((a%2)==0 && a<10){
    c.push_back(a);
    return true;
  }else return false;
}

int main(){
  vector<int> collect;
  vector<int> v({1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12});
  func F(collect);
  v.erase(remove_if(v.begin(), v.end(), F),v.end());
  for(int x : v) cout << x << " "; cout << endl;
  for(int x : collect) cout << x << " ";
}
```

---

## 흔하진 않지만 쓰일수도 있는 잡다한 개념들

다른 클래스의 private에 접근 가능하게 해주는 **friend** 키워드

소괄호 캐스팅이 너무 허술해보여서 새로 만들어진 **명시적 캐스팅**

**파일 읽고쓰기** istream, ostream, ifstream, ofstream, stringstream

모든 동작을 템플릿으로 해결하려고 하는 **템플릿 메타 프로그래밍**

메소드 사용시 생성자 자동호출 방지 **explicit** 키워드

상수(const) 메소드에서도 변경 가능한 필드 변수의 예외조약 **mutable** 키워드

예외 처리 **try - catch - throw** 구조

변수 선언, 초기화 시 대입되는 객체를 보고 타입을 자동으로 추론하는 **auto** 키워드

타입 이름이 긴데 반복적으로 쓰여서 불편할 때 **typedef**로 대체별명 붙여주기

---

