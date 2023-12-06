## Pointers & References to Base Class of Derived Objects

**Concept**
  - In C++, pointers and references to a base class can be used to refer to objects of derived classes.
  - Fundamental in implementing polymorphism, allowing base class interfaces to interact with derived class objects.

**Behavior**
  - When using base class pointers/references with derived objects, only members of the base class are directly accessible.
  - Without virtual functions, calling a method on a base class reference/pointer executes the base class version, not the derived class version.

```
Class Base {
public:
  virtual void show() { std::cout << "Base show" << std::endl; }
};

class Derived : public Base {
public:
  void show() override { std::cout << "Derived show" << std::endl; }
};

// Usage
Derived derivedObj;
Base& baseRef = derivedObj;
baseRef.show(); // Outputs "Derived show" due to virtual function
```

**Use Cases**
  - Useful in scenarios where functions need to operate on a group of related objects differently, depending on their actual derived types.
  - Enables writing generic code that can work with a family of related classes.

---

## Virtual Functions and Polymorphism

**Issue with Base Class References**:
  - Base class references/pointers only access the base class functions, not the derived versions.
  - Virtual functions resolve to the most-derived version for the actual type of the object.
  - Declared with the `virtual` keyword; derived classes override them with matching signatures.

```
class Base {
public:
  virtual std::string_view getName() const { return "Base"; }
};

class Derived: public Base {
public:
  std::string_view getName() const override { return "Derived"; }
};

// Main function
Derived derived;
Base& rBase = derived;
std::cout << rBase.getName(); // Now outputs "Derived"
```

**Important Points**
  - Virtual functions allow polymorphic behavior.
  - Avoid calling virtual functions in constructors/destructors.
  - Virtual function resolution only occurs through pointers or references.
  - If a function is virtual, all matching overrides in derived classes are implicitly virtual.

**Polymorphism and Efficiency**
  - Virtual functions enable runtime polymorphism.
  - There is a performance cost due to dynamic binding.
  - Compiler has to allocate an extra pointer for each class object that has one or more virtual functions.

**Best Practices**
  - Use virtual destructors for classes with virtual functions.
  - Be cautious with the return types and signatures of virtual functions and overrides.

---

## The `override` and `final` Specifiers

`override`
- Ensures a derived class function is actually overriding a base class virtual function.
- A compile-time error is generated if the function doesn't override a base class function.
- No performance penalty -> all virtual override functions should be tagged with specifier.

```
class Base {
public:
    virtual std::string_view getName() { return "Base"; }
};

class Derived : public Base {
public:
    std::string_view getName() override { return "Derived"; } // No virtual keyword, virtual is implied
};
```

`final`
- Prevents further overriding of a virtual function or inheritance from a class.
- Applied to virtual functions or classes to prevent them from being extended.

```
class Base {
public:
    virtual std::string_view getName() { return "Base"; }
};

class Derived final : public Base {
public:
    std::string_view getName() override { return "Derived"; }
};

class FurtherDerived : public Derived { // Error: Derived is final
    std::string_view getName() override { return "FurtherDerived"; }
};
```

## Covariant Return Types
- A special case where derived class overrides can have different return types.
- Applicable when return types are pointers or references to classes.
- The return type in the derived class can be a pointer/reference to a derived type.

```
class Base {
public:
    virtual Base* getThis() { return this; }
};

class Derived : public Base {
public:
    Derived* getThis() override { return this; } // Covariant return type
};

int main() {
    Derived d;
    Base* b = d.getThis(); // Returns Derived* but upcast to Base*
}
```

---

## Pure Virtual Functions and Abstract Base Classes

**Pure Virtual Functions**: Functions with no body, declared by assigning `= 0` in their declaration.

**Abstract Base Class**: A class containing one or more pure virtual functions. It cannot be instantiated (illegal: Base base {};).

```
class Base {
public:
    virtual int getValue() const = 0; // Pure virtual function
};
```

**Consequences**
  - Derived classes must implement all pure virtual functions or they also become an abstract base class.
  - Prevents instantiation of classes that are incomplete (i.e., lacking implementation of essential functions).

