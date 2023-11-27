## Pointers and References to the Base Class of Derived Objects

**Concept**:
  - In C++, pointers and references to a base class can be used to refer to objects of derived classes.
  - Fundamental in implementing polymorphism, allowing base class interfaces to interact with derived class objects.

**Behavior**:
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

**Use Cases**:
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

**Important Points**:
  - Virtual functions allow polymorphic behavior.
  - Avoid calling virtual functions in constructors/destructors.
  - Virtual function resolution only occurs through pointers or references.
  - If a function is virtual, all matching overrides in derived classes are implicitly virtual.

**Polymorphism and Efficiency**:
  - Virtual functions enable runtime polymorphism.
  - There is a performance cost due to dynamic binding.
  - Compiler has to allocate an extra pointer for each class object that has one or more virtual functions.

**Best Practices**:
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

