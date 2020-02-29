+++
title = "Implement LRU Cache"
date = "2020-02-27T08:07:04-05:00"
description = ""
tags = [
    "linkedlist", 
	"hashmap",
]
categories = [
    "leetcode", 
    "interview prep",
]
draft  = false

+++


Problem: [146. LRU Cache](https://leetcode.com/problems/lru-cache/)

> Design and implement a data structure for [Least Recently Used (LRU) cache](https://en.wikipedia.org/wiki/Cache_replacement_policies#LRU). It should support the following operations: `get` and `put`.
>
> `get(key)` - Get the value (will always be positive) of the key if the key exists in the cache, otherwise return -1.  
> `put(key, value)` - Set or insert the value if the key is not already present. When the cache reached its capacity, it should invalidate the least recently used item before inserting a new item.
>
> The cache is initialized with a **positive** capacity.
>
> **Follow up:**
> Could you do both operations in **O(1)** time complexity?
>
> **Example:**
>
> ```
> LRUCache cache = new LRUCache( 2 /* capacity */ );
> 
> cache.put(1, 1);
> cache.put(2, 2);
> cache.get(1);       // returns 1
> cache.put(3, 3);    // evicts key 2
> cache.get(2);       // returns -1 (not found)
> cache.put(4, 4);    // evicts key 1
> cache.get(1);       // returns -1 (not found)
> cache.get(3);       // returns 3
> cache.get(4);       // returns 4
> ```

### Intuition

Looking at the requirements, we know that we need couple things to build this LRU cache:   
- Something to hold the cache 
- Something to keep track the least recently used items and evict them when reaching capacity

### Attempt #1 [Failed]

For holding key and value it's pretty obvious we could use a hashmap to achieve the O(1) get and put. However, a regular hashmap doesn't not have order or priority, which means we cannot keep track the least used items without introducing another attribute to represent when this item was last created or accessed. 

#### How to keep track of the least used items?

First thing came to my mind is to use an integer to represent the least used items, with 0 as the most recently used item To keep track of last used rank, we put 0 as least recently used rank for new item and  increment the rank by 1 for the rest of items to increase their ranks. When an item's rank exeeds the capacity, we remove it from the hashmap.

One obvious mistake in this solution is that we didn't account what to do with get or update operations. And there is no obvious way to recreate the rankings without some kind of sorting which is O(n*log(n)) time.

### Attempt #2 [Succeed] 

In the previous attempt we tried to order these hashmap items by creating an additional attribute to the item but couldn't find an easy way to order them by access time.  What if instead of creating additional attributes, we could find a data structure that both supports ordering, and map-like key value lookup. After some [googling](https://stackoverflow.com/a/663388) we found that `LinkedHashMap` and  `SortedMap` are good candidates to solve our problem.

#### Which one should we use? 

For `SortedMap` we still need to create an additional attribute to represent the least recently used rank, on top of that, each get and put now needs O(log(n)) time to complete.

`LinkedHashMap` maintains a natural ordering as items are added to the map, and, it has a nice method called `removeEldestEnrty` to help us evict the least used items. Let's take a deeper look at this class:

> ### LinkedHashMap
>
> ```java
> public LinkedHashMap(int initialCapacity,
>                      float loadFactor,
>                      boolean accessOrder)
> ```
>
> Constructs an empty LinkedHashMap instance with the specified initial capacity, load factor and ordering mode.
> Parameters:
> 	initialCapacity - the initial capacity
> 	loadFactor - the load factor
> 	accessOrder - the ordering mode - true for access-order, false for insertion-order
> Throws:
> IllegalArgumentException - if the initial capacity is negative or the load factor is nonpositive
>
> ### 

We can see that `LinkedHashMap` even allows us to order by access order, which is exactly we need. Now let's take a look at the method `removeEldestEntry` and how we can use it to evict the least used item when exceeding capacity:

> removeEldestEntry
>
> ```java
> protected boolean removeEldestEntry(Map.Entry<K,V> eldest)
> ```
>
> Returns `true` if this map should remove its eldest entry. This method is invoked by `put` and `putAll` after inserting a new entry into the map. It provides the implementor with the opportunity to remove the eldest entry each time a new one is added. This is useful if the map represents a cache: it allows the map to reduce memory consumption by deleting stale entries.
>
> Sample use: this override will allow the map to grow up to 100 entries and then delete the eldest entry each time a new entry is added, maintaining a steady state of 100 entries.
>
> ```java
>      private static final int MAX_ENTRIES = 100;
> 
>      protected boolean removeEldestEntry(Map.Entry eldest) {
>         return size() > MAX_ENTRIES;
>      }
> ```



#### Put `LinkedHashMap` into action

```java
class LRUCache extends LinkedHashMap<Integer,Integer> {
    int cap;
    public LRUCache(int capacity) {
        // initialize linked hashmap ordered by access time, 0.75F is the load factor
        super(capacity,0.75F, true);
        cap = capacity;

    }
    
    public int get(int key) {
       return super.getOrDefault(key, -1);
    }
    
    public void put(int key, int value) {
        super.put(key, value);
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry entry){
        // evict least used item when cache size is greater than capacity, note this method is invoked by put()
        return this.size() > cap;
    }
}
```

### Attempt #3 [Works, but slow]

`LinkedHashMap` is great, but what if you never used it before or weren't allowed to google during an interview? Well we know we need some list to maintain the order of the cache, and we need to constantly moving elements in the front of the list and popping item at the end of the list. With this in mind, we can build a simple (but inefficient) LRU cache with list and tuple: we store key and value in the tuple and insert (or remove) these tuples to the list as we add (or evict) items. 

Each put() and get() takes O(n) time because it involes searching the entire list to locate the item and to shift items around for add() and remove() calls. 

```java
class LRUCache {
    List<Tuple> cache;
    int cap;
    public LRUCache(int capacity) {
        // initialize linked hashmap ordered by access time, 0.75F is the load factor
        cap = capacity;
        cache = new ArrayList<>();
    }
    
    public int get(int key) {
       for (int i = 0; i < cache.size(); i++) {
           if (cache.get(i).x == key) {
               // we remove current tuple from the list and insert it in the front of list
               Tuple t = new Tuple(key, cache.get(i).y);
               cache.remove(i);
               cache.add(0, t);
               return t.y;
           }
       }

       return -1;
    }
    
    public void put(int key, int value) {
        // check if key exists in cache, update tuple by removing and inserting new tuple in the front
        for (int i = 0; i < cache.size(); i++) {
            if (cache.get(i).x == key) {
                Tuple t = new Tuple(key, value);
                cache.remove(i);
                cache.add(0, t);
                return;
            }
        }
        Tuple t = new Tuple(key, value);
        cache.add(0, t);
        if (cache.size() > cap) {
            cache.remove(cache.size() - 1);
        }

    }

    public class Tuple {
        public final Integer x;
        public final Integer y;
        public Tuple(Integer x, Integer y) {
            this.x = x;
            this.y = y;
        }
    }
}
```

### Attempt #4 [Works, but not what we wanted]

How do we achieve O(1) read and write while maintaining access order? A map can be used to achieve O(1) lookup, however, from the previous attempts we know we need to rearrange the items each time we do put() or get(). This implies list is out of the picture since add(i, item) and remove(i) is O(n). 

#### Which data structure maintains order and has O(1) add and remove?

Answer is LinkedList, to be more specific, Doublely LinkedList since we only need reference of the node itself for removal. Let's implement this. 

```java
class LRUCache {
    LinkedList<Tuple> lruOrder;
    HashMap<Integer, Tuple> cache;
    int cap;
    public LRUCache(int capacity) {
        cap = capacity;
        lruOrder = new LinkedList<>();
        cache = new HashMap<>();
    }

    public int get(int key) {
       if (cache.containsKey(key)){
           Tuple data = cache.get(key);
           lruOrder.remove(data);
           lruOrder.addFirst(data);
           return data.y;
       }
       return -1;
    }

    public void put(int key, int value) {
        if (cache.containsKey(key)){
            Tuple data = cache.get(key);
            lruOrder.remove(data);
            Tuple newData = new Tuple(key, value);
            lruOrder.addFirst(newData);
            cache.put(key, newData);
        } else {
            Tuple data = new Tuple(key, value);
            lruOrder.addFirst(data);
            cache.put(key, data);
            if (lruOrder.size() > cap) {
                Tuple toRemove = lruOrder.removeLast();
                cache.remove(toRemove.x);
            }
        }
    }

    public class Tuple {
        public final Integer x;
        public final Integer y;
        public Tuple(Integer x, Integer y) {
            this.x = x;
            this.y = y;
        }
    }
}
```

Superisingly this is only slightly faster then our attempt #3 which is O(n) time, which meas somethings is not O(1). Upon examining the LinkedList APIs, I found that `remove(Object o)` actully was in O(n) time since according to the documentation, it removes the **first** **occurrence** of the specified element from this list, if it is present.

### Attempt #5 [succeeded, finally O(1) time]

We know the problem is the implemenation of  collection's LinkedList remove() method. So... why not implementing the doubly LinkedList ourselves? LinkedList is a simple concept and our custom doubly linkedlist only needs to achieve the following:

- maintains order (i.e an ordered list)
- addFirst() in O(1) time
- remove(Node node) in O(1) time
- size() in O(1) time

Note that we are constantly inserting to the front of the linkedlist and removing at the end of the linkedlist, it's better to define a dummy head and a dummy tail to make the implementation easier. 

```java
   HashMap<Integer, Node> cache;
   int cap;
   Node head;
   Node tail;
   int size;
   public LRUCache(int capacity) {
       cap = capacity;
       cache = new HashMap<>();

       head = new Node();
       tail = new Node();
       head.next = tail;
       tail.prev = head;
       size = 0;
   }
   
    public class Node {
        int key;
        int value;
        Node prev; // previous pointer
        Node next; // next pointer
        public Node(){}
        public Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }

    private void addFirst(Node node) {
        // inserting new node after dummy head 
        node.prev = head;
        node.next = head.next;
        
        // change dummy head next pointer and original head previous pointer
        head.next.prev = node;
        head.next = node;
        size ++;
    }

    private void remove(Node node) {
        // get previous and next node of current node
        Node prevNode = node.prev;
        Node nextNode = node.next;
        
        // link previous node and next node directly 
        prevNode.next = nextNode;
        nextNode.prev = prevNode;
        size --;
    }
}

```

Add get() and put():

```java
    public int get(int key) {
       if (cache.containsKey(key)){
           Node node = cache.get(key);
           remove(node);
           addFirst(node);
           return node.value;
       }
       return -1;
    }

    public void put(int key, int value) {
        if (cache.containsKey(key)){
            Node node = cache.get(key);
            remove(node);
            Node newNode = new Node(key, value);
            addFirst(newNode);
            cache.put(key, newNode);
        } else {
            Node newNode = new Node(key, value);
            addFirst(newNode);
            cache.put(key, newNode);
            if (size > cap) {
                // using previous of tail to get the last node
                Node toRemove = tail.prev;
                remove(toRemove);
                cache.remove(toRemove.key);
            }
        }
    }
```

Finally we have implemented a LRU cache that performs in O(1) time and uses O(n) space where n is the capacity of the cache. By breaking down the requirements, we successfully improved our O(n) implementation to O(1) and we even implemented a doubly linkedlist along the way. 