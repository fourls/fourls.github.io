---
layout: post
title: Memory Management Strategies in C++
description: "Nowadays there are manifold strategies to manage memory usage in a C++ program. This report analyzes some of the most common, including C-style manual management, C++ style pointer technologies, and two garbage collection algorithms: reference counting, and Mark-Sweep. These methods are tested against each other in terms of speed and memory usage in order to identify the strengths and weaknesses of the various approaches."
---
# Introduction 
Object-oriented programming revolves, quite intuitively, around the concept of the object – an entity consisting of data that can take a wide variety of shapes and sizes, which is an issue when the theory meets the reality of programming. This report analyses some of the multitude of ways of managing the memory footprint of an object-oriented programming and conducts several experiments using the C++ language to illustrate the advantages and disadvantages of these approaches.

# Theoretical Background
## C++ Object Architecture
Although the strategies outlined in this report are not specifically tied to C++, some of the terms and implementation details that will be mentioned will be C++ specific for simplicity. 

Memory in regard to programming is often described in terms of the ‘stack’ and the ‘heap’, but the C++ standard does not actually define whether a certain object should be stored on the stack or the heap. It is rather left up to the implementation of the specific compiler. In fact, the positioning of an object in C++ is decided by the object’s storage duration, which dictates when and how the object’s memory is allocated and deallocated. There are four types of storage duration, but only two will be discussed in this report. 
An object with automatic storage duration is allocated at the beginning of the containing scope and deallocated at the end of it (Prata, 2011).  These are also known as ‘local variables’. This almost always corresponds to being placed on the program call stack (often shortened to the stack).

An object with dynamic storage duration is allocated and deallocated at runtime at the programmer’s discretion. These objects, once created, persist for the lifetime of the program unless explicitly deallocated – potentially longer, although the operating system typically clears program memory after exit (cppreference.com, 2020). This corresponds to being placed in free memory (often termed the heap) – note that in no implementation can a dynamically allocated object be placed on the stack, as they must be expected to outlive their current scope. 

The storage duration of an object is determined at compile time, so these two storage durations correspond to different C++ syntaxes to provide maximum flexibility for the programmer, as seen in Figure 1 below.

<figure markdown="1">
| **Automatic** | `Node x = Node();`      |
| **Dynamic**   | `Node* x = new Node();` |

<figcaption>Figure 1. Basic C++ syntax for allocating automatic and dynamic memory.</figcaption>
</figure>

Because automatic objects are automatically deallocated once the code block they are contained in is left, these objects do not require explicit management of memory. Dynamic objects, however, must be managed, as the exact size and length of time a dynamic block of memory may be used for may not be determinable. As such, this report deals primarily with dynamically allocated objects. It’s important to note that automatic variables contain the actual object data within them, while objects that are dynamic are referenced using pointer types which merely contain a pointer to the object data. This has consequences - for example, it means that large objects are not suitable to be automatic, and that the operators used for dynamic objects are different.

## Manual Memory Management
The most fundamental memory management method a language can employ is to delegate the task to the programmer. In C, C++’s predecessor, dynamic memory allocation is provided through the use of low-level functions that directly allocate and free memory of a specified size: malloc, which allocates memory and free, which deallocates memory, as seen in Figure 2.

<figure markdown="1">

```cpp
Node* my_node = (Node*)malloc(sizeof(Node));

if(my_node != nullptr) {
    // do stuff with object

    free(my_node);
}
```
<figcaption>Figure 2. Using C-style memory allocation functions to manage memory.</figcaption>
</figure>

To enable C++ to be written in a C-style manner, these functions are also available in C++. While this method is fast, it is prone to both size errors and memory leaks – if the size required is miscalculated by the programmer, the memory will overflow into a different block, and if the programmer forgets to free any allocated object then this memory will take up space for the duration of the program (Standard C++ Foundation, 2020). Allocating memory in this way also just creates an empty object – there is no way to call an object’s constructor or destructor to initialize or clean up. Luckily, this system is only included for backwards compatibility - C++ provides an new interface for manual memory management.

