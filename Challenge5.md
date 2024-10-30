## Challenge 5

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch5_1.png)

The function starts by allocating some variables on the stack and storing on in `eax` the value of `arg_0` which is the string given in input to the function. This string is used as parameter for a subroutine, which upon inspection, reveals to simply count the number of characters in the input string and returning such value. Then, the value returned is compared with `6`, if the length of the input string is actually `6` the program goes on by jumping to the next memory location, otherwise the bomb explodes.

$\implies$ The input string must be 6 characters long (6 bytes).

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch5_2.png)

Going on with the analysis we see that a cycle is encountered, with a total of 6 iterations. At each iteration we take the $i$th character of the input string and we perform a bit-wise AND operation with the hexadecimal value `0x0F`, the resulting byte will then be used as an index to recover a byte from a string in memory, such byte will become the $i$th character of another string allocated in the stack at position `ebp-12`, so basically `[ebp-12-i] = byte_404580[ [ebp+input+i] && 0x0F ]`.
Let's take a look at the string in memory we are referring to: 

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch5_3.png)

So, the value of `memoryString` is `isrveawhobpnutfggiants`, note that we add an `i` in the first position because the first byte is `69h (hex) = 105 (dec) = i (ASCII)`.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch5_4.png)

Going on with the execution, we see that a subroutine, which upon closer inspection simply performs the same actions of the `strcmp` C library function,  is called with arguments `giants`, which is nothing more then the memory string `srveawhobpnutfggiants` (this time the first `0x69` byte is not included) take with an offset of `0xF = 15`, and the string we created earlier. If the `strcmp` returns `0` the two strings are the same and we have solved the challenge, otherwise the bomb expodes.

$\implies$ The input string must be made of characters such that the result of the bit-wise AND operation between each character and `0x0F` returns an index that corresponds to the desired character taken from the string in memory (including the inital `i`).

We notice that by applying the mask `0x0F = 0000 1111`, the maximum index that we can reach is `15`, so up until the first `g` encountered. For each index, we ask ourselves, which character when &&'d with `0x0F` returns such integer?

| Index | inStr\[Index\] | memStr\[Index\] |
| ----- | -------------- | --------------- |
| 15    | 6F = o         | g               |
| 0     | 70 = p         | i               |
| 5     | 75 = u         | a               |
| 11    | 6B = k         | n               |
| 13    | 6D = m         | t               |
| 1     | 61 = a         | s               |

The string we need to use as input for the challenge 5 function is: `opukma`!
Note: this is just a possible solution, we could obtain the same result with `opekmq` for instance.

Below is C source code representation of this challenge.

```c
char* inStr;
char newStr[6];
char* memStr = "isrveawhobpnutfggiants";

int inStrLen = myStrCmp(inStr);
if(inStrLen == 6) {
	int i = 0;
	while(i < 6) {
		newStr[i] = memStr[inStr[i] && 0x0F];
		i++;
	}
	if(newStr == memStr + 16) { return; }
	else { bombExplodes(); }
} else { bombExplodes(); }
```
