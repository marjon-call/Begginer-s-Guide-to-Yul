# Begginers-Guide-to-Yul

## What is Yul?
Yul is an intermediate programming language that can be used to write a form of assembly language inside smart contracts. Although we often times see Yul used inside smart contracts, you can write a smart contract entirely in Yul. Understanding Yul can level up your smart contracts and allow you to understand what is going on under the hood in solidity, which in turn can help you save user's gas costs. We can identify Yul in a smart contract with the following syntax. <br>

``` 
  assembly {
    // do stuff
  }
```
<br>
In the remainder of this article we will discuss the basics of using Yul through examples, I encourage you to follow along in remix.


## Variable Assignments, Operations, & Evaluations
The first topic we need to cover is simple operations. Yul has +, -, *, /, %, **, <, >, and =. Notice that >= and <= are not included, Yul does not have those operations. Additonally, instead of evaluations equaling ture or false, they equal 1 or 0 respectivley. With that said, let's get started with learning some Yul!

<br>

|   Instruction   |  Explanation   |
|  :---   | :--- |
|   let   |   This is required before defining a variable. Since all values are bytes, there is no need to assign a value type.   |
|   :=    |  Solidity equivalent: x = y  |
|   add(x,y)    |  Solidity equivalent: x + y |
|   sub(x,y)    |  Solidity equivalent: x - y |
|   mul(x,y)    |  Solidity equivalent: x * y |
|   div(x,y)    |  Solidity equivalent: x / y (or 0 if y equals 0)|
|   mod(x,y)    |  Solidity equivalent: x % y (or 0 if y equals 0)|
|   lt(x,y)    |  Solidity equivalent: x < y |
|   gt(x,y)    |  Solidity equivalent:  x > y |
|   eq(x,y)    |  Solidity equivalent:  x == y |
|   iszero(x)    |  Solidity equivalent:  x == 0 |

Let's take a quick look at an example before moving on.
<br>

```
function addOneAnTwo() external pure returns(uint256) {
    // We can access variables from solidity inside our Yul code
    uint256 ans;

    assembly {

        // assigns variables in Yul
        let one := 1
        let two := 2

        // adds the two variables together
        ans := add(one, two)

    }

    return ans;
}
```
<br>

## For Loops & If Statements
To learn about both let's write a function that counts how many numbers are even in a series.

```
function howManyEvens(uint256 startNum, uint256 endNum) external pure returns(uint256) {
 
    // the value we will return
    uint256 ans;
 
    assembly {
 
        // syntax for for loop
        for { let i := startNum } lt( i, add(endNum, 1)  ) { i := add(i,1) }
        {
            // if i == 0 skip this iteration
            if iszero(i) {
                continue
            }
 
            // checks if i % 2 == 0
            // we could of used iszero, but I wanted to show you eq()
            if  eq( mod( i, 2 ), 0 ) {
                ans := add(ans, 1)
            }
 
        }
 
    }
 
 
    return ans;
 
}
```

The syntax for ``if`` statements is very similar to solidity, however, we do not need to wrap the condition in parentheses. For the ```for``` loop, notice we are using brackets when declaring ```i``` and incrementing ```i```, but not when we evaluate the condition. Additionally, we used a ```continue``` to skip an iteration of the loop. We can also use ```break``` statements in Yul, as well.



## Storage
Before we can dive deeper into how Yul works, we need a good understanding of how storage works in smart contracts. Storage is composed of a series of slots. There are 2^256 slots for a smart contract. When declaring variables, we start with slot 0 and increment from there. Each slot is 256 bits long (32 bytes), that’s where ```uint256``` and ```bytes32``` get their names from. All variables are converted to hexadecimal. If a variable, such as a ```uint128```, is used we do not take an entire slot to store that variable. Instead it is padded with 0’s on the left side. Let’s look at an example to get a better understanding. <br>

```
// slot 0
uint256 var1 = 256;

// slot 1
address var2 = 0x9ACc1d6Aa9b846083E8a497A661853aaE07F0F00;

// slot 2
bytes32 var3 = 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff;

// slot 3
uint128 var4 = 1;
uint128 var5 = 2;
```
<br>

