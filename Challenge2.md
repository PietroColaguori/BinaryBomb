## Challenge 2

Let's analyze challenge number 2. The first things we notice is that a function is called and such function takes as input two arguments, one is a pointer to a `dword`, so a pointer to an `int` value, in other words an array of integers, the other instead is recognized by IDA as a buffer, so a pointer to a chatacter. We can also immediately see that if, after calling the function, the first element of the array is not `1` , the bomb will explode.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch2_1.png)

Let's dive deeper into the function, what this function mainly does is calling the `sscanf` function, which returns the number of values read in the proper format. The arguments passed to `sscanf` are the addresses of each of 4 bytes cells contained in the array of integers. In fact, we know that the array of integers starts at `ebp-12`, so the last element can be found at address `ebp-(12+4*5)=ebp-(12+20)=ebp-32`. Starting from the last cell at `ebp-32` the address of each cell is passed as an argument, then the first argument is set as the formatting string.

Note: once you see a specific function, start reasoning on the parameters based on what are the kinds of arguments such function requires.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch2_2.png)

A control is performed on the number of values parsed correctly by `sscanf`, which is its return value: if the number is less than 6 then the bomb explodes, otherwise the function returns.

After returning the program checks wether the first value of the array located at `ebp-12` is equal to `1`, in such case the program proceeds, otherwise the bomb explodes.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch2_3.png)

The program proceeds by initializing a variable to `1`, to be used as a counter, and then a cycle begins, at each iteration it is checked wether `i >= 6`, in case it is the function returns. If it is not, we save the value of `i+1` and, starting this time at `ebp-20` (so skipping the first and the last value of the array, then optimizing the array in the stack by subtracting two times 4 bytes and thus making the first element be at `ebp-12-4-4 = ebp-20` rather than `ebp-12`). This makes sense, in fact there are two values in the array of integers which we won't need when computing `a[i-1]`:
- The first value, since we have nothign before (i.e. `i-1=-1`).
- The last value, since we have no other value after it for which we are interested in doing this computation. i.e. once the last value is reached the cycle just stops.

The program computes `(i+1) * a[i-1]` and compares it to the value of `a[i]`. If the values are not the same, then the bomb will explode, otherwise the cycle will keep going. Once the end of the cycle is reached, the function returns, as stated earlier.

The takeway to defuse the phase 2 is the following: we must build an array of 6 integers such that each element $a_i$ is equal $(i+1)$ times the previous element $a_{i-1}$, keeping in mind that $a_0=1$. 
We obtain:
- $a_0$ = 1.
- $a_1=2*ì \cdot a_0=2$.
- $a_2=3 \cdot a_1=6$.
- $a_3=4 \cdot a_2 = 24$.
- $a_4=5 \cdot a_3 = 120$.
- $a_5=6 \cdot a_4 = 720$.

```c
int valuesRead = readSixNumbers(char &s, int &a);
if(a[0] == 1) {
	int i = 1;
	while(i <= 6) {
		eax = i + 1;
		ecx = i;
		eax = (i + 1) * a[i - 1];
		edx = i;
		if(a[edx] == eax) {
			i++;
			continue;
		}
		else {
			bombExplodes();
		}
	}
}
else {
	bombExplodes();
}
```
