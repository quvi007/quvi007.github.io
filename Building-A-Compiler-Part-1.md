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
#include <bits/stdc++.h>
using namespace std;

class SymbolInfo {
private:
    string name, type;
    SymbolInfo *next;
    int size, offset;
    bool defined, global;
    vector<Parameter> parameterList;
};
```