Dynamic memory allocation in C++ is typically signified by the use of the new operator. Where constructing an object without new initializes the object as automatic (size automatically allocated), constructing an object with new initializes it as dynamic. Behind the scenes, new operates in exactly the same way as malloc, provides some simple checks to ensure data integrity (like the nullptr check in Figure 2), and calls the object’s constructor to initialize it. The equivalent of free for C++-style allocation is the delete keyword, which is used in exactly the same way and both frees the memory and calls the object’s destructor, if present. See Figure 3 for an example of these syntaxes.

<figure markdown="1">

```cpp
Node* my_node = new Node();

// do stuff with object

delete my_node;
```
<figcaption>Figure 3. Using the C++ new and delete operators to manage memory.</figcaption>
</figure>

Note that although C and C++-style memory allocation methods can coexist within a program, they are not interchangeable – if an object is allocated using malloc, it should be freed with free, and the same for new and delete. This is because malloc and new allocate from different heaps (Standard C++ Foundation, 2020).
 
## Smart Pointers
As the C++ new/delete system still leaves itself open to potential memory leaks – it’s very easy to forget to delete every object in your program! – recent versions of the C++ specification have included wrappers around the basic C++ system that manage deallocation on the programmer’s behalf. In modern C++, ‘raw’ pointers are rarely used, with the safer option of smart pointers preferred. 

Smart pointers follow the resource acquisition is initialization (RAII) paradigm, which centers around the concept that any object on the heap should be ‘owned’ by an object on the stack. This means that when the stack object is initialized, it initializes the heap object, and when the stack object is destructed, it frees the heap object (Microsoft, 2019). In C++ this corresponds to an automatic object that owns a dynamic object – it is important to note that this does mean that the dynamic object’s lifetime is tied to the scope of the automatic object it is declared in, restricting usage to a specific subset of programming patterns.

C++ offers two main types of smart pointers for usage. Using a unique_ptr permits only one owner of the underlying ‘raw’ pointer, while shared_ptr  allows multiple owners using reference counting (see 2.4). Examples of creation of smart pointers can be seen in Figure 4 below, and the objects they reference can be accessed in the same way as a normal pointer.

<figure markdown="1">

```cpp
std::unique_ptr<Node> smart_node(new Node());
// do stuff with object

std::shared_ptr<Node> shared_ptr_a(new Node());
std::shared_ptr<Node> shared_ptr_b(shared_ptr_a);
// do stuff with object
// note both shared ptrs point to the same object
```
<figcaption>Figure 4. Using smart pointers to automatically garbage collect.</figcaption>
</figure>

This is the first encounter in this report with garbage collection: a system to automatically free an object’s memory when it is done being used. The advantage of using smart pointers is that the programmer knows exactly when the dynamic object is freed but does not have to consciously consider it if it is not important. This means that the exact state of the object at any point in the program can be determined, and such the smart pointer method is called a deterministic garbage collection.

## Garbage Collection: Reference Counting

Most garbage collectors are, however, behind the scenes. Rather than force objects to be freed at a specific point, like smart pointers, collectors typically use a particular garbage collection algorithm to periodically examine the current program state for unused objects before freeing them.

Reference counting is one such algorithm and is typically used because of its simplicity rather than any major advantages it holds. As the name implies, whenever a pointer points to an object, a count of how many references that object has is incremented, and when that pointer points elsewhere, the count is decremented. Most implementations free the object immediately once the reference count reaches zero (Schildt, 2004).

While exceptionally easy to implement and reasonably high performance, reference counts fail to handle circular references, where two objects directly or indirectly point to each other, as when pointers are deallocated the reference count never reaches zero (Schildt, 2004). See Figure 5 below for an illustration of why this occurs.

<figure markdown="1">
![Figure 5](/assets/img/posts/2020-11-01-memory-management-cpp/fig5.png)
<figcaption>Figure 5. How circular references create memory leaks when reference counting. Note that A and B will never be garbage collected, even though they are inaccessible.</figcaption>
</figure>

