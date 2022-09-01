# Building a Minimalist C Compiler in C/C++ - Part 1 (Symbol Table)
The purpose of this tutorial series is to construct a simple C compiler. We will build the Compiler using C++.

In the first step to do so, we are going to implement a Symbol Table. A Symbol Table is a data structure maintained by the compilers in order to store information about the occurence of various entities such as identifiers, objects, function names etc. Information of different entities may include type, value, scope etc. At the starting phase of constructing a compiler, we will construct a Symbol Table which maintains a list of Hash Tables (Also known as Scope Tables) where each Hash Table contains information of symbols encountered in a scope of the source program.

We will implement the following three classes.
1. `SymbolInfo`
2. `Parameter`
3. `ScopeTable`
4. `SymbolTable`

### SymbolInfo
This class contains the information regarding a symbol faced in the source program. It has the following members:

1. `name` - Name of the Symbol. This is a `string`.
2. `type` - Type of the Symbol. This is also a `string`.
3. `next` - This is a pointer to the next `SymbolInfo` object as we need to implement a chaining mechanism to resolve collisions in the hash table. This is of type `SymbolInfo *`.
4. `size` - Size of the symbol. If the symbol is a variable, its size will be `0`. If it's an array, its size will be the number of elements in the array. Lastly, if it's a function, its size will be `-1`, for our convenience. This is an `int`.
5. `offset` - Each variable will have a local offset (index) with respect to its scope. This is also an `int`.
6. `defined` - Indicates the state of a function. For a function name, its name is inserted into the Symbol Table when it is declared. `defined` is set to `false` initially. But, after the function has been defined, we set it to `true`. This is a `bool`.
7. `global` - Indicates whether a variable / array is global or not. This is also a `bool`.
8. `parameterList` - For function names, we need to store the parameters (names and types) as well. We do that using this vector. This is of type `vector<Parameter>` (We will define the class `Parameter` shortly).

We create two files for our `SymbolInfo` class: `SymbolInfo.h` and `SymbolInfo.cpp`. Initially `SymbolInfo.h` will look like this->

```cpp
#ifndef SYMBOLTABLE_SYMBOLINFO_H
#define SYMBOLTABLE_SYMBOLINFO_H

#include <bits/stdc++.h>
#include "Parameter.h"
using namespace std;

class SymbolInfo {
private:
    string name, type;
    SymbolInfo *next;
    int size, offset;
    bool defined, global;
    vector<Parameter> parameterList;
};

#endif //SYMBOLTABLE_SYMBOLINFO_H
```

After that, we add the following constructors to our class.

```cpp
SymbolInfo(); // Empty constructor
SymbolInfo(const string &name, const string &type); // Constructor for Variable
SymbolInfo(const string &name, const string &type, int size); // Constructor for Array
SymbolInfo(const string &name, const string &type, const vector<Parameter> &parameterList); // Constructor for Function
SymbolInfo(const SymbolInfo &symbolInfo); // Copy Constructor 
```

We also add the following destructor to our class.

```cpp
~SymbolInfo(); // Destructor
```

Now, `SymbolInfo.h` will look like this:

```cpp
#ifndef SYMBOLTABLE_SYMBOLINFO_H
#define SYMBOLTABLE_SYMBOLINFO_H

#include <bits/stdc++.h>
#include "Parameter.h"
using namespace std;

class SymbolInfo {
private:
    string name, type;
    SymbolInfo *next;
    int size, offset;
    bool defined, global;
    vector<Parameter> parameterList;

public:
    SymbolInfo(); // Empty constructor
    SymbolInfo(const string &name, const string &type); // Constructor for Variable
    SymbolInfo(const string &name, const string &type, int size); // Constructor for Array
    SymbolInfo(const string &name, const string &type, const vector<Parameter> &parameterList); // Constructor for Function
    SymbolInfo(const SymbolInfo &symbolInfo); // Copy Constructor 

    ~SymbolInfo(); // Destructor
};

#endif //SYMBOLTABLE_SYMBOLINFO_H
```

