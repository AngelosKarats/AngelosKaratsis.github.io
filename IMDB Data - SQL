'''
Get Data: https://www.imdb.com/interfaces/
Data used:
- title_data: title.basics.tsv.gz
- name_data: name.basics.tsv.gz
- ratings_data: title.ratings.tsv.gz

Questions to be answered:
1) Import Data and create the tables (only ones we will use afterwards)
2) Implement the sql queries below: 
 a) Find all directors(not including assistant directors etc.) that where born in 1939 and show their names.
 b) Find all Thriller productions (movies, tvseries, etc.) that have the best ratings and at least 1 million reviews. Show titles and rating.
 c) Find 10 longest running tvseries with at least 100.000 reviews. Show titles, their "age" in descending order and if they are still on air.
 d) Find actors (regardless gender) with at least 4 productions that are known for. Calculate thir average rating for the productions that have at least 1.5 million
 reviews. Show their names and avg rating in descending order for those with avg rating bigger than 9.
3) Query optimization with indexes.
...


#1

CREATE TABLE name_basics
(
	nconst varchar PRIMARY KEY,
	primaryName text,
	birthYear int,
	deathYear int,
	primaryProfession varchar,
	knownForTitles varchar
);

SELECT * FROM name_basics;

COPY name_basics
FROM 'C:\Users\Public\Documents\bigdata\name_basics.tsv'
with csv DELIMITER E'\t'
NULL '\N'
HEADER;


CREATE TABLE title_basics
(
	tconst varchar PRIMARY KEY,
	titleType varchar,
	primaryTitle text,
	originalTitle text,
	isAdult boolean,
	startYear smallint,
	endYear	smallint,
	runtimeMinutes int,
	genres text
);

SELECT * FROM title_basics;

COPY title_basics
FROM 'C:\Users\Public\Documents\bigdata\title_basics.tsv'
WITH CSV DELIMITER E'\t'
NULL '\N'
QUOTE E'\b'
HEADER;


CREATE TABLE title_ratings
(
	tconst varchar PRIMARY KEY,
	averageRating real ,
	numVotes int
);

SELECT * FROM title_ratings;

COPY title_ratings
FROM 'C:\Users\Public\Documents\bigdata\title_ratings.tsv'
DELIMITER E'\t'
CSV HEADER;



#2A

SELECT primaryname, primaryprofession
FROM name_basics
WHERE birthYear=1939 AND primaryProfession similar to '(%,director%|director%)' 



#2B

select title_basics.primaryTitle, title_ratings.averagerating
from title_ratings, title_basics
where title_basics.genres similar to '(%Thriller%)'
	and title_ratings.numVotes >= 1000000 
	and title_ratings.tconst = title_basics.tconst
order by title_ratings.averagerating desc;



#2C

ALTER TABLE title_basics
ADD COLUMN still_shooting text;

UPDATE title_basics
SET still_shooting = 'Still playing' WHERE endyear IS NULL;

UPDATE title_basics
SET still_shooting = 'End' WHERE still_or_end IS NULL;

SELECT title_basics.primaryTitle, 
		COALESCE(title_basics.endYear,2021) - title_basics.startYear AS age, still_shooting
FROM title_basics INNER JOIN title_ratings ON title_basics.tconst = title_ratings.tconst
WHERE title_ratings.numvotes > 100000 AND title_basics.titleType LIKE 'tvSeries'
ORDER BY age desc
LIMIT 10;



#2D

select name_basics.primaryname, avg(title_ratings.averagerating) as mo
from name_basics inner join title_ratings on title_ratings.tconst = any (string_to_array(name_basics.knownfortitles,','))
where array_length(string_to_array(name_basics.knownfortitles,','),1)>=4
	and (name_basics.primaryprofession like '%act%')
	and title_ratings.numvotes>=1500000
	group by name_basics.primaryname
	having avg(title_ratings.averagerating)>9
	order by mo desc;
  
  
  
#3

CREATE INDEX birthyear_indx on name_basics USING HASH (birthyear)
CREATE INDEX avg_indx ON title_ratings USING BTREE (numvotes,averagerating)
CREATE INDEX tconst_primarytitle_endyear_startyear_titletype_indx ON title_basics (tconst, primarytitle, endyear, startyear, titletype)
CREATE INDEX prof_prim_indx ON name_basics (primaryprofession,primaryname)
