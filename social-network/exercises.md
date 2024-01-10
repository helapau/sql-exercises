Students at your hometown high school have decided to organize their social network using databases. So far, they have collected information about sixteen students in four grades, 9-12. Here's the schema:

Highschooler ( ID, name, grade )

Friend ( ID1, ID2 )
The student with ID1 is friends with the student with ID2. Friendship is mutual, so if (123, 456) is in the Friend table, so is (456, 123).

Likes ( ID1, ID2 )
The student with ID1 likes the student with ID2. Liking someone is not necessarily mutual, so if (123, 456) is in the Likes table, there is no guarantee that (456, 123) is also present.

* Questions*
1. Find the names of all students who are friends with someone named Gabriel.
<details>
<summary>Answer</summary>

    select name 
    from Highschooler
    join Friend 
    on Friend.ID1 = Highschooler.ID 
    where Friend.ID2 IN (
        select ID 
        from Highschooler
        where name = 'Gabriel'
    )
</details>

2. For every student who likes someone 2 or more grades younger than themselves, return that student's name and grade, and the name and grade of the student they like.
<details>
<summary>Answer</summary>

    
    select H1.ID, H1.name, H2.ID, H2.name
    from Highschooler H1, 
    Highschooler H2
    WHERE H1.ID in (
        select ID1 from Likes
        where ID2 = H2.ID
    ) and H2.grade in (H1.grade - 2, H1.grade - 3)
</details>

3. For every pair of students who both like each other, return the name and grade of both students. Include each pair only once, with the two names in alphabetical order.


