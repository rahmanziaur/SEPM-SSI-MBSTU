
**Lecture # 1: (27/09/2026)**

**Part A: Static VS Non Static **

If your goal is to **count how many objects have been created**, then the field `count` should normally be **`static`**, not non-static. The important point is that **static does not mean the value will not increase**. It means there is **one shared copy of the variable for the entire class**.

### Example: `static count` — correct for object counting

```java
class Student {
    static int count = 0;

    Student() {
        count++;
    }
}

public class Main {
    public static void main(String[] args) {
        Student s1 = new Student();
        Student s2 = new Student();
        Student s3 = new Student();

        System.out.println(Student.count);
    }
}
```

Output:

```text
3
```

Why?

```text
Student class
     |
     +---- static count = 3  ← ONE shared variable
     |
     +---- s1
     +---- s2
     +---- s3
```

Every time the constructor runs:

```java
count++;
```

it modifies the **same shared variable**.

---

### What happens if `count` is non-static?

```java
class Student {
    int count = 0;

    Student() {
        count++;
    }
}
```

Now **every object gets its own `count`**:

```text
s1 → count = 1
s2 → count = 1
s3 → count = 1
```

So:

```java
System.out.println(s1.count); // 1
System.out.println(s2.count); // 1
System.out.println(s3.count); // 1
```

It does **not** actually count the total number of objects. It counts something separately inside each object.

---

### The key difference

|                                  | `static int count` | `int count`        |
| -------------------------------- | ------------------ | ------------------ |
| Copies of variable               | **One per class**  | **One per object** |
| Shared by objects?               | Yes                | No                 |
| `count++` across 3 objects       | 1 → 2 → 3          | Each object: 0 → 1 |
| Suitable for total object count? | **Yes**            | No                 |


**Part B:**

Sure. Since you want to demonstrate **multiple threads**, an **infinite loop**, and then stop each thread after roughly **10 seconds**, I would structure it with a loop controlled by time rather than using a truly uncontrolled infinite loop.

Also, because `CookingTask` extends `Thread`, you don't need to create another `Thread` around it. You can simply call `task.start()`.

```java
public class ThreadMain {
    public static void main(String[] args) {

        CookingTask task1 = new CookingTask("Cooking");
        CookingTask task2 = new CookingTask("Washing");
        CookingTask task3 = new CookingTask("Cleaning");

        task1.start();
        task2.start();
        task3.start();

        System.out.println("All tasks started...");
    }
}

class CookingTask extends Thread {

    private String taskName;

    public CookingTask(String taskName) {
        this.taskName = taskName;
    }

    @Override
    public void run() {

        long startTime = System.currentTimeMillis();

        while (true) {

            System.out.println(
                    Thread.currentThread().getName()
                    + " - Running: " + taskName
            );

            // Wait 1 second before next iteration
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                System.out.println(taskName + " interrupted.");
                break;
            }

            // Stop after 10 seconds
            if (System.currentTimeMillis() - startTime >= 10_000) {
                break;
            }
        }

        System.out.println(taskName + " finished.");
    }
}
```

### What happens

You create **three separate thread objects**:

```java
CookingTask task1 = new CookingTask("Cooking");
CookingTask task2 = new CookingTask("Washing");
CookingTask task3 = new CookingTask("Cleaning");
```

Then:

```java
task1.start();
task2.start();
task3.start();
```

All three can execute concurrently.

You'll see output similar to:

```text
All tasks started...

Thread-0 - Running: Cooking
Thread-1 - Running: Washing
Thread-2 - Running: Cleaning

Thread-0 - Running: Cooking
Thread-1 - Running: Washing
Thread-2 - Running: Cleaning

Thread-0 - Running: Cooking
Thread-2 - Running: Cleaning
Thread-1 - Running: Washing

...

Cooking finished.
Washing finished.
Cleaning finished.
```

