---
layout: post
title: Let's Code A Queue With a Circular Linked List
excerpt_separator: <!--more-->
---

While reading the textbook [Algorithms](https://algs4.cs.princeton.edu/home/) I encountered a problem that was too good to ignore. The goal is to write a queue that uses a circular linked list.<!--more--> On first glance, that goes against everything I know about how queues and linked lists work. So let me show you my thought process while working through it.

Here's the prompt:

> **1.3.29** Write a Queue implementation that uses a circular linked list, which is the same as a linked list except that no links are null and the value of last.next is first whenever the list is not empty. Keep only one Node instance variable (last).

### The Constraints

Since the solution must be a linked list, I need a way to build a collection of nodes. Each node must reference the next node in the list, and each node must hold some value. Since the list must also be a queue, then it uses the "first in first out" rule. That is, the newest item is always last, and the oldest item is always first. 

But to make things fun, the problem says, "No links in the list can be null." You might have had a similar thought as me. I can "link" the first node to a null value, that way I always know where the list begins. Unfortunately for this problem, that isn't an option. Additionally, the last node must always point to the first node. This only makes sense, because the goal is to make the queue circular. 

But the final and perhaps most interesting constraint is that the queue can only hold the last node. 

### Initial Thoughts

To begin, here's a sample of the client code I used to test the `CircularQueue`:

```java
import edu.princeton.cs.algs4.StdOut;

public class ex1329 {
    /* 
     * Sample run configuration:
     * $ java ex1329 some text to use as a test
     */

    public static void main(String[] args) {
        var queue = new CircularQueue<String>();

        for(String s : args) {
            queue.enqueue(s);
        }

        int i = 0;
        for(String s : queue) {
            i++;
            StdOut.printf("Iteration %s: %s\n", i, s);
        }
    }
}
```

My first thought is that I would need to loop through the entire list in order to dequeue the first value. If I can only store the value of last, then I would have to start at last to get to first.

On my first attempt, I started by creating a standard queue. Where normally I would store two nodes, first and last, I only stored last in order to meet the constraints: 

```java
public class CircularQueue<Item> {
    private Node last;
    private int count;

    private class Node {
        Node next;
        Item item;
    }

    public void enqueue(Item item) {
        Node current = last;
        last = new Node();
        last.item = item;
        if (count == 0) first = last;
        else current.next = last;
        count++;
    }

    public Item dequeue() {
        Item item = first.item;
        first = first.next;
        if (count == 0) last = null;
        count--;
        return item;
    }
}
```

When I ran the code, I received a compilation error due to the line `first = last`. Whoops. I can't exactly point an object to another object if it doesn't exist. At this point, it was clear that I needed to work through the problem more slowly.

In order to simplify the process, I decided to focus only on the code responsible for adding the first item. I would ignore everything else for the time being. 

So I ran through the constraints again. I'm only allowed to store the value of last. In this implementation, last always points to first. 

So I drew out some sketches to simulate adding the first element to the queue. If the list is empty, then last is null. This also means that after the first iteration, last and first are the same value. So that means that `last.next` should point to itself after the first item has been added. So the enqueue method has to account for the first iteration so it can handle this edge case:

```java
if(last == null) {
    last = new Node();
    last.item = item;
    last.next = last;
}
```

After the first iteration, last should always point to first. So, I created a new node called temp. I reassigned last to temp at the end of the code block. Because of this, `temp.next` should point to first. And first is always stored as `last.next`. So now last is linked to first. So, we can just reassign last to temp, and we're done.

```java
Node temp = new Node();
Node first = last.next;
temp.item = item;
temp.next = first;
last = temp;
```

Not exactly. When I ran the code, the console printed only the first argument from my test code. On each iteration, the queue reassigned the first arg to the same node. After some time, I realized that I failed to link first to the next node in the list. 

My first thought was that I would need an additional code block to account for the second iteration. Already, I was calling back to my original thought. I would need to loop through the entire list each time in order to achieve this.

### The Solution

In most queues, the value of first is required because that's the starting point of the list. However, since this is a circular queue, last always points to first. So we always have direct access to first!

`last.next = temp;` is the key. "Temp" is effectively the new "last" on each iteration. Since `last.next` points to it, we're telling the program to keep the object in memory until the program exits, or until some other logic reassigns it.

To dequeue the list, I created a new Node called "first" and assigned it to `last.next`, because in this implementation, last always points to first. The list must reassign last to null on the final iteration.

Finally, it's worth mentioning that although I do have a node called "first," since it's not stored inside the class, I'm still meeting the constraints of the problem. "First" is just a temporary variable, only accessible from within the scope of the `enqueue` and `dequeue` methods. 

Here's the final class with an iterator so that I can loop through it:

```java
import java.util.Iterator;

public class CircularQueue<Item> implements Iterable<Item> {
    private Node last;
    private int count;

    private class Node {
        Node next;
        Item item;
    }

    public int getCount(){
        return count;
    }

    public void enqueue(Item item) {
        if(last == null) {
            last = new Node();
            last.item = item;
            last.next = last;
        } else {
            Node temp = new Node();
            Node first = last.next;
            temp.item = item;
            temp.next = first;
            last.next = temp;
            last = temp;
        }
        count++;
    }

    public Item dequeue() {
        Node first = last.next;
        Item item = first.item;
        last.next = first.next;
        if(count <= 1) last = null;
        count--;
        return item;
    }

    @Override
    public Iterator<Item> iterator() {
        return new CircularQueueIterator();
    }

    private class CircularQueueIterator implements Iterator<Item> {
        Node first = last.next;
        int i = 0;

        @Override
        public boolean hasNext() {
            return i < count;
        }

        @Override
        public Item next() {
            Item item = first.item;
            first = first.next;
            i++;
            return item;
        }
    }
}
```
