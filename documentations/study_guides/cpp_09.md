# CPP MODULE 09  

Before starting this project, we must take a moment and read it completely to make decisions.  

As we already know, the STL (Standard Template Library) is divided into three types of containers: `Sequential Containers`, `Associative Containers`, and `Adapter Containers`. Among the sequential ones we have `Vector`, `Deque` and `List`: 
- `std::vector`
- `std::deque`
- `std::list`  

Among the associative ones we have `Map` and `Set`:
- `std::map`
- `std::set`  

Among the adapters are `Stack` and `Queue`:
- `std::stack`
- `std::queue`  

Each of the above has general characteristics, which is why they belong to a certain category, but they also have unit characteristics. For example, vector is sequential, meaning that the information in it is stored sequentially, like deque and list. In all three we will have {hello, goodbye, house, farm, dog, cat} or {56, 48, 3, 6, 45, 8}. The difference is in memory management, since in vector this will be a single block {|56|48| 3| 6| 45| 8} where if I insert a value all elements are shifted to the right, and if I delete a value all elements are shifted to the left, while in deque {56, 48, } {3, 6, 45, 8} the data is grouped into different blocks, and in list {56-> 48-> 3-> 6-> 45-> 8} they are nodes.  

These particularities generate strengths and weaknesses and make them faster or slower for different algorithms. For this reason, this project forces us to look at the characteristics of each container and see which one is the most suitable for the task in each exercise.  

The decision we must make before starting the project is which container to use in each part, since we must use one for `exercise 00`, `another for 01`, and `two for 02`, with the limitation that once a container is used, it cannot be used again in the following exercises.  

With this in mind, we must analyse each exercise and see which is the ideal container for it.  

## Exercise 00  

In this exercise the goal is to build a program that allows us to search and read information about the price of bitcoin on different dates. For this we have a .csv file (comma separated values) in which we must find the information provided by the user in another file. That is, we will basically open and read a .csv file that will be our database and then we will open a .txt that contains the information we are going to search for.  

Keep in mind that the information in data is a date and a value which is a positive integer less than 1000.  

## Exercise 01  

In this one the goal is to create a program that implements RPN (Reverse Polish Notation), which is basically an algorithm that takes data in a particular order, and we will implement it with the basic operations `+` `-` `*` `/`.  

## Exercise 02  

In this exercise the goal is to create a program that implements the Ford Johnson Sort, which is a sequential and indexed sorting algorithm, since we must also implement Jacobsthal, which is an algorithm that generates limits for the creation of indexes.  

## Take decitions 

- `The first exercise` is of the associative type, so we must choose between map or set. Why? Because if we worked with vector, list, or deque, these are sequential, meaning that to search for a value they would have to iterate through everything, making them very slow and expensive compared to map or set.  

In this case the best option would be map, since it allows storing values associated with a key, which fits this problem. Also, set only allows storing unique values, which would not make sense in this exercise.  

- `The second exercise` the logical option would be stack. Being an adapter type, and due to the nature of this algorithm, the most logical is `LIFO (Last In, First Out)`, which stack implements perfectly. For example, 5 6 + means that when you find a token you will pop the values, operate them with the token, and then push the result back onto the stack.  

With vector it could be done, but it would be a waste since we do not need random access nor iteration, and we would also have to manually manage indexes to simulate `LIFO`.  

Queue is not possible since it operates in `FIFO (First In, First Out)`.  

- `The third and last exercise`, since we must iterate and compare in pairs `(so it is sequential)`, and after comparing we must access by index, the options are vector, list, and deque.  

Although list is sequential, it does not allow index access, so our choice is between vector and deque. If we had to choose only one, the option would be vector, since being a system of continuous blocks it is much faster for this algorithm compared to deque, which is a system of multiple blocks.  

In vector  {56, 48, 3, 6, 45, 8, 6} when running the first part of the algorithm, which is the comparison in pairs, everything is in the same memory line, meaning everything is loaded from the moment the vector is loaded {(56, 48), (3, 6), (45, 8), 6}. When it is divided into two vectors it will be  m {....} p {....} and no matter how many times it is divided, it is always a single block created for each one. In contrast, for deque {56, 48, 3, 6, 45, 8, 6} = {56, 48, 3} {6, 45, 8, 6}, several blocks are created, which uses more memory because to compare them we have block1 {(56, 48), (3)} block2 {(6), (45, 8), 6}, and so on, always creating and searching through more blocks.  

