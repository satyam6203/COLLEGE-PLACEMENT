# 🎯 OOP Interview Questions (1–100)

> Obsidian-ready notes. Every answer is written the way you'd actually say it in an interview.

## 1. What is Object-Oriented Programming (OOP)?

**Answer:**
OOP is a way of designing software around **objects** that bundle data (state) and the methods that work on that data (behaviour) into one unit. Instead of writing a long list of functions that take data as parameters, we model the real world as classes, and objects of those classes interact with each other. It makes large programs easier to change, reuse and test.

**Example:**
In an e-commerce app, `Order` is a class with fields `id`, `items`, `total` and methods `addItem()`, `applyDiscount()`, `calculateTotal()`. The order manages its own data, and other services only call its public methods.

**Interview Follow-up:**
- Is Java a pure object-oriented language?
- Why do we still use functional features like streams and lambdas in Java?
- What are the main benefits of OOP?

**Short Interview Answer:**
OOP means designing programs around objects that combine data and behaviour. It gives modularity, code reuse and easier maintenance, because each class has a clear responsibility.

---

## 2. What are the four pillars of OOP?

**Answer:**
The four pillars are **abstraction, encapsulation, inheritance and polymorphism**. Abstraction hides complexity and exposes only essentials, encapsulation hides internal state, inheritance reuses and extends existing behaviour, and polymorphism lets one reference call different implementations. Interviewers usually want one line per pillar with a Java keyword or pattern attached.

| Pillar | Meaning | Java tool | Quick example |
|--------|---------|-----------|---------------|
| Abstraction | Show only what is needed | `abstract class`, `interface` | `List` API hides `ArrayList` internals |
| Encapsulation | Hide data, expose methods | `private` fields, getters/setters | `BankAccount` with private `balance` |
| Inheritance | Reuse and extend | `extends`, `implements` | `Employee extends Person` |
| Polymorphism | One name, many behaviours | Overloading + overriding | `Payment` → UPI / Card |

**Common Misconception:**
Encapsulation and abstraction are different — abstraction is about *design* (what the class offers), encapsulation is about *implementation* (hiding how it stores data).

**Interview Follow-up:**
- Which pillar is the most important in real projects?
- Can you have polymorphism without inheritance?
- Which pillar helps with security the most?

**Short Interview Answer:**
The four pillars are abstraction, encapsulation, inheritance and polymorphism. Abstraction and encapsulation are about hiding things, inheritance is about reuse, and polymorphism lets one interface call many implementations.

---

## 3. What is a class?

**Answer:**
A class is a **blueprint** that defines the data (fields) and behaviour (methods) its objects will have. It is not a real usable thing by itself — it is the template from which objects are created. In Java a class is compiled into a `.class` file and loaded into JVM memory once, while each object gets its own memory.

**Example:**
`class Car { String color; void accelerate() {...} }` — the class is the design of a car (wheels, engine, accelerate method). Every actual car you create is an object of that class.

**Interview Follow-up:**
- Where is a class stored in memory in Java?
- Can a class have multiple objects?
- What is a constructor in a class?

**Short Interview Answer:**
A class is a blueprint that defines the fields and methods its objects will have. It describes what an object looks like and what it can do, but you need to create an object from it to actually use it.

---

## 4. What is an object?

**Answer:**
An object is a **runtime instance** of a class — a real, usable entity with its own state in memory. Every object has three things: an identity (where it lives), state (its current field values), and behaviour (the methods it can run). Objects are created with `new` in Java.

**Example:**
`Car c1 = new Car(); c1.color = "Red";` — the class says a car *has* a colour; the object `c1` actually *has* the colour red right now. Two cars can have different colours.

**Interview Follow-up:**
- Where is an object stored in memory?
- Can an object exist without a class?
- What is the difference between identity and equality of objects?

**Short Interview Answer:**
An object is an instance of a class — a real entity in memory with its own state. It has an identity, the data it currently holds, and the methods defined by its class.

---

## 5. What is the difference between a class and an object?

**Answer:**
A class is the definition/blueprint, an object is a concrete instance of it. One class can produce many objects, each with its own independent field values, but they all share the same methods and the class's code. Think of class as a "car design" and object as the "actual car" you buy.

| Aspect | Class | Object |
|--------|-------|--------|
| Nature | Blueprint / template | Real instance |
| Memory | One copy in metaspace (code) | Separate memory per object |
| Count | One class, many objects | Each object is distinct |
| Keyword used | `class` | `new` |
| Field values | Declared only | Actual values stored |
| Example | `Student` | `new Student("Amit", 90)` |

**Interview Follow-up:**
- Does each object get its own copy of the methods?
- Can two objects share the same class?
- What happens if we don't use `new`?

**Short Interview Answer:**
A class is a blueprint, an object is an instance of that blueprint. One class can create many objects, and each object holds its own separate data.

---

## 6. What is encapsulation?

**Answer:**
Encapsulation means **binding data and the methods that operate on it into one unit and hiding the internal state** from outside code. You make fields `private` and expose them through public methods, so no one can set invalid values directly. This protects the object's invariants and makes the class safe to change later.

**Example:**
In `BankAccount`, `balance` is private. You cannot do `account.balance = -500`; you must call `withdraw(500)`, which checks balance, logs the transaction and throws an exception if funds are insufficient.

```java
class BankAccount {
    private double balance;

    void withdraw(double amt) {
        if (amt > 0 && amt <= balance) {
            balance -= amt;          // rule enforced here
        } else {
            throw new IllegalArgumentException("Invalid amount");
        }
    }
}
```

**Common Misconception:**
Encapsulation is not just "use getters and setters". A plain setter like `setBalance(5000)` still breaks the design — the point is to expose *behaviour*, not raw data.

**Interview Follow-up:**
- How is encapsulation different from abstraction?
- Does encapsulation need to be applied everywhere?
- What is the difference between a data class and an encapsulated class?

**Short Interview Answer:**
Encapsulation is hiding a class's internal data and exposing only the methods that operate on it correctly, using private fields and public methods. That way nobody can put invalid data into the object.

---

## 7. What is abstraction?

**Answer:**
Abstraction is the process of showing **only the essential features** to the user and hiding the implementation details. In Java you do this with abstract classes and interfaces — the caller works with the contract, not with how it is actually implemented. It lets you change the implementation later without affecting the caller.

**Example:**
When you write `List<String> list = new ArrayList<>(); list.add("A");` you never know how `ArrayList` stores or grows its data. You only use the `List` contract — that is abstraction.

**Common Misconception:**
Abstraction and encapsulation are not the same thing. Abstraction is about *what* the class offers (interface-level), encapsulation is about *hiding* how it works internally.

**Interview Follow-up:**
- What is the difference between abstraction and encapsulation?
- Can we achieve abstraction without an interface?
- When should I use an abstract class instead of an interface?

**Short Interview Answer:**
Abstraction means exposing only what a class does and hiding the internal implementation. In Java we do this with interfaces and abstract classes, so the caller depends on the contract, not the implementation.

---

## 8. What is inheritance?

**Answer:**
Inheritance means creating a **new class that reuses and extends an existing class**. The parent (superclass) holds the common code, and the child (subclass) adds or overrides behaviour using `extends`. It models an "is-a" relationship, e.g. `Car is-a Vehicle`.

**Example:**
`Employee extends Person` — `Employee` automatically gets `name`, `age` and `address`, and adds `salary`, `department`. This avoids repeating common fields and methods in every class.

**Common Misconception:**
Inheritance is not only about reusing code. If the relationship is not "is-a" (e.g. `Car has-a Engine`), prefer composition — deep inheritance trees become tightly coupled and hard to maintain.

**Interview Follow-up:**
- Why is inheritance discouraged in modern design?
- Is Java's `class Employee` a good design?
- What is an `is-a` vs `has-a` relationship?

**Short Interview Answer:**
Inheritance lets a child class reuse and extend a parent class using `extends`, which models an is-a relationship. It's useful for shared code, but if the relationship is really "has-a", composition is a better choice.

---

## 9. What is polymorphism?

**Answer:**
Polymorphism means **one name, many forms** — the same method call behaves differently depending on the object type or parameters. It has two types: compile-time (method overloading) and runtime (method overriding). It is what lets your code work with different implementations through a single type.

**Example:**
You write `payment.pay(500)`. If the object is `UpiPayment`, it calls the UPI flow; if it's `CardPayment`, it calls the card flow. Your code does not change.

```java
Payment payment = new UpiPayment();
payment.pay(500);   // JVM calls UpiPayment.pay() at runtime
```

**Interview Follow-up:**
- What is the difference between overloading and overriding?
- Is polymorphism possible without inheritance?
- Which type of polymorphism is used by Spring's `@Autowired`?

**Short Interview Answer:**
Polymorphism is one name with many forms. Compile-time polymorphism is overloading, runtime polymorphism is overriding, and runtime polymorphism is what lets us call different implementations through the same interface reference.

---

## 10. What are the advantages of OOP?

**Answer:**
OOP gives us **modularity, code reuse, easier maintenance and loose coupling**. Each class has one clear job, so changes are local and existing code does not break. It also maps closely to real-world entities, which makes the design easier to explain and unit-test.

**Example:**
When a new payment method is added, you just create `WalletPayment implements Payment`. No existing code changes — only the new class is added. That is the *open/closed principle* in practice.

**Interview Follow-up:**
- Is OOP always better than procedural programming?
- What are the drawbacks of OOP?
- How does OOP help with unit testing?

**Short Interview Answer:**
The main advantages are code reuse, modularity, easier maintenance and loose coupling. Because classes interact through interfaces, you can add or change implementations without touching the rest of the code.

---

## 11. What is method overloading?

**Answer:**
Method overloading means **the same method name with a different parameter list** in the same class — different number, type, or order of parameters. It is also called compile-time or static polymorphism because the compiler decides which method to call. The return type alone is **not** enough to overload.

**Example:**
```java
void add(int a, int b)      { System.out.println(a + b); }
void add(double a, double b) { System.out.println(a + b); }
void add(int a, int b, int c){ System.out.println(a + b + c); }

add(1, 2);        // -> int version
add(1.5, 2.5);    // -> double version
add(1, 2, 3);     // -> 3-arg version
```

**Common Misconception:**
Overloading is not decided by return type. Two methods `int get()` and `String get()` in the same class will **not compile**.

**Interview Follow-up:**
- Can we overload a method by changing only the return type?
- Is overloading possible across different classes?
- What happens if we pass `null` to an overloaded method?

**Short Interview Answer:**
Overloading is having several methods with the same name but different parameter lists, resolved at compile time. The compiler picks the method based on the arguments we pass, not on the return type.

---

## 12. What is method overriding?

**Answer:**
Method overriding means a **subclass provides its own implementation of a method already defined in its parent class**, with the same name and same parameter list. It is runtime (dynamic) polymorphism — the JVM decides at run time which version to execute based on the actual object. It needs inheritance, and the access level cannot be reduced.

**Example:**
```java
class Payment {
    void pay(double amt) { System.out.println("Generic payment"); }
}

class UpiPayment extends Payment {
    @Override
    void pay(double amt) { System.out.println("UPI payment of " + amt); }
}

Payment p = new UpiPayment();
p.pay(100);   // prints "UPI payment of 100.0"
```

**Common Misconception:**
Overriding is not about the parameters — they must be exactly the same (except covariant return type).

**Interview Follow-up:**
- Can we override a `final` or `static` method?
- What is covariant return type?
- Can we override a `private` method?