```var1``` : Since ```uint256``` variables are equal to 32 bytes, ```var1``` takes up the entirety of slot 0. Here is what is being stored in slot 0: ```0x0000000000000000000000000000000000000000000000000000000000000100```. 
<br>

```var2```: Addresses are slightly more complex. Since they only take up 20 bytes of storage, addresses get padded with 0s on the left side. Here is what is being stored in slot 1: ```0x0000000000000000000000009acc1d6aa9b846083e8a497a661853aae07f0f00```. 

<br>

```var3```: This one may seem simple, slot 2 is consumed by the entirety of the ```bytes32``` variable. 

<br>

```var4 & var5```: Remember when I mentioned that ```uint128```’s get padded with 0’s? Well if we order our variables so that the sum of their storage is under 32 bytes, we can fit them into a slot together! This is called packing variables, and it can save you on gas. Let's look at what’s stored in slot 3: ```0x0000000000000000000000000000000200000000000000000000000000000001```. Notice that ```0x000000000000000000000000000002``` and ```0x000000000000000000000000000001``` fit perfectly together in the same slot. That is because they both take up 16 bytes (half of a slot). 

<br>

Now it's time to learn some more Yul!

<br>

|   Instruction   |  Explanation   |
|  :---   | :--- |
| sload(p) |  Loads the variable in slot p from storage.  |
| sstore(p,v) |  Assigns storage slot p value v.  |
| v.slot |  Returns the storage slot of variable v.   |
| v.offset |  Returns the index in bytes of where variable v begins in a storage slot. Variables are packed from right to left. |

<br>

Let's look at another example!

<br>

```
function readAndWriteToStorage() external returns (uint256, uint256, uint256) {

      uint256 x;
      uint256 y;
      uint256 z;

      assembly  {
      
          // gets slot of var5
          let slot := var5.slot
          
          // gets offset of var5
          let offset := var5.offset
          
          // assigns x and y from solidity to slot and offset
          x := slot
          y := offset

          // stores value 1 in slot 0
          sstore(0,1)
          
          // assigns z to the value from slot 0
          z := sload(0)

      }

      return (x, y, z);
}
```

<br>

```x``` = 3. This makes sense since we know that var5 is packed into slot 3. <br>
```y``` = 16. This should also make sense since we know that ```var4``` takes up half of slot 3. Since variables are packed from right to left we get byte 16 as the start index of ```var5```. <br>
```z``` = 1. The ```sstore()``` is assigning slot 0 the value 1. Then, we assign z the value of slot 0 with the ```sload()```.

<br>

Before we move on, you should add this function to your remix file. It will help you see what is being stored at each storage slot. 

<br>

```
// input is the storage slot that we want to read
function getValInHex(uint256 y) external view returns (bytes32) {

  // since Yul works with hex we want to return in bytes
  bytes32 x;
  
  assembly  {
    // assign value of slot y to x
    x := sload(y)
  }
 
 return x;
 
}
```

<br>

Now let's look at some more complex data structures!

```
// slot 4 & 5
uint128[4] var6 = [0,1,2,3];
```
When working with static arrays the EVM knows how many slots to allocate for our data. With this array in particular, we are packing 2 elements per slot. So if you call ```getValInHex(4)``` it will return ```0x0000000000000000000000000000000100000000000000000000000000000000```. As we should expect, reading right to left, we see value 0 and value 1. Slot 5 contains ```0x0000000000000000000000000000000300000000000000000000000000000002```.

<br>

Next we're going to look at dynamic arrays. <br>

```
// slot 6
uint256[] var7;
```
Try calling ```getValInHex(6)```. You will see that it returns ```0x00```. Since the EVM does not know how many storage slots need to be allocated we can not store the array here. Instead, the keccak256 hash of the current storage slot (slot 6) is used as the start index of the array. From here, all we need to do is add the index of the desired element to retrieve the value. 

<br>

Here is a code example demonstrating how to find an element of a dynamic array.
<br>

