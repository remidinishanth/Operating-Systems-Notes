Awesome introduction:

https://www.baeldung.com/lock-free-programming#linked-queue-multiple-threads

Assume there is a linked list

![image](https://user-images.githubusercontent.com/19663316/207507990-adabc69b-4a08-4ada-8547-6a722831ba40.png)

If multiple threads are trying to append an element to the last, there might be race condition:

![image](https://user-images.githubusercontent.com/19663316/207508067-89296ce6-5f4a-4f54-92cd-07d6614bfeea.png)

There is a race based on how we perform the order of operations A, B, C, and D

![image](https://user-images.githubusercontent.com/19663316/207508095-4de6083c-02e6-4864-90d3-851ea3879ced.png)

Using compare and swap atomic operation

```java
class AtomicReference<T> {
 
  T ref;
 
  void set(T newRef) { ref = newRef; }
 
  T get() { return ref; }
 
  // atomic instruction - hardware locks
  //lock cmpxchg
  boolean compareAndSet(T expect, T update) {
    if (expect == ref)
    {
      ref = update;
      return true;
    }
  }
  ```


Basic lock-free queue in Java

```java
public class NonBlockingQueue<T> {

    private final AtomicReference<Node<T>> head, tail;
    private final AtomicInteger size;

    public NonBlockingQueue() {
        head = new AtomicReference<>(null);
        tail = new AtomicReference<>(null);
        size = new AtomicInteger();
        size.set(0);
    }
}
```


Implementation of the Node class:

```java
private class Node<T> {
    private volatile T value;
    private volatile Node<T> next;
    private volatile Node<T> previous;

    public Node(T value) {
        this.value = value;
        this.next = null;
    }

    // getters and setters 
}
```

Lock-free add:

```java
public void add(T element) {
    if (element == null) {
        throw new NullPointerException();
    }

    Node<T> node = new Node<>(element);
    Node<T> currentTail;
    do {
        currentTail = tail.get();
        node.setPrevious(currentTail);
    } while(!tail.compareAndSet(currentTail, node));

    if(node.previous != null) {
        node.previous.next = node;
    }

    head.compareAndSet(null, node); // for inserting the first element
    size.incrementAndGet();
}
```

Lock-free get:

```java
public T get() {
    if(head.get() == null) {
        throw new NoSuchElementException();
    }

    Node<T> currentHead;
    Node<T> nextNode;
    do {
        currentHead = head.get();
        nextNode = currentHead.getNext();
    } while(!head.compareAndSet(currentHead, nextNode));

    size.decrementAndGet();
    return currentHead.getValue();
}
```

https://livebook.manning.com/book/c-plus-plus-concurrency-in-action/chapter-7/73