**Short Interview Answer:**
Overriding is when a child class redefines a parent method with the same signature and provides a different implementation. It enables runtime polymorphism because the JVM picks the method based on the actual object at run time.

---

## 13. What is the difference between method overloading and method overriding?

**Answer:**
Overloading is **same name, different parameters, same class**, resolved at compile time. Overriding is **same signature, different class (child replaces parent)**, resolved at runtime. Overloading is sometimes called static polymorphism, overriding is called dynamic polymorphism.

| Aspect | Overloading | Overriding |
|--------|-------------|------------|
| Meaning | Same name, different params | Same signature, child implementation |
| Class | Same class | Parent & child classes |
| Parameters | Must be different | Must be identical |
| Return type | Can be anything | Same or covariant |
| Binding | Compile time | Run time |
| Also called | Static polymorphism | Dynamic polymorphism |
| `static` methods | Can be overloaded | Cannot be overridden |

**Common Misconception:**
Overloading is not "the same method with a different body" — the parameter list must change. And you cannot overload by changing only the return type.

**Interview Follow-up:**
- Can `main` be overloaded?
- Can we override a method with a `varargs` parameter?
- Why can a static method not be overridden?

**Short Interview Answer:**
Overloading means the same method name with different parameters in the same class, resolved at compile time. Overriding means a child class re-implements a parent method with the same signature, resolved at runtime.

---

## 14. What is a constructor?

**Answer:**
A constructor is a **special method called automatically when an object is created**, used to initialize the object's fields. It has the **same name as the class**, has **no return type** (not even `void`), and runs before any other method on the object. If you do not write one, Java provides a default no-arg constructor.

**Example:**
```java
class Student {
    String name;
    int rollNo;

    Student(String name, int rollNo) {   // parameterized constructor
        this.name = name;
        this.rollNo = rollNo;
    }
}
```
`new Student("Amit", 10)` runs this constructor before anything else.

**Common Misconception:**
A constructor has no return type. Writing `void Student()` makes it a normal method, not a constructor.

**Interview Follow-up:**
- Can a constructor be `static` or `final`?
- Can it be inherited?
- Can it call another constructor?

**Short Interview Answer:**
A constructor is a special method with the class name that runs automatically when we create an object, mainly to initialize its fields. It has no return type and cannot be overridden or inherited.

---

## 15. What are the types of constructors?

**Answer:**
In Java there are **three types**: default (no-arg) constructor, parameterized constructor, and copy constructor (not built-in in Java, but you can write one). Java does not have destructors — object cleanup is handled by the garbage collector. In C++ you also have a move constructor.

| Type | Purpose | Example |
|------|---------|---------|
| Default / no-arg | Initializes fields to default values | `Student()` |
| Parameterized | Initializes fields with given values | `Student(String name)` |
| Copy (custom) | Creates a copy of another object | `Student(Student s)` |

**Interview Follow-up:**
- Why does Java not have a built-in copy constructor?
- What is a constructor initializer list in C++?
- Can a class have more than one constructor?

**Short Interview Answer:**
Java mainly has default and parameterized constructors. Copy and move constructors are not built-in in Java — you write a copy constructor manually if you need deep copying.

---

## 16. What is a destructor?

**Answer:**
A destructor is a method called **when an object is about to be destroyed**, used to release resources like files, sockets and DB connections. **Java does not have destructors** — memory is cleaned by the garbage collector, and `finalize()` was deprecated for removal. In Java the correct approach is to implement `AutoCloseable` / `Closeable` and use **try-with-resources**, which guarantees cleanup.

**Example:**
```java
try (FileInputStream fis = new FileInputStream("data.txt")) {
    // use fis
}   // fis.close() called automatically, even if an exception occurs
```

**Common Misconception:**
`finalize()` is not a reliable way to free resources. It may never run, and it was deprecated in Java 9 and removed later — always use try-with-resources or a framework-managed lifecycle.

**Interview Follow-up:**
- Why did Java remove `finalize()`?
- How do you release resources safely in Java?
- What does the garbage collector actually collect?

**Short Interview Answer:**
Java has no destructors. Cleanup is handled by the garbage collector, and for resources like files and connections we use try-with-resources with `AutoCloseable` instead of relying on `finalize()`.

---

## 17. What is a default constructor?

**Answer:**
A default constructor is the **no-argument constructor that the compiler generates automatically** when a class does not declare any constructor. It initializes all fields to their default values — `0` for primitives, `null` for objects, `false` for `boolean`. Its access level is the same as the class's access level.

**Example:**
```java
class Student {
    String name;     // becomes null
    int age;         // becomes 0
}
// Student s = new Student();  -> compiler creates Student() for us
```

**Common Misconception:**
If you define any constructor yourself, the compiler **no longer** creates the default one. `new Student()` will then be a compile error.

**Interview Follow-up:**
- What is the default value of a `char`?
- What happens if the class has a private constructor?
- Is the default constructor always public?

**Short Interview Answer:**
The default constructor is the no-arg constructor the compiler creates automatically when a class has no explicit constructor, and it sets all fields to their default values like `0` or `null`.

---

## 18. What is a parameterized constructor?

**Answer:**
A parameterized constructor is a constructor that **takes one or more parameters** and uses them to initialize the object's fields. It is the most common way to give a new object its initial, valid state. You can have as many as you like as long as the parameter lists differ.

**Example:**
```java
class BankAccount {
    private String owner;
    private double balance;

    BankAccount(String owner, double openingBalance) {
        this.owner = owner;
        this.balance = openingBalance;
    }
}
```
`new BankAccount("Amit", 5000)` — here both fields get real values instead of `null` and `0`.

**Interview Follow-up:**
- Can a class have only parameterized constructors?
- What is `this()` and how is it used here?
- Why is it better than setters for setting initial state?

**Short Interview Answer:**
A parameterized constructor accepts parameters to initialize the object's fields with real values instead of default values. It guarantees the object is valid from the moment it is created.

---

## 19. What is a copy constructor?

**Answer:**
A copy constructor creates a **new object as a copy of an existing object of the same class**. C++ provides one implicitly; Java does **not** — the default copy only copies field references (shallow copy). In Java we write it manually to get a deep copy when needed.

**Example:**
```java
class Student {
    String name;
    List<String> courses;

    Student(Student other) {                 // copy constructor
        this.name = other.name;
        this.courses = new ArrayList<>(other.courses);  // deep copy
    }
}
```

**Common Misconception:**
In Java, assigning `Student b = a;` does **not** copy the object — it copies the reference, so `a` and `b` point to the same object.

**Interview Follow-up:**
- What is the difference between a copy constructor and `clone()`?
- How do you do a deep copy in Java?
- What is the difference between a shallow and deep copy?

**Short Interview Answer:**
A copy constructor builds a new object from an existing one. Java doesn't provide it by default, so we write it manually, and we must copy nested objects ourselves if we want a deep copy.

---

## 20. What is constructor overloading?

**Answer:**
Constructor overloading means defining **multiple constructors in the same class with different parameter lists**. It lets callers create objects conveniently in different ways, and the right constructor is chosen at compile time. Inside a constructor you can reuse another constructor using `this(...)`.

**Example:**
```java
class Student {
    Student() { this("Unknown", 0); }              // delegates
    Student(String name) { this(name, 0); }       // delegates
    Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```
Now all of `new Student()`, `new Student("Amit")` and `new Student("Amit", 20)` work.

**Common Misconception:**
`this()` must be the **first statement** in a constructor, and you can call only one other constructor from it.

**Interview Follow-up:**
- Why must `this()` be the first statement?
- Can `this()` and `super()` both be used in one constructor?
- What happens if you call `this()` after setting fields?

**Short Interview Answer:**
Constructor overloading means having several constructors with different parameter lists, so we can create objects in different ways. Inside one constructor we use `this(...)` to reuse another constructor's logic.

---

## 21. What is an abstract class?

**Answer:**
An abstract class is a class declared with the `abstract` keyword that **cannot be instantiated** and may contain both abstract methods (no body) and concrete methods. It exists to share common code and define a partial contract for its subclasses. Any subclass must implement all abstract methods unless it is also abstract.

**Example:**
```java
abstract class Vehicle {
    void print() { System.out.println("Every vehicle has wheels"); }  // shared code

    abstract double calculateMileage();                              // contract
}

class Car extends Vehicle {
    @Override double calculateMileage() { return 15.5; }
}
```

**Interview Follow-up:**
- Can an abstract class have constructors?
- Can it have `static` and `final` methods?
- Can it implement an interface without implementing all methods?

**Short Interview Answer:**
An abstract class cannot be instantiated and can contain both abstract and concrete methods. It is used to share common code and force subclasses to follow a contract.

---

## 22. What is an interface?

**Answer:**
An interface is a **pure contract** that specifies *what* a class should do, not *how*. All its methods are abstract by default; since Java 8 it can also contain `default` and `static` methods, and since Java 9 `private` helper methods. A class can implement **multiple** interfaces, which is how Java achieves multiple inheritance.

**Example:**
```java
interface Payment {
    void pay(double amount);              // implicitly public abstract

    default boolean refundable() { return true; }   // default method
}
```

**Common Misconception:**
Interface fields are **not** normal fields. They are implicitly `public static final` constants, so every implementation shares them.

**Interview Follow-up:**
- Can an interface have state/instance variables?
- Can we create an object of an interface directly?
- What are the changes to interfaces in Java 8 and 9?

**Short Interview Answer:**
An interface defines a contract of what a class must do, not how it does it. Java allows a class to implement multiple interfaces, and since Java 8 interfaces can also hold default and static methods.

---

## 23. What is the difference between an abstract class and an interface?

**Answer:**
An abstract class can hold state (instance variables), constructors and both abstract and concrete methods, but a class can extend only **one** abstract class. An interface cannot hold instance state, but a class can implement **many** interfaces. Use an interface for capability/contract and an abstract class when you need shared code plus state.

| Aspect | Abstract Class | Interface |
|--------|----------------|-----------|
| Keyword | `extends` | `implements` |
| Inheritance | Single only | Multiple allowed |
| Instance variables | Allowed | Not allowed (only constants) |
| Constructor | Allowed | Not allowed |
| Method types | Abstract + concrete | Abstract + default + static |
| Access | Any modifier | Members are `public` |
| When to use | Shared code + state | Define capability/contract |

**Interview Follow-up:**
- Can an interface extend multiple interfaces?
- Why did Java add default methods to interfaces?
- Which one should you use for a payment gateway?

**Short Interview Answer:**
Abstract class allows state, constructors and partial implementation with single inheritance, while an interface is a pure contract with no state but supports multiple inheritance. Use an abstract class when you need shared code, and an interface when you only need to define a capability.

---

## 24. Can an abstract class have a constructor?

**Answer:**
Yes — an abstract class **can have constructors**, and they are used to initialize the state inherited by its subclasses. The abstract class itself cannot be instantiated, but the constructor runs when a subclass object is created. It is a good place for common initialization shared by all subclasses.

**Example:**
```java
abstract class Vehicle {
    protected final String registration;

    Vehicle(String registration) {      // called via super() from child
        this.registration = registration;
    }
}
```
`new Car("MH-12-1234")` calls `Car`'s constructor, which calls `super(...)`.

**Common Misconception:**
Having a constructor does not make the class instantiable — an abstract class can never be created with `new`, no matter what constructors it has.

**Interview Follow-up:**
- Is the constructor called if the subclass doesn't define one?
- What is the order of initialization in Java?
- Can the abstract class constructor be private?

**Short Interview Answer:**
Yes, an abstract class can have a constructor, and it runs when a subclass object is created. It is used to initialize common state, even though the abstract class itself can never be instantiated.

