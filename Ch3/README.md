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

### template   
__Query:__
``` sql
```
__Result:__   
<img src = ".\images\.png" class = "center" alt = "._query">