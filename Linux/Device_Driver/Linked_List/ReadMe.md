# Linux Device Driver - Linked List in Linux Kernel

## Overview

The Linux kernel provides a built-in implementation of a **circular doubly linked list**.

Instead of implementing custom linked list structures, kernel developers use:

```c
#include <linux/list.h>
```

Benefits:

* Reusable kernel implementation
* Efficient insertion and deletion
* Widely used throughout kernel subsystems
* Supports dynamic data structures

---

# What is a Linked List?

A linked list consists of nodes connected using pointers.

Traditional implementation:

```c
struct my_node
{
    int data;

    struct my_node *prev;
    struct my_node *next;
};
```

Kernel implementation embeds a `list_head` structure.

```c
struct my_node
{
    struct list_head list;

    int data;
};
```

---

# Linux Kernel Linked List

Linux uses a **Circular Doubly Linked List**.

```text
+------+     +------+     +------+
|Node1 |<--->|Node2 |<--->|Node3 |
+------+     +------+     +------+
    ^                           |
    |___________________________|
```

Characteristics:

* Circular
* Doubly linked
* No NULL termination
* Head node always exists

---

# Header File

```c
#include <linux/list.h>
```

Kernel implementation:

```c
struct list_head
{
    struct list_head *next;
    struct list_head *prev;
};
```

---

# Creating a Linked List Node

User-defined structure:

```c
struct my_list
{
    struct list_head list;
    int data;
};
```

The `list_head` member links the node into the kernel linked list.

---

# Creating the List Head

Before creating nodes, create a list head.

## LIST_HEAD()

```c
LIST_HEAD(my_linked_list);
```

Example:

```c
LIST_HEAD(etx_linked_list);
```

Internally:

```c
struct list_head etx_linked_list =
{
    &etx_linked_list,
    &etx_linked_list
};
```

When:

```text
head->next == head
head->prev == head
```

The list is empty.

---

# Initializing a Node

For dynamically or statically allocated nodes:

```c
INIT_LIST_HEAD(&node.list);
```

Example:

```c
struct my_list node;

INIT_LIST_HEAD(&node.list);

node.data = 10;
```

This initializes:

```text
node.list.next = node.list
node.list.prev = node.list
```

---

# Adding Nodes

## Add After Head

Function:

```c
list_add(
    &node.list,
    &etx_linked_list
);
```

Effect:

```text
HEAD
 |
 v
NODE
 |
 v
OLD_FIRST_NODE
```

Used for:

```text
Stack (LIFO)
```

---

## Add Before Head

Function:

```c
list_add_tail(
    &node.list,
    &etx_linked_list
);
```

Effect:

```text
HEAD
 |
 v
...
 |
 v
NEW_NODE
 |
 v
HEAD
```

Used for:

```text
Queue (FIFO)
```

---

# Deleting Nodes

## list_del()

Removes node from list.

```c
list_del(&node.list);
```

Important:

```text
Removes linkage only
Memory is NOT freed
```

---

## list_del_init()

Deletes and reinitializes node.

```c
list_del_init(&node.list);
```

Node becomes standalone again.

---

# Replacing Nodes

## list_replace()

Replace old node with new node.

```c
list_replace(
    &old.list,
    &new.list
);
```

---

## list_replace_init()

Replace and reinitialize old node.

```c
list_replace_init(
    &old.list,
    &new.list
);
```

---

# Moving Nodes

## list_move()

Move node after head.

```c
list_move(
    &node.list,
    &etx_linked_list
);
```

---

## list_move_tail()

Move node before head.

```c
list_move_tail(
    &node.list,
    &etx_linked_list
);
```

---

# Rotating List

## list_rotate_left()

Rotate list left.

```c
list_rotate_left(
    &etx_linked_list
);
```

Example:

Before:

```text
HEAD -> A -> B -> C
```

After:

```text
HEAD -> B -> C -> A
```

---

# Testing List State

## list_empty()

Check whether list is empty.

```c
if(list_empty(&etx_linked_list))
{
    printk("Empty\n");
}
```

Returns:

```text
1 -> Empty
0 -> Not Empty
```

---

## list_is_last()

Check whether node is last.