---

## 25. Can an interface have variables?

**Answer:**
Interfaces can have variables, but they are **implicitly `public static final` constants** — always. Since Java 9, interface fields can be `private` only in the sense of private helper methods; the fields themselves remain public static final. Interfaces are used to declare shared constants, not per-object state.

**Example:**
```java
interface Payment {
    int MAX_AMOUNT = 100000;    // public static final
    double DEFAULT_TAX = 0.18;
}
```

**Common Misconception:**
You cannot declare a non-final, non-static field in an interface, and you cannot change `MAX_AMOUNT` later — it is baked in at compile time.

**Interview Follow-up:**
- Can an interface have a non-static field?
- Can we modify an interface variable inside an implementing class?
- Where are interface variables stored in memory?

**Short Interview Answer:**
Yes, but interface variables are always public static final constants. They are meant for shared constants, not for storing per-object data.

---

## 26. Can an interface have methods with implementation?

**Answer:**
Yes, since **Java 8** an interface can have `default` methods with an implementation, plus `static` methods; since **Java 9** it can also have `private` methods. Default methods exist mainly for backward compatibility — so adding a new method to an interface doesn't break every existing implementation.

**Example:**
```java
interface Notification {
    void send(String msg);

    default boolean supportsPush() { return false; }   // implementation in interface
    static Notification noop() { return msg -> {}; }  // static method
}
```

**Interview Follow-up:**
- Why were default methods introduced in Java 8?
- What happens if a class implements two interfaces with the same default method?
- Can a default method be `final`?

**Short Interview Answer:**
Yes — Java 8 allows default and static methods with implementations inside interfaces, and Java 9 added private methods. Default methods were added so that new methods could be added to old interfaces without breaking existing classes.

---

## 27. What is multiple inheritance?

**Answer:**
Multiple inheritance means a class **inherits from more than one parent class** at the same time. It increases flexibility but creates ambiguity when both parents define the same member — the language must decide which one wins. Java does **not** allow this for classes; it is only possible through interfaces.

**Example:**
In C++ a class can extend both `FileLogger` and `DbLogger`. In Java you cannot write `class AppLogger extends FileLogger, DbLogger` — instead both would be interfaces, and `AppLogger implements FileLogger, DbLogger`.

**Common Misconception:**
Multiple inheritance of **classes** is not possible in Java, but multiple inheritance of **interfaces** is fully supported.

**Interview Follow-up:**
- Why did Java designers reject multiple inheritance of classes?
- How do interfaces solve this ambiguity?
- Is multiple inheritance good design?

**Short Interview Answer:**
Multiple inheritance means a class inherits from two or more parent classes. Java does not support it for classes because of ambiguity, but it does allow a class to implement multiple interfaces.

---

## 28. Does Java support multiple inheritance?

**Answer:**
Java supports multiple inheritance **of interfaces only**, not of classes. A class can implement any number of interfaces, and if two interfaces provide the same default method, the class **must** override it and use `InterfaceName.super.method()` to resolve it explicitly. Java uses interfaces because they carry behaviour only, with no state to inherit, so conflicts stay small.

**Example:**
```java
class Logger implements FileLogger, DbLogger {
    @Override
    public void log(String msg) {
        FileLogger.super.log(msg);   // explicit disambiguation
    }
}
```

**Interview Follow-up:**
- What is the diamond problem in Java interfaces?
- What happens if a class inherits two identical default methods?
- Can an interface extend multiple interfaces?

**Short Interview Answer:**
Java allows multiple inheritance through interfaces, not classes. If two interfaces have the same default method, the implementing class must override it and call `InterfaceName.super.method()` to be explicit.

---

## 29. What is hybrid inheritance?

**Answer:**
Hybrid inheritance is a **combination of two or more inheritance types** — for example, multiple inheritance plus multilevel inheritance in one hierarchy. C++ supports it directly, which is where the ambiguities become most visible. In Java we approximate it using interfaces plus class inheritance.

**Example:**
In C++, `C` inherits from `A` and `B`, and `A` itself inherits from `Base` — that is multilevel plus multiple, i.e. hybrid.

**Interview Follow-up:**
- Which language allows hybrid inheritance?
- Why is hybrid inheritance risky?
- How is it handled in Java?

**Short Interview Answer:**
Hybrid inheritance mixes two or more inheritance types, like multiple plus multilevel. C++ allows it directly; Java does not, so we achieve the same result using interfaces together with class inheritance.

---

## 30. What is hierarchical inheritance?

**Answer:**
Hierarchical inheritance is when **many child classes inherit from one single parent class**. The parent holds the shared fields and methods, and each child adds its own specific behaviour. It is a very common and safe form of inheritance because there is only one level and no ambiguity.

**Example:**
`Vehicle` as the parent, with `Car`, `Bike` and `Truck` extending it — each child gets common attributes like `numberOfWheels` while adding its own behaviour.

**Interview Follow-up:**
- What is the benefit over duplicating code in each class?
- Why is it safer than multiple inheritance?
- How would you model this in a Spring Boot app?

**Short Interview Answer:**
Hierarchical inheritance is when several classes inherit from one parent class. It is the cleanest inheritance type because shared code lives in one place and there is no ambiguity.

---

## 31. What is single inheritance?

**Answer:**
Single inheritance is when a class inherits from **exactly one parent class** — Java's normal way of reusing code with `extends`. It keeps the class hierarchy simple and avoids the ambiguities of multiple inheritance. Java class hierarchies are always single-inheritance based.

**Example:**
`class Car extends Vehicle` — Car has one parent, and it gets Vehicle's common code plus its own.

**Interview Follow-up:**
- Can a class have an interface and a superclass at the same time?
- Is single inheritance always enough?
- What are the limits of single inheritance in Java?

**Short Interview Answer:**
Single inheritance means a class extends only one parent class. Java always uses single inheritance for classes, and to get the flexibility of multiple inheritance it uses interfaces.

---

## 32. What is multilevel inheritance?

**Answer:**
Multilevel inheritance is when a class **inherits from a class that itself inherits from another class**, forming a chain — Grandparent → Parent → Child. It reuses code at every level, but the hierarchy becomes harder to maintain as the chain gets deeper. This is a common criticism of inheritance in real projects.

**Example:**
`Animal` → `Mammal` → `Dog`. `Dog` gets methods from `Mammal` (breathing, feeding) and also from `Animal` (alive, age) through the chain.

**Common Misconception:**
Deep hierarchies are not automatically good design. If a chain is more than 3–4 levels, it usually means composition would be cleaner.

**Interview Follow-up:**
- Does Java support multilevel inheritance?
- Why is deep inheritance considered bad design?
- What problem does it create for subclasses?

**Short Interview Answer:**
Multilevel inheritance is a chain where a class inherits from a class that inherits from another class. It reuses code well, but deep chains make the code harder to maintain and couple classes tightly.

---

## 33. What is the diamond problem?

**Answer:**
The diamond problem occurs when a class inherits the **same method through two different paths**, so the compiler or runtime cannot decide which implementation to use. It is common in multiple inheritance (C++) and also appears in Java when a class implements two interfaces with the same default method.

**Example:**
```java
interface A { default void show() { print("A"); } }
interface B { default void show() { print("B"); } }

class C implements A, B {
    @Override public void show() { A.super.show(); }  // must resolve manually
}
```
Without the override, `C` is not fully abstract and compilation fails with an ambiguity error.

**Interview Follow-up:**
- How is it solved in C++?
- How does Java solve it?
- Why does Java have this problem even without class multiple inheritance?

**Short Interview Answer:**
The diamond problem is ambiguity when the same method comes through two inheritance paths. In Java you fix it by overriding the method and calling `InterfaceName.super.method()`.

---

## 34. How is the diamond problem solved in C++?

**Answer:**
C++ solves it using **virtual inheritance** (`virtual` keyword). With virtual inheritance, the shared base class exists as **one single object** in memory instead of two separate copies, so there is no ambiguity and no duplicated state. It also needs a **virtual destructor** in the base class so deletion through the base pointer works correctly.

**Example:**
```cpp
class Base { public: int x; virtual ~Base() {} };
class Left  : virtual public Base {};
class Right : virtual public Base {};
class Child : public Left, public Right {};
// Child has only ONE Base sub-object
```

**Interview Follow-up:**
- What happens if the base class does not have a virtual destructor?
- What is the cost of virtual inheritance?
- Is virtual inheritance used in Java?

**Short Interview Answer:**
C++ fixes the diamond problem with virtual inheritance, so the shared base exists only once in memory. A virtual destructor in the base class is also needed to avoid memory leaks.

---

## 35. What are the access modifiers?

**Answer:**
Access modifiers control **who can access a class, field, method or constructor**. Java has four levels: `public`, `protected`, **default (package-private)**, and `private`. They define visibility across packages, which is how Java enforces encapsulation at compile time.

| Modifier | Same class | Same package | Subclass (other package) | Anywhere |
|----------|-----------|--------------|--------------------------|----------|
| `public` | Yes | Yes | Yes | Yes |
| `protected` | Yes | Yes | Yes | No |
| default (none) | Yes | Yes | No | No |
| `private` | Yes | No | No | No |

**Interview Follow-up:**
- Which modifier is used most in practice?
- Why is `default` (package-private) useful?
- Is there a difference between default access and `protected` in the same package?

**Short Interview Answer:**
Access modifiers decide visibility — `public` for everyone, `protected` for the class, its package and subclasses, `private` for only the class itself, and no modifier means package-private.

---

## 36. What is the difference between public, private, and protected access modifiers?

**Answer:**
`public` allows access from anywhere; `private` allows access only inside the same class; `protected` allows access inside the class, its package and its subclasses in other packages. For fields, `private` is the default choice and you expose them through public methods to preserve encapsulation.

**Example:**
A `private` helper method in a service is an internal detail. A `protected` field in an entity can be used by a subclass in another package. A `public` API method is meant for external callers.

**Common Misconception:**
`private` members are not inherited — a subclass cannot access a parent's private field directly, it needs a `protected` field or a getter.

**Interview Follow-up:**
- Can a constructor be `private`?
- Why should fields not be `public`?
- What is the default access level for a class?

**Short Interview Answer:**
`public` means anyone can access it, `private` means only the same class, and `protected` means the same package plus subclasses. In practice I keep fields private and expose behaviour through public methods.

---

## 37. What is the difference between protected and private members?

**Answer:**
`private` members are visible **only inside the declaring class**, while `protected` members are also visible to **subclasses (including in other packages)** and to classes in the same package. So `private` is the tightest level of encapsulation, and `protected` is a controlled way to let child classes reuse state.

**Example:**
`class Account { private double balance; protected String owner; }` — an `Account` subclass can read `owner` but not `balance` directly; it must use a public `getBalance()`.

**Interview Follow-up:**
- Are `protected` and package-private the same within a package?
- Why is `protected` risky in a deep hierarchy?
- What does `protected final` mean?

**Short Interview Answer:**
`private` is visible only inside the class, `protected` also to subclasses and same-package classes. That means `protected` lets child classes reuse parent state without exposing it to the whole world.

---

## 38. What is the difference between instance variables and class variables?

**Answer:**
An instance variable is declared **per object** (without `static`), so every object gets its own copy in heap memory. A class variable is declared with `static`, so only **one shared copy** exists for the whole class, stored with the class in memory. Use class variables for shared counters or constants, not for per-object data.

**Example:**
```java
class Employee {
    String name;              // per object
    static int empCount;      // shared by all
}
```
Every new `Employee` has its own `name`, but `empCount` is one number shared across all employees.

