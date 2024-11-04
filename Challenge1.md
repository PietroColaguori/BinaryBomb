The result of the `parseInput` is used as input for the function `challenge1`. What happens is that the message to display is loaded into memory and we will use two subroutines:
- A first subroutine is called twice, it will be used to check the length of the string we inserted as input as well as the length of the string we loaded into memory, we proceed if the two lengths are the same otherwise it explodes. We can call this subroutine `myStrLen`,
- A second subroutine compares character by character the two strings, the bomb does not explode only if every character is the same. We can call this subroutine `myStrCmp`.
To read one byte at a time we use the move with sign extension instruction, which is called `movsx`, in which we pad the remaining space in front of the byte with its sign to preserve it. In this case, since we are reading strings, preserving the sign was not really necessary, but the compiler for some reasons did it.

```c
int myStrLen(char* s) {
	char* v8 = s;
	int v4 = 0;
	char edx = *v8;
	while(edx != 0) {
		v8++;
		v4++;
	}
	return v4;
}

int processStrings(char* arg_0, char* arg_1) {
	int eax = myStrLen(arg_0);
	int esi = eax;
	eax = myStrLen(arg_1);
	if(esi == eax) {
		char* v4 = arg_0;
		char* v8 = arg_1;
		while(*v4 != 0) {
			char ecx = *v4;
			char eax = *v8;
			if(ecx == eax) {
				v4++;
				v8++;
				continue;
			}
			else {
				return 1;
			}
		}
		return 0;
	}
	else {
		return 1;
	}
}

char* arg_1 = "Public speaking is very easy.";
char* arg_0 = argv[1];
int eax = processStrings(arg_0, arg_1);
if(eax == 0) {
	return;
}
else {
	bombExplodes();
}
```
