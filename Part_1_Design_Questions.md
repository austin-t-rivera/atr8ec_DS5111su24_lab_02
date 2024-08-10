# Part 1 - Design Questions (Total Points: 20)

### 1) (3 PTS) What tables should you build?
- `learning_outcomes`
- `courses`
- `instructors`
- `instructor_assignments`

### 2) (2 PTS) For each table, what field(s) will you use for primary key?
- learning_outcomes: `learning_outcome_id` (int)
- courses: `course_id` (varchar: the course mnemonic)
- instructors: `instructor_id` (int) (assumption that two people could have the same name)
- instructor_assignments: `assignment_id` (int)

### 3) (2 PTS) For each table, what foreign keys will you use?
- learning_outcomes: `course_id`
- courses: `N/A`
- instructors: `N/A`
- instructor_assignments: `course_id`, `instructor_id`

### 4) (2 PTS) Learning outcomes, courses, and instructors need a flag to indicate if they are currently active or not. How will your database support this feature? In particular:

I have added an `is_active` boolean to the `courses`, `instructors`, and `learning_outcomes` tables. While I considered naming these fields in a table-specific way to be more clear, I felt that it caused confusion, and the simplicity of a consistent is_active boolean made the most sense.

### 6) (1 PT) Is there anything to normalize in the database, and if so, how will you normalize it? Recall the desire to eliminate redundancy.

Yes, by seperating the instructors and the courses into separate tables, we are able to minimize redundancy for instructors who have taught a course over multiple terms.

### 8) (1 PT) Are there indexes that you should build? Explain your reasoning.

Yes, indexes should be build for:
- `instructor_id` in case there are multiple lecturers with the same name.
- `learning_outcome_id` because the uniqueness of the course_id and description are too large to act as the primary key.
- `assignment_id` because the uniquness of all the other fields would also be too large to concatenate and for the many instances.

### 7) (2 PTS) Are there constraints to enforce? Explain your answer and strategy.

For example, these actions should not be allowed:
- Entering learning objectives for a course not offered by the School of Data Science
  - In this instance, adding a field to the `course` table to denote the school within UVA the course is associated with
  - Or there could be another table that separately links the `course_id` and `school_id`
  - Additionally, looking at the `is_active` field within courses and checking if the course exists
  - Database triggers could be used to reference the aforementioned fields or table before allowing the data to go in 
- Assigning an invalid instructor to a course
  - Database triggers could be used to reference:
    - The `instructors` table to check if the instructor is in the table
    - The `is_active` field if they are in the table
    - Potentially a new field or table that assins instructors to course elligibility, school association, etc.
- Cannot reuse or duplicate course_id (course mnemonic)
  - There should be rules to avoid duplication
  - Course that are marked as False for `is_active` are not allowed to be copied
  - Only after a course and its associated data in the other tables is completely deleted can the mnemonic be reused
  - This helps to maintain consistency and avoid confusion
  - An alternative to deleting the old version is to change the original `course_id` to denote an older version
      - For example, if "ds5001" were to be dropped, but then repurposed for a different class, you could maintain the old information by renaming it to "deprecated_ds5001_001", and make sure that all tables are updated with the new name. Then you could reuse it.

### 8) (5 PTS) Draw and submit a Relational Model for your project. For an example, see Beginning Database Design Solutions page 115 Figure 5-28.
![DS_5111_Lab_2_EER.png](attachment:1bb76c94-fe67-4d39-a06b-ca8476cdbb67.png)

### 9) (2 PTS) Suppose you were asked if your database could also support the UVA SDS Residential MSDS Program. Explain any issues that might arise, changes to the database structure (schema), and new data that might be needed. Note you won’t actually need to support this use case for the project.

I would say that techincally the design of this database could support UVA SDS Residential MSDS Program in terms of holding the information as it would the Online Program data. However, if the desire is to be able to differentiate and filter on the type of program, then this would require either an additional field or additional table but there is a consideration:
- If the courses are unique to a single program:
    - Additional Field: Add a `program` field to the `courses` table
- If the courses are not unique to a single program:
    - Additional Field: Add a `program` field to the `instructor_assignments` table. We could do this because we already have the sections field. From a useability standpoint, this would likely be preferred. From a Normal Form standpoint, I could see an arguement to store this information else where.
    - Additioanl Table: Which leads to having a `course_sections` table where a course_section_id would be used as the Primary Key, the `course_id` would be used as a Foreign Key, and the `section_number` and `program` fields would be stored.