### To clarificate 

- `The first exercise is of the Associative type and between set and map the best option is map.`  

- `The second exercise is of the Adapter type and between stack and queue the best option is stack.` 

- `The third exercise is of the Sequential type and between vector, deque, and list the options are vector and deque.` 

## Ex00  

### What we need:  

- `Read and load a database from a csv file `

- `Read a .txt file separated by “ | “that contains the information to search for` 

#### What information we have:  

- `The csv file contains rows separated by ',' with date and value`  

- `In our search the value is an int between 0 and 1000`

- `If the searched date is not found, the previous date must be used`  

So, we need to create a map that contains the information from the csv file and allows us to iterate to search whatever is in the txt.  

To load the data (date and value), first we need a function that receives a string and removes leading and trailing spaces.  

Then create another function that splits it into year, month, and day, converting them to int, and verifying that the month is between 1 and 12, and that the date is valid (no November 31 or February 30), including a special validation for leap years such as February 29 in 2000.  

Then validate that the value is not less than 0, not greater than 1000, not empty, and is a valid integer, then convert it to float.  

Once this is done, create another function that opens the .csv and loads the database line by line, verifying that everything meets the requirements.  

Finally, create the function that manages the process: open the .txt, verify date and value, and once everything is validated, iterate through the map and print the requested information.  

## Ex01  

### What we need:  

- `Traverse a string and insert numbers in order until finding a token + - * /`  