We also have to add getters/setters and other necessary related methods for each of the private members.

```cpp
// name
const string &getName() const;
void setName(const string &name);

// type
const string &getType() const;
void setType(const string &type);

// next
SymbolInfo *getNext() const;
void setNext(SymbolInfo *next);

// size
int getSize() const;
void setSize(int size);

// offset
int getOffset() const;
void setOffset(int offset);

// defined
bool isDefined() const;
void setDefined(bool defined);

// global
bool isGlobal() const;
void setGlobal(bool global); 

// parameterList
const vector<Parameter> getParameterList() const;
void setParameterList(const vector<Parameter> &parameterList);

bool isVariable() const;
bool isArray() const;
bool isFunction() const;
```

Finally `SymbolInfo.h` will look like this:

```cpp
#ifndef SYMBOLTABLE_SYMBOLINFO_H
#define SYMBOLTABLE_SYMBOLINFO_H

#include <bits/stdc++.h>
#include "Parameter.h"
using namespace std;

class SymbolInfo {
private:
    string name, type;
    SymbolInfo *next;
    int size, offset;
    bool defined, global;
    vector<Parameter> parameterList;

public:
    SymbolInfo(); // Empty constructor
    SymbolInfo(const string &name, const string &type); // Constructor for Variable
    SymbolInfo(const string &name, const string &type, int size); // Constructor for Array
    SymbolInfo(const string &name, const string &type, const vector<Parameter> &parameterList); // Constructor for Function
    SymbolInfo(const SymbolInfo &symbolInfo); // Copy Constructor 

    ~SymbolInfo(); // Destructor

    // name
    const string &getName() const;
    void setName(const string &name);

    // type
    const string &getType() const;
    void setType(const string &type);

    // next
    SymbolInfo *getNext() const;
    void setNext(SymbolInfo *next);

    // size
    int getSize() const;
    void setSize(int size);

    // offset
    int getOffset() const;
    void setOffset(int offset);

    // defined
    bool isDefined() const;
    void setDefined(bool defined);

    // global
    bool isGlobal() const;
    void setGlobal(bool global); 

    // parameterList
    const vector<Parameter> getParameterList() const;
    void setParameterList(const vector<Parameter> &parameterList);

    bool isVariable() const;
    bool isArray() const;
    bool isFunction() const;
};

#endif //SYMBOLTABLE_SYMBOLINFO_H
```

Now we move on to implement our constructors, destructor and other methods. We add the following codes to `SymbolInfo.cpp`.

### Constructors' Implementations
```cpp
#include "SymbolInfo.h"

SymbolInfo::SymbolInfo() : SymbolInfo("", "") {}

SymbolInfo::SymbolInfo(const string &name, const string &type) : SymbolInfo(name, type, 0) {}

SymbolInfo::SymbolInfo(const string &name, const string &type, int size) {
    this->name = name;
    this->type = type;
    next = nullptr;
    this->size = size;
    defined = false;
    global = false;
}

SymbolInfo::SymbolInfo(const string &name, const string &type, const vector<Parameter> &parameterList) : SymbolInfo(name, type, -1) {
    this->parameterList = parameterList;
}

SymbolInfo::SymbolInfo(const SymbolInfo &symbolInfo) {
    name = symbolInfo.name;
    type = symbolInfo.type;
    next = symbolInfo.next;
    size = symbolInfo.size;
    offset = symbolInfo.offset;
    defined = symbolInfo.defined;
    global = symbolInfo.global;
    parameterList = symbolInfo.parameterList;
}
```

### Destructor Implementation

```cpp
SymbolInfo::~SymbolInfo() {
    // We do nothing :(
}
```

### Methods' Implementations

