# Netflix_Postgressql_project
Netflix Data Analysis with PostgreSQL – Unlocking Streaming Insights 
![netflix_logo](https://raw.githubusercontent.com/krutika123-ak/Netflix_Postgressql_project/refs/heads/main/netflix%20logo.webp)
# overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.


# Objective:
1.Analyze the distribution of content types (movies vs TV shows).
2.Identify the most common ratings for movies and TV shows.
3.List and analyze content based on release years, countries, and durations.
4.Explore and categorize content based on specific criteria and keywords.
# Dataset
The data for this project is sourced from the Kaggle dataset:
Dataset link:(https://github.com/krutika123-ak/Netflix_Postgressql_project/blob/main/netflix_titles.csv)

# schema
```sql
create table netflix(show_id varchar(6),
	type varchar(10),
	title varchar(150),
	director varchar(208),
	casts varchar(1000),
	country	 varchar(150),
    date_added varchar(50),
	release_year	int,
    rating	varchar(10),
    duration varchar(15),
	listed_in varchar(100),
	description varchar(250) );
```

	select * from netflix;


	select count(*) as total_count 
	from netflix;

	-- 15 business problems to solve 
	
--1. count the number of movies and tv shows 
	select distinct type ,
	count(*) as total_count from netflix
	group by type;


	
-- 2 . find the most common rating for movies and tv shows 

	select type,count(*) as total_content
	from netflix 
	group by type

select type , rating from
	(select type,rating,count(*),
	rank() over(partition by type order by count(*) desc) as ranking
	from netflix
	group by 1,2) as t1
	where ranking=1

-- 3 . list all the movies which released in a specific year (eg:2020)

	select show_id, title,release_year
	from netflix 
	where type='Movie' and release_year=2020;

-- 4. find the top 5 countries with the most content on netflix
	select * from netflix

	select 
	unnest(string_to_array(country,','))as new_country,
	count(show_id)as total_content
	from netflix
	group by 1
	order by 2 desc
	limit 5;

-- 5. identify the longest movie 

	select * from netflix 
	where type='Movie' and duration =(select max(duration)from netflix)

-- 6 . find the content added in last 5 years
	select * from netflix 
	where to_date(date_added ,'month DD,YYYY')>= current_date-interval '5 years'

-- 7.FIND ALL THE MOVIES/TV SHOWS BY DIRECTOR 'RAJIV CHILAKA'

	SELECT * FROM NETFLIX WHERE DIRECTOR ilike '%Rajiv Chilaka%'

--8. list all the tvshows with more than 5 seasons
select * from netflix where type ='TV Show' and  split_part(duration,' ',1)::numeric > 5 


-- 9. count the no. of content items in each genre
select unnest(string_to_array(listed_in,',')) as genre,
count(show_id) as total_content
from netflix group by 1

-- 10. find each year and the average no. of content release by india in netflix , return top 5 year with highest avg content release
	select extract(year from to_date(date_added,'month dd,yyyy')) as year,
	count(*) as yearly_content,
	round(count(*)::numeric/(select count(*) from netflix where country='India')::numeric*100,2)
	as avg_content_per_year from netflix
	where country='India'
	group by 1

-- 11.  LIST ALL THE MOVIES THAT ARE DOCUMENTARIES

	SELECT * FROM NETFLIX 
	WHERE TYPE = 'Movie' and listed_in ilike '%Documentaries%'

-- 12 . FIND ALL THE CONTENT WITHOUT DIRECTOR

	SELECT * FROM NETFLIX 
	WHERE DIRECTOR IS NULL

-- 13. FIND HOW MANY MOVIES ACTOR 'SALMAN KHAN' APPEARED IN LAST 10 YEARS

	SELECT * FROM NETFLIX
	WHERE
	casts ILIKE '%Salman Khan%' AND release_year > extract(year from current_date) - 10 

-- 14 . find the top 10 actor who have appeared in the highest no. of movies oroduced in india
	select 
	unnest(string_to_array(casts,',') )as actors,
	count(*) as total_content
	from netflix 
	where country ILIKE '%india%' and type ='Movie'
	group by 1
	order by 2 desc
	limit 10

--15 . Categarize the content based on the present of keyword 'kill' and 'violence' in the description field.
	Label the content containing these keywords as 'bad_content' and 'good_content' .
	Count how many item fall into each category
with new_table 
as (
	select *,
	case 
	when 
	description ILIKE '%kill%' or 
	description ILIKE '&voilence%' then 'bad_content'
	else 'good_content'
	end category 
	from netflix
	)
	select category , count(*) as total_content
	from new_table
	group by 1