## Garbage Collection: Mark-Sweep
The Mark-Sweep garbage collection algorithm is a tracing algorithm. This algorithm ‘traces’ all the references from the program stack to mark the objects that are accessible as ‘alive’. After this, the heap is swept and any objects found that are not alive are freed. This is substantially more performance-intensive than a reference counting algorithm, but is guaranteed to be correct, as a Mark-Sweep algorithm handles circular references just fine (Schildt, 2004). See Figure 6 for a visualization of the mark phase of the Mark-Sweep algorithm.
 
<figure markdown="1">
![Figure 6](/assets/img/posts/2020-11-01-memory-management-cpp/fig6.png)
<figcaption>Figure 6. The state of dynamic objects after the mark phase of a Mark-Sweep garbage collector. Note that A and B will be properly garbage collected, despite having references.</figcaption>
</figure>

Note also that tracing algorithms run at intervals, usually when there are available CPU cycles. As a result, all tracing algorithms are non-deterministic - the exact time heap objects are destructed is impossible to determine, meaning that an object’s destructor cannot be relied on to be called at a certain point in execution. 

# Experiment
## Method
Firstly, a benchmarking program was designed that tests three areas of particular interest to garbage collection mechanisms: handling of circular references (see Figure 7 for details), garbage collecting a large amount of objects at a single point in time at the end, and garbage collecting the same amount of objects with collection distributed throughout the program. This program was initially written to use raw pointers, then modified to use each different memory management method. 

<figure markdown="1">
![Figure 7](/assets/img/posts/2020-11-01-memory-management-cpp/fig7.png)
<figcaption>Figure 7. The heap structure created by the circular reference test.</figcaption>
</figure>

The initial version was easily adjusted into a version that uses smart pointers, as these are included in the C++ standard library. For both the garbage collection methods, custom C++ garbage collectors were written expressly for this report. The reference counting garbage collector, based on the one designed by Schildt (2004), contains a GCPtr class that handles reference counting internals and is intended to be used in a similar way to smart pointers. The mark-sweep garbage collector is roughly based off Dmitry Soshnikov’s 2020 implementation of a Mark-Sweep garbage collector in C++, but omits direct stack-reading, containing a RootPtr class that registers object pointers to quickly trace objects from the stack instead.

In the final programs, a command line argument determined which test was to be run at a particular time. These programs were compiled separately for each memory management method using GCC and optimization disabled (see Figure 8).

<figure markdown="1">
```console
g++ -g -O0 -o RefCount.out RefCount/*.cpp
```
<figcaption>Figure 8. Compilation command for the reference counting GC tests.</figcaption>
</figure>

Any memory leaks from circular references for raw pointers were identified using the default settings of the Memcheck tool in the Valgrind Linux command line test suite (see Figure 9). The memory left over after each test had finished execution was recorded by Memcheck, as well as the number of allocations the program had called. This was repeated for each memory management method.

<figure markdown="1">
```console
valgrind ./RefCount.out circ
```
<figcaption>Figure 9. MemCheck command for the reference counting GC’s circular reference test.</figcaption>
</figure>

The large-object garbage collection tests for raw pointers were then executed and timed. To ensure maximum precision, the C++11 chrono library’s high_resolution_clock was used to determine the execution time of the test in microseconds, which was then printed to the terminal and recorded (see Figure 10). To minimize inaccuracy, these tests were each executed ten times and their results averaged. This was repeated for each memory management method.

<figure markdown="1">
```console
./RefCount.out free_once
```
<figcaption>Figure 10. Command to run the reference counting GC’s single collection test.</figcaption>
</figure>

## Results
 
<figure markdown="1">
![Figure 11](/assets/img/posts/2020-11-01-memory-management-cpp/fig11.png)
<figcaption>Figure 11. The size of the memory leaks caused from the circular references test.</figcaption>
</figure>
 
<figure markdown="1">
![Figure 12](/assets/img/posts/2020-11-01-memory-management-cpp/fig12.png)
<figcaption>Figure 12. Estimates of the number of allocations involved in the allocation tests.</figcaption>
</figure>

<figure markdown="1">
![Figure 13](/assets/img/posts/2020-11-01-memory-management-cpp/fig13.png)
<figcaption>Figure 13. The average execution time of each of the allocation tests.

