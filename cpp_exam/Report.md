# Binary Search Tree

In this report I'm going to describe my implementation of the Binary Search Tree class (BST) using C++. ([Wikipedia - Binary search tree](https://en.wikipedia.org/wiki/Binary_search_tree))

In this version each node contains two arguments: a key of template type K, which uniquely identifies the node, and a value of template type V, which is the actual content of the node.
For the sake of simplicity, in the following lines we'll set all values equal to zero.

Let's illustrate by some examples how this class works.
First of all we create a BST object and then we insert some nodes using the method `void AddLeaf(const K key, V val)`.
~~~~
	int v[9] = { 4,2,1,3,9,5,7,6,8 };
	BST<int,double> bst;
	for (int i = 0; i < 9; ++i) { bst.AddLeaf(v[i], 0);	}
~~~~
The tree created in this way is the following.
```
         4 
      /     \ 
     2       9
    / \     /
   1   3   5
             \
              7
             / \
            6   8
```
We can visualize a tree using the public method `void PrintInOrder()`, which shows us the "family" of each node, printing according to the order of the keys.
~~~~
bst.PrintInOrder();
~~~~
Output:
~~~~
Node -> 1       dad -> 2        SX -> NULL      DX -> NULL
Node -> 2       dad -> 4        SX -> 1         DX -> 3
Node -> 3       dad -> 2        SX -> NULL      DX -> NULL
Node -> 4       dad -> root     SX -> 2         DX -> 9
Node -> 5       dad -> 9        SX -> NULL      DX -> 7
Node -> 6       dad -> 7        SX -> NULL      DX -> NULL
Node -> 7       dad -> 5        SX -> 6         DX -> 8
Node -> 8       dad -> 7        SX -> NULL      DX -> NULL
Node -> 9       dad -> 4        SX -> 5         DX -> NULL
~~~~
Other usefull implemented methods to visualize the structure of the tree are:
* `void PrintFamily(const K key)`: as above but only refered to the node with the indicated key;
* `void PrintRoot()`: it prints the key of the root node.

It's possible to pass the object directly to `std::cout`, printing out a list with all the nodes (ordered by keys) with the format `{key : value}`.

If we want to get the value contained in a node with a specific key we can just type `bst[key]`.

Classes `BST<K,V>::Iterator` and `BST<K,V>::ConstIterator` allows us to explore the tree using iterators.
The following code shows how it works:
~~~~
for (BST<int, double>::Iterator it = bst.begin(); it != bst.end(); ++it)
{
	std::cout << *it << " --> " << bst[*it] << std::endl;
}
~~~~
and it's equivalent to
~~~~
for (auto& x : bst) 
{ 
    std::cout << x << " --> " << bst[x] << std::endl; 
}
~~~~
If we wish to stop the iteration before a certain key, we could use the method `Iterator find(K key)` it this way
~~~~
for (auto it = bst.begin(); it != bst.find(4); ++it)
{
	std::cout << *it << " --> " << bst[*it] << std::endl;
}
~~~~

An important method in the class is `void RemoveNode(const K key)`, which allows us to remove from the tree the node with the indicated key.
Every time a node is removed, if it has only one "child" its place is taken by this, while if it has two children it's replaced by the node with the smallest key in the right subtree.
For example with this line
~~~~
bst.RemoveNode(4);
~~~~
our tree becomes
```
         5 
      /      \ 
     2         9
    / \       /
   1   3     7
           /   \
          6      8
```
Let's now illustrate the implementation of the `void Balance()` method.
A balanced tree is a tree that has the lowest possible height.

In implementing this method I assumed that there was a big difference between the memory space occupied by the key (which I assumed very small) and that occupied by the value of the node (which I imagined potentially very large).
Consequently, this algorithm allows to balance the tree without copying any value contained in the nodes but simply reorganizing the tree structure by modifying the connections from one node to another.

Suppose we want to balance the following tree (which it's actually a linked list):
```
1
  \
    2
      \
        3
          \
            4
             \
               5
```

The first step is made by the usage of the private function `MakeVector`, which returns an ordered vector with all the keys of the tree.
~~~~
[1,2,3,4,5]
~~~~
Then we create another vector by rearranging the content of the newly created one using the private method `MakeOrder`.
The order we get is the one that should be followed to insert the nodes in order to obtain a balanced tree.
~~~~
[3,2,1,4,5]
~~~~
The crucial step is accomplished by the private method ` Connect`, which replicates the process of inserting the nodes in the BST (following the "balanced order" just found) and memorizing all the connections that would be created.
After that, we just have to remodel our object on the newly found structure.

At the end we get this
```
       3
    /     \    
   2        4
 /           \
1             5
```
Maintaining the tree balanced is very important for ensuring speed in access to individual nodes.
If we try to run this code
~~~~
for (int n = 10; n < pow(10, 5); n *= 10)
	{
		BST<int, double> bst;
		for (int i = 0; i < n; ++i) { bst.AddLeaf(i, 0); }

		auto t1 = std::chrono::high_resolution_clock::now();
		for (int i = 0; i < n; ++i) { bst[i]; }
		auto t2 = std::chrono::high_resolution_clock::now();

		bst.Balance();

		auto q1 = std::chrono::high_resolution_clock::now();
		for (int i = 0; i < n; ++i) { bst[i]; }
		auto q2 = std::chrono::high_resolution_clock::now();

		std::chrono::duration<double> T = t2 - t1;
		std::chrono::duration<double> Q = q2 - q1;
	}
~~~~
we get these values for T (unbalanced) and Q (balanced)

| log(N) |     1     |      2     |      3      |      4      |
|:------:|:---------:|:----------:|:-----------:|:-----------:|
|    T   | 1.437e-06 | 6.8272e-05 | 6.90659e-03 | 4.80687e-01 |
|    Q   |  8.66e-07 |  7.005e-06 |  6.5145e-05 | 9.69973e-04 |




