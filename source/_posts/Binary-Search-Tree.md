---
title: Binary Search Tree
date: 2022-07-18 21:59:38
tags: [Data Structure]
category: [Data Structure, Binary Search Tree & Balance Tree]
---
\ \ BST(Binary Search Tree) is a widely used data structure, which can reduce the time complex of opration to average $O(log n)$.
Many problem can be efficient solved with BST, e.g. whether an element existed, find the N-th biggest/smallest element, etc.
We can implement other data structure with BST, such as set, map.
<!--more-->

## Definition  

\ \ BST is a binary tree data structure which has the following properties:  
- An empty tree is a BST.   
- The left subtree of root node is a BST and **all** nodes in subtrer **less** than root.  
- The right subtree of root is also a BST and **all** nodes in subtree **greater** than root.

\ \ A BST may look like below:
![BST](/images/Binary_Search_Tree/bst.svg)

## Impementation
\ \ Here I post my implementation of BST, the complete code can be find [here](https://github.com/HikawaRin/data_structure/blob/master/binary_search_tree_and_balanced_tree/binary_search_tree.cc)

### Node Structure 
\ \ The tree node structure defined as below:
```cpp
struct { 
  int e[3]; // Store the index of left son, right son and parent
  int key; // The key of that node
  int cnt; // The number of duplicate key
  int size; // The number of nodes contains in this subtree
}nodes[tree_size];
static int node_cnt{1}; // The index of unused node, 0 reference to NULL node
static int root{0}; // The root of bst, initial to not exist

// Help function
int *ls(int n) { return &(nodes[n].e[0]); } // Get left son
int *rs(int n) { return &(nodes[n].e[1]); } // Get right son
int *p(int n) { return &(nodes[n].e[2]); } // Get parent
```
\ \ Notice that we stored the parent index in each node, that would be very useful in delete and range search.

### Query Key
\ \ Query a key in BST is just like binary search.
We start at BST root, if key smaller than root key, we let the left son of root be the new root; else if key greater than root key, we let the right son be the new root.
Recursive the step above until we find the key or reached null node.  
\ \ Here\`s the code:
```cpp
int query(int key) { 
  int it = root;
  while (it != 0) { 
    int cmp = key - nodes[it].key;
    if (cmp < 0) { it = *ls(it); }
    else if (cmp > 0) { it = *rs(it); }
    else { break; }
  }
  return it;
}
```

### Insert  
\ \ Each time we insert a key into BST we need first query the key, if node with that key exist, we simply increase it\`s count; 
Otherwise we need construct a new node with that key and set it to the proper position.  
\ \ With the help of pointer, we can perform insert in **one pass**.  
```cpp
void insert(int key) { 
  int *ip = &root, pa = 0; // we can update new node`s index in BST with the help of pointer
  while (*ip != 0) { 
    ++nodes[*ip].size; // each insert would increase the subtree size by 1
    // travel the BST, update iterator and parent info
    int cmp = key - nodes[*ip].key;
    if (cmp < 0) { pa = *ip; ip = ls(pa); }
    else if (cmp > 0) { pa = *ip; ip = rs(pa); }
    else {
      // duplicate key, increase the node counter
      ++nodes[*ip].cnt; return;
    }
  }
  // now the ip pointer to null node, but ip itself is the last node`s e[0]/e[1], 
  // which need hold our new node`s index
  *ip = node_cnt++; // fetch an unused index
  // Update node information
  nodes[*ip].key = key; nodes[*ip].cnt = 1; nodes[*ip].size = 1;
  *p(*ip) = pa; *ls(*ip) = 0; *rs(*ip) = 0;
}
```

### Delete
\ \ Similar to insert, first we need to locate the node, which hold the key. 
If the counter greater than 1, we just decrease the counter. 
But if the counter become 0, we need perform a node delete, and there are three case:  
1. the node is a leaf  
just delete it.  
2. one son of the node is null  
replace that node with the not null son, then delete it.
3. Both son of the node is not null  
In this scenario we can not delete that node without broken BST. 
We will need the remove successor strategy, 
which find the successor of the node, swap node information and delete the successor node.  
Why it works, because in BST the left son of successor must be null, if it\`s not null, then the left son will be the really successor.  
Let\`s take a look at code, I assume the key to delete exist in BST:
```cpp
void remove(int key) { 
  int *ip = &root; // Again make our life easier with pointer
  while (*ip != 0) { 
    --nodes[*ip].size; // the subtree would decrease by 1
    int cmp = key - nodes[*ip].key;
    if (cmp < 0) { ip = ls(*ip); }
    else if (cmp > 0) { ip = rs(*ip); }
    else { --nodes[*ip].cnt; break; }
  }
  if (nodes[*ip].cnt == 0) { 
    // need remove node
    if (*ls(*ip) == 0) { 
      // left son is null
      if (*rs(*ip) != 0) { *p(*rs(*ip)) = *p(*ip); }

      *ip = *rs(*ip); // like insert, ip is the parent node`s e[0]/e[1]
    } else if (*rs(*ip) == 0) { 
      // right son is null
      *p(*ls(*ip)) = *p(*ip);

      *ip = *ls(*ip); 
    } else { 
      // Third case
      // find successor
      int *sip = rs(*ip);
      while (*ls(*sip) != 0) { sip = ls(*sip); }

      // swap information
      nodes[*ip].key = nodes[*sip].key; nodes[*ip].cnt = nodes[*ip].cnt;
      // decrease subtree by successor counter
      ip = rs(*ip);
      while (ip != sip) { nodes[*ip].size -= nodes[*sip].cnt; ip = ls(*ip); }
      
      if (*rs(*ip) != 0) { *p(*rs(*ip)) = *p(*ip); }
      *ip = *rs(*ip);
    }
  }
}
```
With the help of pointer and parent information, we can remove node in **one pass**, just like doubly linked list.

### Find Rank
If key greater than node key, the left subtree must have lower rank, so add up left subtree`s size to rank. 
This method works even the key not exist in BST.
```cpp
int query_rank(int key) { 
  int rank = 1, it = root;
  while (it != 0) { 
    int cmp = key - nodes[it].key;
    if (cmp > 0) { 
      rank += (nodes[*ls(it)].size + nodes[it].cnt);
      it = *rs(it); 
    } else { it = *ls(it); }
  }
  return rank;
}
```

### Find node with rank
If left subtree\`s size smaller than rank, than rank cannot be in left subtree, decrease rank by the size of left subtree.
If rank in root, it cannot greater than root node counter now.
```cpp
int query(int rank) { 
  int it = root;
  while (it != 0) { 
    if (rank > nodes[*ls(it)].size) { 
      rank -= (nodes[*ls(it)].size + nodes[it].cnt);
      if (rank <= 0) { return it; }
      it = *rs(it);
    } else { it = *ls(it); }
  }
  return -1; // rank bigger than BST size, interrupt directly
}
```

### Get Previous and Successor
We can use parent information to find the previous or successor.
But there already exist a silver bullet.
```cpp
// Previous
// the query_rank will ignore the duplicate key, so decrease it by 1 to get the previous key rank
// query that rank we can get the previous node
nodes[query(query_rank(key)-1)].key;

// Successor
// Simillar to previous, this time we first take the rank of (key + 1) that will return the rank of successor node 
// query that rank we can get the successor node
nodes[query(query_rank(key+1))].key;
```
