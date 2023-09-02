# Chapter 3: Solutions to Exercises
<p> <font size = "4"> Write the following queries in SQL, using the university schema. (We suggest you actually run these queries on a database, using the sample data that we provide on the web site of the book, db-book.com. Instructions for setting up a database and loading sample data, are provided on the above web site.) </font> <p>

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
    
```
__Result:__   
<img src = ".\images\3.png" class = "center" alt = "3._query">

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
