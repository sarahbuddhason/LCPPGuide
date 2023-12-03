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

