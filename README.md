# SEPM-SSI-MBSTU

Servlet Running:
https://www.geeksforgeeks.org/advance-java/how-to-create-servlet-in-myeclipse-ide/

package com.example;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/Login")
public class Login extends HttpServlet {
    private static final long serialVersionUID = 1L;
       
    public Login() {
        super();
    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html");
        PrintWriter out=response.getWriter();
        String userName=request.getParameter("userName");
        String password=request.getParameter("password");
        out.print("<html><body>");
        out.print("<h1>Hello!!!</h1>");
        out.printf("<h3>User Name with : %s </h3><h3>Your password is : %s</h3>",userName,password);
        out.print("</body></html>");
    }
}



To implement this, you'll need:

1. A **Student table** in a database (e.g., MySQL, SQLite, etc.).
2. A **Servlet** that connects to the database and retrieves student records.
3. A **JSP page** that displays the student data.

---

## ✅ Step 1: Create the `Student` Table (Example for MySQL)

```sql
CREATE TABLE Student (
    ID INT PRIMARY KEY,
    Name VARCHAR(100),
    Email VARCHAR(100)
);

INSERT INTO Student (ID, Name, Email) VALUES
(1, 'Alice', 'alice@example.com'),
(2, 'Bob', 'bob@example.com'),
(3, 'Charlie', 'charlie@example.com');
```

---

## ✅ Step 2: `Student` Java Class (Java Bean)

```java
package com.example;

public class Student {
    private int id;
    private String name;
    private String email;

    // Constructor
    public Student(int id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    // Getters
    public int getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
}
```

---

## ✅ Step 3: `StudentServlet.java` — Fetch from DB and Forward to JSP

```java
package com.example;

import java.io.IOException;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.*;

@WebServlet("/StudentServlet")
public class StudentServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        List<Student> studentList = new ArrayList<>();

        try {
            // Load JDBC driver
            Class.forName("com.mysql.cj.jdbc.Driver");

            // Connect to database
            Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/yourdb", "youruser", "yourpassword");

            // Query
            Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery("SELECT * FROM Student");

            while (rs.next()) {
                int id = rs.getInt("ID");
                String name = rs.getString("Name");
                String email = rs.getString("Email");
                studentList.add(new Student(id, name, email));
            }

            rs.close();
            stmt.close();
            conn.close();
        } catch (Exception e) {
            e.printStackTrace();
        }

        // Set list to request scope
        request.setAttribute("students", studentList);

        // Forward to JSP
        RequestDispatcher dispatcher = request.getRequestDispatcher("students.jsp");
        dispatcher.forward(request, response);
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }
}
```

---

## ✅ Step 4: `students.jsp` — Display the Table

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="com.example.Student" %>
<%@ page import="java.util.List" %>

<html>
<head>
    <title>Student List</title>
</head>
<body>
    <h2>Student Records</h2>

    <table border="1" cellpadding="10">
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Email</th>
        </tr>
        <%
            List<Student> students = (List<Student>) request.getAttribute("students");
            if (students != null) {
                for (Student s : students) {
        %>
        <tr>
            <td><%= s.getId() %></td>
            <td><%= s.getName() %></td>
            <td><%= s.getEmail() %></td>
        </tr>
        <%
                }
            } else {
        %>
        <tr><td colspan="3">No students found.</td></tr>
        <% } %>
    </table>
</body>
</html>
```

---

## ✅ Final Checklist

* ✅ Add MySQL JDBC driver (or appropriate driver) to your project’s `lib/` directory or Maven dependencies.
* ✅ Use your correct database URL, username, and password.
* ✅ Make sure your servlet is mapped correctly (`@WebServlet("/StudentServlet")`).
* ✅ Access the servlet: `http://localhost:8080/YourAppContext/StudentServlet`

---

Let me know if you want to do the same using **SQLite**, **DAO pattern**, or **JSTL tags** instead of scriptlets in JSP.
