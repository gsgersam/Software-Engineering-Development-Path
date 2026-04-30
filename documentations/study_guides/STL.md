# STL STANDARD TEMPLATE LIBRARY  

The STL has three fundamental pillars which are: Containers (where data is stored), Iterators (the bridge to access them), and Algorithms (functions to manipulate them).  

 
## CLASSIFICATION OF CONTAINERS  

In C++, containers are divided into three main categories depending on how they organize data in memory and how they allow access to it:  


### Sequential Containers  

They maintain the order in which you insert the elements. The position of when and where you placed them.  

- `std::vector: is a dynamic array.`  

- `std::deque: is a double-ended queue (Double-Ended Queue).`  

- `std::list: is a doubly linked list.`  


### Associative Containers  

They do not maintain chronological order but are based on keys. The data is automatically ordered (usually through binary trees) to allow fast searches. 

- `std::set : is a collection of unique elements.`  

- `std::map : key-value pairs.`  


### Container Adapters  

They are not “real” containers by themselves but imitate the interface of a sequential container so that it behaves in a specific way.  

- `std::stack : LIFO (Last In, First Out).`  

- `std::queue : FIFO (First In, First Out).`

 
## COMPARISON OF SEQUENTIAL CONTAINERS  

 

### std::vector (Dynamic Array)  

Everything is in a contiguous block. If you want the element v[2], the processor simply jumps to the exact memory address.  
```
Memory: [ 2 | 5 | 26 | 48 ]  
          ^  ^     ^      ^   
Index:    0  1      2     3  
```

Behaviour: If you insert at the beginning, it has to “push” all the others to the right. Very expensive (O(n)).  

 

### std::deque (Double-Ended Queue)  

It uses a map of pointers to different blocks.  
```
Central Map: [ Block A   |   Block B ]  
                    |              |   
                    v              v   
Block A:  [ 2 | 5 | 26 ] Block B: [ 98 | 55 | 13 ]  
```

Behaviour: It can add new blocks at the beginning or at the end without moving the existing data. That is why it is fast on both ends (O(1)).  

 

### std::list (Doubly Linked List)  

There are no blocks. Each element is a node that lives anywhere in memory and points to the next and the previous.  

[ nullptr | 2 | * ] <-> [ * | 5 | * ] <-> [ * | 26 | nullptr ]  

Behaviour: You cannot jump directly to position 10. You must follow the pointers one by one (O(n)). But inserting only means changing a couple of pointers.  

 

## COMPARISON OF ASSOCIATIVE CONTAINERS  

Here the memory is not linear, but hierarchical.  

### std::map and std::set  

They are organized as a Binary Tree. Each node has a value (or key-value) and two “children”.  
```
[ Key: 26 ]  <-- Root   
    /            \   
[ Key: 5 ] [ Key: 48 ]  
```

Behaviour: They are always sorted. To search for 48, the program asks: “Is it greater than 26? Yes, go right”. This is very efficient for large sets (O(log n)).  

 

## ADAPTERS (The concept of “WRAPPING”)  

 

### std::stack and std::queue  

They do not have their own memory structure. By default, they are like a “box” that wraps a std::deque.  

### STACK:  
```
[ Input/Output ] <-- Only the top is visible  
       |   
       v   
[ Deque Block ] <-- The data is down here, protected.  
```

Behaviour: The stack tells the deque: “Only let me use your push_back and pop_back functions”. It hides everything else (like iterators) to ensure that nobody breaks the LIFO rule.  

## ITERATION AND DELETION METHODS  

| Method | Purpose |¿Who uses it? |
| :--- | :--- | :--- |
| `push_back()` | Adds to the end | vector, deque, list |

| `push_front()` | Adds to the beginning | deque, list | 

| `pop_back()` | Removes the last | vector, deque, list | 

| `pop_front()` | Removes the first | deque, list | 

| `insert()` | Inserts at a specific position | vector, deque, list, map, set | 

| `erase()` | Removes a specific element | All containers |
 

## ACCESS AND SEARCH METHODS  

| Method | Purpose |¿Who uses it? | 

|`at() o []` | Direct access by index | vector, deque | 

| `front()` | Looks at the first element | vector, deque, list | 

| `back()` | Looks at the last element | vector, deque, list | 

| `top()` | Looks at the top element | stack | 

| `find()` | Searches for a specific key | map, set | 

 
## CAPACITY AND STATE METHODS  

| Method | Purpose |¿Who uses it? |

| `size()` | How many elements there are | All | 

| `empty()` | Is it empty? (returns true/false) | All |

| `clear()` | Deletes all content | All  (except adaptaters) |

## SPECIAL CASES  

- `std::stack and std::queue have adapters and specific methods to reinforce their logic.`  

- `push() To insert data, equivalent to push_back.`  

- `pop() To remove data (in stack the last one and in queue the first one)` 

The STL also includes unordered associative containers (unordered_map, unordered_set), which were introduced in C++11. 

However, since the project uses C++98, only ordered associative containers (map, set) are available. 