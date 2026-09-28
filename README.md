
Lecture # 1: Static VS Non Static 

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


**Previous Year Recommended Topic:**

Servlet Learning Project (Lab): https://github.com/rahmanziaur/TestServlet19
Spring Boot Helps: https://youtu.be/-Fe0zk-F4OA
Spring Boot Projects: https://youtu.be/HYGnVeCs0Yg
https://github.com/RameshMF/student-management-system-springboot
GFG 100 Days Tute: https://www.geeksforgeeks.org/blogs/100-days-of-spring-boot/