**Interview Follow-up:**
- Where are static variables stored in memory?
- Are static variables thread-safe?
- Can a static variable be `final`?

**Short Interview Answer:**
Instance variables belong to each object and have separate copies, while class variables are declared `static` and shared by all instances of that class.

---

## 39. What is a static variable?

**Answer:**
A static variable is declared with the `static` keyword and belongs to the **class, not to any object**. It is created once when the class is loaded and shared by every instance. It is mainly used for shared counters, constants and configuration-like values.

**Example:**
```java
class Counter {
    static int count = 0;      // one shared copy
    Counter() { count++; }
}
```
Creating 10 objects leaves `count` at 10.

**Common Misconception:**
Static variables are **not** stored on the stack. They live with the class in the JVM's method area/metaspace, which is why they are shared across threads.

**Interview Follow-up:**
- Is a static variable thread-safe?
- Can we access a static variable without creating an object?
- What is the default value of a static variable?

**Short Interview Answer:**
A static variable belongs to the class rather than the objects, so one copy is created and shared by all instances. It is useful for shared counters and constants.

---

## 40. What is a static method?

**Answer:**
A static method belongs to the **class** and can be called without creating an object, using `ClassName.method()`. Inside it we cannot use `this` or access instance variables and methods directly, because there is no current object. Utility methods and factory methods are usually static.

**Example:**
```java
class Util {
    static boolean isValidEmail(String email) {
        return email != null && email.contains("@");   // no instance data used
    }
}
String result = Util.isValidEmail("a@b.com");
```

**Common Misconception:**
A static method cannot be inherited, but it can be **inherited** (used by subclasses) — it just cannot be *overridden*.

**Interview Follow-up:**
- Can a static method be called from an instance method?
- Can a static method use `this`?
- Can we override a static method?

**Short Interview Answer:**
A static method belongs to the class and can be called without creating an object, so it cannot use `this` or instance variables directly. It is mainly used for utility and factory methods.

---

## 41. Can static methods be overridden?

**Answer:**
No. Static methods **cannot be overridden**, they can only be **hidden**. Since a static method belongs to the class and not the object, there is no runtime object to decide which version to call, so the compiler uses the reference type. If a subclass declares the same static method, the parent's version is simply hidden.

**Example:**
```java
class Parent { static void show() { print("Parent"); } }
class Child  extends Parent { static void show() { print("Child"); } }

Parent ref = new Child();
ref.show();      // "Parent"  <- compile-time decision by reference type
```

**Interview Follow-up:**
- Can static methods be overloaded?
- What is the difference between hiding and overriding?
- Are `private` methods virtually inherited?

**Short Interview Answer:**
Static methods cannot be overridden, only hidden, because they belong to the class rather than the object. The compiler decides which one to call based on the reference type, at compile time.

---

## 42. What is method hiding?

**Answer:**
Method hiding happens when a subclass declares a **static method with the same signature as a static method in the parent** — the child's method hides the parent's. The version that runs is chosen at **compile time** based on the reference type, not the actual object. It is the static counterpart of overriding.

**Example:**
```java
class A { static void display() { print("A"); } }
class B extends A { static void display() { print("B"); } }

A obj = new B();
obj.display();     // "A" — because obj is of type A (compile time)
```

**Common Misconception:**
The child's static method does not "override" the parent's — the parent's is not even inherited by instances in a dynamic sense.

**Interview Follow-up:**
- What happens with instance methods in the same situation?
- Can an instance method hide a static method?
- How is hiding different from overriding?

**Short Interview Answer:**
Method hiding is when a child class declares a static method with the same signature as the parent's, and the parent's version is hidden rather than overridden. The call is resolved at compile time using the reference type.

---

## 43. What is dynamic binding?

**Answer:**
Dynamic (late) binding means the **method call is resolved at runtime** based on the actual type of the object, not the reference type. This is exactly what happens with method overriding — the JVM uses the object's real class and its method table (vtable) to find the right implementation. It is the mechanism behind runtime polymorphism.

**Example:**
```java
Animal a = new Dog();
a.speak();     // binds to Dog.speak() at runtime
```
Even though the variable type is `Animal`, the JVM executes `Dog`'s version.

**Common Misconception:**
Fields are **not** dynamically bound — they always resolve using the reference type, which is why `a.name` gives the `Animal` version.

**Interview Follow-up:**
- What is a vtable?
- Are final methods dynamically bound?
- Is method overloading dynamic or static?

**Short Interview Answer:**
Dynamic binding means the JVM decides at runtime which method to execute, based on the actual object type rather than the reference type. That is how overriding and runtime polymorphism work.

---

## 44. What is static binding?

**Answer:**
Static (early) binding means the **method call is resolved at compile time**, using the reference type and the method signature. It applies to method overloading, `static` methods, `final` methods, `private` methods and field access. Since the decision happens before the program runs, it is also faster.

**Example:**
```java
Animal a = new Dog();
a.display();     // static method -> Animal.display(), decided at compile time
a.name;          // field access -> Animal's name, not Dog's
```

**Interview Follow-up:**
- Why is static binding faster?
- Which methods are bound at compile time?
- Can dynamic binding be avoided?

**Short Interview Answer:**
Static binding means the compiler decides which method to call before the program runs, based on the reference type. Overloaded methods, static methods, final methods and private methods are all bound at compile time.

---

## 45. What is runtime polymorphism?

**Answer:**
Runtime polymorphism is when the **actual method to execute is chosen at runtime** based on the object's real type, which happens through method overriding. It lets you keep a collection of different types under one reference type and still get the right behaviour.

**Example:**
```java
List<Payment> payments = List.of(new UpiPayment(), new CardPayment());
for (Payment p : payments) {
    p.pay(500);     // each runs its own implementation
}
```

**Interview Follow-up:**
- What is needed to achieve runtime polymorphism?
- Can it work without inheritance?
- How does Spring use runtime polymorphism?

**Short Interview Answer:**
Runtime polymorphism is the same call resolving to different methods at runtime based on the actual object type. It requires overriding, and it's how we call different implementations through a single interface reference.

---

## 46. What is compile-time polymorphism?

**Answer:**
Compile-time polymorphism is when the method call is resolved **during compilation**, which is exactly what method overloading does — the compiler picks the method based on the argument types. Generic types in Java also resolve at compile time. It is fast and has no run-time overhead.

**Example:**
```java
void show(int x)    { print("int"); }
void show(String s) { print("String"); }

show(10);       // "int"     — decided while compiling
show("hello");  // "String"  — decided while compiling
```

**Interview Follow-up:**
- Why is it also called static polymorphism?
- Is overloading an example of runtime polymorphism?
- Which is better, compile-time or runtime polymorphism?

**Short Interview Answer:**
Compile-time polymorphism is method overloading, where the compiler chooses the right method while compiling based on the parameters passed. It is also called static polymorphism because nothing is decided at runtime.

---

## 47. What is a virtual function?

**Answer:**
A virtual function is a base-class function that can be **overridden by a derived class**, and the derived version is called when the call is made through a base reference. In C++ you must mark it `virtual`. In Java, **every instance method is virtual by default**, except `static`, `final` and `private` methods.

**Example:**
```java
class Parent { void show() { print("Parent"); } }        // virtual by default
class Child extends Parent { @Override void show() { print("Child"); } }

Parent p = new Child();
p.show();     // "Child" — virtual dispatch
```

**Common Misconception:**
In C++, a virtual function is not virtual unless you write `virtual` — in Java you don't write anything.

**Interview Follow-up:**
- What is a pure virtual function?
- Why is the destructor virtual in base classes?
- Are private methods virtual in Java?

**Short Interview Answer:**
A virtual function is one whose implementation can be overridden by a child class and resolved at runtime through the base reference. In Java every normal instance method is virtual by default.

---

## 48. What is a pure virtual function?

**Answer:**
A pure virtual function is declared in C++ with `= 0` and has **no implementation** — the derived class is forced to provide it. A class containing a pure virtual function becomes **abstract** and cannot be instantiated, which is C++'s way of expressing an interface. If such a class also owns resources, its destructor should be `virtual`.

**Example:**
```cpp
class Shape {
public:
    virtual double area() const = 0;     // pure virtual
    virtual ~Shape() {}                  // virtual destructor
};
```

**Interview Follow-up:**
- Can a pure virtual function have a body?
- How does it map to Java's abstract method?
- What happens if a derived class does not override it?

**Short Interview Answer:**
A pure virtual function in C++ is declared with `= 0` and has no body, so the class becomes abstract and every derived class must override it. It is similar to a Java abstract method.

---

## 49. What is an abstract method?

**Answer:**
An abstract method is declared **without a body** and has no implementation — just a signature ending with a semicolon. Any non-abstract class that inherits it must implement it, otherwise the class must also be declared abstract. In interfaces, methods are abstract implicitly, so the `abstract` keyword is optional.

**Example:**
```java
abstract class Shape {
    abstract double area();       // no body
}

class Circle extends Shape {
    @Override double area() { return Math.PI * r * r; }
}
```

**Common Misconception:**
An abstract method cannot be `private`, `static` or `final` — those modifiers conflict with the idea of overriding it in a subclass.

**Interview Follow-up:**
- Can an abstract method be `synchronized` or `native`?
- Who is allowed to call an abstract method?
- Is it possible to have a class with no abstract methods declared `abstract`?

**Short Interview Answer:**
An abstract method has only a signature and no implementation, so it just declares that subclasses must provide the behaviour. If a class does not implement it, that class must be abstract too.

---

## 50. What is a final class?

**Answer:**
A class declared `final` **cannot be extended** by any subclass — it is the complete implementation with no extensions allowed. This gives security and predictability: the compiler can trust the class's behaviour and optimise it. Java's `String` and the wrapper classes like `Integer` are final.

**Example:**
```java
final class Utility { }      // compile error: class Utility is final
// class Helper extends Utility { }
```

**Common Misconception:**
`final` on a class means its methods and fields are automatically final too — actually it does prevent overriding, but the class can still have non-final fields.

**Interview Follow-up:**
- Why is `String` final?
- Can a final class have non-final methods?
- What is the benefit of a final class for the JIT?

**Short Interview Answer:**
A final class cannot be inherited, so its behaviour is guaranteed and cannot change. Java makes `String` final for thread safety and to allow safe optimizations.

---

## 51. What is a final method?

**Answer:**
A method declared `final` **cannot be overridden** by a subclass, though it can still be inherited and called normally. It is used to lock down critical logic so subclasses cannot break the rules or invariants of the parent. `Object`'s `getClass()` and `wait()`/`notify()` are final for exactly this reason.

**Example:**
```java
class Payment {
    public final void audit() { print("cannot be changed"); }
}
class UpiPayment extends Payment {
    @Override public void audit() { }     // compile error
}
```

**Interview Follow-up:**
- Can a `final` method be overloaded?
- Can we call a final method from a constructor?
- Why is `getClass()` final?

**Short Interview Answer:**
A final method cannot be overridden by a child class, so it fixes the behaviour of that method. It is used for critical logic that subclasses must not change.

---

## 52. What is object slicing?

**Answer:**
Object slicing is a **C++ problem** that happens when a derived object is assigned to a base class object **by value** — the copy only contains the base part, and the derived-specific data is silently cut off. In Java this cannot happen because objects are always referenced, so assignment copies the reference.

**Example:**
```cpp
Base b = derived;      // slicing: derived members lost
Base& b = derived;     // no slicing, use a reference
```

**Common Misconception:**
Java has no slicing — `Base b = new Derived();` keeps the full object because `b` is just a reference to it.

