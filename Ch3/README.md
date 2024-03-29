# Chapter 3: Solutions to Exercises

### 2019331019, 2019331065

## 3.1
### a. Find the titles of courses in the Comp. Sci. department that have 3 credits
__Query:__
``` sql
    select title from course
    where dept_name = "Comp. Sci." and credits = 3
```
__Result:__   
<img src = ".\images\3.1.a.png" class = "center" alt = "3.1.a_query">
   
### b.Find the IDs of all students who were taught by an instructor named Einstein; make sure there are no duplicates in the reesult.

__Query:__
``` sql
    select distinct takes.ID from takes, teaches, instructor
    where
        instructor.name = 'Einstein' and
        instructor.id = teaches.id and
        takes.course_id = teaches.course_id
```
__Result:__   
<img src = ".\images\3.1.b.png" class = "center" alt = "3.1.b_query">   

### c. Find the highest salary of any instructor.
__Query:__
``` sql
    select max(salary) as max_salary from instructor
```
__Result:__   
<img src = ".\images\3.1.c.png" class = "center" alt = "3.1.c_query">

### d. Find all instructors earning the highest salary (there may be more than one with the same salary).

__Query:__
``` sql
    select * from instructor
    where salary = (select max(salary) from instructor)
```
__Result:__   
<img src = ".\images\3.1.d.png" class = "center" alt = "3.1.d_query">

### e. Find the enrollment of each section that was offered in Fall 2017.
__Query:__
``` sql
    select course_id, sec_id, (
        select count(ID) from takes
            where takes.year = section.year
                and takes.sec_id = section.sec_id
                and takes.course_id = section.course_id
                and takes.semester = section.semester) 
            as enrollment
    from section
    where semester = 'Fall'
	and year = 2017
```
__Result:__   
<img src = ".\images\3.1.e.png" class = "center" alt = "3.1.e_query">

### f. Find the maximum enrollment, across all sections, in Fall 2017.
__Query:__
``` sql
    select max(enrollment) from 
        (select count(ID) as enrollment
        from section, takes
        where takes.year = section.year
            and takes.semester = section.semester
            and takes.sec_id = section.sec_id
            and takes.semester = 'Fall'
            and takes.year = 2017
            group by takes.course_id, takes.sec_id
        )
```
__Result:__   
<img src = ".\images\3.1.f.png" class = "center" alt = "3.1.f_query">

### g. Find the sections that had the maximum enrollment in Fall 2017.
__Query:__
``` sql
    WITH enrollment_in_fall_2017(course_id, sec_id, enrollment) AS (
    SELECT course_id, sec_id, COUNT(id) 
    FROM takes
    WHERE semester = 'Fall' AND year = 2017
    GROUP BY course_id, sec_id
) 
SELECT course_id, sec_id
FROM enrollment_in_fall_2017
WHERE enrollment = (SELECT MAX(enrollment) FROM enrollment_in_fall_2017);
```
__Result:__   
<img src = ".\images\3.1.g.png" class = "center" alt = "3.1.g_query">
---

## 3.2

