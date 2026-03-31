# JVM Memory: Stack vs Heap

Understanding the difference between the Stack and the Heap is crucial for writing efficient Java code and diagnosing memory-related issues like performance bottlenecks or crashes. In the JVM, these two memory areas have distinct purposes, lifetimes, and management rules.

---

## 1. The Stack (Java Virtual Machine Stack)

The Stack is a per-thread memory area that stores temporary data related to method execution. Each time a thread invokes a method, a new **Stack Frame** is created and pushed onto the stack. When the method returns, the frame is popped.

### 1.1 Structure of a Stack Frame
Each frame contains:
1. **Local Variable Table**: Primitive types (`int`, `boolean`, etc.) and object references.
2. **Operand Stack**: Workspace for performing calculations (e.g., adding two numbers).
3. **Frame Data**: Return address, references to the constant pool, and exception dispatch info.

```mermaid
graph TD
    subgraph StackFrame[Stack Frame Detail]
    LVT[Local Variable Table: x=10, y=20, ref_z]
    OS[Operand Stack: Push 10, Push 20, Add]
    FD[Frame Data: Return PC, Dyn Link, Exceptions]
    end
```

### 1.2 Characteristics
- **Dynamic LIFO**: Last-In, First-Out (LIFO) access.
- **Speed**: Extremely fast due to its simple access pattern.
- **Safety**: Each thread has its own stack, so data is naturally thread-safe.
- **Limited Size**: Controlled by the `-Xss` parameter.

**Example 1: Recursive Method Call**
In a recursive call like `calculateFactorial(5)`, a new frame is pushed for each level ($5, 4, 3, \dots, 1$). If the recursion is too deep (e.g., infinite loop), the stack runs out of space, resulting in a `StackOverflowError`.

**Example 2: Method Logic Execution**
```java
public void compute() {
    int x = 10; // Stored in Local Variable Table of current Frame
    int y = 20; // Stored in Local Variable Table
    int sum = x + y; // Intermediated via Operand Stack
}
```
When `compute()` finishes, its entire Stack Frame (including `x`, `y`, and `sum`) is wiped from memory instantly.

---

## 2. The Heap (Java Heap)

The Heap is the largest memory area in the JVM, shared across all threads. It is used for the runtime allocation of objects.

### 2.1 Object Storage
When you create an object using the `new` keyword, the memory is allocated on the Heap. The local variable on the Stack merely holds a **reference** (pointer) to the object's memory address on the Heap.

### 2.2 Heap Generations
To improve the efficiency of Garbage Collection (GC), the JVM divides the heap into generations:
- **Young Generation**: Where new objects are born. Most objects die here.
- **Old (Tenured) Generation**: Where long-lived objects are moved after surviving multiple GC cycles.

```mermaid
graph LR
    subgraph YoungGen[Young Generation]
    Eden[Eden Area - New Objects] --> S0[Survivor 0]
    S0 <--> S1[Survivor 1]
    end
    YoungGen --> OldGen[Old Generation - Long-lived]
    OldGen --> Meta[MetaSpace - Class Metadata]
```

```mermaid
graph LR
    A[New Object] --> Eden[Eden Area]
    Eden -- Survivor of Minor GC --> S0[Survivor 0]
    S0 -- Survivor of Minor GC --> S1[Survivor 1]
    S1 -- Survivor of Minor GC --> S0
    S1 -- Reaches Max Age --> Old[Old Generation]
    Old -- Full GC Reclaims --> Dead[Removed from Memory]
```

### 2.3 Characteristics
- **Shared Access**: Accessible by all parts of the application. Not inherently thread-safe.
- **Slower Access**: More complex than stack operations.
- **Garbage Collection**: Reclaimed automatically by the GC when no more references point to an object.
- **Flexible Size**: Controlled by `-Xms` (initial) and `-Xmx` (max).

**Example 1: Object Allocation**
```java
User user = new User("Huy"); 
```
- `user`: A reference (pointer) stored in the Stack Frame.
- `new User("Huy")`: The actual object data (name, ID, etc.) resides on the Heap.

**Example 2: GC Trigger**
If `user = null;` is called, the reference in the stack is cleared. The object on the heap becomes "unreachable" and will be reclaimed during the next Minor GC cycle in the Young Generation.

---

## 3. Key Differences at a Glance

| Feature | Stack | Heap |
|---|---|---|
| **Scope** | Per-Thread (Isolated) | Application-wide (Shared) |
| **Object Lifetime** | Lives only until the method returns | Lives until explicitly reclaimed by GC |
| **Storage** | Primitives and References | Full Objects and Class metadata |
| **Access Speed** | Very Fast | Slower (requires pointer dereferencing) |
| **Common Failure** | `StackOverflowError` | `OutOfMemoryError` |
| **Management** | Manual (LIFO) by JVM instructions | Automatic by Garbage Collector |

---

## 4. Interaction Visualization

```mermaid
graph LR
    subgraph Thread_1_Stack
    Frame_A[Method A Frame]
    Frame_B[Method B Frame - local_ref_x]
    end

    subgraph Thread_2_Stack
    Frame_C[Method C Frame - local_ref_y]
    end

    subgraph Shared_Heap
    Obj_1[User Object 'Huy']
    Obj_2[Account Object 'Active']
    end

    Frame_B -- points to --> Obj_1
    Frame_C -- points to --> Obj_1
    Frame_C -- points to --> Obj_2
```
*In the diagram above, both Thread 1 and Thread 2 can see the 'Huy' object through their local references, while only Thread 2 can see the 'Active' account.*

---

## 5. Troubleshooting Strategy

- **If you see `StackOverflowError`**: 
    - Check for infinite recursion.
    - Check for excessively large local arrays.
    - Increase stack size using `-Xss`.
- **If you see `OutOfMemoryError: Java heap space`**:
    - Check for memory leaks (objects held in a list but never removed).
    - Check for large data sets loaded into memory at once.
    - Increase heap size using `-Xmx`.
