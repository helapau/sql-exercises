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
<details>
<summary>Answer</summary>

    select H1.name, H1.grade, H2.name, H2.grade
    from Highschooler H1
    join ( select L1.ID1, L1.ID2
    from Likes L1
    where exists (
        select L2.ID2
        from Likes L2
        where L2.ID2 = L1.ID1
        and L2.ID1 = L1.ID2
    ) ) as MutualLikes
    on MutualLikes.ID1 = H1.ID
    join Highschooler H2 
    on H2.ID = MutualLikes.ID2
    where H1.name < H2.name

explanation:
MutualLikes where condition is looking for the mutual connection.
If Id1 likes Id2 (pair: Id1, Id2) then in L2 table must appear 
the pair (L1.id2, L1.id1).
</details>

4. Find all students who do not appear in the Likes table (as a student who likes or is liked) and return their names and grades. Sort by grade, then by name within each grade.
<details>
<summary>Answer</summary>

    select H.name, H.grade 
    from Highschooler H
    where H.ID not in (
        select ID1
        from Likes
        union
        select ID2
        from Likes
    )
    order by H.grade, H.name
</details>

5. For every situation where student A likes student B, but we have no information about whom B likes (that is, B does not appear as an ID1 in the Likes table), return A and B's names and grades.
<details>
<summary>Answer</summary>

    select H1.name as 'admirer', H1.grade,
    H2.name as 'crush', H2.grade
    from Highschooler H1
    join Likes L1
    on H1.ID = L1.ID1
    join Highschooler H2
    on H2.ID = L1.ID2
    where H2.ID not in (select id1 from Likes)
</details>


