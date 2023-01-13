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
The first topic we need to cover is simple operations. Yul has +, -, *, /, %, **, <, >, & =. Notice that >= and <= are not included, Yul does not have those operations. Additonally, instead of evaluations equaling ture or false, they equal 1 or 0 respectivley. With that said, let's get started with learning some Yul!

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


## Storage
Before we can dive deeper into how Yul works, we need a good understanding of how storage works in smart contracts. Storage is composed of a series of slots. There are 2^256 slots for a smart contract. When declaring variables, we start with slot0 and increment from there. Each slot is 256 bits long (32 bytes), that’s where uint256 and bytes32 get their names from. All variables are converted to hexadecimal. If a variable, such as a uint128, is used we do not take an entire slot to store that variable. Instead it is padded with 0’s on the left side. Let’s look at an example to get a better understanding. <br>

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

```var1``` : Since uint256 variables are equal to 32 bytes, var 1 takes up the entirety of slot 0. Here is what is being stored in slot 0: ```0x0000000000000000000000000000000000000000000000000000000000000100```. 
<br>

```var2```: Addresses are slightly more complex. Since they only take up 20 bytes of storage, addresses get padded with 0s on the left side. Here is what is being stored in slot 1: ```0x0000000000000000000000009acc1d6aa9b846083e8a497a661853aae07f0f00```. 

<br>

```var3```: This one may seem simple, slot 2 is consumed by the entirety of the bytes32 variable. 

<br>

```var4 & var5```: Remember when I mentioned that uint128’s get padded with 0’s? Well if we order our variables so that the sum of their storage is under 32 bytes, we can fit them into a slot together! This is called packing variables, and it can save you on gas. Let's look at what’s stored in slot 3: ```0x0000000000000000000000000000000200000000000000000000000000000001```. Notice that ```0x000000000000000000000000000002``` and ```0x000000000000000000000000000001``` fit perfectly together in the same slot. That is because they both take up 16 bytes (half of a slot). 

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
```y``` = 16. This should also make sense since we know that var4 takes up half of slot 3. Since variables are packed from right to left we get byte 16 as the start index of var5. <br>
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
Mappings behave similarly to dynamic arrays except that we hash the slot along with the key value.

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
Congradulations, you completed the section on storage with Yul.

<br>

## Reading & Writing Packed Variables

Suppose you want to change ```var5``` to 4. We know that var5 is located in slot 3, so you might try something like this:

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

Let's start with ```and()```. We are going to take two bytes32 and try the and opperator and see what it returns.
<br>

```
function getAnd() external view returns (bytes32) {

    bytes32 randVar = 0x0000000000000000000000009acc1d6aa9b846083e8a497a661853aae07f0f00;
    bytes32 mask = 0x1111111111111111111111110000000000000000000000000000000000000000;

    bytes32 ans;

    assembly {

        ans := and(mask, randVar)

    }

    return ans;

}
```

To be conintued.













