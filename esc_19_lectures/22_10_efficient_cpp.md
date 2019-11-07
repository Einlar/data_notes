A pure pointer does not include critical information, such as ownership.
Also, it is easy to forget to delete a pointer, generating a leak, or even to make a double remove.
This is expecially true in the presence of expections: a throw could bypass an instruction to delete the pointer (e.g. if it is not managed, that is not catched)
Also, a pointer introduces a certain amount of overhead (for allocation, or for memory jumps)

**How to debug memory problems**
Valgrind Memcheck -> detects everything at runtime, but slow:
```sh
$ g++ leak.cpp
$ valgrind ./a.out
```
Address Sanitizer (ASan), specified at compile time, much faster:
$ g++ -fsanitize=address leak.cpp
$ ./a.out

Example:
```
d esc/hands-on/cpp/
g++ -fsanitize=address leak.cpp - o leak -g
./leak // => detected memory leaks!
```
Usage of a pure pointer T*:
- Object not owned
- The link can be rebound
If object is owened, and the link needs to be always valid, do not use a pointer!
Use instead a copy, or a (const) reference (if rebinding is not needed)

```cpp
T& tr = t1; //create reference
tr = t2; //corresponds to t1 = t2, does not re-assign the reference!
```

If rebinding is needed, use an object: `std::array`, `std::vector`, `std::string`, smart pointers...

**Smart pointers**
Objects like pointers, but that manage the lifetime of the pointee
RAII idiom (Resource Acquisition Is Initialization) - a resource is acquired in the constructor, and then released in the destructor

```cpp
template<typename Pointee>
class SmartPointer {
    Pointee* m_p; //pointer
  public:
    explicit SmartPointer(Pointee* p): m_p{p} {}
    ~SmartPoionter() { delete m_p; }

    //can define (overload) -> operator -> and *
    Pointee* operator->() { return m_p; }
    Pointee& operator*() { return *m_p; }
};


class Histo { ... };

{
    SmartPointer<Histo> sp{new Histo{}};
} //at the end of the scope, the pointer is automatically destroyed (as with the pointee)
```

There are two types of smart pointers in the STL:
- The standard pointer (exclusive ownership, no overhead, non-copyable): `std::unique_pointer`.
```cpp
class Histo { ... };

void take(std::unique_ptr<Histo> ph);

std::unique_ptr<Histo> ph{new Histo{}};
auto ph = std::make_unique<Histo>();
take(ph); //error - pointer is passed by value, but this kind of smart pointer cannot be copied (it is unique!)

//instead do:
take(std::move(ph)); //"move" the smart pointer to the new function
```
- Shared pointers: a counter of number of owners is kept, and after the last owner deletes the pointer, the pointer is deleted for good. Of course, this introduces some overhead, but allows copies to be made

```cpp
class Histo { ... };
void take(std::shared_ptr<Histo> px);

std::shared_ptr<Histo> ph{new Histo{}};
auto px = std::make_shared<Histo>();
take(px); //allowed

take(std::move(px)); //also allowed
```

Pass smart pointers to function only if necessary. As before, `unique_ptr` needs to be moved, while `shared_ptr` can be copied.

# Move Semantics
Suppose we want to use a temporary string to create a new object, like in the following:
```cpp
String get_string() { return "Hi!"; }
String s{ get_string() };
```
The problem is that `get_string()` generates a temporary object which is then _copied_ to the new string, and then deleted. This operation is redundant and unnecessary: it would be better to simply assign a pointer to temporary object, and move it to the new (definitive) object.

This is done by implementing a `move constructor` for the object, that takes as argument a `r-value` (i.e. a temporary object):
```cpp
class String {
    //copy constructor
    String(String const& other) { ... }

    //move constructor
    String(String&& tmp) { ... }
}
```
Analogous constructors are available for assignment (copy vs move). The main rule is to implement them all, or no one (leave them to the compiler).