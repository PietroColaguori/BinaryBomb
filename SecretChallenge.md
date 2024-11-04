## Secret Challenge

How do we find the secret phase? Let's start by taking a look at the `checkCompletion` function.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/chs_1.png)

We can see that this function checks the number of challenges that we have solved and if they are not 6 then it will return, otherwise it will go on and load two addresses from memory. Then it will prompt us with a `sscanf("%d %s", &Buffer)` and if the number of parameters matches is equal to 2 it will proceed otherwise it will congratulate us for defusing the bomb and exit. 

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/chs_2.png)

The string `austinpowers` is then loaded from memory (we examine the stack with hex-view to verify the value IDA is suggesting) and is compared with the password we inserted during the `sscanf`, while the number we inserted seems to have no relevance for now. If we inserted `austinpowers` as password the sample curses at us for finding the secret phase, otherwise it will congratulate us and exit.

However, the buffer the `sscanf` is reading from is not a simple buffer, it is the same buffer to which we have written the solution for the Fibonacci challenge (phase 4), this time not only the number `9` is parsed but also the password, which we set to `austinpowers`.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/chs_3.png)

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/chs_4.png)

We enter the `secretChallenge()` function, this function takes as input a string, converts it into an integer and the bomb will explode unless the obtained integer belongs to the set $[1,1001]$.
The conversion is performed using the C library function `atoi`. Then a subroutine is called, taking as arguments a pointer to what seems like a data structure on the stack and the integer that resulted from the earlier conversion. We do not yet know what this data structure is and we do not know what this subroutine does, we only know that if the result of the subroutine is not 7 the bomb will explode, otherwise we have successfully defused the bomb.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/chs_5.png)

We look at the layout of the data structure in the memory, it has an integer and two pointers, each pointer points to another node, exploring a bit we can deduce it is a binary tree (and not a doubly linked list as one may think in the beginning).

```c
struct TreeNode {
	int value;           // node + 0
	TreeNode* leftSon;   // node + 4
	TreeNode* rightSon;  // node + 8
};

/*
After some analysis, we deduce the layout to be the following:

/
└── 36
    ├── 8
    │   ├── 6
    │   │   ├── 1
    │   │   └── 7
    │   └── 22
    │       ├── 20
    │       └── 35
    └── 50
        ├── 45
        │   ├── 40
        │   └── 47
        └── 107
            ├── 99
            └── 1001

Note: values have been converted to decimal, the order of sons is, from top to bottom,
      first left son and then right son.
*/
```

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/chs_6.png)

Let's analyze the code related to the function, I renamed it `recTree`, that interacts with the tree data structure. If the pointer to the tree node is NULL we have reached the fourth level and we return -1.
Otherwise if the input integer (the one previously converted from string using `atoi`) is greater or equal than the current value of the node pointed at by the tree pointer (located at `[ebp+treePtr+0]`), we go on with an additional comparison, checking if the input integer is different from the current value or not, if it is different we jump to a location we will analyze later, otherwise, if the values are equal we return 0.
If in the first comparison `convertedToInt < treePtr -> value == True`, then we perform a recursive call using the left son of the current node, i.e. `recTree(treePtr -> leftSon, convertedToInt)`, and we return the value returned by the previous recursive call shifted to the left by one, as we know:

$$
x << y = x \cdot 2^y
$$
So in our case we return the return value of the previous recursive call multiplied by 2.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/chs_7.png)

If instead, going back to the second comparison, `convertedToInt != treePtr -> value`, we will execute the branch shown above. Which basically simply makes the recursive call `recTree(treePtr -> rightSon, convertedToInt)` and returns the value returned by the recursive call multiplied by two and incremented by one.

All we know is that we want this function to return `7`, how can we do so? We know that the starting address passed at the very beginning will be `0x00404688` (in `.data`), containing the first node of the tree. At this point we can simulate the behaviour of the function and see how we can steer it into the direction we want.

To get `7` we should reason in a backwards manner, as often happens with recursive function.
We have two base cases:
- We reach a node with value equal to the value passed as input, and we return `0`.
- We reach a leaf and perform an additional recursive call, because no previous node had the same value as the one given by us as input, and we return `-1`.

Clearly, if we start we `-1` there is no way that by doing $\cdot 2$ or $\cdot 2 + 1$ operations we ever reach `7`. The only possibility for us is starting with `0`, meaning we want to trigger the base case, i.e. we give in input a value already present in a node. 

| Value | Direction | newValue            |
| ----- | --------- | ------------------- |
| 0     | RIGHT     | $0 \cdot 2 + 1 = 1$ |
| 1     | RIGHT     | $1 \cdot 2 + 1 = 3$ |
| 3     | RIGHT     | $3 \cdot 2 + 1 = 7$ |

If we start with `0`, the only way to reach `7` is to perform the $\cdot 2 + 1$ transformation 3 times, meaning that we need to go to the right node 3 times *and* choose as input value the value of the last node, to trigger the base case. In this case, the last node reached contains value `1001` and so this will be our input.

Let's add `1001` to our `input.txt` file and we verify that we indeed defused the bomb.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/chs_8.png)

Below I show some pseudo C code that I have written to better understand the general behaviour of the functions used inside this challenge.


```c
struct treeNode {
  int value;           // treeNode + 0
  treeNode* leftSon;   // treeNode + 4
  treeNode* rightSon;  // treeNode + 8
}

treeNode* binTree;

int recTree(treeNode* nodePtr, int input) {
	if(nodePtr == NULL) { return -1; }  // base case 1
	if(input >= nodePtr -> value) {
		if(input != nodePtr -> value) {
			return recTree(nodePtr -> rightSon, input) * 2 + 1;
		}
		else {
			return 0;  // base case 2
		}
	}
	else {
		return recTree(nodePtr -> leftSon, input) * 2;
	}
}

void secretChallenge(char* inputString) {
	int input = atoi(inputString);
	if(input < 1 || input > 1001) { bombExplodes(); }
	int result = recTree(binTree, input);
	if(result == 7) { 
		printf("Wow! You've defused the secret stage.\n")
		return;
	}
	bombExplodes();
}
```



