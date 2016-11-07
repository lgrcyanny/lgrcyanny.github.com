title: Scala Collections
date: 2016-11-07 10:09:10
tags:
    - Scala
    - Learning
categories:
      - Learning
---

In scala there are many fancy collections with great utilities. Here are some key notes for scala collections which did a great help to me.
<!--more-->

# Collections Hierarchy
![collection hierarchy](http://ww1.sinaimg.cn/mw1024/761b7938jw1f9jbrdeugkj20ku0q041j.jpg)

# Description
Collections have two kinds:

	- mutable collections
	- immutable collections

## Immutable Collections	

- Lists are finite immutable sequences. They provide constant-time access to their first element as well as the rest of the list

- A stream is like a list except that its elements are computed lazily. Because of this, a stream can be infinitely long. for example:

```scala
	 val s = 1 #:: 2 #:: 3 #:: Stream.empty
	 def fib(m: Int, n: Int): Stream[Int] = m #:: fib(n, m + n)
```

- Vectors are a new collection type in Scala 2.8 that give efficient access to elements beyond the head.
    - Access to any elements of a vector take only “effectively constant time,” as defined below.
    - shallow trees
    - when only one level, store 32 elements in an array
    - if lager than 32, grow to 2 levels, each node in level 2 has 32 elements, and level 1 store 32 pointers, now level 2 has 2^10 elements
    - level 3 has 2^ 15 elements
    - to access an element, the complexity is log32(N)
    - Vector have very decent random access performance
    - The default implementation to immutable IndexedSeq
    
- Stack
	- first-in-last-out
	- you can use ArrayBuffer to implement a Stack 
	```scala
		val emtpyStack = Stack.empty
		val hasOne = emptyStack.push(1)
	```
- Immutable queues

- Ranges

	```scala
	val range = 1 to 10 
	```
	
- Hash Tries
    - Hash tries4 are a standard way to implement immutable sets and maps efficiently.
    
- RedBlackTrees
    - Red-black trees are a form of balanced binary trees where some nodes are designated “red” and others “black.”
    - TreeSet
    - TreeMap
    - default implementation for SortedSet
    
-  Immutable bit sets
![bit set example](http://ww2.sinaimg.cn/mw690/761b7938jw1f9jc4ydow0j20r208276p.jpg)
    - Operations on bit sets are very fast. Testing for inclusion takes constant time. Adding an item to the set takes time proportional to the number of Longs in the bit set’s array, which is typically a small number.
    
- ListMap
    -  The only possible difference is if the map is for some reason constructed in such a way that the first elements in the list are selected much more often than the other elements.

## Mutable Collections

- Array buffer: operations simply access and modify the underlying array.

-  List Buffer: A list buffer is like an array buffer except that it uses a linked list internally instead of an array.
    - you plan to convert the buffer to a list once it is built up, use a list buffer instead of an array buffer.
    
- StringBuilder:  a string builder is useful for building strings

- LinkedList: Linked lists are mutable sequences that consist of nodes that are linked with next pointer
    - use  LinkedList.empty.isEmpty for empty list
    - linked lists are best operated on sequen- tially. In addition, linked lists make it easy to insert an element or linked list into another linked list.

- Double Linked List: The main benefit of that additional link is that it makes element removal very fast

- Mutable List:
    - A MutableList consists of a single linked list together with a pointer that refers to the terminal empty node of that list.
    - The default implementation for LinearSeq

- Queue
    - the dequeue method will just remove the head element from the queue and return it

- Array Sequences
    - A class for polymorphic arrays of elements that's represented internally by an array of objects
    - Array sequences are mutable sequences of fixed size that store their elements internally in an Array[AnyRef]

- Stack
    - It works exactly the same as the immutable version except that modifications happen in place

- ArrayStack
    - ArrayStack is an alternative implementation of a mutable stack, which is backed by an Array that gets resized as needed
    - It provides fast indexing and is generally slightly more efficient for most operations than a normal mutable stack.

- HashTable
    - A hash table stores its elements in an underlying array, placing each item at a position in the array determined by the hash code of that item.
    - As a result, the default mutable map and set types in Scala are based on hash tables.
    - HashMap, HashSet implements with hash tables in array
    - Iteration over a hash table is not guaranteed to occur in any particular order.
        - To get a guaranteed iteration order, use a linked hash map or set instead of a regular one.
        - Iteration over such a collection is always in the same order that the elements were initially added.

- Weak Hash Maps
    - A weak hash map is a special kind of hash map in which the garbage collector does not follow links from the map to the keys stored in it
    - This means that a key and its associated value will disappear from the map if there is no other reference to that key
    - Weak hash maps are useful for tasks such as caching, where you want to re-use an expensive function’s result if the function is called again on the same key
    - Weak hash maps in Scala are implemented as a wrapper of an underlying Java implementation, java.util.WeakHashMap.

- Concurrent Maps
    - A concurrent map can be accessed by several threads at once.
    -  Currently, its only implementation is Java’s java.util.concurrent.ConcurrentMap

- BitSet
    - Mutable bit sets are slightly more efficient at updating than immutable ones, because they don’t have to copy around Longs that haven’t changed.