```cpp
const string &SymbolInfo::getName() const {
    return name;
}

void SymbolInfo::setName(const string &name) {
    this->name = name;
}

const string &SymbolInfo::getType() const {
    return type;
}

void SymbolInfo::setType(const string &type) {
    this->type = type;
}

SymbolInfo *SymbolInfo::getNext() const {
    return next;
}

void SymbolInfo::setNext(SymbolInfo* next) {
    this->next = next;
}

int SymbolInfo::getSize() const {
    return size;
}

void SymbolInfo::setSize(int size) {
    this->size = size;
}

int SymbolInfo::getOffset() const {
    return offset;
}

void SymbolInfo::setOffset(int offset) {
    this->offset = offset;
}

bool SymbolInfo::isDefined() const {
    return defined;
}

void SymbolInfo::setDefined(bool defined) {
    this->defined = defined;
}

bool SymbolInfo::isGlobal() const {
    return global;
}

void SymbolInfo::setGlobal(bool global) {
    this->global = global;
}

const vector<Parameter> SymbolInfo::getParameterList() const {
    return parameterList;
}

void SymbolInfo::setParameterList(const vector<Parameter> &parameterList) {
    this->parameterList = parameterList;
}

bool SymbolInfo::isVariable() const {
    return size == 0;
}

bool SymbolInfo::isArray() const {
    return size > 0;
}

bool SymbolInfo::isFunction() const {
    return size == -1;
}
```

Finally, `SymbolInfo.cpp` will look like this:

```cpp
#include "SymbolInfo.h"

SymbolInfo::SymbolInfo() : SymbolInfo("", "") {}

SymbolInfo::SymbolInfo(const string &name, const string &type) : SymbolInfo(name, type, 0) {}

SymbolInfo::SymbolInfo(const string &name, const string &type, int size) {
    this->name = name;
    this->type = type;
    next = nullptr;
    this->size = size;
    defined = false;
    global = false;
}

SymbolInfo::SymbolInfo(const string &name, const string &type, const vector<Parameter> &parameterList) : SymbolInfo(name, type, -1) {
    this->parameterList = parameterList;
}

SymbolInfo::SymbolInfo(const SymbolInfo &symbolInfo) {
    name = symbolInfo.name;
    type = symbolInfo.type;
    next = symbolInfo.next;
    size = symbolInfo.size;
    offset = symbolInfo.offset;
    defined = symbolInfo.defined;
    global = symbolInfo.global;
    parameterList = symbolInfo.parameterList;
}

SymbolInfo::~SymbolInfo() {
    // We do nothing :(
}

const string &SymbolInfo::getName() const {
    return name;
}

void SymbolInfo::setName(const string &name) {
    this->name = name;
}

const string &SymbolInfo::getType() const {
    return type;
}

void SymbolInfo::setType(const string &type) {
    this->type = type;
}

SymbolInfo *SymbolInfo::getNext() const {
    return next;
}

void SymbolInfo::setNext(SymbolInfo* next) {
    this->next = next;
}

int SymbolInfo::getSize() const {
    return size;
}

void SymbolInfo::setSize(int size) {
    this->size = size;
}

int SymbolInfo::getOffset() const {
    return offset;
}

void SymbolInfo::setOffset(int offset) {
    this->offset = offset;
}

bool SymbolInfo::isDefined() const {
    return defined;
}

void SymbolInfo::setDefined(bool defined) {
    this->defined = defined;
}

bool SymbolInfo::isGlobal() const {
    return global;
}

void SymbolInfo::setGlobal(bool global) {
    this->global = global;
}

const vector<Parameter> SymbolInfo::getParameterList() const {
    return parameterList;
}

void SymbolInfo::setParameterList(const vector<Parameter> &parameterList) {
    this->parameterList = parameterList;
}

bool SymbolInfo::isVariable() const {
    return size == 0;
}

bool SymbolInfo::isArray() const {
    return size > 0;
}

bool SymbolInfo::isFunction() const {
    return size == -1;
}
```