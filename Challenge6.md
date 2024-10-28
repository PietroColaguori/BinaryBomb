## Binary Bomb - Challenge 6

The first thing we notice is that the `challenge6` function passes to a subroutine two pointers, upon inspecting such subroutine we realise that it calls the `sscanf` C library function to read 6 space-separated integers from the input file. These six numbers (array of 6 integers) is saved in memory at the location `ebp-20h = ebp-32`, we will call this variable `sixNumbers`, I will refer to this variable also as `input` in IDA comments and in the pseudo source code later.

Then a counter is initialized to 0 and compared to the value 6, if `i >= 6` then the program jumps to another location (out of the loop) otherwise the execution continues. If the execution continues, the `i`-th element of the input array is checked, if it is not in the set $[1,6]$ then the bomb explodes, otherwise the value of `i` is increased by 1 and a new counter is initialized to the value of `++i`.

This new counter, we will call it `j`, will compare the value of `input[i]` with all the remaining values of the array with indexes in the set $[i+1,5]$, effectively ensuring that no two values in the input array are the same. Below is a snippet showing how this last check is performed.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch6_1.png)

Going on with the execution, we encounter another loop. For each value of $i \in [0,5]$ we set the value of the local variable `ebp-4` to `ebp-60`, in particular, by examining the stack we can see that while `ebp-4` seems to be some sort of pointer, `ebp-60` seems to be placed right at the beginning of some sort of array, curiously of the same size as our input array! We will call the first variable `nodePtr` and the second variable `initialAddress`, for reasons that will be clearer later. I will refer to `initialAddress` also as `headNode` in IDA comments and pseudo source code later. For each value of $i \in [0,5]$, $j$ will be initialized to `1` and we will check the following:
- If `j >= sixNumbers[i]`, then we will set the $i$th value of the local array `elements` to the value of `nodePtr`, which was initialized to `headNode`.
- Otherwise, we will increase by 1 the value of `j` and increase by `8 bytes` the value of the `nodePtr`. This is interesting, it represents a hint of the fact that we are navigating some sort of data structure, to find out which we should observe the stack, beginning at address `initialAddress`.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch6_2.png)

Let's observe the stack, as stated before, to understand which kind of data structure we are talking about, we observe the following:
- The first 4 bytes contain a numerical value. We will call this `value`.
- The second 4 bytes contain the index, starting from `1`, in memory, of the node considered.
- The third 4 bytes (the cell we access when performing `nodePtr+8`) contains the address in memory of the next node. We will call this `next`.
Clearly, this is linked list.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch6_3.png)

Note: remeber that we must refer to little-endian, the first 4 bytes of this node contain the value `002D5`, which in decimal is `725`. As we can see this is the second node in the stack and the third cell contains the pointer to the next one.

To recap, we have now ended up with an array containing addresses of nodes on the stack. Nodes of which we know the value and layout thanks to IDA. E.g. if `input[0] == 1`, then `elements[0]` will contain the address to the first node of the linked list. So, the question is, how we set the values of `input` so that we obtain a desired linked list? For that we have to first check the remaning loops!

The next loop is quite basic, we start by setting $i=1$ and `nodePtr = headNode`, then we perform 5 iterations, during these iterations we simply set each time the next node in the linked list as the $i$th element of the array of pointers we have built in the previous 2 phases, when we reach the final node, we set the next node as `NULL`. With this very easy cycle, we just link together the nodes in the order we have chosen, effectively a new linked list.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch6_4.png)

Time for the last cycle, this one is also fairly easy to understand, for each node in the linked list just created, we check wether the value of the current node is greater or equal than that of the next node, if this condition is not met, the bomb explodes. This loop enforced descending order on the way in which we choose the nodes in phase 2.

![](https://github.com/PietroColaguori/BinaryBomb/blob/main/assets/ch6_5.png)

It should be now clear how to build the linked list in such a way that the bomb does not explode. We need to explore the values stored in the nodes on the stack and create the `input` array with their offset (in terms of nodes) between the desired node and the head of the linked list.

| Displacement | Value (Hex) | Value (Dec) |
| ------------ | ----------- | ----------- |
| 1            | FD          | 253         |
| 2            | 2D5         | 725         |
| 3            | 12D         | 301         |
| 4            | 3E5         | 997         |
| 5            | D4          | 212         |
| 6            | 1B0         | 432         |

The input sequence we are looking for is `4 2 6 3 1 5`!

Below is a some pseudo code I wrote in C to showcase how the program seems to behave, this is probably not very precise but should give you an idea of the way the sample is behaving.

```c
int input[6];
int elements[6];
struct node nodes[6];
int* nodePtr = &nodes[0];

// Numbers must be in [1,6] and all different
for(int i = 0; i <= 6; i++) {
	if(input[i] < 1) { bombExplodes(); }
	if(input[i] > 6) { bombExplodes(); }
	for(int j = i+1; j < 6; j++) {
		if(input[i] == input[j]) { bombExplodes(); }
	}
}

// Populate array of pointers (decide on ordering)
i = 0;
while(i < 6) {
	nodePtr = headNode;
	j = 1;
	while(j < input[i]) {
		nodePtr = nodePtr -> next;
		j++;
	}
	elements[i] = nodePtr
	i++;
}

// Link the nodes of the LL based on ordering
i = 1;
nodePtr = headNode;
while(i < 6) {
	nodePtr -> next = elements[i];
	nodePtr = nodePtr -> next;
	i++;
}
nodePtr -> next = NULL;

// Ensure descending order inside the LL
i = 0;
nodePtr = headNode;
while(i < 5) {
	if(nodePtr -> value < nodePtr -> next -> value) { bombExplodes(); }
	i++;
	nodePtr = nodePtr -> next;
}
```