**Advantages**
- Ensures that derived classes provide specific functionality that the base class was expecting them to.
- Allows for polymorphic behavior with a common interface.

```
class Animal {
public:
    Animal() { std::cout << "Animal created" << std::endl; }
    virtual ~Animal() { std::cout << "Animal destroyed" << std::endl; }

    virtual std::string_view speak() const = 0; 
};

class Cat : public Animal {
public:
    Cat() { std::cout << "Cat created" << std::endl; }
    ~Cat() override { std::cout << "Cat destroyed" << std::endl; }

    std::string_view speak() const override { return "Meow"; }
};
```

**PVF with Definitions**
- Pure virtual functions can have definitions, but the class remains abstract.
- Useful for providing default implementations while still requiring overrides.

```
class Animal {
public:
    virtual std::string_view speak() const = 0;
};

std::string_view Animal::speak() const {
    return "Default Sound";
}
```

---

## Interface Classes
- Classes with only pure virtual functions (no data members).
- Used to define a set of functions that derived classes must implement.

```
class IErrorLog { // Starts with 'I'!
public:
    virtual bool openLog(std::string_view filename) = 0;
    virtual bool closeLog() = 0;
    virtual bool writeError(std::string_view errorMessage) = 0;
    virtual ~IErrorLog() {} // Virtual destructor
};
```

---

## Multiple Inheritance

- Allows a derived class to inherit from more than one base class.
- Enables the creation of a derived class that combines features or behaviors from multiple parent classes.

```
class Person {
    // Person class members
};

class Employee {
    // Employee class members
};

class Teacher : public Person, public Employee {
    // Teacher inherits from both Person and Employee
};
```

**Mixins**
- A small class that adds properties or functionalities to a class through inheritance.
- Intended for adding specific features without direct instantiation.

```
class Box { /* ... */ };
class Label { /* ... */ };

class Button : public Box, public Label {};
```

**Problems with Multiple Inheritance**
1. **Ambiguity**
    - Arises when multiple base classes have a function with the same name.
    - **Example**: If both `USBDevice` and `NetworkDevice` have a `getID()` method, `WirelessAdapter` inheriting from both can lead to ambiguity.
    - **Solution**: Explicitly specify the class to use the method from, e.g., `c54G.USBDevice::getID()`.

3. **Diamond Problem**
    - Occurs in a diamond-shaped inheritance pattern, where a class inherits from two classes that both inherit from the same base class.
    - **Example**: A `Copier` class inheriting from both `Scanner` and `Printer`, which both inherit from `PoweredDevice`.
    - **Issues**: Whether `Copier` should have one or two copies of `PoweredDevice`, and resolving ambiguous references.
    - Addressed in C++ using virtual base classes.

---

## Virtual Base Classes

- Solution to the diamond problem.
- Use the `virtual` keyword to prevent duplication of the base class.
- Ensures a single shared instance of the base class among all derived classes.

```
class PoweredDevice { /* ... */ };

class Scanner : virtual public PoweredDevice { /* ... */ };

class Printer : virtual public PoweredDevice { /* ... */ };

class Copier : public Scanner, public Printer { /* ... */ };
```

- Only one instance of `PoweredDevice` is created and shared between `Scanner` and `Printer`.
- The most derived class is responsible for constructing the virtual base class.
- Constructors of intermediate classes call the virtual base class constructor, but is ignored.

```
class Copier : public Scanner, public Printer {
public:
    Copier(int scanner, int printer, int power) : PoweredDevice{ power }, // Constructs PoweredDevice
                                                  Scanner{ scanner, power },
                                                  Printer{ printer, power } { /* ... */ }
};
```
- In this case, `Copier` directly constructs `PoweredDevice`.

**Important Points**
1. **Order of Construction**: Virtual base classes are constructed before non-virtual base classes.
2. **Responsibility of Construction**: The most derived class is responsible for constructing the virtual base class, even in single inheritance scenarios.
3. **Virtual Table**: Classes inheriting a virtual base class will have a virtual table, increasing the object size by a pointer.

---

## Virtual Destructors

- Essential in base classes to ensure proper cleanup of derived class resources.
- A non-virtual destructor can lead to resource leaks when deleting derived objects through base pointers.