```c
list_is_last(
    &node.list,
    &etx_linked_list
);
```

---

## list_is_singular()

Check whether list contains only one entry.

```c
list_is_singular(
    &etx_linked_list
);
```

---

# Splitting a List

## list_cut_position()

Split one list into two.

```c
list_cut_position(
    &new_list,
    &old_list,
    &entry->list
);
```

Example:

Before:

```text
HEAD
 |
 A -> B -> C -> D
```

After:

```text
LIST1:
A -> B

LIST2:
C -> D
```

---

# Joining Lists

## list_splice()

Join two linked lists.

```c
list_splice(
    &list2,
    &list1
);
```

Result:

```text
LIST1 + LIST2
```

Used commonly for:

* Stack implementation
* Queue implementation
* Batch insertion

---

# Traversing Linked Lists

## list_entry()

Convert list_head pointer to container structure.

```c
list_entry(
    ptr,
    struct my_list,
    list
);
```

Parameters:

| Parameter | Description       |
| --------- | ----------------- |
| ptr       | list_head pointer |
| type      | structure type    |
| member    | list_head member  |

---

# Forward Traversal

## list_for_each()

```c
struct list_head *pos;

list_for_each(
        pos,
        &etx_linked_list)
{
}
```

---

## list_for_each_entry()

Preferred method.

```c
struct my_list *node;

list_for_each_entry(
        node,
        &etx_linked_list,
        list)
{
    printk("%d\n",
           node->data);
}
```

---

# Safe Traversal

## list_for_each_entry_safe()

Safe while deleting nodes.

```c
struct my_list *node;
struct my_list *tmp;

list_for_each_entry_safe(
        node,
        tmp,
        &etx_linked_list,
        list)
{
    list_del(&node->list);
}
```

Used when removing nodes during traversal.

---

# Reverse Traversal

## list_for_each_prev()

```c
list_for_each_prev(
        pos,
        &etx_linked_list)
{
}
```

---

## list_for_each_entry_reverse()

```c
list_for_each_entry_reverse(
        node,
        &etx_linked_list,
        list)
{
    printk("%d\n",
           node->data);
}
```

---

# Typical Driver Use Cases

Linux kernel linked lists are heavily used in:

* Process management
* Device driver queues
* Workqueue management
* Timer management
* Networking subsystem
* USB subsystem
* Block layer
* Scheduler

Kernel developers frequently embed `struct list_head` inside driver-specific structures rather than storing data directly in the list node.

---

# Example Flow

```text
Create List Head
       |
       v
Create Node
       |
       v
INIT_LIST_HEAD()
       |
       v
list_add()
       |
       v
Traverse
       |
       v
list_del()
       |
       v
Free Memory
```

---

# Important APIs Summary

## Initialization

```c
LIST_HEAD()
INIT_LIST_HEAD()
```

## Add

```c
list_add()
list_add_tail()
```

## Delete

```c
list_del()
list_del_init()
```

## Replace

```c
list_replace()
list_replace_init()
```

## Move

```c
list_move()
list_move_tail()
```

## Test

```c
list_empty()
list_is_last()
list_is_singular()
```

## Split / Join

```c
list_cut_position()
list_splice()
```

## Traversal

```c
list_for_each()
list_for_each_entry()
list_for_each_entry_safe()
list_for_each_prev()
list_for_each_entry_reverse()
```

---

# Key Takeaways

* Linux kernel provides a built-in circular doubly linked list implementation.
* `struct list_head` is embedded inside user-defined structures.
* List heads are created using `LIST_HEAD()`.
* Nodes are initialized using `INIT_LIST_HEAD()`.
* `list_add()` inserts after head (stack behavior).
* `list_add_tail()` inserts before head (queue behavior).
* `list_del()` removes nodes but does not free memory.
* `list_for_each_entry()` is the preferred traversal method.
* `list_for_each_entry_safe()` should be used when deleting nodes during traversal.
* Linked lists are extensively used throughout Linux kernel subsystems and device drivers.


# Conclusion
This is just a basic linux device driver which explains about the linked list in linux device driver.

Please refer this URL for the complete tutorial of this example source code.
https://embetronicx.com/tutorials/linux/device-drivers/example-linked-list-in-linux-kernel/
