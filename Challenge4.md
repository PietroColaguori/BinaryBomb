## Challenge 4

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch4_1.png)

This function starts by saving into register `ecx` the input provided by the user and then `sscanf` is called with a `%d` format string, implying that the function reads a single integer value. The result of `sscanf`, i.e. the number of matches in the format string (in our case we are expecting 1), is compared to 1, if it is less than 1 the bomb explodes otherwise the execution goes on by comparing local variable `ebp-4` with `1`, if `ebp-4 >= 1` then the execution continues, otherwise the bomb explodes. Note: we saved the scanned integer at the address of `ebp-4` (used later). 

$\implies$ The input must be an integer (4 bytes).

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch4_2.png)

The program uses the integer given as input stored at `ebp-4` as argument for a subroutine that we will later easily identify as a recursive function. The result of this recursive function is then compared to `55`, if `result == 55` then the function returns otherwise the bomb explodes.

$\implies$ The input must be an integer such that the output of `recFun` is $55$.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch4_3.png)

The integer we insterted as input is compared to `1`, this is the base case for which the recursive function returns the contents of `eax`. Otherwise, the result of `recFun` called with argument `inputInt-1` and then stored in `esi` is summed up with the result of `recFun` called with argument `inputInt-2`, the overall sum is stored in `eax` and returned. This is very familiar, it is the famous *Fibonacci function*!

$\implies$ The input must be an integer such that the result of the Fibonacci sequence in position returns $55$.

| inputInt | Fibonacci(inputInt) |
| -------- | ------------------- |
| 1        | 1                   |
| 2        | 1+1=2               |
| 3        | 2+1=3               |
| 4        | 3+2=5               |
| 5        | 5+3=8               |
| 6        | 8+5=13              |
| 7        | 13+8=21             |
| 8        | 21+13=34            |
| 9        | 34+21=55            |

The right input to provide to the function is `9`!
Below is some C pseudo-code written based on the observed disassembled.

```c
int fibonacci(int v) {
	if(v == 1) { return 1; }
	int firstSum = fibonacci(v-1);
	int secondSum = fibonacci(v-2);
	return firstSum + secondSum;
}


void challenge4(char* input) {
	int inInt;
	int num_scanned = sscanf("%d", &inInt);
	if(num_scanned == 1) {
		if(inInt >= 1) {
			int result = fibonacci(inInt);
			if(result == 55) { return; }
			else { bombExplodes(); }
		}
		else { bombExplodes(); }
	}
	else { bombExplodes(); }
}
```