**Interview Follow-up:**
- Does Java have object slicing?
- How do you avoid slicing in C++?
- What happens with `auto b = derived;` in C++?

**Short Interview Answer:**
Object slicing is when a derived object is copied into a base variable by value, and the extra derived data is lost. It only happens in C++ — in Java assignment copies the reference, so nothing is lost.

---

## 53. What is the difference between stack memory and heap memory?

**Answer:**
Stack memory is a **small, fast, thread-local** area used for local variables, method frames and the references (pointers) themselves. Heap memory is a **large, shared** area where all objects are actually stored and managed by the garbage collector. In Java, objects always live in the heap, while local variable references and primitives are held in the stack frame of the running thread.

**Example:**
```java
void process() {
    int x = 10;              // primitive -> stack
    User u = new User();     // reference -> stack, object -> heap
}   // frame popped: x and the reference vanish, but u stays in heap until GC
```

**Common Misconception:**
Variables are not "stored in stack memory" — the reference is on the stack, the object is on the heap. Each thread has its own stack, so stack data is automatically thread-safe.

**Interview Follow-up:**
- What is a stack overflow and how is it different from OOM?
- Where are static variables stored?
- Which memory is managed by the GC?

**Short Interview Answer:**
Stack holds method frames, local variables and references and is thread-local, while the heap holds actual objects and is shared and managed by the garbage collector. In Java objects always go on the heap.

---

## 54. What is shallow copy?

**Answer:**
A shallow copy copies the object's own field values, but for reference fields it just **copies the reference**, not the object it points to. So both objects end up **sharing** the same nested object, and changing it through one is visible through the other. This is the default behaviour of `clone()` and of the assignment `b = a` in Java.

**Example:**
```java
Employee e1 = new Employee("Amit");
Employee e2 = e1;              // shallow: same reference

e1.name = "Amit2";
System.out.println(e2.name);   // "Amit2" — they point to the same object
```

**Interview Follow-up:**
- What is the difference between a shallow and deep copy?
- How do you create a deep copy in Java?
- Is Java assignment a shallow copy or an object copy?

**Short Interview Answer:**
A shallow copy copies only the top-level values, and nested object references are shared between the two objects. So modifying a nested object through one copy also affects the other.

---

## 55. What is deep copy?

**Answer:**
A deep copy **recursively copies** every nested object as well, so the two objects share nothing in memory. Changing any part of the copy leaves the original completely untouched. In Java you implement it manually with a copy constructor or `clone()` that also copies the nested fields, and it matters whenever your object is mutable and shared across threads.

**Example:**
```java
Student deep(Student s) {
    Student copy = new Student(s.name);
    copy.courses = new ArrayList<>(s.courses);   // new list, not shared
    return copy;
}
```

**Interview Follow-up:**
- Which is more expensive, shallow or deep copy?
- Is `clone()` a deep copy?
- How do immutable objects help here?

**Short Interview Answer:**
A deep copy recursively copies all nested objects, so the two objects are completely independent. It is more memory and time costly, but changing one never affects the other.

---

## 56. What is a friend function?

**Answer:**
A friend function is a **non-member function of a class** that is given access to that class's private and protected members. It is declared inside the class with the `friend` keyword but is not actually a member — it has no `this` pointer and cannot be inherited or overridden. In Java there is no `friend`, and we achieve similar access using package-private members or getters/setters.

**Example:**
```cpp
class Box {
    int value;
    friend void printValue(Box b);   // allowed to read b.value
};
void printValue(Box b) { cout << b.value; }
```

**Common Misconception:**
Friend functions break encapsulation — use them sparingly, only when a helper is genuinely needed by two classes.

**Interview Follow-up:**
- Does Java have friend classes?
- What is the difference between a friend function and a static member function?
- Why are friend functions discouraged?

**Short Interview Answer:**
A friend function in C++ is a non-member function that can access a class's private members. Java has no such keyword — we use getters, setters or package-private access instead.

---

## 57. What is operator overloading?

**Answer:**
Operator overloading means defining a **custom behaviour for an existing operator** (`+`, `-`, `==`) for your own user-defined class. C++ allows it for most operators; **Java does not** allow operator overloading for classes — we instead use method names like `add()`, `equals()`, `compareTo()`. So in Java you see overloading of *methods*, not of operators.

**Example:**
```cpp
class Complex {
    Complex operator+(const Complex& o) { return Complex(r + o.r, i + o.i); }
};
```
In Java you would write `c1.add(c2)` instead.

**Common Misconception:**
Java supports overloading (many methods, same name) but **not** operator overloading. Lambdas in Java 8 are not operator overloading.

**Interview Follow-up:**
- Which operators can be overloaded in C++ and which cannot?
- How does Java achieve `+` for its own types?
- What is `+` used for on strings in Java?

**Short Interview Answer:**
Operator overloading lets you define what `+`, `-` or `==` do for your own class, and C++ supports it while Java does not. In Java we use method names like `add()` and `equals()` to get the same effect.

---

## 58. What is function overloading?

**Answer:**
Function (method) overloading is having **multiple functions with the same name but different parameter lists**, resolved at compile time by the compiler. In Java this works within one class and the return type alone is not enough to distinguish methods. It is also called static polymorphism.

**Example:**
```java
void print(int n)          { }
void print(String s)       { }
void print(int a, int b)   { }
```
All three can be called based on what you pass in.

**Interview Follow-up:**
- Is overloading possible with the same parameter types but different return types?
- Can we overload `main`?
- Is function overloading the same as overriding?

**Short Interview Answer:**
Function overloading means several methods share the same name but differ in parameters, and the compiler picks the right one at compile time. It is also called static polymorphism.

---

## 59. What is constructor chaining?

**Answer:**
Constructor chaining is when one constructor **calls another constructor of the same class** using `this(...)`, or of the parent using `super(...)`. It lets you centralize common initialization instead of repeating it in every constructor. `this(...)` must be the **first statement**, and a constructor can call only one other constructor in that way.

**Example:**
```java
class Employee {
    Employee() { this("Unknown", 0); }                 // same class
    Employee(String name, double sal) { this.name = name; this.sal = sal; }

    Employee(double sal) { super(); this.sal = sal; }  // parent class
}
```

**Common Misconception:**
`super()` is implicitly added by the compiler if you write nothing — so you cannot write `super()` yourself and also let the compiler add it.

**Interview Follow-up:**
- Can `this()` and `super()` be used in the same constructor?
- Why must `this()` be the first statement?
- What is an initializer block and when does it run?

**Short Interview Answer:**
Constructor chaining means calling one constructor from another using `this()` for the same class or `super()` for the parent. It avoids duplicate initialization code, and `this()` must be the first statement.

---

## 60. What is the this pointer?

**Answer:**
`this` in Java is a **reference to the current object**, used to disambiguate fields when a parameter has the same name, or to pass the current object to another method. In C++ it is literally a pointer; in Java it is an implicit reference available inside instance methods. It **cannot** be used inside a `static` method because there is no current object.

**Example:**
```java
class Student {
    String name;
    Student(String name) {
        this.name = name;          // LHS = field, RHS = parameter
    }
}
```

**Common Misconception:**
`this` cannot be used in static methods or static blocks — there is no object context there.

**Interview Follow-up:**
- Can we use `this` in a constructor?
- How do we use `this` inside an anonymous inner class?
- What is the difference between `this` and `this()`?

**Short Interview Answer:**
`this` refers to the current object. I use it to disambiguate when a field and parameter have the same name, and it cannot be used inside static methods.

---

## 61. What is the super keyword?

**Answer:**
`super` is a **reference to the parent class object**. It is used to access a parent's fields and methods when the child has hidden them, and `super()` calls the parent's constructor — it is added implicitly if you don't write it. It cannot be used in a `static` context, and unlike `this`, `super` cannot be re-assigned.

**Example:**
```java
class Employee extends Person {
    Employee() { super("Amit", 22); }      // parent constructor

    void display() {
        super.display();                    // call parent's version
        System.out.println(getClass().getSimpleName());
    }
}
```

**Interview Follow-up:**
- What happens if the parent has no no-arg constructor?
- Can we use `super` more than once in a constructor?
- What is `Interface.super.method()`?

**Short Interview Answer:**
`super` refers to the parent object. I use it to call the parent's constructor and to access parent members that the child has hidden, and it cannot be used inside static methods.

---

## 62. What is the difference between this and super?

**Answer:**
`this` refers to the **current object** and `super` refers to the **parent object**. `this()` calls another constructor of the same class, while `super()` calls a constructor of the parent. `this(...)` must be the first statement, and only one of `this()` / `super()` can appear in a constructor.

| Aspect | `this` | `super` |
|--------|--------|---------|
| Refers to | Current object | Parent object |
| Constructor call | `this()` — same class | `super()` — parent class |
| Use in static | Not allowed | Not allowed |
| Reassignment | Not allowed | Not allowed |
| Position | First statement only | `super()` also first |
| Typical use | Field vs parameter ambiguity | Reusing parent code |

**Interview Follow-up:**
- What happens if the parent class has no no-arg constructor?
- Can we call `this.field` for a static field?
- Can `super` be used in an interface?

**Short Interview Answer:**
`this` points to the current object and `super` points to the parent. `this()` calls another constructor in the same class, and `super()` calls the parent's constructor, and both must be the first statement.

---

## 63. What is an inner class?

**Answer:**
An inner class is a **non-static nested class** defined inside another class. It holds an implicit reference to its enclosing object, so it can directly access the outer class's private fields and methods. It is used when the class is only meaningful inside the outer class, like a helper or a listener.

**Example:**
```java
class University {
    private String name;

    class Printer {                       // inner class
        void print() {
            System.out.println(name);     // direct access to private field
        }
    }
}
```

**Common Misconception:**
An inner class cannot be used without an instance of the outer class — you need `university.new Printer()`.

**Interview Follow-up:**
- What is the difference between inner and static nested classes?
- Can an inner class access outer class static fields?
- Why were non-static inner classes restricted before Java 16?

**Short Interview Answer:**
An inner class is a class defined inside another class without `static`, so it keeps a reference to the outer object and can access its private members directly. Use it when the class is only useful inside the outer class.

---

## 64. What is a nested class?

**Answer:**
A nested class is a class defined inside another class; in Java the term usually refers to a **`static` nested class**. It does **not** hold a reference to the outer object, so it must access outer data through an explicit object reference or via static members only. It behaves like a top-level class apart from its name.

**Example:**
```java
class Order {
    private double total;

    static class Comparator {              // static nested class
        boolean isBig(Order o) { return o.total > 1000; }
    }
}
```
`new Order.Comparator().isBig(order)` — created without any `Order` instance.

**Common Misconception:**
Since Java 16, inner classes can also declare static members, so the old rule "no static in inner class" is outdated.

**Interview Follow-up:**
- Can a static nested class access outer instance fields?
- When should you use a static nested class over an inner class?
- Can a nested class be `private`?

**Short Interview Answer:**
A nested class is a class declared inside another class, usually with `static`. Unlike an inner class, it does not hold a reference to the outer object and is independent of it.

---

## 65. What is an anonymous class?

**Answer:**
An anonymous class is a **one-off unnamed class created inline** where you need to implement an interface or extend an abstract class. It is written at the point of use, so you cannot reuse it elsewhere. Since Java 8, lambdas and method references are preferred for single-method interfaces, since they are shorter and clearer.

**Example:**
```java
Runnable r = new Runnable() {
    @Override public void run() { System.out.println("running"); }
};

// Java 8+ preferred:
Runnable r2 = () -> System.out.println("running");
```

