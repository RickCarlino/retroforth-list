# Dynamic Lists in Retro

This library creates lists that can shrink and grow at runtime
(similar to the `std::vector` template in C++).

Although the overall number of nodes in the list is flexible, the size of an
individual list node is fixed (currently 1 cell). If you need to store values larger than one cell in length, consider allocating the memory via `mem:alloc` and storing the resulting address in the list.

# The List Node

A list is a chain of nodes linked together via pointers.

    Node 0    Node 1     Node 2
    ┌──────┐  ┌──────┐   ┌──────┐
    │Value │  │Value │   │Value │  <-- Cell Zero
    ├──────┤  ├──────┤   ├──────┤
    │Next  │  │Next  │   │Next  │  <-- Cell One
    │ Addr.│  │ Addr.│   │ Addr.│
    └────┬─┘  └─────┬┘   └─────┬┘ And so on...
         │    ▲     │    ▲     │    ▲
         └────┘     └────┘     └────┘

A single node in a list is 16 bytes. (64 Bit value + 64 bit "next" value).

~~~
#16 'node:SIZE const
~~~

The memory layout for a single node is as follows:

|Cell Number (64 Bit) | Contents                             |
|---------------------|--------------------------------------|
|0                    | Node's Value (called "head")         |
|1                    | Address to next node ("tail")        |

I will use the term "head" to mean "a node's value slot".
I will use the term "tail" to mean "a node's link to its next sibling".
The last node in a list has a `node:tail` value of `FALSE`.

The following words are added to simplify CRUD operations on list nodes:

~~~
:node:head (a-a) #0 mem:cell+ ;
:node:tail (a-a) #1 mem:cell+ ;

:node:fetch-head (a-n)  node:head mem:fetch ;
:node:fetch-tail (a-n)  node:tail mem:fetch ;
:node:fetch-both (a-nn) dup node:fetch-tail swap node:fetch-head ;

:node:store-head (an-)   swap node:head swap mem:store ;
:node:store-tail (an-)   swap node:tail swap mem:store ;

:node:reset-head (a-) #0 node:store-head ;
:node:reset-tail (a-) #0 node:store-tail ;
:node:reset      (a-) &node:reset-head sip node:reset-tail ;

:node:link-pair (ht-t) tuck node:store-tail ;
:node:last?     (a-f)  node:fetch-tail FALSE -eq? ;
:node:inspect   (a-)   node:fetch-both '(%n,_%n) s:format s:put ;
:node:new       (-a)   node:SIZE mem:alloc &node:reset sip ;
~~~

Dynamic memory for a single node is allocated via `node:new`:
TODO: Should I add an explicit `node:reset` call for allocators that don't zero memory?
# Iterating Over Lists

~~~
{{
  :qa->qaaq dup-pair swap ;
  :call-iterator  (q(a-)a-q(a-)a) qa->qaaq call ;

  :cycle (q(a-)a-q(a-)a)
    call-iterator
    node:fetch-tail 0;
    cycle
  ; tail-recurse

---reveal---
  :list:for-each (aq(a-)-) swap cycle drop ;
}}
~~~

---

The rest of this stuff is not complete.

# Appending to a List with `list,`

# Removing the last element (pop)

# Counting the Length

# Finding the Tail

# Accessing List Elements

# Reassigning Values

# Destroying Nodes / Shrinking the List