Note: The ‘reference counting single collection’ value is cut off; it is 18534.3.</figcaption>
</figure>

## Discussion
As shown in Figure 11, only the reference counting garbage collection method was unable to successfully handle objects with circular references. This is in accordance with what the theory suggests: Figure 14 shows the objects expected to be leaked in the circular reference test along with their size in bytes, which adds up to the leak measured. Note that neither raw or smart pointers are garbage collectors, but rather methods of manual memory management, so the memory leaked is entirely dependent on the programmer’s acumen. 
 
<figure markdown="1">
![Figure 14](/assets/img/posts/2020-11-01-memory-management-cpp/fig14.png)
<figcaption>Figure 14. Objects leaked by the reference counting GC in the circular reference test.</figcaption>
</figure>

The number of allocations recorded (Figure 12) also matches up with the theoretical overhead of the different methods - raw pointers are the most concise, requiring no additional allocations than those explicitly instructed, while smart pointers and reference counting incur a memory impact due to the overhead of managing pointers. The Mark-Sweep method’s practice of creating object headers and its costly marking process result in it having almost three times as many allocations as using raw pointers.

The most substantial cause for performance concern is the time impact of these memory management methods (Figure 13). Raw and smart pointers were similar in terms of performance in both allocation tests, giving credence to the Modern C++ recommendation in most situations, use of raw pointers should be phased out in preference of smart pointers (Microsoft, 2019). In the distributed collection test, the reference counting method was only slower by five milliseconds, showing that reference counting is a viable garbage collection method with only a small performance impact. In the single collection test, however, the reference counting method took orders of magnitude longer than any other method - clocking at 18.5 seconds, it was 18.3 seconds slower than the next slowest test result. This is due to the fact that reference counting is not intended to have ‘sweeps’ in the same way a tracing algorithm might, and as such the algorithm is not optimized for it. While adjusting the reference counting code may deliver a performance increase, the magnitude of this result upholds the theory that reference counting is best used as a distributed collection method.

The Mark-Sweep garbage collector was, as expected, the most time-costly management method. Although Mark-Sweep collectors are not intended to be run frequently, in the collection tests it performed better in the distributed collection than when a single collection was issued. Although unintuitive at first glance, this is likely because of the nature of the test rather than the collector itself. In the distributed collection test, a single variable was allocated before the garbage collector was  called, resulting in a very small list of stack variables to pore through. This is unrealistic to a real program, where several stack variables would be allocated before collection expected. The single collection was about 50ms slower than the distributed collection, likely because of the size of the set of stack variables allocated resulting in a large tree of objects to manage.

# Conclusion
While manual memory allocation techniques were once the only way to manage memory in a program, nowadays there are many techniques and algorithms that help take this burden out of the programmer’s hands. Although not all methods were examined in this report, comparing manual allocation, smart pointers, and two basic garbage collection methods with each other delivered a series of results that largely matched the literature on the subject: for raw speed, manual management was best, followed narrowly by smart pointers and further behind by reference counting. For maximum reliability, however, performance must be sacrificed, as seen by the results of the Mark-Sweep garbage collector.

# References
cppreference.com. (2020, April 3). Storage class specifiers. Retrieved from cppreference.com: <https://en.cppreference.com/w/cpp/language/storage_duration>

Microsoft. (2019). C++ Language Reference. Retrieved October 30, 2020, from Microsoft Docs: <https://docs.microsoft.com/en-us/cpp/cpp/cpp-language-reference>

Prata, S. (2011). C++ Primer Plus. Addison-Wesley Professional.

Schildt, H. (2004). A Simple Garbage Collector for C++. In H. Schildt, The Art of C++ (pp. 7-55). McGraw-Hill/Osborne.

Soshnikov, D. (2020, March 15). Writing a Mark-Sweep Garbage Collector. Retrieved from dmitrysoshnikov.com: <http://dmitrysoshnikov.com/compilers/writing-a-mark-sweep-garbage-collector/>

Standard C++ Foundation. (2020). Memory Management. Retrieved from ISO C++: <https://isocpp.org/wiki/faq/freestore-mgmt>