**Interview Follow-up:**
- What is the difference between an anonymous class and a lambda?
- Can an anonymous class have constructors?
- When is an anonymous class still preferred over a lambda?

**Short Interview Answer:**
An anonymous class is an unnamed class created inline to implement an interface, used when you need a one-time implementation. With Java 8+, if the interface has a single method, a lambda is usually a better choice.

---

## 66. What is object cloning?

**Answer:**
Cloning means **creating a copy of an existing object** at runtime without going through the constructor. In Java, `Object.clone()` is `protected`, so a class must implement the **`Cloneable` marker interface** and typically override `clone()` as `public`. Note that `clone()` performs a **shallow copy**, so nested objects are still shared.

**Example:**
```java
class Student implements Cloneable {
    @Override public Object clone() throws CloneNotSupportedException {
        return super.clone();     // shallow copy
    }
}
Student s2 = (Student) s1.clone();
```

**Common Misconception:**
Calling `clone()` without implementing `Cloneable` throws `CloneNotSupportedException` — `Cloneable` is a marker interface with no methods.

**Interview Follow-up:**
- Is `clone()` a deep copy?
- Why is `Cloneable` a marker interface?
- Is cloning preferred over copy constructors?

**Short Interview Answer:**
Cloning creates a copy of an object at runtime using `clone()`, and the class must implement `Cloneable`. `clone()` gives a shallow copy by default, so deep copying needs manual code.

---

## 67. What is the Cloneable interface?

**Answer:**
`Cloneable` is a **marker interface** in `java.io` with no methods, whose only purpose is to **signal the JVM that this class supports `clone()`**. It does not provide the cloning logic — you still must override `clone()` yourself. Without it, `Object.clone()` throws `CloneNotSupportedException`.

**Example:**
```java
class Address implements Cloneable, Serializable { ... }
```

**Common Misconception:**
`Cloneable` does not make cloning work automatically. Many developers consider cloning a **broken design** and prefer copy constructors or serialization-based copying.

**Interview Follow-up:**
- Why was `Cloneable` designed this way?
- Why is cloning considered a bad API?
- What is the alternative to `clone()`?

**Short Interview Answer:**
`Cloneable` is an empty marker interface that only tells the JVM a class supports `clone()`. It has no methods and does not provide the copying logic — you still override `clone()` yourself.

---

## 68. What is serialization?

**Answer:**
Serialization is converting an object into a **byte stream** so it can be saved to a file, stored in a database, or sent over the network. In Java a class must implement the `Serializable` marker interface, and you use `ObjectOutputStream` to write it. Declaring `serialVersionUID` helps with version compatibility, and in most real projects JSON with Jackson is preferred for API payloads.

**Example:**
```java
class User implements Serializable {
    private static final long serialVersionUID = 1L;
    String name;
}

try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("u.ser"))) {
    oos.writeObject(new User());
}
```

**Common Misconception:**
Java's `Serializable` is easy to use but fragile and slow; it can also leak sensitive fields like passwords because all non-transient fields are serialized.

**Interview Follow-up:**
- Why should `password` be marked `transient`?
- What happens if `serialVersionUID` changes?
- How is serialization used in session management?

**Short Interview Answer:**
Serialization converts an object into a byte stream so it can be stored or sent over the network, and the class must implement `Serializable`. In real APIs I usually prefer JSON with Jackson because it is safer and more interoperable.

---

## 69. What is deserialization?

**Answer:**
Deserialization is the reverse — reading a byte stream back into a Java object using `ObjectInputStream`. It is the exact opposite of serialization. The main security concern is that a malicious byte stream can trigger nested object graphs, so untrusted input should be filtered (look-ahead `ObjectInputStream` or Jackson with safe defaults).

**Example:**
```java
try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("u.ser"))) {
    User u = (User) ois.readObject();
}
```

**Common Misconception:**
Deserialization is not risk-free — it is a well-known attack vector when reading untrusted data, so never deserialize data you did not produce yourself.

**Interview Follow-up:**
- What is a deserialization gadget chain?
- How do you prevent insecure deserialization?
- How do you make a field skip serialization?

**Short Interview Answer:**
Deserialization converts a byte stream back into an object, using `ObjectInputStream`. It is risky with untrusted input, so for APIs I use JSON with Jackson instead of native Java serialization.

---

## 70. What is runtime type identification (RTTI)?

**Answer:**
RTTI is the ability to **identify an object's actual type at runtime** and act on it. Java supports it through `instanceof`, `getClass()`, the `Class` object and reflection. Java 16+ improved this with **pattern matching for instanceof**, which removes the need for a separate cast.

**Example:**
```java
if (obj instanceof String s) {       // Java 16+
    System.out.println(s.length());
}
```
Older style: `if (obj instanceof String) { String s = (String) obj; ... }`

**Interview Follow-up:**
- How is RTTI done in C++?
- What is reflection and how does it differ from RTTI?
- Is there a performance cost to using `instanceof`?

**Short Interview Answer:**
RTTI is the ability to check an object's real type at runtime, and Java provides it through `instanceof`, `getClass()` and reflection. With Java 16 pattern matching, `instanceof` also safely binds the variable, avoiding a manual cast.

---

## 71. What is covariant return type?

**Answer:**
Covariant return type means an overridden method may **return a more specific type** than the parent method declared. For example, `Object.clone()` returns `Object`, but a class can override it to return `ArrayList` because `ArrayList` is a subtype of `Object`. This is valid because it still satisfies the parent's contract and returns an object the caller can use as the parent type.

**Example:**
```java
@Override
public ArrayList<String> clone() { ... }   // overriding Object.clone()
```

**Common Misconception:**
Changing only the return type to create an **overload** is not allowed — covariant return type only applies when you are also overriding an existing method.

**Interview Follow-up:**
- Can we do the opposite (return a supertype)?
- When was covariant return type introduced in Java?
- How is it used with generics?

**Short Interview Answer:**
Covariant return type allows an overridden method to return a more specific type than its parent, such as returning `ArrayList` instead of `Object`. It is safe because the returned object can still be used as the parent's return type.

---

## 72. What is dependency injection?

**Answer:**
Dependency injection means a class **does not create its own dependencies** — they are supplied from outside, usually through the constructor. This keeps classes loosely coupled and makes them easy to test with mocks. In Spring Boot, the container does this automatically for `@Autowired` fields, constructors and `@Bean` definitions.

**Example:**
```java
class OrderService {
    private final PaymentGateway gateway;      // depends on abstraction

    OrderService(PaymentGateway gateway) { this.gateway = gateway; }
}
```
In a unit test you pass a mock gateway instead of the real one.

**Common Misconception:**
`new OrderService(new RazorpayClient())` inside the class is **not** DI — the class still creates its own dependency, so it is tightly coupled and hard to test.

**Interview Follow-up:**
- What are the three types of DI in Spring?
- Which type of DI is recommended and why?
- What is the difference between DI and IoC?

**Short Interview Answer:**
Dependency injection means the dependencies of a class are provided from outside instead of the class creating them itself. It reduces coupling and makes unit testing easy with mocks, and Spring does it automatically.

---

## 73. What is tight coupling?

**Answer:**
Tight coupling is when classes **depend strongly on each other's concrete implementation**, so changing one forces changes in others. It usually appears when a class directly creates an object of a specific class with `new`. The result is code that is hard to test, reuse or modify.

**Example:**
```java
class OrderService {
    private MySQLConnection db = new MySQLConnection();   // tight coupling
}
```
If you move to PostgreSQL, `OrderService` must be edited.

**Interview Follow-up:**
- What is the effect of tight coupling on testing?
- How do we break tight coupling?
- Is inheritance a form of tight coupling?

**Short Interview Answer:**
Tight coupling means classes depend directly on each other's specific implementation, usually by creating objects with `new`. It makes code hard to change and test, and it is the opposite of loose coupling.

---

## 74. What is loose coupling?

**Answer:**
Loose coupling means classes depend on **abstractions like interfaces**, not on concrete classes, so implementations can be swapped or changed without touching the caller. It is achieved with interfaces, dependency injection and programming to the abstraction. This is the core of testable, maintainable design.

**Example:**
```java
interface PaymentRepository { void save(Order o); }

class OrderService {
    private final PaymentRepository repo;      // any implementation works
}
```
Swapping a MySQL repository for a Mongo one needs no change in the service.

**Interview Follow-up:**
- How do interfaces reduce coupling?
- What is the difference between tight and loose coupling?
- How does Spring promote loose coupling?

**Short Interview Answer:**
Loose coupling means classes depend on interfaces rather than concrete classes, so the implementation can be changed independently. It is achieved through interfaces and dependency injection, and it makes code testable and maintainable.

---

## 75. What is association?

**Answer:**
Association is a **general relationship between two classes** where one uses or knows the other, but neither class owns the other. It is the weakest form of object relationship and is modelled by holding a reference to the other class in a field. It covers one-to-one, one-to-many and many-to-many relationships.

**Example:**
`class Student { Address address; }` — the Student has an Address, but the Address can exist independently (e.g. for a delivery) and does not die with the student.

**Interview Follow-up:**
- Is association bidirectional or unidirectional?
- How is association represented in a database?
- What is the difference between association and dependency?

**Short Interview Answer:**
Association is a general "uses-a" or "knows-a" relationship between classes where there is no ownership. It is modelled by having a reference of one class inside the other.

---

## 76. What is aggregation?

**Answer:**
Aggregation is a **weak has-a relationship** where one class (the whole) holds a reference to other objects (parts), but the parts **can exist independently** of the whole. It is modelled by passing objects into the whole's constructor or setter. A classic example is a Department having Employees — an employee can leave the department and still exist.

**Example:**
```java
class Department {
    private final List<Employee> employees;    // reference is held, not owned
}
```
Deleting the department does not delete its employees.

**Interview Follow-up:**
- Is aggregation represented by composition or association?
- What is a whole-part relationship?
- How do you represent aggregation in a database?

**Short Interview Answer:**
Aggregation is a weak has-a relationship where the parts can exist independently of the whole, and the whole only holds references to them. For example, a department has employees, but an employee can move to another department.

---

## 77. What is composition?

**Answer:**
Composition is a **strong has-a relationship** where the part **cannot exist without the whole**, because its lifecycle is tightly bound to it. The whole creates the part and is responsible for destroying it. In practice it is still implemented by holding a reference, and Java has no built-in ownership keyword for it, so it is a design decision, not a language feature.

**Example:**
```java
class Car {
    private final Engine engine = new Engine();   // created by Car itself
}
```
The engine is meaningless outside the car, so it is a part, not an independent collaborator.

**Interview Follow-up:**
- Which is preferred, composition or inheritance?
- How does composition help with reusability?
- What is the difference between composition and aggregation in code?

**Short Interview Answer:**
Composition is a strong has-a relationship where the part cannot exist without the whole, and the whole controls the part's lifecycle. For example, a Car owns an Engine that it creates itself.

---

## 78. What is the difference between aggregation and composition?

**Answer:**
The difference is **ownership and lifecycle**. In composition the whole creates and destroys the part, so the part cannot exist alone. In aggregation the whole only holds a reference; the part is created and owned externally and can survive the whole.

| Aspect | Aggregation | Composition |
|--------|-------------|-------------|
| Relationship | Weak has-a | Strong has-a |
| Ownership | Part owned elsewhere | Part owned by whole |
| Part lifetime | Independent | Depends on whole |
| Created by | Outside code | Inside the whole class |
| Example | Department + Employee | Car + Engine |
| Deleting whole | Part survives | Part is destroyed |