```
function getValFromDynamicArray(uint256 targetIndex) external view returns (uint256) {
 
    // get the slot of the dynamic array
    uint256 slot;
 
    assembly {
        slot := var7.slot
    }
 
    // get hash of slot for start index
    bytes32 startIndex = keccak256(abi.encode(slot));
 
    uint256 ans;
 
    assembly {
        // adds start index and target index to get storage location. Then loads corresponding storage slot
        ans := sload( add(startIndex, targetIndex) )
    }
 
    return ans;
}

```
Here we retrieve the slot of the array, then perform an ```add()``` operation along with a ```sload()``` to get our desired array element’s value.

You may be asking what prevents us from having a collision with another variable’s slot? This is entirely possible, however, extremely unlikely since 2^256 is a very large number.


<br>
Mappings behave similarly to dynamic arrays except that we hash the slot along with the key.

```
// slot 7
mapping(uint256 => uint256) var8;
```

For this demonstration I set the mapping value ```var8[1] = 2```.
Now let’s look at an example of how to get the value of a key for a mapping.
<br>

```
function getMappedValue(uint256 key) external view returns(uint256) {
 
    // get the slot of the mapping
    uint256 slot;
 
    assembly {
        slot := var8.slot
    }
 
    // hashs the key and uint256 value of slot
    bytes32 location = keccak256(abi.encode(key, slot));
 
 
    uint256 ans;
 
    // loads storage slot of location and returns ans
    assembly {
        ans := sload(location)
    }
 
 
    return ans;
 
}
```

As you can see, the code looks very similar to when we found an element from a dynamic array. The main difference is that we hash the key and slot together. 

<br>

The final part to our section on storage is learning about nested mappings. Before reading on, I encourage you to write your own implementation of how to read a nested map value based off of what you have learned so far.
<br>

```
// slot 8
mapping(uint256 => mapping(uint256 => uint256)) var9;
```
For this example I set the mapping value ```var9[0][1] = 2```.
Here is the code, let's dive in!
<br>

```
function getMappedValue(uint256 key1, uint256 key2) external view returns(uint256) {

    // get the slot of the mapping
    uint256 slot;

    assembly {
        slot := var9.slot
    }

    // hashs the key and uint256 value of slot
    bytes32 locationOfParentValue = keccak256(abi.encode(key1, slot));

    // hashs the parent key with the nested key
    bytes32 locationOfNestedValue = keccak256(abi.encode(key2, locationOfParentValue));



    uint256 ans;

    // loads storage slot of location and returns ans
    assembly {
        ans := sload(locationOfNestedValue)
    }


    return ans;

}
```


Recall that we are reading from right to left when packing varibales. Nested mappings are no differnt. We first get the hash of the first key (0). Then we take the hash of that with the second key (1). Finally, we load the slot from storage to get our value.
<br>
Congradulations, you completed the section on storage with Yul!

<br>

## Reading & Writing Packed Variables

Suppose you want to change ```var5``` to 4. We know that ```var5``` is located in slot 3, so you might try something like this:

```
function writeVar5(uint256 newVal) external {
 
    assembly {
        sstore(3, newVal)
    }
 
}


``` 

Using ```getValInHex(3)``` we see that slot 3 has been rewritten to ```0x0000000000000000000000000000000000000000000000000000000000000004```. That’s a problem because now ```var4``` has been rewritten to 0. In this section we are going to go over how to read and write packed variables, but first we need to learn a little more about Yul syntax.


<br>

|   Instruction   |  Explanation   |
|  :---   | :--- |
| and(x, y)  |  bitwise “and” of x and y |
| or(x, y) |  bitwise “or” of x and y |
| xor(x,  y) |  bitwise “xor” of x and y  |
| shl(x, y)  |  a logical shift left of y by x bits |
| shr(x, y)  |  a logical shift right of y by x bits|

<br>

If you're unfamiliar with these operations don’t worry, we are about to go over them with examples.

Let's start with ```and()```. We are going to take two ```bytes32``` and try the ```and()``` opperator and see what it returns.
<br>

