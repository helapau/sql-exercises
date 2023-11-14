There are 3 tables containing information about reviewers' ratings of various movies.
The tables are:

1. Movie ( mID, title, year, director )
   There is a movie with ID number mID, a title, a release year, and a director.

2. Reviewer ( rID, name )
   The reviewer with ID number rID has a certain name.

3. Rating ( rID, mID, stars, ratingDate )
   The reviewer rID gave the movie mID a number of stars rating (1-5) on a certain ratingDate.

*Questions*
1. Find the titles of all movies directed by Steven Spielberg.
<details>
<summary>Answer</summary>

	select title from Movie 
	where director = 'Steven Spielberg';
</details>

2. Find all years that have a movie that received a rating of 4 or 5, and sort them in increasing order.
<details>
<summary>Answer</summary>

	select year from Movie 
	join Rating on 
	Movie.mId = Rating.mId	
	where stars > 3
	group by year
	order by year

</details>

3. Find the titles of all movies that have no ratings.
<details>
<summary>Answer</summary>

	Select Movie.title from Movie 
	LEFT JOIN Rating ON 
	Rating.mId = Movie.mId
	WHERE Rating.stars IS NULL;

or:

	SELECT title
	FROM Movie
	WHERE mID not in (SELECT mID FROM Rating);

</details>

4. Some reviewers didn't provide a date with their rating. Find the names of all reviewers who have ratings with a NULL value for the date.

<details>
<summary>Answer</summary>

	SELECT name from Reviewer
	JOIN Rating
	ON Rating.rId = Reviewer.rId
	WHERE Rating.ratingDate IS NULL

</details>


5. Write a query to return the ratings data in a more readable format: reviewer name, movie title, stars, and ratingDate. Also, sort the data, first by reviewer name, then by movie title, and lastly by number of stars.
<details>
<summary>Answer</summary>

	SELECT name as [reviewer name], title as [movie title], stars, ratingDate as [rating date]
	from Movie
	JOIN Rating on 
	Rating.mID = Movie.mId
	JOIN Reviewer ON 
	Reviewer.rId = Rating.rId
	ORDER BY [reviewer name], [movie title], stars;

</details>

6. For all cases where the same reviewer rated the same movie twice and gave it a higher rating the second time, return the reviewer's name and the title of the movie.
<details>
<summary>Answer</summary>

	SELECT name as reviewerName, title as movieTitle
	from Movie 
	JOIN Rating ON 
	Rating.mId = Movie.mId
	JOIN Reviewer ON 
	Rating.rId = Reviewer.rId
	JOIN Rating Rating2 on 
	Rating2.rId = Rating.rId and Rating2.mId = Rating.mID 
	WHERE Rating2.stars > Rating.stars AND 
	Rating2.ratingDate > Rating.ratingDate;

</details>

7. For each movie that has at least one rating, find the highest number of stars that movie received. Return the movie title and number of stars. Sort by movie title.

<details>
<summary>Answer</summary>

	SELECT DISTINCT title, MAX(Rating.stars) over(partition by Rating.mId) as maxStars
	FROM Rating 
	JOIN Movie
	ON Movie.mId = Rating.mID
	ORDER BY title;

</details>

8. For each movie, return the title and the 'rating spread', that is, the difference between highest and lowest ratings given to that movie. Sort by rating spread from highest to lowest, then by movie title.
<details>
<summary>Answer</summary>

	SELECT DISTINCT 
	title, 
	MAX(stars) over(partition by Movie.mId) - MIN(stars) over(partition by Movie.mId) as [rating spread]
	from Movie
	JOIN Rating ON Movie.mId = Rating.mID
	ORDER BY [rating spread] DESC, title;

</details>

9. Find the difference between the average rating of movies released before 1980 and the average rating of movies released after 1980. (Make sure to calculate the average rating for each movie, then the average of those averages for movies before 1980 and movies after. Don't just calculate the overall average rating before and after 1980.)
<details>
<summary>Answer</summary>

	SELECT AVG(before1980.avg - after1980.avg) 
	FROM (
		SELECT AVG(CAST(stars as FLOAT)) as avg, Rating.mID
		FROM Movie
		JOIN Rating ON Movie.mId = Rating.mID
		WHERE Movie.year < 1980
		GROUP BY Rating.mId
		-- AVG() is applied to each group separately!
	) as before1980,
	(
		SELECT AVG(CAST(stars as FLOAT)) as avg, Rating.mID
		FROM Movie
		JOIN Rating ON Movie.mId = Rating.mID
		WHERE Movie.year > 1980
		GROUP BY Rating.mId
	) as after1980

</details>

10. Find the name of all reviewers who rated Gone With the Wind.
<details>
<summary>Answer</summary>

    SELECT name
    FROM Reviewer
    JOIN Rating ON
    Rating.rId = Reviewer.rId
    JOIN Movie ON
    Movie.mId = Rating.mID
    WHERE title = 'Gone with the Wind'
    GROUP BY name;

</details>

11. For any rating where the reviewer is the same as the director of the movie, 
return the reviewer name, movie title, and number of stars. 
<details>
<summary>Answer</summary>
    
    SELECT Reviewer.name AS reviewerName,
    Movie.title AS movieTitle,
    Rating.stars
    from Rating
    JOIN Movie ON
    Rating.mID = Movie.mId
    JOIN Reviewer ON 
    Reviewer.rId = Rating.rId
    WHERE Reviewer.name = Movie.director;
</details>

12. Return all reviewer names and movie names together in a single list, alphabetized.
    (Sorting by the first name of the reviewer and first word in the title is fine;
    no need for special processing on last names or removing "The".) 
<details>
<summary>Answer</summary>

    (SELECT title as c from Movie) 
    UNION
    (SELECT name as c from Reviewer)
    ORDER BY c;
</details>

13. Find the titles of all movies not reviewed by Chris Jackson.
<details>
<summary>Answer</summary>

    SELECT title
    FROM Movie
    WHERE Movie.mId NOT IN (
        SELECT mID FROM Rating
        JOIN Reviewer ON
        Reviewer.rId = Rating.rId
        WHERE Reviewer.name = 'Chris Jackson'
    );
</details>

14. For all pairs of reviewers such that both reviewers gave a rating to the same movie, return the names of both reviewers.
    Eliminate duplicates, don't pair reviewers with themselves, and include each pair only once.
    For each pair, return the names in the pair in alphabetical order. 
<details>
<summary>Answer</summary>

    WITH cte (revName, revId, mId) AS (
    SELECT name as revName,
    Reviewer.rId,
    mID
    FROM
    Reviewer
    JOIN Rating ON
    Rating.rId = Reviewer.rId
    )
    
    SELECT T1.revName as reviewerA, T2.revName as reviewerB
    from cte T1, cte T2
    where T1.mId = T2.mId and
    -- this alphabetizes the pair
    T1.revName < T2.revName
    group by T1.revName, T2.revName;    
</details>