- `When we find a token, operate the numbers using Reverse Polish Notation'   

#### What information we have:  

- `We need to implement Reverse Polish Notation`

- `The numbers used cannot be greater than 10, meaning from 1 to 9`  

- `If any error occurs, print an error message`

- `We must support + - * /`

### Reverse Polish Notion

Reverse Polish notation is a way of writing expressions where values come first and operations come afterward, allowing them to be evaluated directly without precedence rules example: `5 6 3 * - -> 5 6 * 3 -> 5 -18 -> 13`

First, create an auxiliary function that performs calculations, receiving two int values and a char token.  

Verify that the numbers are not greater than 10 and that tokens exist. First verify that the string is not empty using. empty; if it has content, loop through it, casting to int and skipping spaces, starting from 1 since 0 is the program name. Once cast, add the number to the stack.  

If a token is found, check if we have more than one number in the stack; if so, pop them and pass them to the auxiliary calculate function.  

Check the special case to avoid division by zero and push the result of the operation to the stack.  

At the end, check that the stack size is exactly 1; otherwise, report an error.  

## Ex02  

### What we need:  

- `Be able to iterate through a chain of positive integers to sort it`  

- `Implement the fordJohnsonSort sorting algorithm` 

- `Measure and compare the time of each container` 

#### What information we have:  

- `We must use positive numbers`

- `We most use fordjohnsonSort algorithm and jacobStall algorithm`

### FordJohnsonSort algorithm

The fordJohnsonSort is an algorithm that sorts sequences of numbers by comparing them in pairs and dividing them into two chains, a main and a secondary one. The secondary one will execute another algorithm, jacobSthal, which generates limits to generate indexes. It continues comparing and dividing the main chain until it has a single element, then it starts inserting the secondary chains by index order until it is fully sorted. Example:  

`./PmergeMe 3 5 9 7 4` 

`vector` { (3, 5), (9, 7), 4 } compare if the first is greater than the second and swap the larger ones so that the larger is in front and the smaller behind: {(5, 3), (9, 7), 4}. Now divide into two vectors, main and pend; the leftover number goes directly to main.  

std::vector main {5, 9, 4} 

std::vector pend {3, 7} 

main must be compared again  

main {(5, 9), 4} -> main {(9, 5), 4} -> main {9, 4} pend1 {5}  

And again main {(9, 4)} -> main {(9, 4)} -> main {9} pend2 {4(1)} insertion by index and binary search begins.  

main {9}  

pend2 {4(1)}  

main {4, 9}  

pend1 {5(1)}  

main {4, 5, 9}  

pend {3(2), 7(1)}  

main {4, 5, 7, 9}  

main {3, 4, 5, 7, 9} sorted vector  

As is evident, to fully implement this algorithm it is also necessary to implement jacobSthal, which receives an int and applies the algorithm formula n = n + 2 * n - 1 to generate limits and then indexes. For example:  

### JacobSthal algorithm 

- `We have a vector with 6 elements, so the limit is n = 6. Applying the formula value = current value + 2 * previous value we get:`

J(0) = 0  

J(1) = 1  

J(2) = J(1) + 2 * J(0) = 1 + 2*0 = 1  

J(3) = J(2) + 2 * J(1) = 1 + 2*1 = 3  

J(4) = J(3) + 2 * J(2) = 3 + 2*1 = 5  

J(5) = J(4) + 2 * J(3) = 5 + 2*3 = 11 -> `discarded because it exceeds n` 

J {0, 1, 1, 3, 5}  

- `Then we take the indices up to 5 since 6 exceeds:` 

(5 -> 3) : [5, 4] 

(3 -> 1) : [3, 2] 

(1 -> 1) : [] 

(1 -> 0) : [1] 

Final: [5, 4, 3, 2, 1]  

- `Now we iterate in a loop to create the indexes using the limits generated by jacobSthal`

For j = [0, 1, 1, 3, 5]:  

- `Initial variables:` 

pend = [p0, p1, p2, p3, p4, p5] 

n = 6 

used = [false, false, false, false, false, false] 

seq = [] (to build)  

- `Iteration 1:`

i = 1 -> j[1] = 1 

used[1] = false -> insert -> seq = [1] 

mark used[1] = true  

- `Iteration 2:`

i = 2 -> j[2] = 1 

used[1] = true -> already used -> not inserted  

- `Iteration 3:` 

i = 3 -> j[3] = 3 

used[3] = false -> insert -> seq = [1, 3] 

mark used[3] = true  

- `Iteration 4:` 

i = 4 -> j[4] = 5 

used[5] = false -> insert -> seq = [1, 3, 5] 

mark used[5] = true  

- `Now we add the remaining unused indexes:`

i = 0 -> used[0] = false -> insert -> seq = [1, 3, 5, 0] 

i = 1 -> used[1] = true -> skip 

i = 2 -> used[2] = false -> insert -> seq = [1, 3, 5, 0, 2] 

i = 3 -> used[3] = true -> skip 

i = 4 -> used[4] = false -> insert -> seq = [1, 3, 5, 0, 2, 4] 

i = 5 -> used[5] = true -> skip  

- `Final insertion sequence`

[1, 3, 5, 0, 2, 4]  

### Iteration by iteration summary:  

| `Step` | `i` | `j[n]/Action` | `List` | `Status`|
| :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | j[1]=1, insert | [1] | [F,T,F,F,F,F] |
| 2 | 2 | j[2]=1, used | [1] | [F,T,F,F,F,F] | 
| 3 | 3 | j[3]=3, insert | [1,3] | [F,T,F,T,F,F] |
| 4 | 4 | j[4]=5, insert | [1,3,5] | [F,T,F,T,F,T] | 
| 5 | 0 | resto, insert 0 | [1,3,5,0] | [T,T,F,T,F,T] |
| 6 | 2 | resto, insert 2 | [1,3,5,0,2] | [T,T,T,T,F,T] | 
| 7 | 4 | resto, insert 4 | [1,3,5,0,2,4] | [T,T,T,T,T,T] | 

This way we create a sequence without duplicates and ready to insert the elements into pend.  

Now we start implementing fordJohnsonSort. We check that we have more than one element, then we create and initialize two vectors: main and pend.  

In main we separate by pairs and compare them, keeping the larger ones in main and the smaller ones in pend. Any leftover goes to main. We recursively apply fordJohnsonSort to main until it has a single element. All pend chains generated are sent to jacobSthal to generate insertion indexes.  

Finally, we insert pend into main using lower_bound (as in ex00) so each value is placed correctly: 3 before 4, 10 between 8 and 18, etc.  

Now we can begin. We need addValue to fill the original vector.  

Then we implement sort, which initializes a timer and calls fordJohnsonSort until it finishes.  

We also implement print to print the sorted vector, and printBefore to print the original vector.  

Finally, process calls sortVector.  

In main we validate that all inputs are numbers.  

If they are valid, we call addNumbers, then printBefore, then process, which calls sort, starts the timer, calls fordJohnsonSort, splits the chains, calls jacobSthal for each pend, inserts using the generated indexes, stops the timer, and process finally calls printAfter. 