**Assignment Operators**
- `MyClass& operator=(const MyClass& other)`
- Generally, they should not be made virtual to avoid complexity.

**Guidelines**
1. A base class destructor should be either public and virtual, or protected and non-virtual.
2. For classes not intended for inheritance, marking them as final can prevent the need for virtual destructors.

---

## Early vs. Late Binding

**Binding**
- Process that is used to convert identifiers (variable, function names) into machine language addresses.

**Early Binding**
- Process that resolves direct function calls (i.e., statement that directly calls a function).
- Also called static binding, the compiler or linker directly associates a function call with a function's memory address.
- In a program with direct function calls (like add()), the compiler uses early binding.
- Slightly more efficient as the CPU can jump difrectly to the function address.

**Late Binding**
- Also called dynamic binding, this process is used when the function call cannot be resolved until runtime.
- In C++, late binding often involves function pointers,
- CPU has to read the address held in the pointer and then jump to that address.
- Slightly less efficient due to the extra indirection step, but more flexible as the decision on which function to call is made at runtime.

**Function Pointers**
- Also called indirect function call.

```cpp
int add(int x, int y) { return x + y; }
int (*FuncPointer)(int, int) { add };
int result = FuncPointer(5, 3) // result = 8
```

---

## Virtual Tables

- Each class with virtual function (or derived) has a vtable, a static array created at compile time.
- Contains function pointers to the most-derived functions accessible by that class.
- Classes with virtual functions include hidden pointer (`*__vptr`) to the class' vtable.
- Pointer is set when class object is created, making each object larger by the size of one pointer.
- Allows C++ to dynamically bind function calls to the correct function at runtime.