```
function getAnd() external pure returns (bytes32) {

    bytes32 randVar = 0x0000000000000000000000009acc1d6aa9b846083e8a497a661853aae07f0f00;
    bytes32 mask = 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff;

    bytes32 ans;

    assembly {

        ans := and(mask, randVar)

    }

    return ans;

}
```

If you look at the output we see ```0x0000000000000000000000009acc1d6aa9b846083e8a497a661853aae07f0f00```. The reason for this is because what the ```and()``` does is it looks at each bit from both inputs and compares their values. If both bits are a 1 (not literally a 1, think of it in terms of binary: active or inactive), then we keep the bit as it is. Otherwise it gets set to 0. <br>

Now look at the code for ```or()```.

```
function getOr() external pure returns (bytes32) {
 
    bytes32 randVar = 0x000000000000000000000000ffffffffffffffffffffffffffffffffffffffff;
    bytes32 mask = 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff;
 
    bytes32 ans;
 
    assembly {
 
        ans := or(mask, randVar)
 
    }
 
    return ans;
 
}
```
This time the output is ```0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff```. This is because it looks to see if either bit is active. Let’s look at what happens if we change the mask variable to ```0x00ffffffffffffffffffffff0000000000000000000000000000000000000000
```. As you can see the output changes to ```0x00ffffffffffffffffffffff9acc1d6aa9b846083e8a497a661853aae07f0f00```. Notice the first byte is ```0x00```, because neither input has an active bit for the first byte. 
<br>

```xor()``` is a little bit different. It requires one bit to be active (1) and the other bit to be inactive (0). Here is a code demonstration.
```
function getXor() external pure returns (bytes32) {
 
    bytes32 randVar = 0x00000000000000000000000000000000000000000000000000000000000000ff;
    bytes32 mask =    0xffffffffffffffffffffffff00000000000000000000000000000000000000ff;
 
    bytes32 ans;
 
    assembly {
 
        ans := xor(mask, randVar)
 
    }
 
    return ans;
 
}

```
 The output is ```0xffffffffffffffffffffffff0000000000000000000000000000000000000000```. The key difference is apparent when we see the only active bits in the output are when ```0x00``` and ```0xff``` are aligned.


<br>

```shl()``` and ```shr()``` operate very similarly to each other. Both shift the input value by an input amount of bits. ```shl()``` shifts to the left and ```shr()``` shifts to the right. Let’s take a look at some code!

```
function shlAndShr() external pure returns(bytes32, bytes32) {
   
    bytes32 randVar = 0xffff00000000000000000000000000000000000000000000000000000000ffff;
 
    bytes32 ans1;
    bytes32 ans2;
 
    assembly {
 
        ans1 := shr(16, randVar)
        ans2 := shl(16, randVar)
 
    }
 
    return (ans1, ans2);
 
}
```
Output: <br>
```ans1```: ```0x0000ffff00000000000000000000000000000000000000000000000000000000``` <br>
```ans2```:  ```0x00000000000000000000000000000000000000000000000000000000ffff0000``` <br>

Let’s start by looking at ```ans1```. We perform ```shr()``` by 16 bits (2 bytes). As you can see the last two bytes change from ```0xffff``` to ```0x0000```, and the first two bytes are shifted two bytes to the right. Knowing this, ```ans2``` seems self explanatory; all that happens is the bits are shifted to the left. 

<br>

Before we write to ```var5```, let's write a function that reads ```var4``` and ```var5``` first.

```
function readVar4AndVar5() external view returns (uint128, uint128) {
 
        uint128 readVar4;
        uint128 readVar5;
 
        bytes32 mask = 0x00000000000000000000000000000000ffffffffffffffffffffffffffffffff;
 
        assembly {
 
            let slot3 := sload(3)
 
            // the and() operation sets var5 to 0x00
            readVar4 := and(slot3, mask)
 
 
            // we shift var5 to var4's position
            // var5's old position becomes 0x00
            readVar5 := shr( mul( var5.offset, 8 ), slot3 )
 
        }
 
        return (readVar4, readVar5);
 
    }
```