Notice that the order is **not guaranteed**. Sometimes `Thread-2` may print before `Thread-0`, etc. That's an important part of understanding Java multithreading.

### If you specifically want an infinite `for` loop

For teaching purposes, you can use:

```java
for (;;) {
    System.out.println("Running " + taskName);

    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        break;
    }

    if (System.currentTimeMillis() - startTime >= 10_000) {
        break;
    }
}
```

`for (;;)` is Java's traditional **infinite loop**:

```java
for (;;) {
    // runs forever unless something breaks it
}
```

The `break` after 10 seconds is what terminates it.

**One more important correction to your original code:** you had:

```java
CookingTask task = new CookingTask("task");
Thread thread = new Thread(task);
thread.start();
```

Since `CookingTask extends Thread`, this creates a `Thread` **inside another Thread object**, which isn't what you want. Use:

```java
CookingTask task = new CookingTask("task");
task.start();
```

Yes. This is a good experiment because with **3 thread objects**, you can see exactly why `static` and non-static fields behave differently.

```java
public class ThreadMain {

    public static void main(String[] args) {

        CookingTask task1 = new CookingTask("Cooking");
        CookingTask task2 = new CookingTask("Washing");
        CookingTask task3 = new CookingTask("Cleaning");

        task1.start();
        task2.start();
        task3.start();
    }
}


class CookingTask extends Thread {

    private String taskName;

    // ONE copy shared by ALL CookingTask objects
    static int staticCount = 0;

    // ONE copy for EACH CookingTask object
    int nonStaticCount = 0;


    public CookingTask(String taskName) {
        this.taskName = taskName;
    }


    @Override
    public void run() {

        long startTime = System.currentTimeMillis();

        for (;;) {

            // Increase both counters
            staticCount++;
            nonStaticCount++;

            System.out.println(
                    Thread.currentThread().getName()
                    + " | " + taskName
                    + " | Static Count = " + staticCount
                    + " | Non-Static Count = " + nonStaticCount
            );

            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                break;
            }

            // Stop after approximately 10 seconds
            if (System.currentTimeMillis() - startTime >= 10_000) {
                break;
            }
        }

        System.out.println(
                taskName + " finished. "
                + "Final Non-Static Count = " + nonStaticCount
        );
    }
}
```

### What you should observe

There are **3 objects**:

```text
task1
task2
task3
```

The non-static variable exists separately:

```text
task1 → nonStaticCount
task2 → nonStaticCount
task3 → nonStaticCount
```

But `staticCount` has only **one copy**:

```text
             ┌── task1
             │
staticCount ─┼── task2
             │
             └── task3
             ↑
       ONE shared variable
```

So after the first iteration, you might see something like:

```text
Thread-0 | Cooking  | Static Count = 1 | Non-Static Count = 1
Thread-1 | Washing  | Static Count = 2 | Non-Static Count = 1
Thread-2 | Cleaning | Static Count = 3 | Non-Static Count = 1

Thread-0 | Cooking  | Static Count = 4 | Non-Static Count = 2
Thread-1 | Washing  | Static Count = 5 | Non-Static Count = 2
Thread-2 | Cleaning | Static Count = 6 | Non-Static Count = 2
```

The important observation is:

**Static:**

```text
1 → 2 → 3 → 4 → 5 → 6 → ...
```

All three threads are modifying the **same variable**.

**Non-static:**

```text
Cooking:  1 → 2 → 3 → 4 ...
Washing:  1 → 2 → 3 → 4 ...
Cleaning: 1 → 2 → 3 → 4 ...
```

Each object has its **own copy**.

### One more interesting experiment

At the end of `main()`, try:

```java
System.out.println(CookingTask.staticCount);
```

You can access the static variable using the **class name** because it belongs to the class:

```java
CookingTask.staticCount
```

But you cannot do:

```java
CookingTask.nonStaticCount;  // ERROR
```

because `nonStaticCount` belongs to an **object**, not the class.

For example:

```java
System.out.println(task1.nonStaticCount);
System.out.println(task2.nonStaticCount);
System.out.println(task3.nonStaticCount);
```

will give three potentially different values.

Absolutely. This is the key concept behind why your `staticCount++` experiment can produce surprising results.

## 1. What is a race condition?

A **race condition** occurs when multiple threads access and modify the **same shared data at the same time**, and the final result depends on the timing/order of the threads.

In your example:

```java
static int staticCount = 0;
```

There is only **one `staticCount` shared by all three threads**.

Each thread executes:

```java
staticCount++;
```

The problem is that:

```java
staticCount++;
```

looks like one operation, but internally it is approximately:

```text
1. READ  staticCount
2. ADD   1
3. WRITE staticCount
```

It is therefore **not atomic**.

---

## 2. How can the problem happen?

Suppose:

```text
staticCount = 10
```

Two threads execute `staticCount++` at nearly the same time.

### Thread 1

```text
READ → 10
```

### Thread 2

```text
READ → 10
```

Both threads have now read `10`.

Then:

```text
Thread 1: 10 + 1 = 11
Thread 2: 10 + 1 = 11
```

And both write:

```text
Thread 1 → WRITE 11
Thread 2 → WRITE 11
```

You expected:

```text
10 → 11 → 12
```

But you got:

```text
10 → 11
```

One increment was effectively **lost**.

This is called a **lost update**.

---

# 3. Let's do the experiment

Instead of running for 10 seconds, let's make the experiment easier to measure.

We'll create three threads, and each thread will increment the counter **1,000,000 times**.

### Experiment 1 — No synchronization

```java
public class ThreadMain {

    public static void main(String[] args) throws InterruptedException {

        CookingTask task1 = new CookingTask("Cooking");
        CookingTask task2 = new CookingTask("Washing");
        CookingTask task3 = new CookingTask("Cleaning");

        task1.start();
        task2.start();
        task3.start();

        // Wait for all threads to finish
        task1.join();
        task2.join();
        task3.join();

        System.out.println("Expected Count = " + (3 * 1_000_000));
        System.out.println("Actual Count   = " + CookingTask.staticCount);
    }
}


class CookingTask extends Thread {

    static int staticCount = 0;

    private String taskName;

    public CookingTask(String taskName) {
        this.taskName = taskName;
    }

    @Override
    public void run() {

        for (int i = 0; i < 1_000_000; i++) {
            staticCount++;
        }

        System.out.println(taskName + " finished.");
    }
}
```

You might expect:

```text
Expected Count = 3000000
Actual Count   = 3000000
```

But you may get something like:

```text
Expected Count = 3000000
Actual Count   = 1847291
```

or:

```text
Expected Count = 3000000
Actual Count   = 2478392
```

The exact result can vary.

That's the **race condition**.

---

# 4. Solution 1 — `synchronized`

We can protect the increment using `synchronized`.

Change:

```java
staticCount++;
```

to:

```java
incrementCount();
```

and create:

```java
static synchronized void incrementCount() {
    staticCount++;
}
```

Complete version:

```java
public class ThreadMain {

    public static void main(String[] args) throws InterruptedException {

        CookingTask task1 = new CookingTask("Cooking");
        CookingTask task2 = new CookingTask("Washing");
        CookingTask task3 = new CookingTask("Cleaning");

        task1.start();
        task2.start();
        task3.start();

        task1.join();
        task2.join();
        task3.join();

        System.out.println("Expected Count = " + (3 * 1_000_000));
        System.out.println("Actual Count   = " + CookingTask.staticCount);
    }
}


class CookingTask extends Thread {

    static int staticCount = 0;

    private String taskName;

    public CookingTask(String taskName) {
        this.taskName = taskName;
    }

    @Override
    public void run() {

        for (int i = 0; i < 1_000_000; i++) {
            incrementCount();
        }

        System.out.println(taskName + " finished.");
    }

    static synchronized void incrementCount() {
        staticCount++;
    }
}
```

