# Part 2 - Use Case Questions (Total Points: 14)

```sql
USE ROLE DS5111_DBT;
USE DATABASE DS5111_SU24;
-- CREATE SCHEMA ATR8EC;

USE SCHEMA ATR8EC;
```

### 1) (1 PT) Which courses are currently included (active) in the program? Include the course mnemonic and course name for each.
```sql
SELECT course_id, title
FROM courses
WHERE is_active;
```


### 2) (1 PT) Which courses were included in the program, but are no longer active? Include the course mnemonic and course name for each.
```sql
SELECT course_id, title
FROM courses
WHERE NOT is_active;
```


### 3) (1 PT) Which instructors are not current employees?
```sql
SELECT instructor_id, instructor_name
FROM instructors
WHERE NOT is_active;
```


### 4) (1 PT) For each course (active and inactive), how many learning outcomes are there?
```sql
SELECT course_id, count(*) AS "Number of Learning Outcomes"
FROM learning_outcomes
GROUP BY 1;
```


### 5) (2 PTS) Are there any courses with no learning outcomes? If so, provide their mnemonics and names.
```sql
SELECT c.course_id, c.title
FROM courses c
LEFT JOIN learning_outcomes lo ON c.course_id = lo.course_id
WHERE lo.course_id IS NULL;
```


### 6) (2 PTS) Which courses include SQL as a learning outcome? Include the learning outcome descriptions, course mnemonics, and course names in your solution.
```sql
SELECT c.course_id, c.title, lo.description
FROM courses c
LEFT JOIN learning_outcomes lo ON c.course_id = lo.course_id
WHERE LOWER(lo.description) LIKE '%sql%';
```


### 7) (1 PT) Who taught course ds5100 in Summer 2021?
```sql
SELECT instructor_name
FROM instructor_assignments ins_a
LEFT JOIN instructors ins ON ins.instructor_id = ins_a.instructor_id
WHERE course_id = 'ds5100'
    AND term_semester = 'Summer'
    AND term_year = 2021;
    ```


### 8) (1 PT) Which instructors taught in Fall 2021? Order their names alphabetically, making sure the names are unique.
```sql
SELECT DISTINCT instructor_name
FROM instructor_assignments ins_a
LEFT JOIN instructors ins ON ins.instructor_id = ins_a.instructor_id
WHERE term_semester = 'Fall'
    AND term_year = 2021
ORDER BY 1 asc;
```


### 9) (1 PT) How many courses did each instructor teach in each term? Order your results by term and then instructor.
```sql
SELECT term_semester, term_year, instructor_name, count(*) AS "Number of Courses Taught"
FROM instructor_assignments ins_a
LEFT JOIN instructors ins ON ins.instructor_id = ins_a.instructor_id
GROUP BY 1,2,3
ORDER BY 2,1,3;
```


### 10)(a) (2 PTS) Which courses had more than one instructor for the same term? Provide the mnemonic and term for each. Note this occurs in courses with multiple sections.
```sql
SELECT DISTINCT course_id, term_semester, term_year
FROM instructor_assignments
WHERE course_section > 1
ORDER BY 1,2;

-- I realize the above solution is cheating a bit since it is convenient the way I set up the database, so I have provided an alternative below:
WITH counts AS (
    SELECT course_id, term_semester, term_year, count(*) AS num_courses
    FROM instructor_assignments
    GROUP BY 1,2,3
)
SELECT course_id, term_semester, term_year
FROM counts
WHERE num_courses > 1
ORDER BY 1,2;
```


### 10)(b) (1 PT) For courses with multiple sections, provide the term, course mnemonic, and instructor name for each. Hint: You can use your result from 10a in a subquery or WITH clause.
```sql
with mutli_sections AS (
    SELECT DISTINCT course_id, term_semester, term_year
    FROM instructor_assignments
    WHERE course_section > 1
)
SELECT ins_a.term_semester, ins_a.term_year, ins_a.course_id, ins.instructor_name
FROM instructor_assignments ins_a
INNER JOIN mutli_sections ms
    ON ins_a.term_semester = ms.term_semester
    AND ins_a.term_year = ms.term_year
    AND ins_a.course_id = ms.course_id
LEFT JOIN instructors ins ON ins_a.instructor_id = ins.instructor_id
ORDER BY 1,2,3,4;
```