The output is 1 & 2 as expected. For retrieving ```var4``` we just need to use a mask to set the value to ```0x0000000000000000000000000000000000000000000000000000000000000001
```. Then we return a ```uint128``` set equal to 1. When reading ```var5```, we need to shift ```var4``` off by shifting right. This leaves us with ```0x0000000000000000000000000000000000000000000000000000000000000002```, which we can return. It is important to note that sometimes you will have to shift and mask in unison to read a value that has more than 2 varibales packed into a storage slot.

<br>


Ok, we’re finally ready to change the value of ```var5``` to 4!


```
function writeVar5(uint256 newVal) external {
 
    assembly {
 
        // load slot 3
        let slot3 := sload(3)
 
        // mask for clearing var5
        let mask := 0x00000000000000000000000000000000ffffffffffffffffffffffffffffffff
 
        // isolate var4
        let clearedVar5 := and(slot3, mask)
 
        // format new value into var5 position
        let shiftedVal := shl( mul( var5.offset, 8 ), newVal )
 
        // combine new value with isolated var4
        let newSlot3 := or(shiftedVal, clearedVar5)
 
        // store new value to slot 3
        sstore(3, newSlot3)
    }
 
}
```

The first step is to load storage slot 3. Next, we need to create a mask. Similarly to when we read ```var4```, we want to isolate the value to ```0x0000000000000000000000000000000000000000000000000000000000000001
```. The next step is formatting our new value to be in ```var5```’s slot position so it looks like this ```0x0000000000000000000000000000000400000000000000000000000000000000```. Unlike when we read ```var5```, we are going to shift our value to the left this time. Finally, we are going to use ```or()``` to combine our values into 32 bytes of hexadecimal, and store that value to slot 3. We can check our work by calling ```getValInHex(3)```. This is going to return ```0x0000000000000000000000000000000400000000000000000000000000000001```, which is what we are expecting to see.

<br>

Great, you now know how to read and write to packed storage slots!

<br>



## Memory

Alright, we are finally ready to learn about memory!

<br>

Memory behaves differently than storage. Memory is not persistent. Meaning that once the function is done executing all of the variables are cleared. Memory is comparable to heap in other languages, but there is no garbage collector. Memory is a lot cheaper than storage. The first 22 words of memory costs are calculated linearly, but be careful because after that memory costs become quadratic. Memory is laid out in 32 byte sequences. We will get a better understanding of this later, but for now understand ```0x00``` - ```0x20``` is one sequence (you can think of it like a slot if that helps, but they are different). Solidity allocates ```0x00``` - ```0x40``` as ```scratch space```. This area of memory is not guaranteed to be empty, and is used for certain operations. ```0x40``` - ```0x60``` is known as the ```free memory pointer```, and is used to write something new to memory. ```0x60``` - ```0x80``` is typcally left empty. ```0x80``` is where we begin our opperations. Memory does not pack values. Retrieving values from storage will be stored in their own 32 byte sequence (i.e ```0x80-0xa0```).

<br> 

Memory is used for the following operations: <br>
  - Return values for external calls <br>
  - Set function values for external calls <br>
  - Get values from external calls <br>
  - Revert with an error string <br>
  - Log messages <br>
  - Hash with ```keccak256()``` <br>
  - Create other smart contracts <br>

<br>

Here are some useful Yul instructions for memory!

|   Instruction   |  Explanation   |
|  :---   | :--- |
| mload(p)  |  Similar to ```sload()```, but we are saying load the next 32 bytes after p |
| mstore(p, v) |  Similar to ```sstore()```, but we are saying store value v in p plus 32 bytes |
| mstore8(p, v) |  Similar to ```mstore()```, but only for a single byte |
| msize()  |  Returns the largest accessed memory index |
| pop(x)  |  Discard value x |
| return(p, s)  |  End execution, and return data from memory locations p - v |
| revert(p, s)  |  End execution without saving state changes, and return data from memory locations p - v |

<br>

Let's check out some more data structures!
<br>

Structs and fixed arrays actually behave the same but since we already looked at fixed arrays in the storage section, we are going to look at structs here. Look at the following struct.