Now:

```text
Expected Count = 3000000
Actual Count   = 3000000
```

### Why?

`synchronized` makes sure that **only one thread at a time** can execute the synchronized method.

Conceptually:

```text
Thread 1 → 🔒 increment → 🔓
Thread 2 → waits
Thread 2 → 🔒 increment → 🔓
Thread 3 → waits
Thread 3 → 🔒 increment → 🔓
```

So the read → add → write sequence is protected.

---

# 5. Solution 2 — `AtomicInteger`

Java provides a special class for this kind of situation:

```java
AtomicInteger
```

Import it:

```java
import java.util.concurrent.atomic.AtomicInteger;
```

Then:

```java
static AtomicInteger staticCount = new AtomicInteger(0);
```

Instead of:

```java
staticCount++;
```

use:

```java
staticCount.incrementAndGet();
```

Complete example:

```java
import java.util.concurrent.atomic.AtomicInteger;

public class ThreadMain {

    public static void main(String[] args) throws InterruptedException {

        CookingTask task1 = new CookingTask("Cooking");
        CookingTask task2 = new CookingTask("Washing");
        CookingTask task3 = new CookingTask("Cleaning");

        task1.start();
        task2.start();
        task3.start();

        task1.join();
        task2.join();
        task3.join();

        System.out.println("Expected Count = " + (3 * 1_000_000));
        System.out.println("Actual Count   = " + CookingTask.staticCount.get());
    }
}


class CookingTask extends Thread {

    static AtomicInteger staticCount = new AtomicInteger(0);

    private String taskName;

    public CookingTask(String taskName) {
        this.taskName = taskName;
    }

    @Override
    public void run() {

        for (int i = 0; i < 1_000_000; i++) {
            staticCount.incrementAndGet();
        }

        System.out.println(taskName + " finished.");
    }
}
```

Now again:

```text
Expected Count = 3000000
Actual Count   = 3000000
```

---

## 6. `synchronized` vs `AtomicInteger`

|                               | `staticCount++` | `synchronized`            | `AtomicInteger`     |
| ----------------------------- | --------------- | ------------------------- | ------------------- |
| Thread-safe?                  | ❌ No            | ✅ Yes                     | ✅ Yes               |
| Race condition?               | Possible        | Prevented                 | Prevented           |
| Lock required?                | No              | Yes                       | No explicit lock    |
| Suitable for simple counters? | ❌               | ✅                         | ✅                   |
| Operation                     | `++`            | synchronized method/block | `incrementAndGet()` |

A useful way to remember it:

```text
static
  ↓
ONE shared variable

multiple threads
  ↓
simultaneous access

staticCount++
  ↓
READ → MODIFY → WRITE

READ/MODIFY/WRITE can overlap
  ↓
RACE CONDITION
```

Whereas:

```text
AtomicInteger
      ↓
incrementAndGet()
      ↓
atomic increment
      ↓
safe concurrent counter
```

### One subtle point

The `join()` calls in the examples are **not** what makes the counter thread-safe.

```java
task1.join();
task2.join();
task3.join();
```

They simply make `main()` **wait until the worker threads finish** before printing the result.

So there are actually **two separate concurrency concepts** in this experiment:

1. **`synchronized` / `AtomicInteger`** → protects the shared counter.
2. **`join()`** → makes the main thread wait for the worker threads.

That distinction is very important when learning Java concurrency.


**Previous Year Recommended Topics:**

Servlet Learning Project (Lab): https://github.com/rahmanziaur/TestServlet19
Spring Boot Helps: https://youtu.be/-Fe0zk-F4OA
Spring Boot Projects: https://youtu.be/HYGnVeCs0Yg
https://github.com/RameshMF/student-management-system-springboot
GFG 100 Days Tute: https://www.geeksforgeeks.org/blogs/100-days-of-spring-boot/
