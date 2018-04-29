# memoryjs
Node add-on for memory reading and writing! (finally!)

# Features

- open a process
- close the process (handle)
- list all open processes
- list all modules associated with a process
- find a certain module associated with a process
- read memory
- write to memory
- pattern scanning

TODO:
- nothing (suggestions welcome)

# Install

This is a Node add-on (last tested to be working on `v6.9.1`) and therefore requires [node-gyp](https://github.com/nodejs/node-gyp) to use.

You may also need to [follow these steps](https://github.com/nodejs/node-gyp#user-content-installation).

`npm install memoryjs`

When using memoryjs, the target process should match the platform architecture of the Node version running.
For example if you want to target a 64 bit process, you should try and use a 64 bit version of Node.

You also need to recompile the library and target the platform you want. Head to the memoryjs node module directory, open up a terminal and to run the compile scripts, type:

`npm run build32` if you want to target 32 bit processes

`npm run build64` if you want to target 64 bit processes

# Usage

For a complete example, view `index.js` and `example.js`.

Initialise:
``` javascript
const memoryjs = require('memoryjs');
const processName = "csgo.exe";
```

### Processes

Open a process (sync):
``` javascript
const processObject = memoryjs.openProcess(processName);
```

Open a process (async):
``` javascript
memoryjs.openProcess(processName, (error, processObject) => {

});
```

Get all processes (sync):
``` javascript
const processes = memoryjs.getProcesses();
```

Get all processes (async):
``` javascript
memoryjs.getProcesses((error, processes) => {

});
```

See the [Documentation](#user-content-documentation) section of this README to see what a process object looks like.

### Modules

Find a module (sync):
``` javascript
const module = memoryjs.findModule(moduleName, processId);
```

Find a module (async):
``` javascript
memoryjs.findModule(moduleName, processId, (error, module) => {

});
```

Get all modules (sync):
``` javascript
const modules = memoryjs.getModules(processId);
```

Get all modules (async):
``` javascript
memoryjs.getModules(processId, (error, modules) => {

});
```

See the [Documentation](#user-content-documentation) section of this README to see what a module object looks like.

### Memory

Read from memory (sync):
``` javascript
const value = memoryjs.readMemory(address, dataType);
```

Read from memory (async):
``` javascript
memoryjs.readMemory(address, dataType, (error, result) => {

});
```

Write to memory:
``` javascript
memoryjs.writeMemory(address, value, dataType);
```

See the [Documentation](#user-content-documentation) section of this README to see what values `dataType` can be.

### Pattern scanning

Pattern scanning (sync):
``` javascript
const offset = memoryjs.findPattern(moduleName, signature, signatureType, patternOffset, addressOffset);
```

Pattern scanning (async):
``` javascript
memoryjs.findPattern(moduleName, signature, signatureType, patternOffset, addressOffset, (error, offset) => {

})
```

# Documentation

### Process object:
``` javascript
{  cntThreads: 47,
   szExeFile: "csgo.exe",
   th32ProcessID: 10316,
   th32ParentProcessID: 7804,
   pcPriClassBase: 8,
   handle: 808,
   modBaseAddr: 1673789440 }
```

The `handle` and `modBaseAddr` properties are only available when opening a process and not when listing processes.

### Module object:
``` javascript
{ modBaseAddr: 468123648,
  modBaseSize: 80302080,
  szExePath: 'c:\\program files (x86)\\steam\\steamapps\\common\\counter-strike global offensive\\csgo\\bin\\client.dll',
  szModule: 'client.dll',
  th32ProcessID: 10316 }
  ```

### Data Type:

When using the write or read functions, the data type (dataType) parameter can either be a string and be one of the following:

`"int", "dword", "long", "float", "double", "bool", "boolean", "ptr", "pointer", "str", "string", "vec3", "vector3", "vec4", "vector4"`

or can reference constants from within the library:

`memoryjs.INT, memoryjs.DWORD, memoryjs.LONG, memoryjs.FLOAT, memoryjs.DOUBLE, memoryjs.BOOL, memoryjs.BOOLEAN, memoryjs.PTR, memoryjs.POINTER, memoryjs.STR, memoryjs.STRING, memoryjs.VEC3, memoryjs.VECTOR3, memoryjs.VEC4, memoryjs.VECTOR4`

This is simply used to denote the type of data being read or written.

Vector3 is a data structure of three floats:

``` javascript
const vector3 = { x: 0.0, y: 0.0, z: 0.0 };
memoryjs.writeMemory(address, vector3);
```

Vector4 is a data structure of four floats:

```javascript
const vector4 = { w: 0.0, x: 0.0, y: 0.0, z: 0.0 };
memoryjs.writeMemory(address, vector4);
```

### Strings:

You can use this library to read either a "string", or "char*" and to write a string.

In both cases you want to get the address of the char array:

```c++
std::string str1 = "hello";
std::cout << "Address: 0x" << hex << (DWORD) str1.c_str() << dec << std::endl;

char* str2 = "hello";
std::cout << "Address: 0x" << hex << (DWORD) str2 << dec << std::endl;
```

From here you can simply use this address to write and read memory.

There is one caveat when reading a string in memory however, due to the fact that the library does not know
how long the string is, it will continue reading until it finds the first null-terminator. To prevent an
infinite loop, it will stop reading if it has not found a null-terminator after 1 million characters.

One way to bypass this limitation in the future would be to allow a parameter to let users set the maximum
character count.

### Signature Type:

When pattern scanning, flags need to be raised for the signature types. The signature type parameter needs to be one of the following:

`0x0` or `memoryjs.NORMAL` which denotes a normal signature.

`0x1` or `memoryjs.READ` which will read the memory at the address.

`0x2` or `memoryjs.SUBSTRACT` which will subtract the image base from the address.

To raise multiple flags, use the bitwise OR operator: `memoryjs.READ | memoryjs.SUBTRACT`.

---

#### openProcess(processName[, callback])

opens a process to be able to read from and write to it

- **processName** *(string)* - the name of the process to open
- **callback** *(function)* - has two parameters:
  - **err** *(string)* - error message (empty if there were no errors)
  - **processObject** *(JSON [process object])* - information about the process

**returns** *process object (JSON)* either directly or via the callback

---

#### closeProcess()

closes the handle on the opened process

---

#### getProcesses([callback])

collects information about all the running processes

- **callback** *(function)* - has two parameters:
  - **err** *(string)* - error message (empty if there were no errors)
  - **processes** *(array)* - array of *process object (JSON)*

**returns** an array of *process object (JSON)* for all the running processes

---

#### findModule(moduleName, processId[, callback])

finds a module associated with a given process

- **moduleName** *(string)* - the name of the module to find
- **processId** *(int)* - the id of the process in which to find the module
- **callback** *(function)* - has two parameters:
  - **err** *(string)* - error message (empty if there were no errors)
  - **module** *(JSON [module object])* - information about the module

**returns** *module object (JSON)* either directly or via the callback

---

#### getModules(processId[, callback])

gets all modules associated with a given process

- **processId** *(int)* - the id of the process in which to find the module
- **callback** *(function)* - has two parameters:
  - **err** *(string)* - error message (empty if there were no errors)
  - **modules** *(array)* - array of *module object (JSON)*

**returns** an array of *module object (JSON)* for all the modules found

---

#### readMemory(address, dataType[, callback])

reads the memory at a given address

- **address** *(int)* - the address in memory to read from
- **dataType** *(string)* - the data type to read into (definitions can be found at the top of this section)
- **callback** *(function)* - has two parameters:
  - **err** *(string)* - error message (empty if there were no errors)
  - **value** *(any data type)* - the value stored at the given address in memory

**returns** the value that has been read from memory

---

#### writeMemory(address, value, dataType[, callback])

writes to an address in memory

- **address** *(int)* - the address in memory to write to
- **value** *(any data type)* - the data type of value must be either `number`, `string` or `boolean` and is the value that will be written to the address in memory
- **dataType** *(string)* the data type of the value (definitions can be found at the top of this section)
- **callback** *(function)* - has one parameter:
  - **err** *(string)* - error message (empty if there were no errors)

---

#### findPattern(moduleName, signature, signatureType, patternOffset, addressOffset[, callback])

pattern scans memory to find an offset

- **moduleName** *(string)* - the name of the module to pattern scan (module.szModule)
- **signature** *(string)* - the actual signature mask (in the form `A9 ? ? ? A3 ?`)
- **signatureType** *(int)* - flags for [signature types](#user-content-signature-type) (definitions can be found at the top of this section)
- **patternOffset** *(int)* - offset will be added to the address (before reading, if `memoryjs.READ` is raised)
- **addressOffset** *(int)* - offset will be added to the address returned
- **callback** *(function)* - has two parameters:
  - **err** *(string)* - error message (empty if there were no errors)
  - **offset** *(int)* - value of the offset found (will return -1 if the module was not found, -2 if the pattern found no address)

**returns** the value of the offset found