```
struct Var10 {
    uint256 subVar1;
    uint256 subVar2;
}
```

Nothing unordinary about this, just a simple struct. Now let’s look at some code!

```
function getStructValues() external pure returns(uint256, uint256) {
 
    // initialize struct
    Var10 memory s;
    s.subVar1 = 32;
    s.subVar2 = 64;
 
    assembly {
        return( 0x80, 0xc0 )
    }
 
}
 
```

Here we are setting ```s.subVar1``` to memory location ```0x80``` - ```0xa0``` and  ```s.subVar2``` to memory location ```0xa0``` - ```0xc0```. That is why we are returning ```0x80``` - ```0xc0```. Here is a table of the memory layout right before the end of the transaction.


|   Memory Location   |  Value Stored  |
|  :---   | :--- |
| ```0x00``` | Scratch Space (Empty) |
| ```0x20``` |  Scratch Space (Empty) |
| ```0x40``` |  ```0xc0``` (Free Memory Pointer) |
| ```0x60``` |  Empty |
| ```0x80```  |  s.subVar1: ```0x20``` (32 in decimal) |
| ```0xa0```  |  s.subVar1: ```0x40``` (64 in decimal)  |   
| ```0xc0```  |  New Free Memory Pointer. This is what ```msize()``` would return. (Empty)  | 

Things to take away from this: <br>
  - ```0x00``` - ```0x40``` are empty for scratch space <br>
  - ```0x40``` gives us the free memory pointer <br>
  - Solidity leaves a gap for ```0x60``` <br>
  - ```0x80``` and ```0xa0``` are used for storing the values of the struct <br>
  - ```0xc0``` is the new free memory pointer. <br>

<br>

In this last part of the memory section I want to show you how dynamic arrays work in memory. We are going to pass ```[0, 1, 2, 3]``` as the parameter ```arr``` for this example. As an added bonus for this example we are going to add an extra element to the array. Be careful doing this in production as you may overwrite a different memory variable. Here is the code!

```
function getDynamicArray(uint256[] memory arr) external view returns (uint256[] memory) {
 
    assembly {
 
        // where array is stored in memory (0x80)
        let location := arr
 
        // length of array is stored at arr (4)
        let length := mload(arr)
 
        // gets next available memory location
        let nextMemoryLocation := add( add( location, 0x20 ), mul( length, 0x20 ) )
 
        // stores new value to memory
        mstore(nextMemoryLocation, 4)
 
        // increment length by 1
        length := add( length, 1 )
 
        // store new length value
        mstore(location, add( length, 1 ) )
 
        // update free memory pointer
        mstore(0x40, 0x140)
 
        return ( add( location, 0x20 ) , mul( length, 0x20 ) )
 
    }
 
}
 

```

What we are doing here is getting where the array is stored in memory. Then, we get the length of the array, which is stored in the first memory location of the array. To see the next available location we are adding 32 bytes to the location (skip the length of the array) , and multiplying the length of the array by 32 bytes. This advances us to the next memory location after our array. Here, we will store our new value (4). Next, we update the length of the array by one. After that, we have to update the free memory pointer. Finally, we return the array.

<br> 
Let’s look at the memory layout once again. 

|   Memory Location   |  Value Stored  |
|  :---   | :--- |
| ```0x00``` | Scratch Space (Empty) |
| ```0x20``` |  Scratch Space (Empty) |
| ```0x40``` |  ```0x140``` (Free Memory Pointer) |
| ```0x60``` |  Empty |
| ```0x80```  | New length of the array (6) |
| ```0xa0```  | arr[0] (0)  |    
| ```0xc0```  |  arr[1] (1) | 
| ```0xe0```  |  arr[2] (2) | 
| ```0x100```  |  arr[3] (3) | 
| ```0x120```  |  arr[4] (4) | 
| ```0x140```  |  Free Memory Pointer (Empty) | 

<br>

That concludes the section on memory!


## Contract Calls

In the final section of this article we will lok at how contract calls work in Yul.

To be continued.