### a.  Find the total grade points earned by the student with ID '12345', across all courses taken by the student.
__Query:__
``` sql
    (
    SELECT SUM(c.credits * g.points)
    FROM takes AS t 
        INNER JOIN course AS c ON t.course_id = c.course_id
        INNER JOIN grade_points AS g ON t.grade = g.grade
    WHERE t.ID = '12345'
    )
    UNION 
    (
    SELECT 0
    FROM student 
    WHERE ID = '12345' AND NOT EXISTS (SELECT * FROM takes WHERE ID = '12345')
    )
```
__Result:__   
(The relation grade_points deesn't exist on db-book.com)

### b. Find the grade point average (GPA) for the above student, that is, the total grade-points divided by the total credits for the associated courses.
__Query:__
``` sql
    UNION 
    (
        SELECT null AS GPA 
        FROM student 
        WHERE ID = '12345' AND 
            NOT EXISTS (SELECT * FROM takes WHERE ID='12345')
    )
```
__Result:__   

### c. Find the ID and the grade-point average of each student.
__Query:__
``` sql
    UNION 
    (
        SELECT ID, null AS GPA 
        FROM student 
        WHERE NOT EXISTS (SELECT * FROM takes WHERE takes.ID = student.ID)
    )
```
__Result:__   

---

## 3.3

### a. Increase the salary of each instructor in the Comp. Sci. department by 10%.   
__Query:__ 
``` sql
    UPDATE instructor
    SET salary = salary * 1.10
    WHERE dept_name = 'Comp. Sci.'
```

### b. Delete all courses that have never been offered (i.e., do not occur in the section relation).
__Query:__
``` sql
    DELETE FROM course
    WHERE course_id NOT IN (SELECT course_id FROM section)
```

### c. Insert every student whose tot_cred attribute is greater than 100 as an instructor in the same department, with a salary of $10,000.
__Query:__
``` sql
    INSERT INTO instructor
    SELECT ID, name, dept_name, 10000
    FROM student
    WHERE tot_cred > 100
```
---
## 3.4

### a. Find the total number of people who owned cars that were involved in accidents in 2017. 

__Query:__
``` sql
    SELECT COUNT(DISTINCT person.driver_id)
    FROM accident, participated, person, owns
    WHERE accident.report_number = participated.report_number
        AND owns.driver_id = person.driver_id 
        AND owns.license_plate = participated.license_plate
        AND year = 2017
```
__Result:__   
(database doesn't exist in db-book.com)

### b. Delete all year-2010 cars belonging to the person whose ID is '12345'.

__Query:__
``` sql
    DELETE FROM car
    WHERE year = 2010 AND license_plate IN 
        (SELECT license_plate FROM owns WHERE driver_id = '12345')
```
__Result:__   (database doesn't exist in db-book.com)    

---

## 3.5

### a. Display the grade for each student, based on the marks relation.
__Query:__
``` sql
    SELECT ID, 
        CASE
            WHEN score < 40 THEN 'F'
            WHEN score < 60 THEN 'C'
            WHEN score < 80 THEN 'B'
            ELSE 'A' 
        END
    FROM marks
```
__Result:__   
(the relation marks doesn't exist)


### b. Find the number of students with each grade.
__Query:__
``` sql
    WITH grades(ID,grade) AS (
        SELECT ID, 
            CASE
                WHEN score < 40 THEN 'F'
                WHEN score < 60 THEN 'C'
                WHEN score < 80 THEN 'B'
                ELSE 'A' 
            END
        FROM marks
    ) 
    SELECT grade, COUNT(ID)
    FROM grades
    GROUP BY grade
```   
---
## 3.6

__Query:__
``` sql
    SELECT dept_name
    FROM department
    WHERE LOWER(dept_name) LIKE '%sci%'
```
__Result:__   
<img src = ".\images\3.6.png" class = "center" alt = "3.6_query">
---

## 3.8
(database doensn't exist in db-book.com)
### a. Find the ID of each customer of the bank who has an account but not a loan.

__Query:__
``` sql
    (SELECT ID FROM depositor)
    EXCEPT 
    (SELECT ID FROM borrower)
```
__Result:__   

### b. Find the ID of each customer who lives on the same street and in the same city as customer '12345'

__Query:__
``` sql
    SELECT F.ID
    FROM customer AS F, customer AS S
    WHERE F.customer_street = S.customer_street
        AND F.customer_city = S.customer_city
        AND S.customer_id = '12345';
```
__Result:__   

### c. Find the name of each branch that has at least one customer who has an account in the bank and who lives in "Harrison".
__Query:__
``` sql
    SELECT DISTINCT branch_name
    FROM account, depositor, customer 
    WHERE customer.id = depositor.id
        AND depositor.account_number = account.account_number 
        AND customer_city = 'Harrison'
```
__Result:__   

---

## 3.9

### a. Find the ID, name, and city of residence of each employee who works for "First Bank Corporation".

__Query:__
``` sql
    SELECT e.ID, e.person_name, city
    FROM employee AS e, works AS w
    WHERE w.company_name = 'First Bank Corporation' AND w.ID = e.ID
```

### b. Find the ID, name, and city of residence of each employee who works for "First Bank Corporation" and earns more than $10000.

__Query:__
``` sql
    SELECT ID, name, city
    FROM employee 
    WHERE ID IN (
        SELECT ID
        FROM works
        WHERE company_name = 'First Bank Corporation' AND salary > 10000
    ) 
```

### c. Find the ID of each employee who does not work for "First Bank Corporation".  
__Query:__
``` sql
    SELECT ID
    FROM works
    WHERE company_name <> 'First Bank Corporation' 
```

### d. Find the ID of each employee who earns more than every employee of "Small Bank Corporation".
__Query:__
``` sql
    SELECT ID
    FROM works
    WHERE salary > ALL (
        SELECT salary
        FROM works
        WHERE company_name = 'Small Bank Corporation'
    )
```

### e. Assume that companies may be located in several cities. Find the name of each company that is located in every city in which "Small Bank Corporation" is located.

__Query:__
``` sql
    SELECT S.company_name 
    FROM company AS S 
    WHERE NOT EXISTS (
        (
            SELECT city
            FROM company
            WHERE company_name = 'Small Bank Corporation'
        )
        EXCEPT
        (
            SELECT city
            FROM company AS T
            WHERE T.company_name = S.company_name
        )
    )
```

### f. Find the name of the company that has the most employees (or companies, in the case where there is a tie for the most).

__Query:__
``` sql
    SELECT company_name 
    FROM works
    GROUP BY company_name
    HAVING COUNT(DISTINCT ID) >= ALL (
        SELECT COUNT(DISTINCT ID)
        FROM works
        GROUP BY company_name
    )
```

### g. Find the name of each company whose employees earn a higher salary, on average, than the average salary at "First Bank Corporation".  

__Query:__
``` sql
    SELECT company_name
    FROM works
    GROUP BY company_name 
    HAVING AVG(salary) >  (
        SELECT AVG(salary)
        FROM works
        WHERE company_name = 'First Bank Corporation'
    )
```   
---

## 3.10
### a. Modify the database so that the employee whose ID is '12345' now lives in "Newtown".

__Query:__
``` sql
    UPDATE employee
    SET city = 'Newtown'
    WHERE ID = '12345' 
```

### b. Give each manager of "First Bank Corporation" a 10 percent raise unless the salary becomes greater than $100000; in such cases, give only a 3 percent raise.

__Query:__
``` sql
    UPDATE works T
    SET T.salary = T.salary * 1.03
    WHERE T.ID IN (SELECT manager_id FROM manages)
        AND T.salary * 1.1 > 100000
        AND T.company_name = 'First Bank Corporation';

    UPDATE works T
    SET T.salary = T.salary * 1.1
    WHERE T.ID IN (SELECT manager_id FROM manages)
        AND T.salary * 1.1 <= 100000
        AND T.company_name = 'First Bank Corporation';
```
---

## 3.11

### a. Find the ID and name of each student who has taken at least one Comp. Sci. course; make sure there are no duplicates names in the result.
__Query:__
``` sql
    SELECT DISTINCT student.ID, student.name
    FROM student INNER JOIN takes  ON student.ID = takes.ID 
                INNER JOIN course ON takes.course_id = course.course_id
    WHERE course.dept_name = 'Comp. Sci.';
```
__Result:__   
<img src = ".\images\3.11.a.png" class = "center" alt = "3.11.a_query">

### b. Find the ID and name of each student who has not taken any course offered before 2017.
__Query:__
``` sql
    SELECT ID, name 
    FROM student AS S
    WHERE NOT EXISTS (
        SELECT * 
        FROM takes AS T
        WHERE year < 2017 AND T.ID = S.ID 
    )
```
__Result:__   
<img src = ".\images\3.11.b.png" class = "center" alt = "._query">

### c. For each department, find the maximum salary of instructors in that department. You may assume that every department has at least one instructor.
__Query:__
``` sql
    SELECT dept_name, MAX(salary)
    FROM instructor
    GROUP BY dept_name 
```
__Result:__   
<img src = ".\images\3.11.c.png" class = "center" alt = "3.11.c_query">

### d. Find the lowest, across all departments, of the per-department maximum salary computed by the preceding query.
__Query:__
``` sql
    WITH maximum_salary_within_dept(dept_name, max_salary) AS (
        SELECT dept_name, MAX(salary)
        FROM instructor
        GROUP BY dept_name 
    ) 
    SELECT MIN(max_salary) 
    FROM maximum_salary_within_dept
```
__Result:__   
<img src = ".\images\3.11.d.png" class = "center" alt = "3.11.d_query">
---

## 3.12

### a. Create a new course "CS-001", titled "Weekly Seminar", with 0 credits.
__Query:__
``` sql
    INSERT INTO course (course_id, title,dept_name, credits) 
    VALUES ('CS-001','Weekly Seminar','Comp. Sci.', 0);
```

### b. Create a section of this course in Fall 2017, with sec_id of 1, and with the location of this section not yet specified.
__Query:__
``` sql
    INSERT INTO section (course_id, sec_id, semester, year) VALUES 
    ('CS-001', '1', 'Fall', 2017)
```

### c. Enroll every student in the Comp. Sci. department in the above section.
__Query:__
``` sql
    INSERT INTO takes (id, course_id, sec_id, semester, year) 
    SELECT student.id,'CS-001', '1', 'Fall', 2017
    FROM student 
    WHERE student.dept_name = 'Comp. Sci.'
```

### d. Delete enrollments in the above section where the student's ID is 12345.
__Query:__
``` sql
    DELETE FROM takes 
    WHERE ID = '12345' AND (course_id, sec_id, semester, year) = ('CS-001', '1', 'Fall', 2017)
```

### e. Delete the course CS-001. What will happen if you run this delete statement without first deleting offerings (sections) of this course? 
__Query:__
``` sql
    DELETE FROM course
    WHERE course_id = 'CS-001'; 
```

### f. Delete all takes tuples corresponding to any section of any course with the word "advanced" as a part of the title; ignore case when matching the word with the title.    
__Query:__
``` sql
    DELETE FROM takes
    WHERE course_id IN (
        SELECT course_id
        FROM course
        WHERE LOWER(title) LIKE '%advanced%'
    )
```
---

## 3.13


### template   
__Query:__
``` sql
    CREATE TABLE person (
        driver_id VARCHAR(15),
        name VARCHAR(30) NOT NULL, 
        address VARCHAR(40), 
        PRIMARY KEY (driver_id)
    );

    CREATE TABLE car (
        license_plate VARCHAR(8), 
        model VARCHAR(7), 
        year NUMERIC(4,0) CHECK (year > 1701 AND year < 2100), 
        PRIMARY KEY (license_plate)
    );

    CREATE TABLE accident ( 
        report_number VARCHAR(10), 
        year NUMERIC(4,0) CHECK (year > 1701 AND year < 2100),
        location VARCHAR(30), 
        PRIMARY KEY (report_number)
    );

    CREATE TABLE owns (
        driver_id VARCHAR(15),
        license_plate VARCHAR(8),
        PRIMARY KEY (driver_id, license_plate), 
        FOREIGN KEY (driver_id) REFERENCES person(driver_id) 
            ON DELETE CASCADE, 
        FOREIGN KEY (license_plate) REFERENCES car(license_plate)
            ON DELETE CASCADE
    );

    CREATE TABLE participated ( 
        report_number VARCHAR(10), 
        license_plate VARCHAR(8), 
        driver_id VARCHAR(15),
        damage_amount NUMERIC(10,2),
        PRIMARY KEY(report_number, license_plate),
        FOREIGN KEY (report_number) REFERENCES accident(report_number), 
        FOREIGN KEY (license_plate) REFERENCES car(license_plate)
    );
```
---

## 3.14

### a. Find the number of accidents involving a car belonging to a person named "John Smith".
__Query:__
``` sql
    WITH all_cars_owned_by_john_smith(license_plate) AS (
        SELECT license_plate 
        FROM person INNER JOIN owns ON person.driver_id = owns.driver_id
        WHERE person.name = 'John Smith'
    )
    SELECT COUNT(DISTINCT report_number)
    FROM participated 
    WHERE license_plate IN (SELECT license_plate FROM all_cars_owned_by_john_smith);
```


### b. Update the damage amount for the car with license_plate "AABB2000" in the accident with report number "AR2197" to $3000.
__Query:__
``` sql
    UPDATE participated 
    SET damage_amount = 3000
    WHERE report_number = 'AR2197' AND license_plate = 'AABB2000';
```
----

## 3.15
### a. Find each customer who has an account at every branch located in "Brooklyn"
__Query:__
``` sql
    WITH all_branches_in_brooklyn(branch_name) AS (
        SELECT branch_name 
        FROM branch
        WHERE branch_city = 'Brooklyn'
    )
    SELECT ID, customer_name 
    FROM customer AS c1
    WHERE NOT EXISTS (
        (SELECT branch_name FROM all_branches_in_brooklyn)
        EXCEPT
        (
            SELECT branch_name
            FROM account INNER JOIN depositor 
                ON account.account_number = depositor.account_number
            WHERE depositor.ID = c1.ID
        )
    )
```

### b. Find the total sum of all loan amounts in the bank.
__Query:__
``` sql
    SELECT SUM(amount)
    FROM loan
```

### c. Find the names of all branches that have assets greater than those of at least one branch located in "Brooklyn".
__Query:__
``` sql
    SELECT branch_name
    FROM branch
    WHERE assets > SOME (
        SELECT assets
        FROM branch
        WHERE branch_city = 'Brooklyn'
    );
```
---

## 3.16

### a. Find ID and name of each employee who lives in the same city as the location of the company for which the employee works.
__Query:__
``` sql
    SELECT employee.id, employee.person_name
    FROM employee INNER JOIN works ON employee.id = works.id
                INNER JOIN company ON works.company_name = company.company_name
    WHERE employee.city = company.city
```

### b. Find ID and name of each employee who lives in the same city and on the same street as does her or his manager.   
__Query:__
``` sql
    SELECT E.id, E.person_name
    FROM employee AS E INNER JOIN manages ON E.id = manages.id
                    INNER JOIN employee AS manager_of_E ON manages.manager_id = manager_of_E.id
    WHERE E.street = manager_of_E.street AND 
        E.city = manager_of_E.city;
```

### c. Find ID and name of each employee who earns more than the average salary of all employees of her or his company.
__Query:__
``` sql
    WITH average_salary_per_company(company_name, avg_salary) AS (
        SELECT company_name, AVG(salary) 
        FROM works
        GROUP BY company_name
    ) 
    SELECT E.id, E.person_name
    FROM employee AS E INNER JOIN works ON E.id = works.id
    WHERE works.salary > (
        SELECT avg_salary 
        FROM average_salary_per_company 
        WHERE company_name = works.company_name
    )
```

### d. Find the company that has the smallest payroll.
__Query:__
``` sql
    SELECT company_name, SUM(salary) AS total_payroll
    FROM works
    GROUP BY company_name
    ORDER BY total_payroll ASC
    LIMIT 1
```
---

## 3.17

### a. Give all employees for "First Bank Corporation" a 10 percent raise.   
__Query:__
``` sql
    UPDATE works 
    SET salary = salary * 1.1
    WHERE company_name = 'First Bank Corporation'
```

### b. Give all managers of "First Bank Corporation" a 10 percent raise.
__Query:__
``` sql
    UPDATE works
    SET salary = salary * 1.1
    WHERE company_name = 'First Bank Corporation' AND id IN (
        SELECT manager_id
        FROM manages
    )
```

### c. Delete all tuples in the works relation for employees of "Small Bank Corporation".

__Query:__
``` sql
    DELETE FROM works
    WHERE company_name = 'Small Bank Corporation'
```
---

## 3.18
   
__Query:__
``` sql
    CREATE TABLE employee ( 
        id VARCHAR(8),
        person_name VARCHAR(30) NOT NULL, 
        street VARCHAR(40), 
        city VARCHAR(30),
        PRIMARY KEY (id)
    ); 

    CREATE TABLE company ( 
        company_name VARCHAR(40), 
        city VARCHAR(30), 
        PRIMARY KEY (company_name)
    ); 

    CREATE TABLE works ( 
        id VARCHAR(8), 
        company_name VARCHAR(40), 
        salary NUMERIC(10,2) CHECK (salary > 10000), 
        PRIMARY KEY (id), 
        FOREIGN KEY (id) REFERENCES employee(id)
            ON DELETE CASCADE, 
        FOREIGN KEY (company_name) REFERENCES company(company_name)
            ON DELETE CASCADE
    );

    CREATE TABLE manages (
        id VARCHAR(8),
        manager_id VARCHAR(8), 
        PRIMARY KEY (id), 
        FOREIGN KEY (id) REFERENCES employee (id), 
        FOREIGN KEY (manager_id) REFERENCES employee (id)
    );
```
---
### 3.19. List two reasons why null values might be introduced into the database  
__Query:__
``` sql
```
__Result:__   
<img src = ".\images\3.19.PNG" class = "center" alt = "3.19._query">

<p> <font size = "4">3.21 Consider the library database of Figure 3.20. Write the following queries in SQL.</font> <p>

### a. Find the member number and name of each member who has borrowed at least one book published by “McGraw-Hill”. 
__Query:__
``` sql
    SELECT memb_no, name
    FROM member AS m
    WHERE EXISTS (
    SELECT *
    FROM book INNER JOIN borrowed ON book.isbn = borrowed.isbn
    WHERE book.publisher = 'McGraw-Hill' AND borrowed.memb_no = m.memb_no
)

```
__Result:__   
<!-- <img src = ".\images\3.21.a.png" class = "center" alt = "3.21.a._query"> -->

### b. Find the member number and name of each member who has borrowed every book published by “McGraw-Hill”   
__Query:__
``` sql
    SELECT memb_no, name
    FROM member AS m
    WHERE NOT EXISTS (
    (
    SELECT isbn
    FROM book
    WHERE publisher = 'McGraw-Hill'
    )
    EXCEPT
    (
    SELECT isbn
    FROM borrowed
    WHERE memb_no = m.memb_no
    )
    )
```
__Result:__   
<!-- <img src = ".\images\3.21.b.png" class = "center" alt = "3.21.b._query"> -->

### c. For each publisher, find the member number and name of each member who has borrowed more than five books of that publisher   
__Query:__
``` sql
    WITH member_borrowed_book(memb_no,
    memb_name,isbn,title,authors,publisher,date) AS (
    SELECT member.memb_no, name, book.isbn, title, authors, publisher,
    date
    FROM member INNER JOIN borrowed ON member.memb_no =
    borrowed.memb_no
    INNER JOIN book ON borrowed.isbn = book.isbn
    )
    SELECT memb_no, memb_name, publisher, COUNT(isbn)
    FROM member_borrowed_book
    GROUP BY memb_no, memb_name, publisher
    HAVING COUNT(isbn) > 5;
```
__Result:__   
<!-- <img src = ".\images\3.21.c.png" class = "center" alt = "3.21.c._query"> -->

### d. Find the average number of books borrowed per member. Take into account that if a member does not borrow any books, then that member does not appear in the borrowed relation at all, but that member still counts in the average.   
__Query:__
``` sql
    WITH number_of_books_borrowed(memb_no, memb_name,
    number_of_books) AS (
    SELECT memb_no, name, (
    CASE
    WHEN NOT EXISTS (SELECT * FROM borrowed WHERE
    borrowed.memb_no = member.memb_no) THEN 0
    ELSE (SELECT COUNT(*) FROM borrowed WHERE
    borrowed.memb_no = member.memb_no)
    END
    )
    FROM member
    )
    SELECT AVG(number_of_books) AS
    average_number_of_books_borrowed_per_member
    FROM number_of_books_borrowed
```
__Result:__   
<!-- <img src = ".\images\3.21.d.png" class = "center" alt = "3.21.d._query"> -->

### 3.22 Rewrite the where clause where unique (select title from course) without using the unique construct.   
__Query:__
``` sql
    SELECT *
    FROM course 
    WHERE title IN (
    SELECT title
    FROM course
    GROUP BY title
    HAVING COUNT(*) = 1
);

```
__Result:__   
<img src = ".\images\3.22.PNG" class = "center" alt = "3.22._query">

### 3.23 Consider the query:
```
    with dept total (dept name, value) as
    (select dept name, sum(salary)
    from instructor
    group by dept name),
    dept total avg(value) as
    (select avg(value)
    from dept total)
    select dept name
    from dept total, dept total avg
    where dept total.value >= dept total avg.value;
```
### Rewrite this query without using the with construct.  
__Query:__
``` sql
    SELECT dept_name
    FROM (SELECT dept_name, SUM(salary) AS value FROM instructor
    GROUP BY dept_name) AS dept_total,
    (SELECT AVG(value) AS value FROM (SELECT dept_name,
    SUM(salary) AS value FROM instructor GROUP BY dept_name)) AS
    dept_total_avg
    WHERE dept_total.value >= dept_total_avg.value
```
__Result:__   
<img src = ".\images\3.23.PNG" class = "center" alt = "3.23._query">

### 3.24. Using the university schema, write an SQL query to find the name and ID of those Accounting students advised by an instructor in the Physics department.   
__Query:__
``` sql
    SELECT s.id, s.name
    FROM student AS s
    INNER JOIN advisor AS a ON s.id = a.s_id
    INNER JOIN instructor AS i ON a.i_id = i.id
    WHERE s.dept_name = 'Accounting' AND i.dept_name = 'Physics';
```
__Result:__   
<img src = ".\images\3.24.PNG" class = "center" alt = "3.24._query">

### 3.25. Using the university schema, write an SQL query to find the names of those departments whose budget is higher than that of Philosophy. List them in alphabetic order.  
__Query:__
``` sql
    SELECT dept_name
    FROM department
    WHERE budget > (SELECT budget FROM department WHERE
    dept_name = 'Philosophy')
    ORDER BY dept_name ASC;
```
__Result:__   
<img src = ".\images\3.25.PNG" class = "center" alt = "3.25._query">

### 3.26. Using the university schema, use SQL to do the following: For each student who has retaken a course at least twice (i.e., the student has taken the course at least three times), show the course ID and the student's ID. Please display your results in order of course ID and do not display duplicate rows
__Query:__
``` sql
    WITH retakers AS (
        SELECT t1.id, t1.course_id
        FROM takes t1
        JOIN takes t2 ON t1.id = t2.id AND t1.course_id = t2.course_id
        WHERE t1.grade IS NOT NULL AND t2.grade IS NOT NULL
        GROUP BY t1.id, t1.course_id
        HAVING COUNT(*) >= 3
        SELECT DISTINCT r.course_id, r.id
        FROM retakers r
        ORDER BY r.course_id, r.id;
    )
```
__Result:__   
<img src = ".\images\3.26.PNG" class = "center" alt = "3.26._query">

### 3.27. Using the university schema, write an SQL query to find the IDs of those students who have retaken at least three distinct courses at least once (i.e, the student has taken the course at least two times).
__Query:__
``` sql
    WITH retakers(id,course_id,frequency) AS (
        SELECT id,course_id,COUNT(*)
        FROM takes
        GROUP BY id,course_id
        HAVING COUNT(*) > 1
    )
    SELECT id
    FROM retakers
    GROUP BY id
    HAVING COUNT(*) >= 3;
```
__Result:__   
<img src = ".\images\3.27.PNG" class = "center" alt = "3.37._query">

### 3.28. Using the university schema, write an SQL query to find the names and IDs of those instructors who teach every course taught in his or her department (i.e., every course that appears in the course relation with the instructor’s department name). Order result by name.   
__Query:__
``` sql
    SELECT ID, name
    FROM instructor
    WHERE NOT EXISTS (
        SELECT course_id
        FROM course
        WHERE dept_name = instructor.dept_name
        EXCEPT
        SELECT course_id
        FROM teaches
        WHERE teaches.ID = instructor.ID
    )
    ORDER BY name;
```
__Result:__   
<img src = ".\images\3.28.PNG" class = "center" alt = "3.28._query">

### 3.29. Using the university schema, write an SQL query to find the name and ID of each History student whose name begins with the letter 'D' and who has not taken at least five Music courses.   
__Query:__
``` sql
    SELECT S.ID, S.name
    FROM student S
    JOIN takes T ON S.ID = T.ID
    JOIN course C ON T.course_id = C.course_id
    WHERE S.dept_name = 'History'
    AND S.name LIKE 'D%'
    AND C.dept_name = 'Music'
    GROUP BY S.ID, S.name
    HAVING COUNT(DISTINCT T.course_id) < 5;

```
__Result:__   
<img src = ".\images\3.29.PNG" class = "center" alt = "3.29._query">

### 3.30 Consider the following SQL query on the university schema:
```
select avg(salary)-(sum(salary) / count(*))
from instructor
```
### We might expect that the result of this query is zero since the average of a set of numbers is defined to be the sum of the numbers divided by the number of numbers. Indeed this is true for the example instructor relation in Figure 2.1. However, there are other possible instances of that relation for which the result would not be zero. Give one such instance, and explain why the result would not be zero.   
__Query:__
``` sql
```
__Result:__   
<img src = ".\images\3.30.PNG" class = "center" alt = "3.30._query">

### 3.31 Using the university schema, write an SQL query to find the ID and name of each instructor who has never given an A grade in any course she or he has taught. (Instructors who have never taught a course trivially satisfy this condition.)  
__Query:__
``` sql
    SELECT id, name
    FROM instructor AS i
    WHERE 'A' NOT IN (
        SELECT takes.grade
        FROM takes INNER JOIN teaches
        ON (takes.course_id,takes.sec_id,takes.semester,takes.year) =
        (teaches.course_id,teaches.sec_id,teaches.semester,teaches.year)
        WHERE teaches.id = i.id
    )
```
__Result:__   
<img src = ".\images\3.31.PNG" class = "center" alt = "3.31._query">

### 3.32 Rewrite the preceding query, but also ensure that you include only instructors who have given at least one other non-null grade in some course.   
__Query:__
``` sql
    SELECT id, name
    FROM instructor AS i
    WHERE 'A' NOT IN (
    SELECT takes.grade
    FROM takes INNER JOIN teaches
    ON (takes.course_id,takes.sec_id,takes.semester,takes.year) =
    (teaches.course_id,teaches.sec_id,teaches.semester,teaches.year)
    WHERE teaches.id = i.id
    )
    AND
    (
    SELECT COUNT(*)
    FROM takes INNER JOIN teaches
    ON (takes.course_id,takes.sec_id,takes.semester,takes.year) =
    (teaches.course_id,teaches.sec_id,teaches.semester,teaches.year)
    WHERE teaches.id = i.id AND takes.grade IS NOT NULL
    ) >= 1
```
__Result:__   
<img src = ".\images\3.32.PNG" class = "center" alt = "3.32._query">

### 3.33 Using the university schema, write an SQL query to find the ID and title of each course in Comp. Sci. that has had at least one section with afternoon hours (i.e., ends at or after 12:00). (You should eliminate duplicates if any.)  
__Query:__
``` sql
    SELECT course_id,title
    FROM course AS c
    WHERE dept_name = 'Comp. Sci.' AND
    EXISTS (
    SELECT *
    FROM section
    WHERE section.course_id = c.course_id AND
    time_slot_id IN (SELECT time_slot_id FROM time_slot WHERE
    end_hr >= 12)
    )
```
__Result:__   
<img src = ".\images\3.33.PNG" class = "center" alt = "3.33._query">

### 3.34 Using the university schema, write an SQL query to find the number of students in each section. The result columns should appear in the order “courseid, secid, year, semester, num”. You do not need to output sections with 0 students.  
__Query:__
``` sql
    SELECT course_id, sec_id, year, semester, COUNT(DISTINCT ID) AS
    num
    FROM takes
    GROUP BY course_id, sec_id, year, semester;
```
__Result:__   
<img src = ".\images\3.34.PNG" class = "center" alt = "3.34._query">

### 3.35 Using the university schema, write an SQL query to find section(s) with maximum enrollment. The result columns should appear in the order “courseid, secid, year, semester, num”. (It may be convenient to use the with construct.)  
__Query:__
``` sql
    WITH section_student_frequency(courseid, sec_id, year, semester, num)
    AS (
    SELECT course_id, sec_id, year, semester, COUNT(DISTINCT ID)
    FROM takes
    GROUP BY course_id, sec_id, year, semester
    )
    SELECT *
    FROM section_student_frequency
    WHERE num = (SELECT MAX(num) FROM section_student_frequency);
```
__Result:__   
<img src = ".\images\3.35.PNG" class = "center" alt = "3.35._query">

### template   
__Query:__
``` sql
```
__Result:__   
<img src = ".\images\.PNG" class = "center" alt = "._query">
