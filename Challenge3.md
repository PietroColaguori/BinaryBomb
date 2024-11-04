## Challenge 3

We can now start analyzing the third challenge. We immediately see some interesting variables placed in the stack, among these we see two integer pointers and one character pointer (denoted as `byte ptr` by IDA). The first thing that happens is that the addresses of these three variables are pushed on the stack and used as arguments for `sscanf` together with the format string `%d %c %d`.  After calling `sscanf` we check wether the number of values parsed correctly is indeed 3, as we would expect, if it is not 3 then the bomb explodes.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch3_1.png)

The program proceeds by comparing the first integer with `7`, if the first number is greater then `7` then the bomb will explode, otherwise the program will enter a switch jumptable and jump to the case corresponding to the first number inserted, for a total of `7` cases.
All the cases are quite similar so we will analyze jump `case 0: ...` as an example.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch3_2.png)

Analyzing case 0 we see that the character `q` is saved in a variable for later use and then the second integer is compared to the number `777`, if they are not equal then the bomb explodes otherwise the execution continues.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch3_3.png)

The program proceeds by comparing the previosuly saved character `q` with the input character inserted by us, if they are not equal the bomb explodes, otherwise we the function returns.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch3_4.png)

In conclusion, considering the `case 0`, to create the right input to win this challenge we have to simply follow the "instructions" and write `0 q 777`.

```c
int eax = sscanf("%d %c %d", int &var_C, char &var_D, char &v8;
if(eax >= 3) {
	int ecx = var_C;
	int v18 = ecx;
	if(v18 > 7) {
		bombExplodes();
	}
	else {
		switch(v18) {
		
			# ...
			case 7:
				int v1 = 98;
				if(v8 == 524) {
					int eax = v1;
					int ecx = var_D;
					if(var_D == v1) {
						return;
					}
					else {
						bombExplodes();
					}
				}
				else {
					bombExplodes();
				}
		}
	}
}
else {
	bombExplodes();
}
```