```cpp
class Base {
public:
    VirtualTable* __vptr;
    virtual void function1() {};
    virtual void function2() {};
};

class D1: public Base {
public:
    void function1() override {};
};

class D2: public Base {
public:
    void function2() override {};
};
```
![image](https://github.com/sarahbuddhason/LCPPGuide/assets/55853717/696d8f4a-c3c8-4b94-9429-4b83acd7dd51)

- If a derived class overrides a virtual function, its virtual table points to its own implementation rather than the base class'.
- When a virtual function is called on a base pointer pointing to a derived object, the function call is resolved to the derived class's implementation via the *__vptr and the virtual table.
- By using vtables, the compiler and program are able to ensure function calls resolve to the appropriate virtual function, even if only using a pointer or reference to a base class.
- Calling a virtual function is marginally slower than a non-virtual function due to the additional steps of accessing the virtual table and resolving the function to call.

---

## Object Slicing

- When a derived class object is assigned to a base class object, only the base part of the derived object is copied, and the derived part is "sliced off".
- Leads to a loss of the derived class information.

```cpp
Derived derived = 5;
Base base = derived;
```

## Consequences
1. Functions taking base class objects by value (instead of reference) can cause slicing.
   - Avoided by making the function parameter pass by reference instead of pass by value.
2. Vectors of base class objects can result in slicing when derived objects are added.
   - `vector<Base&> v;`: Elements cannot be references (must be assignable).
   - `vector<Base*> v;`: Avoid with a vector of pointers.
   - `vector<reference_wrapper<Base>> v;`: Avoid with a vector of reassignable references to Base.
  
## Best Practices
- Make sure function paramters are references or pointers to prevent slicing.
- Avoid pass-by-value for derived classes.

---

## Dynamic Casting

**Upcasting**
- Implicitly converting a Derived ppointer into a Base pointer.

**Downcasting**
- Most commonly used for converting base-class pointers into derived-class pointers.
- Uses casting operator: `dynamic_cast`.
- Ensures type safety by checking at runtime if conversion is valid.

**`dynamic_cast` Failure**
- When actual object is not the target Derived type, result is a null pointer.
- For references, result throws a `std::bad_cast` exception.
- Always check that `dynamic_cast` succeeded by checking for a null pointer.

**Limitations**
- Has a runtime performance penalty due to type-checking.
- Does not work with classes with no virtual functions (and thus no vtable).
- Does not work with protected or private inheritance.
- Does not work in some calses with virtual base classes.

**`dynamic_cast` vs. `static_cast`**
- `static_cast` is faster but does not perform runtime checks.
- `dynamic_cast` is safer for downcasting.

**Best Practices**
- Use virtual functions over casting.
- Use `dynamic_cast` when you need to access derived-specific members or when you cannot modify Base class to add virtual functions.
- Do not turn off run-time type information (RTTI, reveals object data type) when using `dynamic_cast`.

---

## Function Pointers

- Stores the address of a function.
- Allows functions to be called indirectly.

**Pointer to Function**
```cpp
int (*funcPointer)(int); // Pointer to function with 1 parameter (int), returns an int.
```

**Assign Function to Function Pointer**
```cpp
int (*funcPointer)(int) { &foo }; // funcPointer points to foo.
funcPointer = &goo;               // funcPointer now points to goo.
```

**Call Function using Function Pointer**
```cpp
int (*funcPointer)(int) { &foo }; // funcPointer points to foo.
(*funcPointer)(5)                 // Explicit dereference.
funcPointer(5)                    // Implicit dereference.
```

- Should ensure that `funcPointer` is not nullptr before calling it.
- Dereferencing a null function pointer leads to undefined behaviour.

**Callback Functions**
```cpp
void selectionSort(int* array, int size, bool comparisonFcn(int, int));
```

- Pass a function as an argument to another function.

**Default Parameters**
```cpp
void selectionSort(int* array, int size, bool comparisonFcn(int, int) = ascending);
```

- Defaults the sort to `ascending` sort function.
- `ascending` must be declared prior to this point.

**Type Aliases**
```cpp
using ValidateFunction = bool(*)(int, int);

bool validate(int x, int y, bool (*funcPointer)(int, int)); // Before
bool validate(int x, int y, ValidateFunction funcPointer)   // After
```

**`std::function`**
```cpp
#include <functional>
bool validate(int x, int y, std::function<bool(int, int)> func);
bool sort(int* array, std::function<bool()> func);
using FuncType = std::function<bool(int, int)>;
```

**Type Reference with `auto`**
- `auto` can infer the type of a function pointer.

**Best Practices**
- For repeated use, `std::function` and type aliases are recommended.

---

## Stack and Heap

**Memory Segments**
- Code Segment: Where compiled program sits in memory.
- BSS Segment: Where zero-initialized global and static variables are stored.
- Data Segment: Where initialized global and static variables are stored.
- Heap: Where dynamically allocated variables are allocated from.
- Call Stack: Where function-related information are stored.

**Heap**
- Used for dynamic memory allocation, managed via `new` and `delete` operators.
- Memory is allocated and freed manually, leading to potential memory leaks.
- Useful for allocating large data or unknown sized data at compile time.
- Slower than stack allocation due to dynamic nature.

**Stack**
- Used for static memory allocation, including function parameters, local variables, etc.
- Follows LIFO: functions are pushed onto the stack when called and popped off when complete.
- Faster than heap allocation since size is known at compile time.
- Limited in size, leading to stack overflow if too much memory is used.

**Stack Frames**
- Stack is a fixed-size block of memory addresses.
- The items pushed and popped on the stack are called stack frames.
- A stack frame keeps track of all the data associated with one function call.
- The stack pointer keeps track of where the top of the call stack currently is.

**Example of Call Stack**
1. Program has a function call.
2. Stack frame constructed. Pushed onto stack. Includes:
   - Return address which tells CPU where to return after called function exits.
   - All function arguments.
   - Memory for any local variables.
   - Saved copies of any registers modified that need to be restored after function returns.
3. CPU jumps to the function start point.
4. Instructions inside function begin execution.
5. Function terminates.
6. Registers are restored from the call stack.
7. Stack frame is popped off the stack, freeing memory for all local variables and arguments.
8. Return value handled.
9. CPU resumes execution at the return address.

**Stack Overflow**
- Occurs when all the memory in the stack has been allocated.
- Result of deep recursion, nested function calls.

**Best Practices**
- Stack: For small, short-lived data, or when exact data size known at compile time.
- Heap: Large data or data whose size is determined at runtime.

---

## Smart Pointers & Move Semantics

**Smart Pointers**
- Classes designed to manage dynamically allocated memory.
- Automatically deallocates memory when the smart pointer object goes out of scope, regardless of how the function exits.
- Wrap classes with a pointer, destructor, and overloaded operators * and ->.

```cpp
class SmartPointer {
  T* pointer;
public:
  SmartPointer(T* p = nullptr) { p = pointer };
  ~SmartPointer() { delete (pointer) };
  T& operator*() const { return *pointer };
  T* operator->() const { return pointer };
}

void main() {
  SmartPointer pointer(new int());
}
```

**Flaw in Smart Pointers**
- Copy constructor and assignment operator perform shallow copies.
- Leads to dangling pointers, which occurs when pointing to a memory location that has already been freed.

**Move Semantics**
- Addresses smart pointer flaws.
- Class will transfer ownership of resources from one object to another, instead of copying them.

```cpp
class SPMoveSemantics {
  T* pointer;
public:
  SPMoveSemantics(T* p = nullptr) { p = pointer };
  ~SPMoveSemantics() { delete (pointer) };

  // Copy constructor with move semantics.
  SPMoveSematics(SPMoveSemantics& o) {
    pointer = o.pointer;    // Transfer source pointer to local object.
    o.pointer = nullptr;    // Source should no longer own the pointer.
  }

  // Assignment operator with move semantics.
  SPMoveSemantics& operator=(SPMoveSemantics& o) {
    if (&o == this) return *this;
    delete pointer;         // Deallocate old pointer.
    pointer = o.pointer;    // Transfer source pointer to local object.
    o.pointer = nullptr;    / Source should no longer own the pointer.
    return *this;
  }

  // Overload dereference and assignment operators.
  T& operator*() const { return *pointer };
  T* operator->() const { return pointer };

  // Disable copy constructor and assignment operator.
  SPMoveSemantics(const SPMoveSemantics&) = delete;
  SPMoveSemantics& operator=(const SPMoveSemantics&) = delete;
}
```

**`std::auto_ptr`**
- A standardized smart pointer in C++ which incorrectly nullified source pointers during copies.
- Deprecated in C++11, removed in C++17.

**Move-Aware Smart Pointers**
- In C++11, `std::auto_ptr` was replaced by `std::unique_ptr`, `std::weak_ptr`, `std::shared_ptr`.

---

## Value Categories

**L-Values**
- Expressions that evaluate to an identifiable object with a distinct memory address.
- Persists beyond the end of the expression.
- Can either be modifiable (non-`const`) or non-modifiable (`const`).

**R-Values**
- Expressions that evaluate to a literals or values returned by functions that are later discarded.
- Not addressable and only exist within scope of the expression.

**Tip**
- Try taking its address using operator&, which requires its operand to be an lvalue.
- If &(expression); compiles, expression must be an lvalue.

```cpp
int foo() { return 5; }
int x = 5;
&x;    // Compiles: x is lvalue.
&5;    // Does not compile: 5 is rvalue.
&foo;  // Does not compile: foo() is rvalue.
```

---

## Resizing Vectors

- Resizing vectors is a trade off between the cost of computation, so copying over its elements to a resized vector, and the space used by it.
- It can be done two main ways: resizing by a constant and resizing by a scalar.
- For both cases, there are conditions to be met in order to be resized. A vector would be resized if it becomes full, or below or above a certain threshold, such as only being half-filled with elements.
- For resizing by a constant, when the vector reaches capacity, you would increase its size by a constant, allocate space for the new vector, copy its elements over, and free the space associated with the old vector.
- The process is very similar for resizing by a scalar.
- There are, however, significant differences in memory and computational time costs for each approach.
- Scalar resizing is generally more memory efficient for large vectors, although it may allocate more than needed during each resize.
- Constant resizing can be memory inefficient if we have few elements and the vector is resized by a large constant.
- Constant resizing can also be time inefficient if we repeated hit capacity and have to spend time copying.
- Scalar resizing generally has fewer resizing events, so there is less overhead, but the cost of each resizing is typically higher.
- In general, scalar resizing is better for large-scale or quickly-growing data, and constant resizing is better for more predictable behaviour and smaller amounts of data.
