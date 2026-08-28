# C++: OOP, Memory, Smart Pointer và Thuật toán phỏng vấn

Tài liệu tổng hợp các nội dung đã tìm hiểu, sử dụng C++17 trở lên.

## Mục lục

1. [Lập trình hướng đối tượng](#1-lập-trình-hướng-đối-tượng)
2. [Upcasting và downcasting](#2-upcasting-và-downcasting)
3. [Vtable và vptr](#3-vtable-và-vptr)
4. [Các vùng nhớ trong C++](#4-các-vùng-nhớ-trong-c)
5. [Smart pointer](#5-smart-pointer)
6. [Câu hỏi thuật toán phỏng vấn](#6-câu-hỏi-thuật-toán-phỏng-vấn)
7. [Lời giải thuật toán](#7-lời-giải-thuật-toán)

---

# 1. Lập trình hướng đối tượng

OOP (Object-Oriented Programming) tổ chức chương trình thành các đối tượng. Mỗi đối tượng thường có:

- Trạng thái: các thuộc tính.
- Hành vi: các hàm thành viên.
- Danh tính: hai đối tượng có dữ liệu giống nhau vẫn là hai thực thể riêng.

```cpp
#include <iostream>
#include <string>
using namespace std;

class Student {
private:
    string name;
    double gpa;

public:
    Student(string name, double gpa)
        : name(std::move(name)), gpa(gpa) {}

    void display() const {
        cout << name << " - GPA: " << gpa << '\n';
    }
};
```

## 1.1. Đóng gói — Encapsulation

Đóng gói kết hợp dữ liệu và hành vi trong một lớp, đồng thời kiểm soát quyền truy cập bằng `private`, `protected` và `public`.

```cpp
class BankAccount {
private:
    double balance;

public:
    explicit BankAccount(double initialBalance)
        : balance(initialBalance) {
        if (initialBalance < 0) {
            throw std::invalid_argument("Invalid balance");
        }
    }

    void deposit(double amount) {
        if (amount <= 0) {
            throw std::invalid_argument("Invalid amount");
        }
        balance += amount;
    }

    bool withdraw(double amount) {
        if (amount <= 0 || amount > balance) {
            return false;
        }
        balance -= amount;
        return true;
    }

    double getBalance() const {
        return balance;
    }
};
```

Mục đích chính:

- Ngăn trạng thái không hợp lệ.
- Buộc thay đổi dữ liệu thông qua nghiệp vụ rõ ràng.
- Cho phép thay đổi cách triển khai bên trong mà ít ảnh hưởng mã bên ngoài.

## 1.2. Trừu tượng — Abstraction

Trừu tượng chỉ công khai những thao tác cần thiết và che giấu chi tiết triển khai.

```cpp
class CoffeeMachine {
private:
    void heatWater() {}
    void grindCoffee() {}
    void pumpWater() {}

public:
    void makeCoffee() {
        heatWater();
        grindCoffee();
        pumpWater();
    }
};
```

Lớp trừu tượng có ít nhất một hàm thuần ảo:

```cpp
class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() = default;
};

class Rectangle : public Shape {
private:
    double width;
    double height;

public:
    Rectangle(double width, double height)
        : width(width), height(height) {}

    double area() const override {
        return width * height;
    }
};
```

## 1.3. Kế thừa — Inheritance

Kế thừa cho phép lớp con sử dụng và mở rộng lớp cha.

```cpp
class Employee {
protected:
    string name;
    double baseSalary;

public:
    Employee(string name, double salary)
        : name(std::move(name)), baseSalary(salary) {}

    virtual ~Employee() = default;
};

class Manager : public Employee {
private:
    double allowance;

public:
    Manager(string name, double salary, double allowance)
        : Employee(std::move(name), salary), allowance(allowance) {}

    double salary() const {
        return baseSalary + allowance;
    }
};
```

Chỉ nên dùng kế thừa cho quan hệ **is-a**:

- `Manager` là một `Employee`.
- `Dog` là một `Animal`.

Với quan hệ **has-a**, ưu tiên composition:

```cpp
class Engine {};

class Car {
private:
    Engine engine;
};
```

## 1.4. Đa hình — Polymorphism

Đa hình cho phép cùng một giao diện có nhiều cách thực hiện.

### Đa hình lúc biên dịch

Nạp chồng hàm:

```cpp
class Calculator {
public:
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
};
```

### Đa hình runtime

```cpp
class Animal {
public:
    virtual void sound() const = 0;
    virtual ~Animal() = default;
};

class Dog : public Animal {
public:
    void sound() const override {
        cout << "Woof\n";
    }
};

class Cat : public Animal {
public:
    void sound() const override {
        cout << "Meow\n";
    }
};
```

```cpp
void playSound(const Animal& animal) {
    animal.sound();
}
```

| Đặc tính | Câu hỏi chính | Công cụ C++ |
|---|---|---|
| Đóng gói | Ai được truy cập dữ liệu? | `private`, `protected`, `public` |
| Trừu tượng | Người dùng cần nhìn thấy gì? | Interface, pure virtual |
| Kế thừa | Lớp mới có quan hệ gì với lớp cũ? | `class Child : public Parent` |
| Đa hình | Giao diện chung hoạt động khác nhau thế nào? | `virtual`, `override` |

---

# 2. Upcasting và downcasting

## 2.1. Upcasting

Upcasting chuyển con trỏ hoặc tham chiếu lớp con thành lớp cha:

```cpp
Dog dog;
Animal* animalPointer = &dog;
Animal& animalReference = dog;
```

Upcasting tự động và an toàn vì mọi `Dog` đều là `Animal`. Đối tượng thật vẫn là `Dog`; chỉ giao diện truy cập bị giới hạn theo kiểu tĩnh `Animal`.

Tránh sao chép theo giá trị:

```cpp
Animal animal = dog; // Object slicing nếu Animal không abstract
```

Slicing loại bỏ phần riêng của lớp con. Muốn giữ đa hình, dùng con trỏ hoặc tham chiếu.

## 2.2. Downcasting

Downcasting chuyển từ lớp cha về lớp con. Nó cần kiểm tra vì không phải mọi `Animal` đều là `Dog`.

```cpp
Animal* animal = new Dog();

if (Dog* dog = dynamic_cast<Dog*>(animal)) {
    dog->sound();
}

delete animal;
```

Với con trỏ, `dynamic_cast` thất bại trả về `nullptr`. Với tham chiếu, nó ném `std::bad_cast`:

```cpp
try {
    Dog& dog = dynamic_cast<Dog&>(animalReference);
} catch (const std::bad_cast&) {
    // Không phải Dog
}
```

Lớp nguồn phải đa hình, tức có ít nhất một hàm `virtual`.

## 2.3. `dynamic_cast` và `static_cast`

```cpp
Dog* checked = dynamic_cast<Dog*>(animal); // Kiểm tra runtime
Dog* unchecked = static_cast<Dog*>(animal); // Không kiểm tra runtime
```

| Đặc điểm | `dynamic_cast` | `static_cast` |
|---|---|---|
| Kiểm tra kiểu động | Có | Không |
| Downcast sai bằng con trỏ | `nullptr` | Undefined behavior khi sử dụng |
| Yêu cầu lớp đa hình | Có | Không nhất thiết |
| Mục đích | Downcast chưa chắc chắn | Chỉ dùng khi chắc chắn kiểu thật |

Downcast quá thường xuyên thường là dấu hiệu interface lớp cha còn thiếu. Nếu hành vi có thể đưa thành hàm ảo, hãy gọi thông qua lớp cha thay vì kiểm tra từng lớp con.

---

# 3. Vtable và vptr

> Tiêu chuẩn C++ quy định hành vi của đa hình nhưng không bắt buộc compiler phải dùng vtable/vptr. Đây là cách triển khai phổ biến của GCC, Clang và MSVC.

## 3.1. Khái niệm

- **vtable**: bảng dùng chung cho một lớp đa hình, thường chứa địa chỉ hàm ảo và metadata.
- **vptr**: con trỏ ẩn trong mỗi đối tượng đa hình, trỏ tới vtable thích hợp.

```text
Dog object                         Dog vtable
┌────────────────────┐            ┌────────────────────┐
│ vptr ──────────────────────────►│ RTTI: Dog          │
│ Animal data        │            │ &Dog::sound        │
│ Dog data           │            │ &Dog::~Dog         │
└────────────────────┘            └────────────────────┘
```

Nhiều đối tượng cùng lớp thường dùng chung một vtable nhưng mỗi đối tượng có vptr riêng.

## 3.2. Virtual dispatch

```cpp
Animal* animal = new Dog();
animal->sound();
```

Khái niệm thực thi:

1. Đọc vptr từ đối tượng.
2. Truy cập vtable.
3. Lấy slot của `sound()`.
4. Gọi địa chỉ hàm trong slot và truyền con trỏ `this`.

Có thể hình dung gần giống:

```cpp
animal->vptr[SOUND_SLOT](animal); // Chỉ là mô hình minh họa
```

Nếu hàm không có `virtual`, compiler chọn hàm bằng kiểu tĩnh và không cần tra vtable.

## 3.3. Constructor và destructor

Trong constructor lớp cha, phần lớp con chưa hoàn chỉnh nên lời gọi ảo chỉ gọi phiên bản lớp cha:

```cpp
class Base {
public:
    Base() { identify(); }
    virtual void identify() { cout << "Base\n"; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void identify() override { cout << "Derived\n"; }
};
```

Khi tạo `Derived`, lời gọi trong `Base()` in `Base`. Destructor hoạt động theo chiều ngược lại: sau khi phần lớp con bị hủy, destructor lớp cha chỉ dispatch trong phạm vi lớp cha.

Không nên dựa vào virtual dispatch xuống lớp con trong constructor/destructor.

## 3.4. Destructor ảo

```cpp
Animal* animal = new Dog();
delete animal;
```

Destructor lớp cha phải là `virtual` để `~Dog()` chạy trước `~Animal()`. Nếu không, việc xóa đối tượng lớp con qua con trỏ lớp cha là undefined behavior.

## 3.5. RTTI và downcasting

`dynamic_cast` dùng RTTI để:

1. Xác định kiểu động của toàn bộ đối tượng.
2. Kiểm tra quan hệ kế thừa.
3. Phát hiện trường hợp nhập nhằng.
4. Điều chỉnh địa chỉ con trỏ nếu cần.

Trong implementation phổ biến, RTTI được liên kết với vtable. Vtable không tự “bảo đảm downcast”; `dynamic_cast` được bảo đảm bởi quy tắc ngôn ngữ và dùng RTTI.

## 3.6. Đa kế thừa

```cpp
class Printer { public: virtual ~Printer() = default; };
class Scanner { public: virtual ~Scanner() = default; };
class Machine : public Printer, public Scanner {};
```

Một `Machine` có thể có nhiều subobject và nhiều vptr. `Printer*` và `Scanner*` trỏ đến các địa chỉ khác nhau trong cùng một đối tượng. Compiler có thể dùng **thunk** để điều chỉnh `this` trước khi gọi hàm override.

## 3.7. Chi phí

- Mỗi đối tượng đa hình thường tốn thêm một hoặc nhiều vptr.
- Lời gọi ảo cần đọc vptr, đọc slot và gọi gián tiếp.
- Lời gọi gián tiếp có thể khó inline hơn.
- Compiler có thể **devirtualize** nếu chứng minh được kiểu thật; `final` có thể hỗ trợ việc này.

Không nên tự đọc, ghi hoặc sao chép vptr bằng `memcpy`/`memset`. Vptr là chi tiết implementation và phá hỏng nó gây undefined behavior.

---

# 4. Các vùng nhớ trong C++

Một tiến trình C++ thường có các vùng nhớ khái niệm sau. Bố cục chính xác phụ thuộc hệ điều hành, compiler và linker.

## 4.1. Code/Text segment

Chứa mã máy của chương trình và thường chỉ đọc, có quyền thực thi.

```cpp
int add(int a, int b) {
    return a + b;
}
```

Mã máy của `add` thường nằm trong text segment.

## 4.2. Data segment

Chứa biến global/static đã được khởi tạo với giá trị khác 0.

```cpp
int globalCount = 10;
static int limit = 100;
```

Tồn tại trong toàn bộ thời gian chạy chương trình.

## 4.3. BSS segment

Chứa biến global/static chưa khởi tạo tường minh hoặc được khởi tạo bằng 0.

```cpp
int globalValue;
static int counter = 0;
```

Hệ thống khởi tạo chúng bằng 0 trước khi `main()` chạy.

## 4.4. Read-only data

Thường chứa string literal và một số hằng số:

```cpp
const char* message = "Hello";
```

Không được sửa string literal:

```cpp
// const_cast<char*>(message)[0] = 'h'; // Undefined behavior
```

## 4.5. Stack

Chứa stack frame của lời gọi hàm, thường gồm biến cục bộ, tham số, địa chỉ trả về và dữ liệu lưu tạm.

```cpp
void function() {
    int localValue = 42;
    Student student("An", 8.5);
}
```

Đặc điểm:

- Cấp phát và thu hồi tự động theo scope/lời gọi hàm.
- Rất nhanh.
- Dung lượng có giới hạn.
- Đệ quy quá sâu hoặc mảng local quá lớn có thể gây stack overflow.

## 4.6. Heap/Free store

Chứa đối tượng có thời gian sống động:

```cpp
int* number = new int(42);
delete number;
```

Trong C++ hiện đại, ưu tiên:

```cpp
auto number = std::make_unique<int>(42);
```

Đặc điểm:

- Thời gian sống không phụ thuộc trực tiếp vào scope tạo đối tượng.
- Cấp phát chậm hơn stack và có thể phân mảnh.
- Quên giải phóng gây memory leak.
- Giải phóng hai lần hoặc dùng sau khi giải phóng gây undefined behavior.

## 4.7. Thread-local storage

Mỗi thread có một bản sao riêng:

```cpp
thread_local int requestCount = 0;
```

Biến sống theo vòng đời thread và không được dùng chung trực tiếp giữa các thread.

## 4.8. So sánh stack và heap

| Tiêu chí | Stack | Heap/free store |
|---|---|---|
| Quản lý | Tự động theo scope | RAII hoặc thủ công |
| Tốc độ cấp phát | Rất nhanh | Thường chậm hơn |
| Dung lượng | Thường nhỏ hơn | Thường lớn hơn |
| Thời gian sống | Theo scope | Do chủ sở hữu quyết định |
| Rủi ro | Stack overflow | Leak, dangling pointer, fragmentation |
| Cách ưu tiên | Object trực tiếp | Smart pointer/container |

`new` không đồng nghĩa tuyệt đối với “heap” theo thuật ngữ chuẩn; chuẩn C++ gọi vùng cấp phát bởi `new` là **free store**. Trong thực tế, free store thường được triển khai trên heap.

---

# 5. Smart pointer

Smart pointer áp dụng RAII: nhận tài nguyên khi khởi tạo và tự giải phóng trong destructor.

```cpp
#include <memory>
```

## 5.1. `std::unique_ptr`

Biểu diễn quyền sở hữu duy nhất:

```cpp
auto first = std::make_unique<int>(42);
auto second = std::move(first);
```

Sau `move`, `second` sở hữu đối tượng và `first` rỗng. `unique_ptr` không copy được vì hai chủ sở hữu độc quyền sẽ gây double delete.

### Truyền vào hàm

```cpp
void inspect(const Student& student);          // Chỉ dùng
void consume(std::unique_ptr<Student> value); // Nhận sở hữu
```

```cpp
auto student = std::make_unique<Student>("An", 8.5);
inspect(*student);
consume(std::move(student));
```

### Factory đa hình

```cpp
std::unique_ptr<Animal> createAnimal() {
    return std::make_unique<Dog>();
}
```

Ứng dụng:

- Cây: node cha sở hữu duy nhất node con.
- PImpl.
- Thành phần động có một chủ sở hữu rõ ràng.
- Factory trả về đối tượng đa hình.

Các hàm quan trọng:

- `get()`: lấy raw pointer không chuyển sở hữu.
- `reset()`: hủy đối tượng hiện tại và có thể nhận đối tượng mới.
- `release()`: nhả quyền sở hữu; người gọi phải tự quản lý, dễ gây leak.

## 5.2. `std::shared_ptr`

Cho phép nhiều chủ sở hữu:

```cpp
auto first = std::make_shared<int>(42);
auto second = first;
```

`shared_ptr` dùng control block, thường chứa:

- Strong reference count.
- Weak reference count.
- Deleter và allocator metadata.

Đối tượng bị hủy khi strong count về 0. Control block chỉ bị hủy khi không còn strong hoặc weak reference.

Ưu tiên `make_shared` vì thường an toàn hơn và có thể gộp đối tượng với control block trong một lần cấp phát.

Ứng dụng:

- Tài nguyên được nhiều đối tượng thực sự cùng sở hữu.
- Tác vụ bất đồng bộ phải giữ đối tượng sống.
- Cache hoặc texture dùng chung.

Không dùng `shared_ptr` mặc định cho mọi thứ vì nó có chi phí reference counting và có thể che giấu quyền sở hữu.

## 5.3. `std::weak_ptr`

`weak_ptr` quan sát đối tượng do `shared_ptr` quản lý nhưng không giữ đối tượng sống.

```cpp
std::weak_ptr<Student> observer = owner;

if (auto student = observer.lock()) {
    student->display();
}
```

`lock()` trả `shared_ptr` hợp lệ nếu đối tượng còn sống, ngược lại trả con trỏ rỗng.

Ứng dụng quan trọng: phá vòng tham chiếu.

```cpp
struct Node {
    std::vector<std::shared_ptr<Node>> children;
    std::weak_ptr<Node> parent;
};
```

Cha sở hữu con bằng `shared_ptr`; con chỉ quan sát cha bằng `weak_ptr`.

## 5.4. So sánh

| Đặc điểm | `unique_ptr` | `shared_ptr` | `weak_ptr` |
|---|---|---|---|
| Sở hữu | Duy nhất | Chia sẻ | Không sở hữu |
| Copy | Không | Có | Có |
| Move | Có | Có | Có |
| Reference count | Không | Có | Theo dõi control block |
| Hủy đối tượng | Khi owner duy nhất mất | Khi strong count = 0 | Không hủy |

Quy tắc lựa chọn:

1. Nếu không cần cấp phát động, dùng object trực tiếp.
2. Nếu cần cấp phát động, ưu tiên `unique_ptr`.
3. Chỉ dùng `shared_ptr` khi có nhiều chủ sở hữu thật sự.
4. Dùng `weak_ptr` cho quan hệ quan sát hoặc phá vòng sở hữu.
5. Dùng raw pointer/reference cho truy cập không sở hữu.

## 5.5. Custom deleter

```cpp
struct FileCloser {
    void operator()(FILE* file) const {
        if (file != nullptr) {
            std::fclose(file);
        }
    }
};

using FilePointer = std::unique_ptr<FILE, FileCloser>;

FilePointer file(std::fopen("data.txt", "r"));
```

Custom deleter cho phép RAII quản lý `FILE*`, socket, OS handle hoặc tài nguyên thư viện C.

## 5.6. Sai lầm thường gặp

```cpp
int* raw = new int(10);
std::shared_ptr<int> first(raw);
std::shared_ptr<int> second(raw); // Sai: hai control block, double delete
```

Đúng:

```cpp
auto first = std::make_shared<int>(10);
auto second = first;
```

Không tự `delete` kết quả của `get()`. Không dùng một `unique_ptr` sau khi đã move. Không capture `shared_ptr` trong callback do chính đối tượng sở hữu nếu điều đó tạo vòng tham chiếu; hãy cân nhắc `weak_ptr`.

---

# 6. Câu hỏi thuật toán phỏng vấn

| STT | Bài toán | Kỹ thuật | Kỳ vọng |
|---:|---|---|---|
| 1 | Two Sum | Hash map | `O(n)` |
| 2 | Palindrome | Two pointers | `O(n)` |
| 3 | Reverse Linked List | Pointer | `O(n)`, `O(1)` space |
| 4 | Linked List Cycle | Floyd | `O(n)`, `O(1)` space |
| 5 | Valid Parentheses | Stack | `O(n)` |
| 6 | Most Frequent Element | Frequency map | `O(n)` trung bình |
| 7 | Top K Frequent | Bucket/heap | `O(n)` hoặc `O(n log k)` |
| 8 | Binary Search | Chia đôi | `O(log n)` |
| 9 | Merge Sorted Arrays | Two pointers | `O(n+m)` |
| 10 | Maximum Subarray | Kadane | `O(n)` |
| 11 | Longest Unique Substring | Sliding window | `O(n)` |
| 12 | Closest Pair Sum | Sort + two pointers | `O(n log n)` |
| 13 | Merge Intervals | Sort | `O(n log n)` |
| 14 | Product Except Self | Prefix/suffix | `O(n)` |
| 15 | Kth Largest | Heap/Quickselect | `O(n log k)` |
| 16 | Tree Level Order | BFS | `O(n)` |
| 17 | Validate BST | DFS + range | `O(n)` |
| 18 | Lowest Common Ancestor | DFS | `O(n)` |
| 19 | Number of Islands | DFS/BFS | `O(rows*cols)` |
| 20 | Shortest Maze Path | BFS | `O(rows*cols)` |
| 21 | Course Schedule | Topological sort | `O(V+E)` |
| 22 | Dijkstra | Min-heap | `O((V+E)log V)` |
| 23 | Coin Change | Dynamic programming | `O(amount*coins)` |
| 24 | Longest Increasing Subsequence | Binary search | `O(n log n)` |
| 25 | LRU Cache | Hash map + list | `O(1)` trung bình |

---

# 7. Lời giải thuật toán

Các lời giải sau giả định đã include những thư viện chuẩn cần thiết và có `using namespace std;`.

## 7.1. Two Sum

```cpp
vector<int> twoSum(const vector<int>& nums, int target) {
    unordered_map<int, int> indexByValue;

    for (int i = 0; i < static_cast<int>(nums.size()); ++i) {
        int needed = target - nums[i];
        auto found = indexByValue.find(needed);

        if (found != indexByValue.end()) {
            return {found->second, i};
        }
        indexByValue[nums[i]] = i;
    }
    return {};
}
```

## 7.2. Palindrome

```cpp
bool isPalindrome(const string& text) {
    int left = 0;
    int right = static_cast<int>(text.size()) - 1;

    while (left < right) {
        while (left < right &&
               !isalnum(static_cast<unsigned char>(text[left]))) ++left;
        while (left < right &&
               !isalnum(static_cast<unsigned char>(text[right]))) --right;

        if (tolower(static_cast<unsigned char>(text[left])) !=
            tolower(static_cast<unsigned char>(text[right]))) return false;

        ++left;
        --right;
    }
    return true;
}
```

## 7.3. Reverse Linked List

```cpp
struct ListNode {
    int value;
    ListNode* next = nullptr;
};

ListNode* reverseList(ListNode* head) {
    ListNode* previous = nullptr;

    while (head != nullptr) {
        ListNode* next = head->next;
        head->next = previous;
        previous = head;
        head = next;
    }
    return previous;
}
```

## 7.4. Linked List Cycle

```cpp
bool hasCycle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;

    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}
```

## 7.5. Valid Parentheses

```cpp
bool isValidParentheses(const string& expression) {
    stack<char> brackets;

    for (char character : expression) {
        if (character == '(' || character == '[' || character == '{') {
            brackets.push(character);
            continue;
        }

        if (brackets.empty()) return false;
        char opening = brackets.top();
        brackets.pop();

        if (!((opening == '(' && character == ')') ||
              (opening == '[' && character == ']') ||
              (opening == '{' && character == '}'))) return false;
    }
    return brackets.empty();
}
```

## 7.6. Most Frequent Element

```cpp
int mostFrequentElement(const vector<int>& nums) {
    if (nums.empty()) throw invalid_argument("Empty array");

    unordered_map<int, int> frequency;
    int answer = nums.front();
    int best = 0;

    for (int value : nums) {
        int current = ++frequency[value];
        if (current > best) {
            best = current;
            answer = value;
        }
    }
    return answer;
}
```

## 7.7. Top K Frequent

```cpp
vector<int> topKFrequent(const vector<int>& nums, int k) {
    unordered_map<int, int> frequency;
    for (int value : nums) ++frequency[value];

    vector<vector<int>> buckets(nums.size() + 1);
    for (const auto& [value, count] : frequency) {
        buckets[count].push_back(value);
    }

    vector<int> result;
    for (int count = static_cast<int>(nums.size());
         count > 0 && static_cast<int>(result.size()) < k; --count) {
        for (int value : buckets[count]) {
            result.push_back(value);
            if (static_cast<int>(result.size()) == k) break;
        }
    }
    return result;
}
```

## 7.8. Binary Search

```cpp
int binarySearch(const vector<int>& nums, int target) {
    int left = 0;
    int right = static_cast<int>(nums.size()) - 1;

    while (left <= right) {
        int middle = left + (right - left) / 2;
        if (nums[middle] == target) return middle;
        if (nums[middle] < target) left = middle + 1;
        else right = middle - 1;
    }
    return -1;
}
```

## 7.9. Merge Sorted Arrays

```cpp
vector<int> mergeSortedArrays(const vector<int>& a,
                              const vector<int>& b) {
    vector<int> result;
    result.reserve(a.size() + b.size());
    size_t i = 0, j = 0;

    while (i < a.size() && j < b.size()) {
        if (a[i] <= b[j]) result.push_back(a[i++]);
        else result.push_back(b[j++]);
    }
    while (i < a.size()) result.push_back(a[i++]);
    while (j < b.size()) result.push_back(b[j++]);
    return result;
}
```

## 7.10. Maximum Subarray

```cpp
long long maxSubarraySum(const vector<int>& nums) {
    if (nums.empty()) throw invalid_argument("Empty array");

    long long current = nums[0];
    long long best = nums[0];

    for (size_t i = 1; i < nums.size(); ++i) {
        current = max<long long>(nums[i], current + nums[i]);
        best = max(best, current);
    }
    return best;
}
```

## 7.11. Longest Unique Substring

```cpp
int longestUniqueSubstring(const string& text) {
    vector<int> lastPosition(256, -1);
    int left = 0;
    int answer = 0;

    for (int right = 0; right < static_cast<int>(text.size()); ++right) {
        unsigned char character = static_cast<unsigned char>(text[right]);
        left = max(left, lastPosition[character] + 1);
        lastPosition[character] = right;
        answer = max(answer, right - left + 1);
    }
    return answer;
}
```

## 7.12. Closest Pair Sum

```cpp
pair<int, int> closestPairSum(vector<int> nums, int target) {
    if (nums.size() < 2) throw invalid_argument("Need two values");
    sort(nums.begin(), nums.end());

    int left = 0;
    int right = static_cast<int>(nums.size()) - 1;
    long long bestDifference = LLONG_MAX;
    pair<int, int> answer;

    while (left < right) {
        long long sum = static_cast<long long>(nums[left]) + nums[right];
        long long difference = llabs(sum - target);

        if (difference < bestDifference) {
            bestDifference = difference;
            answer = {nums[left], nums[right]};
        }
        if (sum < target) ++left;
        else if (sum > target) --right;
        else break;
    }
    return answer;
}
```

## 7.13. Merge Intervals

```cpp
vector<vector<int>> mergeIntervals(vector<vector<int>> intervals) {
    if (intervals.empty()) return {};
    sort(intervals.begin(), intervals.end());

    vector<vector<int>> result{intervals.front()};
    for (size_t i = 1; i < intervals.size(); ++i) {
        if (intervals[i][0] <= result.back()[1]) {
            result.back()[1] = max(result.back()[1], intervals[i][1]);
        } else {
            result.push_back(intervals[i]);
        }
    }
    return result;
}
```

## 7.14. Product Except Self

```cpp
vector<long long> productExceptSelf(const vector<int>& nums) {
    vector<long long> result(nums.size(), 1);
    long long prefix = 1;

    for (size_t i = 0; i < nums.size(); ++i) {
        result[i] = prefix;
        prefix *= nums[i];
    }

    long long suffix = 1;
    for (int i = static_cast<int>(nums.size()) - 1; i >= 0; --i) {
        result[i] *= suffix;
        suffix *= nums[i];
    }
    return result;
}
```

## 7.15. Kth Largest

```cpp
int findKthLargest(const vector<int>& nums, int k) {
    if (k <= 0 || k > static_cast<int>(nums.size())) {
        throw invalid_argument("Invalid k");
    }

    priority_queue<int, vector<int>, greater<int>> heap;
    for (int value : nums) {
        heap.push(value);
        if (static_cast<int>(heap.size()) > k) heap.pop();
    }
    return heap.top();
}
```

## 7.16. Tree Level Order

```cpp
struct TreeNode {
    int value;
    TreeNode* left = nullptr;
    TreeNode* right = nullptr;
};

vector<vector<int>> levelOrder(TreeNode* root) {
    if (root == nullptr) return {};
    queue<TreeNode*> nodes;
    nodes.push(root);
    vector<vector<int>> result;

    while (!nodes.empty()) {
        int levelSize = static_cast<int>(nodes.size());
        vector<int> level;

        while (levelSize-- > 0) {
            TreeNode* node = nodes.front();
            nodes.pop();
            level.push_back(node->value);
            if (node->left) nodes.push(node->left);
            if (node->right) nodes.push(node->right);
        }
        result.push_back(std::move(level));
    }
    return result;
}
```

## 7.17. Validate BST

```cpp
bool validateBST(TreeNode* node, long long minimum, long long maximum) {
    if (node == nullptr) return true;
    if (node->value <= minimum || node->value >= maximum) return false;

    return validateBST(node->left, minimum, node->value) &&
           validateBST(node->right, node->value, maximum);
}

bool isValidBST(TreeNode* root) {
    return validateBST(root, LLONG_MIN, LLONG_MAX);
}
```

## 7.18. Lowest Common Ancestor

```cpp
TreeNode* lowestCommonAncestor(TreeNode* root,
                               TreeNode* first,
                               TreeNode* second) {
    if (root == nullptr || root == first || root == second) return root;

    TreeNode* left = lowestCommonAncestor(root->left, first, second);
    TreeNode* right = lowestCommonAncestor(root->right, first, second);

    if (left != nullptr && right != nullptr) return root;
    return left != nullptr ? left : right;
}
```

## 7.19. Number of Islands

```cpp
int countIslands(vector<vector<char>>& grid) {
    if (grid.empty() || grid[0].empty()) return 0;
    int rows = static_cast<int>(grid.size());
    int columns = static_cast<int>(grid[0].size());
    int answer = 0;
    const int dr[] = {-1, 1, 0, 0};
    const int dc[] = {0, 0, -1, 1};

    for (int row = 0; row < rows; ++row) {
        for (int column = 0; column < columns; ++column) {
            if (grid[row][column] != '1') continue;
            ++answer;
            stack<pair<int, int>> cells;
            cells.push({row, column});
            grid[row][column] = '0';

            while (!cells.empty()) {
                auto [r, c] = cells.top();
                cells.pop();
                for (int direction = 0; direction < 4; ++direction) {
                    int nr = r + dr[direction];
                    int nc = c + dc[direction];
                    if (nr >= 0 && nr < rows && nc >= 0 && nc < columns &&
                        grid[nr][nc] == '1') {
                        grid[nr][nc] = '0';
                        cells.push({nr, nc});
                    }
                }
            }
        }
    }
    return answer;
}
```

## 7.20. Shortest Maze Path

```cpp
int shortestPath(const vector<vector<int>>& grid,
                 pair<int, int> start,
                 pair<int, int> destination) {
    if (grid.empty() || grid[0].empty()) return -1;
    int rows = static_cast<int>(grid.size());
    int columns = static_cast<int>(grid[0].size());
    vector<vector<int>> distance(rows, vector<int>(columns, -1));
    queue<pair<int, int>> cells;
    const int dr[] = {-1, 1, 0, 0};
    const int dc[] = {0, 0, -1, 1};

    auto [sr, sc] = start;
    auto [tr, tc] = destination;
    if (sr < 0 || sr >= rows || sc < 0 || sc >= columns ||
        tr < 0 || tr >= rows || tc < 0 || tc >= columns ||
        grid[sr][sc] == 1 || grid[tr][tc] == 1) return -1;

    cells.push(start);
    distance[sr][sc] = 0;

    while (!cells.empty()) {
        auto [row, column] = cells.front();
        cells.pop();
        if (row == tr && column == tc) return distance[row][column];

        for (int direction = 0; direction < 4; ++direction) {
            int nr = row + dr[direction];
            int nc = column + dc[direction];
            if (nr >= 0 && nr < rows && nc >= 0 && nc < columns &&
                grid[nr][nc] == 0 && distance[nr][nc] == -1) {
                distance[nr][nc] = distance[row][column] + 1;
                cells.push({nr, nc});
            }
        }
    }
    return -1;
}
```

## 7.21. Course Schedule

```cpp
bool canFinish(int courseCount,
               const vector<pair<int, int>>& prerequisites) {
    vector<vector<int>> graph(courseCount);
    vector<int> indegree(courseCount, 0);

    for (const auto& [course, prerequisite] : prerequisites) {
        graph[prerequisite].push_back(course);
        ++indegree[course];
    }

    queue<int> available;
    for (int course = 0; course < courseCount; ++course) {
        if (indegree[course] == 0) available.push(course);
    }

    int completed = 0;
    while (!available.empty()) {
        int course = available.front();
        available.pop();
        ++completed;

        for (int next : graph[course]) {
            if (--indegree[next] == 0) available.push(next);
        }
    }
    return completed == courseCount;
}
```

## 7.22. Dijkstra

```cpp
vector<long long> dijkstra(
    const vector<vector<pair<int, int>>>& graph,
    int source) {
    const long long infinity = LLONG_MAX;
    vector<long long> distance(graph.size(), infinity);
    using State = pair<long long, int>;
    priority_queue<State, vector<State>, greater<State>> heap;

    distance[source] = 0;
    heap.push({0, source});

    while (!heap.empty()) {
        auto [currentDistance, vertex] = heap.top();
        heap.pop();
        if (currentDistance != distance[vertex]) continue;

        for (const auto& [next, weight] : graph[vertex]) {
            if (weight < 0) throw invalid_argument("Negative edge");
            long long candidate = currentDistance + weight;
            if (candidate < distance[next]) {
                distance[next] = candidate;
                heap.push({candidate, next});
            }
        }
    }
    return distance;
}
```

## 7.23. Coin Change

```cpp
int coinChange(const vector<int>& coins, int amount) {
    if (amount < 0) return -1;
    vector<int> dp(amount + 1, amount + 1);
    dp[0] = 0;

    for (int current = 1; current <= amount; ++current) {
        for (int coin : coins) {
            if (coin > 0 && coin <= current) {
                dp[current] = min(dp[current], dp[current - coin] + 1);
            }
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
```

## 7.24. Longest Increasing Subsequence

```cpp
int lengthOfLIS(const vector<int>& nums) {
    vector<int> tails;

    for (int value : nums) {
        auto position = lower_bound(tails.begin(), tails.end(), value);
        if (position == tails.end()) tails.push_back(value);
        else *position = value;
    }
    return static_cast<int>(tails.size());
}
```

## 7.25. LRU Cache

```cpp
class LRUCache {
private:
    using Entry = pair<int, int>;
    using Iterator = list<Entry>::iterator;

    size_t capacity;
    list<Entry> entries; // Front: mới nhất, back: cũ nhất
    unordered_map<int, Iterator> positionByKey;

    void moveToFront(Iterator position) {
        entries.splice(entries.begin(), entries, position);
    }

public:
    explicit LRUCache(size_t capacity) : capacity(capacity) {}

    int get(int key) {
        auto found = positionByKey.find(key);
        if (found == positionByKey.end()) return -1;
        moveToFront(found->second);
        return found->second->second;
    }

    void put(int key, int value) {
        if (capacity == 0) return;

        auto found = positionByKey.find(key);
        if (found != positionByKey.end()) {
            found->second->second = value;
            moveToFront(found->second);
            return;
        }

        if (entries.size() == capacity) {
            positionByKey.erase(entries.back().first);
            entries.pop_back();
        }

        entries.push_front({key, value});
        positionByKey[key] = entries.begin();
    }
};
```

---

# 8. Chiến lược trả lời phỏng vấn

Với mỗi bài thuật toán:

1. Xác nhận input, output và constraints.
2. Nêu cách đơn giản trước.
3. Phân tích thời gian và bộ nhớ.
4. Đề xuất cách tối ưu.
5. Viết code rõ ràng, đặt tên có ý nghĩa.
6. Chạy tay với một ví dụ.
7. Kiểm tra edge case: input rỗng, một phần tử, phần tử trùng, số âm, overflow và `nullptr`.

Với câu hỏi thiết kế C++:

- Nói rõ ai sở hữu tài nguyên.
- Ưu tiên RAII và Rule of Zero.
- Dùng `override` khi ghi đè.
- Lớp cơ sở đa hình cần destructor ảo nếu có thể bị xóa qua con trỏ lớp cha.
- Ưu tiên composition hơn inheritance nếu quan hệ không phải **is-a**.
- Không tối ưu theo cảm tính; đo đạc trước khi đánh đổi độ rõ ràng.
