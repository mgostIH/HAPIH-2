# HAPIH-2
API for supporting C++14 external memory hacking. Complete redesign from HAPIH.

## Elements
HAPIH 2 uses 3 objects in order to do its job:
1. HandleIH
2. PointerIH
3. HackIH

### HandleIH
Used to store and close either generic or OpenProcess handles upon object destruction, but can't be copied.
If initialized from a DWORD, it will open the process with enough privileges to fully support HAPIH 2.
Usage example is inside source code, the object basically works like a normal HANDLE.


### PointerIH
Defines the internal structure of a pointer. It can either be initialized by a `void*` or by a numeric type ( `size_t` ).

Using Cheat Engine pointer notation, the PointerIH object will point to an address following this scheme:
`[[[[[BaseAddr + Offs1] + Offs2] + Offs3] + Offs4] + ...]`

- Offsets can be added by the `<<` operator, following the offset or by initializing the pointer directly with it. They can be accessed by the [] operator.
```cpp
#include "HAPIH.h"
int main() {
	PointerIH Pointer1 = { 0xDEADBEEF,1,2,3,4 };	//Initialization
	PointerIH Pointer2 = 0xDEADBEEF;
	Pointer2 << 1 << 2 << 3 << 4;         //Adding offsets after initialization
	Pointer1[0] == Pointer2[0];           //Will access the first offsets of both pointers, the result is 1.
}
```

- PointerIH supports final addition and substraction using the operators `+, -, +=, -=`, the final offset will be added **AFTER** the pointer is read from the process.
### HackIH
Handles the entire API, it's responsible to do the direct memory operations, search for open processes and binding to them.

- Once initialized it will store internally all the running processes, thru which you can index them easily with binding or vector manipulation.
```cpp
#include "HAPIH.h"
int main() {
  HackIH MyObj;
  MyObj.WriteProcesses(std::cout);
  std::cin.get();
}
```

```cpp
#include "HAPIH.h"
int main() {
	HackIH MyObj;
	for (auto & proc : MyObj.GetProcesses()) {
		std::cout << std::get<1>(proc) << std::endl;	//Iterates thru every process, only by its name.
		//Use std::get<0>(proc) to get the corresponding process ID
	}
}
```

- HackIH can also enable logging features, using any stream object you desire (Files, STDOUT or any custom wrapper).
```cpp
#include "HAPIH.h"
int main() {
  HackIH MyObj;
  MyObj.SetDebugOutput(std::cout);  //From now on, the operations happening inside HAPIH 2 will write what's happening on STDOUT
  //Code...
}
```

- You can then use the `.bind` function to bind to a process, using it's process name or process ID.
```cpp
#include "HAPIH.h"
int main() {
  HackIH MyObj;
  MyObj.SetDebugOutput(std::cout);  
  MyObj.bind("GeometryDash.exe");   //Binding to the Geometry Dash process.
  //Code...
}
```

```cpp
#include "HAPIH.h"
int main() {
  HackIH MyObj;
  MyObj.SetDebugOutput(std::cout);  
  MyObj.bind(GetCurrentProcessId());   //Binding to itself.
  //Code...
}
```

- After binding, the HackIH object is guaranteed access to the process and can now handle modules, memory spaces and operations, such as: 

1. Reading
    
```cpp
#include "HAPIH.h"
int main() {
	HackIH MyObj;
	MyObj.SetDebugOutput(std::cout);
	MyObj.bind("GeometryDash.exe");		//Binding to the Geometry Dash process.
	int IconID;			//Icon to read from memory
	IconID = MyObj.Read<int>({ MyObj.BaseAddress , 0x303118 , 0x1E8 });
        //IconID = MyObj.Read<int>({ MyObj.GetModuleAddress("GeometryDash.exe") , 0x303118 , 0x1E8 }); //Alternative
        //.GetModuleAddress can be used on every process' module (DLLs)
	std::cout << IconID << std::endl;
	std::cin.get();
}
```
2. Writing 
    
```cpp
#include "HAPIH.h"
int main() {
	HackIH MyObj;
	MyObj.SetDebugOutput(std::cout);
	MyObj.bind("GeometryDash.exe");		//Binding to the Geometry Dash process.
	int IconID= 33;						//Icon to write to memory
	PointerIH IconPtr = { MyObj.BaseAddress , 0x303118 , 0x1E8 };
	MyObj.Write(IconPtr, IconID);
	MyObj.Write(IconPtr - 8, MyObj.Read<int>(IconPtr - 4) + IconID);	
	std::cin.get();
}
```


## Other Features
HAPIH 2 is able to perform DLL injection using the `.DllInject` and `.DllEject` functions, which work by spawning a thread on the needed kernel32.dll function, thus this feature only work for a target with the same bits as the compiled executable.

Memory allocation, thread spawning and chunk memory I/O is also made available (`.ReadBytes` and `.WriteBytes`), a function for computing an hash of data inside a vector is also made available (`DJBHash`)

## Todo
Adding a binary searcher for the executable and some other memory manipulation functions.
Also completing this readme since it's trash.