**Interview Follow-up:**
- How is this implemented in code?
- Which one should you prefer and why?
- How does it relate to the `final` keyword in Java?

**Short Interview Answer:**
In composition the whole owns the part and the part cannot exist alone, while in aggregation the whole only references a part that exists independently. The code for both is a field reference — the difference is the intent of the lifecycle.

---

## 79. What is the difference between association and aggregation?

**Answer:**
Association is the **broad term** for any relationship where classes know or use each other, and it does not imply containment. Aggregation is a **specific, weaker form** of association where the whole holds a reference to the part but the part is not owned. In short, aggregation is a kind of association; a plain association has no "whole" at all.

| Aspect | Association | Aggregation |
|--------|-------------|-------------|
| Type | General relationship | Weak has-a |
| Ownership | None | Reference only |
| Contains a "whole"? | No | Yes |
| Part can exist alone | Yes | Yes |
| Example | Student uses Address | Department has Employees |

**Interview Follow-up:**
- Is aggregation always an association?
- How do you identify aggregation in a UML diagram?
- What is a dependency relationship?

**Short Interview Answer:**
Association is any relationship between classes where one knows or uses the other, and aggregation is a specific weak has-a case of it where the whole holds a reference but does not own the part.

---

## 80. What is the difference between "is-a" and "has-a" relationships?

**Answer:**
`is-a` means a class **is a specialized form** of another, so it should use inheritance (`extends` / `implements`). `has-a` means a class **contains or uses** another, so it should use composition or aggregation. The rule of thumb is: use inheritance only when the child can fully substitute for the parent, otherwise prefer `has-a`.

**Example:**
`Dog is-a Animal` → `class Dog extends Animal`. `Car has-a Engine` → `class Car { Engine engine; }`.

**Common Misconception:**
Inheritance should not be used only to reuse code. If the relationship is not truly substitutable, inheritance leads to fragile deep hierarchies.

**Interview Follow-up:**
- When is inheritance better than composition?
- How do you decide between composition and aggregation?
- Is "BankAccount has-a Customer" a good design?

**Short Interview Answer:**
`is-a` is a specialization relationship handled by inheritance, while `has-a` is a containment relationship handled by composition. I prefer `has-a` unless the child can truly replace the parent.

---

## 81. What are SOLID principles?

**Answer:**
SOLID is a set of five design principles for writing maintainable object-oriented code. It stands for Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation and Dependency Inversion. Together they encourage classes that are small, loosely coupled and easy to extend without breaking existing code.

| Letter | Principle | One-line meaning |
|--------|-----------|------------------|
| S | Single Responsibility | One class, one reason to change |
| O | Open/Closed | Open for extension, closed for modification |
| L | Liskov Substitution | A subclass must work wherever the parent works |
| I | Interface Segregation | Many small interfaces beat one big interface |
| D | Dependency Inversion | Depend on abstractions, not concrete classes |

**Interview Follow-up:**
- Which one is violated most often in real projects?
- Is SOLID always required?
- How do these principles help in testing?

**Short Interview Answer:**
SOLID is five design principles — Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation and Dependency Inversion. Following them keeps classes small, loosely coupled and easy to extend without breaking old code.

---

## 82. What is the Single Responsibility Principle?

**Answer:**
SRP says a class should have **only one reason to change** — it should do one job well. If a class handles database access, business logic and email sending, then any change in email format forces you to touch the DB class too. Splitting responsibilities makes code easier to test, change and reuse.

**Example:**
Instead of one `UserService` doing everything, split it into `UserService`, `UserRepository`, `EmailService` and `UserValidator`.

**Common Misconception:**
SRP does not mean "one method per class" — it means one clear *responsibility* or reason to change.

**Interview Follow-up:**
- Does SRP force one method per class?
- How do you decide the responsibilities of a class?
- How does SRP help with unit testing?

**Short Interview Answer:**
SRP means a class should have only one reason to change, so it handles one clear responsibility. Splitting database, business and notification logic into separate classes makes the code easier to test and maintain.

---

## 83. What is the Open/Closed Principle?

**Answer:**
OCP means software entities should be **open for extension but closed for modification** — you should be able to add new behaviour by adding new code, without changing existing working code. In practice this means using polymorphism instead of editing `if-else` or `switch` statements every time a new type is added.

**Example:**
A `switch (paymentType)` that calls each new payment method must be edited for every new type. Instead, create `interface Payment` and add `WalletPayment implements Payment` — the existing service code stays untouched.

**Interview Follow-up:**
- How do `if-else` statements violate OCP?
- Which design pattern implements OCP?
- How does OCP interact with the Strategy pattern?

**Short Interview Answer:**
OCP says a class should be open for extension but closed for modification, so new behaviour comes from adding new classes rather than editing old ones. Polymorphism and the Strategy pattern are the practical ways to achieve it.

---

## 84. What is the Liskov Substitution Principle?

**Answer:**
LSP says a subclass must be usable **everywhere the parent is expected**, without breaking the parent's contract or changing its behaviour. So you must not tighten preconditions, loosen postconditions, throw new checked exceptions, or change what existing methods promise. Violating it makes code that works with the parent fail unpredictably with the child.

**Example:**
If the parent `List.add()` allows `null`, a child that throws `NullPointerException` on `null` breaks LSP — code written against `List` would break.

**Common Misconception:**
"It compiles" does not mean LSP is satisfied. Overriding must preserve the parent's behavioural contract, not just the signature.

**Interview Follow-up:**
- How can a child violate LSP?
- Why is `ArrayList`/`LinkedList` a good LSP example?
- How do you detect an LSP violation?

**Short Interview Answer:**
LSP means a subclass must be substitutable for its parent without changing the contract, so any code written against the parent must work correctly with the child. Violating it breaks code that depends only on the parent's type.

---

## 85. What is the Interface Segregation Principle?

**Answer:**
ISP says a client should **never be forced to depend on methods it does not use**, so prefer several small, specific interfaces over one big interface. A fat interface forces every implementer to write methods they don't need, and often dummy `throw new UnsupportedOperationException()` code.

**Example:**
Instead of one `Machine` interface with `print()`, `scan()` and `fax()`, split it into `Printable`, `Scannable` and `Faxable`. A `SimplePrinter` implements only `Printable`.

**Interview Follow-up:**
- What happens if a class implements an interface it does not need?
- How does ISP relate to interface design in APIs?
- What is the "fat interface" problem?

**Short Interview Answer:**
ISP says clients should not be forced to implement methods they do not use, so it is better to have several small interfaces instead of one big interface. It keeps implementers clean and avoids unnecessary dummy methods.

---

## 86. What is the Dependency Inversion Principle?

**Answer:**
DIP says high-level modules should **not depend on low-level modules directly** — both should depend on abstractions, and those abstractions should not depend on details. So a business service should depend on a `PaymentGateway` interface rather than a concrete `StripeClient`. It is the principle behind dependency injection and Spring's IoC container.

**Example:**
```java
class OrderService {
    private final PaymentGateway gateway;      // abstraction, not StripeClient
}
```
Now the service has no idea whether Stripe or PayPal is running behind it.

**Common Misconception:**
DIP is not the same as "use interfaces everywhere" — the point is that the **direction of dependency** should point toward abstractions, away from concrete details.

**Interview Follow-up:**
- How does Spring implement DIP?
- What is the difference between DIP and dependency injection?
- Is an interface mandatory for DIP?

**Short Interview Answer:**
DIP says high-level modules should depend on abstractions, not on concrete low-level classes. So a service depends on a `PaymentGateway` interface rather than a specific payment client, which is what Spring's IoC container supports.

---

## 87. What is the Singleton design pattern?

**Answer:**
Singleton ensures a class has **exactly one instance** and provides a global access point to it, usually through a private constructor plus a static `getInstance()` method. It is useful for things that are expensive to create and stateful, like a connection pool or a configuration cache. The main drawbacks are hidden global state, thread-safety issues and difficulty in unit testing.

**Example:**
```java
class DatabaseConnection {
    private static DatabaseConnection instance;
    private DatabaseConnection() { }                       // private ctor

    public static synchronized DatabaseConnection getInstance() {
        if (instance == null) instance = new DatabaseConnection();
        return instance;
    }
}
```

**Common Misconception:**
Spring's beans are singletons by default, but they are managed by the container, not by a private constructor — so they are testable and overridable, unlike classic Singleton.

**Interview Follow-up:**
- How do you make Singleton thread-safe?
- What are the problems with Singleton?
- Which classes in Java are singletons?

**Short Interview Answer:**
Singleton allows only one instance of a class and provides a global access point through a static method with a private constructor. It saves resources, but hidden global state makes testing difficult, so in Spring we prefer container-managed singleton beans.

---

## 88. What is the Factory design pattern?

**Answer:**
Factory encapsulates **object creation logic** in one place so the client does not use `new` directly. It centralises the "which class do I actually need?" decision, which keeps code flexible and makes it easy to switch implementations. In Spring this is everywhere — `BeanFactory` and `@Component` scanning are factory-based creation.

**Example:**
```java
class ShapeFactory {
    public static Shape create(String type) {
        return switch (type) {
            case "circle"  -> new Circle();
            case "square"  -> new Square();
            default -> throw new IllegalArgumentException("Unknown");
        };
    }
}
```
The client asks the factory for a `Shape` and never names a concrete class.

**Interview Follow-up:**
- What is the difference between Factory Method and Abstract Factory?
- How is the factory pattern used in Spring?
- Why is `new` in business code considered bad?

**Short Interview Answer:**
The Factory pattern creates objects through a dedicated factory instead of using `new` in client code. It hides creation logic, makes implementations swappable, and is heavily used in Spring's container.

---

## 89. What is the Observer design pattern?

**Answer:**
Observer defines a **one-to-many dependency** — when one object changes state, all registered dependents are notified automatically. The subject does not know who its listeners are, only that they exist, which keeps it loosely coupled. It is the basis of event-driven systems, Spring's application events, and Kafka consumers.

**Example:**
A Spring Boot service publishes `OrderPlacedEvent`; notification and analytics components listen via `@EventListener` and react without the service knowing about them.

**Common Misconception:**
Observer is not the same as a callback. A callback is one listener; Observer is a one-to-many notification mechanism where the subject keeps a list of listeners.

**Interview Follow-up:**
- What are the pros and cons of Observer?
- Is it thread-safe?
- How does it differ from Pub/Sub messaging?

**Short Interview Answer:**
Observer is a one-to-many pattern where the subject notifies all registered listeners whenever its state changes, so it does not depend on them. In Spring Boot this is what application events and `@EventListener` are based on.

---

## 90. What is the Strategy design pattern?

**Answer:**
Strategy puts a **family of algorithms into separate classes** and makes them interchangeable at runtime through a common interface. The client chooses which algorithm to use, or receives it injected. This avoids huge `if-else` chains and lets the algorithm vary independently.

**Example:**
```java
interface PricingStrategy { double apply(Order o); }
// CouponPricing, FestivalPricing, FlatDiscountPricing
```
The service holds a `PricingStrategy` and swaps it without touching the ordering logic.

**Interview Follow-up:**
- How is Strategy different from Factory?
- What is the relationship between Strategy and Open/Closed?
- Where have you used Strategy in a real project?

**Short Interview Answer:**
Strategy is a pattern where a family of algorithms implements a common interface, and the client can swap them at runtime. It replaces long if-else chains with interchangeable strategy objects.

---

## 91. What is the Adapter design pattern?

**Answer:**
Adapter converts the interface of an existing class into **another interface the client expects**, so incompatible classes can work together without changing their code. It is very common in real integration work — wrapping a third-party SDK so the rest of the application does not depend on that vendor.

**Example:**
`InputStreamReader` adapts a byte-based `InputStream` into a character-based `Reader`. Similarly, a `RazorpayAdapter implements PaymentGateway` hides Razorpay's own SDK from our services.

**Interview Follow-up:**
- What is the difference between Adapter and Facade?
- Is there a type of Adapter that uses inheritance?
- Where do we use Adapter in Spring Boot?

**Short Interview Answer:**
Adapter makes an existing incompatible class fit another interface the client expects, so the two can work together without modifying either. It is heavily used when integrating third-party SDKs and in the Java IO stream hierarchy.

---

## 92. What is the Decorator design pattern?

**Answer:**
Decorator adds new behaviour to an object by **wrapping it with another object of the same type**, rather than by creating a subclass. Behaviour from multiple decorators can be stacked, so it is a flexible alternative to inheritance. The trade-off is that deep wrapping chains become harder to debug.

**Example:**
```java
InputStream in = new GZIPInputStream(
                  new BufferedInputStream(
                  new FileInputStream("data.txt")));   // compressed + buffered
```
Each layer adds one responsibility without changing the original class.

**Common Misconception:**
Decorator is not inheritance — the decorator holds a reference to the wrapped object and delegates to it, rather than being a subclass.

**Interview Follow-up:**
- Decorator vs inheritance — when do you use which?
- How does Java IO use this pattern?
- What is the drawback of deep decorator chains?

**Short Interview Answer:**
Decorator adds behaviour by wrapping the original object in another object of the same type, so extra responsibilities can be stacked without subclassing. Java IO streams and Spring's `BeanPostProcessor` are classic examples.

---

## 93. What is the Prototype design pattern?

**Answer:**
Prototype creates a new object by **copying an existing object** instead of constructing it from scratch. It is useful when object creation is expensive — a long constructor, database or file loading, heavy memory use. In Java this maps to `clone()` or a copy constructor, and to Spring's prototype-scoped beans.

**Example:**
In Spring, `@Scope("prototype")` returns a **new instance every time** the bean is requested, whereas the default singleton scope reuses the same instance.

**Common Misconception:**
Prototype pattern does not mean "a class prototype in JavaScript". In Java it means cloning an existing instance to avoid expensive construction.

**Interview Follow-up:**
- What is a shallow vs deep prototype copy?
- What is the difference between prototype and singleton scope in Spring?
- When is cloning preferred over a constructor?

**Short Interview Answer:**
Prototype creates new objects by copying an existing instance instead of constructing them again, which is useful when creation is expensive. In Java it is done with `clone()` or a copy constructor, and it matches Spring's prototype bean scope.

---

## 94. What are immutable objects?

**Answer:**
An immutable object has **state that cannot change after construction**. Because it never changes, it is automatically thread-safe and safe to share across threads, which is why `String` is immutable. To make a class immutable, make it `final`, make all fields `private final`, remove setters, and return defensive copies from getters.

**Example:**
```java
final class Money {
    private final BigDecimal amount;         // final, set only in constructor
    Money(BigDecimal amount) { this.amount = amount; }

    public Money add(Money other) {          // returns a NEW object
        return new Money(this.amount.add(other.amount));
    }
}
```

**Common Misconception:**
A class with only final fields is not fully immutable — if it exposes its mutable internals (like a `List`), the "immutable" object can still be changed from outside.

**Interview Follow-up:**
- How do you make a class immutable in Java?
- Why is `String` immutable?
- What are the performance benefits of immutability?

**Short Interview Answer:**
An immutable object cannot change after creation, so it is thread-safe and safe to share. In Java you make the class final, the fields private final, remove setters, and return copies from getters — that is why `String` is immutable.

---

## 95. What is the difference between == and equals()?

**Answer:**
`==` compares **references** — whether two variables point to the same object in memory. `equals()` compares **values/contents**, based on the fields we decide are meaningful. `equals()` is defined in `Object` and only works properly after you **override it**; by default `Object.equals()` just does a reference comparison.

**Example:**
```java
String a = new String("hello");
String b = new String("hello");

System.out.println(a == b);       // false - different objects
System.out.println(a.equals(b));  // true  - same content
```
For strings coming from the DB or user input, always use `.equals()` (or `Objects.equals`), never `==`.

| Aspect | `==` | `equals()` |
|--------|------|-----------|
| Compares | Reference / memory address | Logical value |
| Where defined | Language operator | Method in `Object` |
| Default behaviour | — | Same as `==` until overridden |
| Null safety | NPE if compared to null | Use `Objects.equals` |
| Overridable | No | Yes |
| Use for | Primitives / same reference | Objects and value comparison |

**Interview Follow-up:**
- How do you override `equals()` correctly?
- Why must `hashCode()` be overridden with `equals()`?
- Is `==` valid for primitive types?

**Short Interview Answer:**
`==` compares references, whether two variables point to the same object, while `equals()` compares values and must be overridden to be meaningful. For objects like Strings I always use `equals()`, never `==`.

---

## 96. What is object comparison?

**Answer:**
Comparing objects means deciding whether two instances are equal and, for collections, deciding **which comes first**. Java gives two ways: implement `Comparable<T>` for a natural ordering defined by the class itself, or provide a `Comparator<T>` for ordering defined outside the class. In both cases you must write the comparison logic based on the fields that matter.

**Example:**
```java
class Student implements Comparable<Student> {
    @Override public int compareTo(Student o) { return Integer.compare(rollNo, o.rollNo); }
}

List<Student> list = new ArrayList<>();
Collections.sort(list);                          // uses compareTo
list.sort(Comparator.comparing(Student::getName)); // external ordering
```

| Aspect | `Comparable` | `Comparator` |
|--------|--------------|--------------|
| Defined | Inside the class | Passed as an object |
| Fields used | Class decides | Caller decides |
| Multiple orderings | Only one natural order | Unlimited |
| Type param | `Comparable<T>` | `Comparator<T>` |
| Sorting call | `sort(list)` | `sort(list, cmp)` |

**Common Misconception:**
You cannot have multiple `compareTo` implementations, which is why `Comparator` is the flexible option for sorting by different fields.

**Interview Follow-up:**
- What happens if `compareTo()` returns 0?
- How does `TreeSet` use `Comparator`?
- Is `Comparator` thread-safe?

**Short Interview Answer:**
Object comparison is done with `Comparable` for a natural ordering defined by the class, or with `Comparator` for orderings chosen by the caller. For sorting a list by different fields, `Comparator` is the flexible choice.

---

## 97. What are circular dependencies?

**Answer:**
A circular dependency occurs when **class A depends on class B and B also depends on A**, directly or through a chain. The JVM can load such classes fine, but it creates a construction deadlock — you cannot create either object because each needs the other first. It also makes the code hard to test and hides poor design boundaries.

**Example:**
`A` needs a `B` in its constructor and `B` needs an `A` in its constructor. In Spring this usually shows up as a startup failure like *"BeanCurrentlyInCreationException"* when using constructor injection.

**Interview Follow-up:**
- Why does constructor injection fail with circular dependencies?
- Is field injection free from this problem?
- How do you detect circular dependencies?

**Short Interview Answer:**
A circular dependency is when class A needs class B and B needs class A, forming a loop. It creates a construction deadlock, so the application can fail to start, and it usually points to poor separation of responsibilities.

---

## 98. How can circular dependencies be avoided?

**Answer:**
Circular dependencies are best avoided by **restructuring the design** — extract the shared behaviour into a third class, or move one side behind an interface. If that is not possible, decouple using **event-driven communication** (A publishes, B listens), which removes the direct reference. In Spring, `@Lazy` on one side, `ApplicationEventPublisher`, and setter injection are the practical workarounds, but they hide the problem rather than solving it.

**Example:**
Instead of `OrderService` ↔ `NotificationService` calling each other, `OrderService` publishes an `OrderPlacedEvent` and `NotificationService` listens to it — no direct dependency at all.

**Common Misconception:**
Spring's setter injection and `@Lazy` only postpone the problem; they do not remove the circular design. The real fix is to change the structure or use events.

**Interview Follow-up:**
- How does `@Lazy` break the cycle?
- What is the best architectural fix?
- When are circular dependencies acceptable?

**Short Interview Answer:**
To avoid circular dependencies, restructure the classes by extracting shared logic into a third component or decoupling them with events. `@Lazy` and setter injection are temporary workarounds, not real solutions.

---

## 99. What are best practices for OOP design?

**Answer:**
Good OOP design means **small focused classes that depend on interfaces, favour composition over deep inheritance, and keep state immutable where possible**. Keep methods short and named clearly, avoid global/static mutable state, make fields `private final` and classes `final` when possible, and use dependency injection so classes stay testable. Follow SOLID where it helps, not as a checklist.

**Example:**
A well-designed order service takes a `PaymentGateway` and a `OrderRepository` in its constructor, keeps them `private final`, and its methods are short — you can test it by passing mocks.

**Common Misconception:**
More abstraction layers is not automatically better design. If a class has one method and one implementation, an extra interface may just be noise.

**Interview Follow-up:**
- When is abstraction unnecessary?
- How do you review code for good design?
- How do you balance SOLID with simplicity?

**Short Interview Answer:**
The best practices are small classes with one responsibility, depending on interfaces instead of concrete classes, preferring composition over inheritance, using immutable state, and injecting dependencies so the code is testable. Keep it simple and avoid unnecessary abstraction.

---

## 100. How would you design a real-world system using OOP principles?

**Answer:**
A good answer is to show **layered design with clear responsibilities**: domain classes that model the business, interfaces for anything external, and concrete implementations injected in. For an e-commerce order flow, I would model `Order`, `Payment`, `Customer` as entities, define `OrderRepository` and `PaymentGateway` as interfaces, implement them with JPA and a vendor SDK, and orchestrate everything in an `OrderService` with an `OrderController` on top. New requirements like a new payment provider or a discount rule are then added as new classes implementing existing interfaces, with no change to the service.

**Example:**
```java
@Entity
class Order {
    @Id Long id;
    @Enumerated(EnumType.STRING) OrderStatus status;
    @OneToMany List<LineItem> items;
    // getters, behaviour like place(), cancel()
}

interface PaymentGateway { PaymentResult pay(Order o); }   // abstraction

@Service
class OrderService {                      // orchestration, constructor DI
    private final OrderRepository repo;    // interface
    private final PaymentGateway gateway; // interface
    private final AppEventPublisher events;

    @Transactional
    Order place(Order o) {
        PaymentResult r = gateway.pay(o);
        if (r.success()) { o.place(); repo.save(o); events.publish(new OrderPlacedEvent(o)); }
        return o;
    }
}
```

**Which principles are used here:**
- **Encapsulation** — `status` changes only through `place()` / `cancel()`, never directly.
- **Abstraction** — the service depends on `PaymentGateway`, not on Razorpay's SDK.
- **Polymorphism** — swapping UPI for card is just a different implementation.
- **Composition** — an `Order` has `LineItem`s rather than inheriting behaviour.
- **SRP** — controller, service, repository and gateway each have one job.
- **Open/Closed** — a new payment provider is a new class; no existing code changes.

**Interview Follow-up:**
- How would you handle failure if the payment succeeds but the DB save fails?
- Where would you apply transactions and why?
- How would you test this design?

**Short Interview Answer:**
I would split it into layers: entities that model the business, interfaces for external things like payment and repository, concrete implementations injected in, a service that orchestrates the flow, and a controller for the API. Each class has one responsibility, so a new payment method is just a new class implementing the interface.

---