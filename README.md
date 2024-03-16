# SQL-Murder-Mystery

Here I'll present my solution to SQL Murder Mystery by Northwestern University Knight Lab.
Avaible in https://mystery.knightlab.com/

The presentation of my solution to the problem was inspired by the gist of https://gist.github.com/bearloga/cfc8099223d1dace2604c8737dcbb4c3 [@bearloga](https://github.com/bearloga)

## Prompt
There's been a Murder in SQL City! The SQL Murder Mystery is designed to be both a self-directed lesson to learn SQL concepts and commands and a fun game for experienced SQL users to solve an intriguing crime.
A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a ​murder​ that occurred sometime on ​Jan.15, 2018​ and that it took place in ​SQL City​. Start by retrieving the corresponding crime scene report from the police department’s database.

## Crime Scene Report
```sql
SELECT * FROM crime_scene_report WHERE date = '20180115' AND city = 'SQL City' AND type = 'murder'  
```
> Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".

## Finding the witness
```sql
SELECT *, MAX(address_number) FROM person WHERE address_street_name = 'Northwestern Dr'
```

|  id | name                                               | license_id | address_number | address_street_name | ssn | MAX(address_number) |
| :-: | :------------------------------------------------- | :--------: | :------------: | :-----------------: | :-: | :-----------------: |
| 14887 |	Morty | Schapiro |	118009 |	4919 |	Northwestern Dr |	111564949 |	4919 |

```sql
SELECT * FROM person WHERE address_street_name = 'Franklin Ave' AND name LIKE 'Annabel %'
```

|  id | name                                               | license_id | address_number | address_street_name | ssn |
| --: | :------------------------------------------------- | :--------: | :------------: | :-----------------: | :-: |
| 16371 |	Annabel Miller |	490173 |	103 |	Franklin Ave |	318771143 |

## Suspects

The main suspects now are all the people who checked in at the same place as the two witnesses on January 15, 2018

```sql
SELECT * FROM facebook_event_checkin AS FB
JOIN person
ON FB.person_id = person.id
WHERE person.id = 16371 OR person.id = 14887 AND date = '20180115'
```

| person_id |	event_id |	event_name |	date	| id |	name |	license_id	| address_number |	address_street_name	| ssn |
| :-------: | :------: | :---------: | :----: | :-: | :--: | :----------- | :------------: | :------------------- | :-: |
| 16371 |	4719 |	The Funky Grooves Tour | 20180115 |	16371 |	Annabel Miller |	490173 | 103  |	Franklin Ave    |	318771143
| 14887	| 4719 |	The Funky Grooves Tour | 20180115	| 14887	| Morty Schapiro |	118009 | 4919 | Northwestern Dr	| 111564949

```sql
SELECT * FROM facebook_event_checkin AS FB
JOIN person
ON FB.person_id = person.id
WHERE event_id = 4719 AND date = '20180115'
```

| person_id |	event_id |	event_name |	date	| id |	name |	license_id	| address_number |	address_street_name	ssn |
| :-------: | :------: | :---------: | :----: | :-: | :--: | :----------: | :------------: | :----------------------: |
| 14887	| 4719 | The Funky Grooves Tour | 20180115 | 14887 | Morty Schapiro | 118009 | 4919 | Northwestern Dr | 111564949 |
| 16371 | 4719 | The Funky Grooves Tour | 20180115 | 16371 | Annabel Miller | 490173 | 103 | Franklin Ave | 318771143  |
| 67318 | 4719 | The Funky Grooves Tour | 20180115 | 67318 | Jeremy Bowers | 423327 | 530 | Washington Pl, Apt 3A | 871539279 |


The main suspect is Jeremy Bowers

```sql
INSERT INTO solution VALUES (1, 'Jeremy Bowers');
SELECT value FROM solution;
```

> Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villain behind this crime. If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries. Use this same INSERT statement with your new suspect to check your answer.

## Further Investigation

```sql
SELECT * FROM interview WHERE person_id = 67318
```

| person_id	| transcript |
| :-------: | :--------- |
| 67318 | I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017. |

```sql
SELECT person.id, person.name FROM person
JOIN drivers_license 
ON person.license_id = drivers_license.id
WHERE height >= 65 AND height <= 67 AND hair_color = 'red' AND gender = 'female' AND car_make = 'Tesla' AND car_model = 'Model S'
```

| id | name | 
| :---: | :--------- |
| 78881 | Red Korb |
| 90700	| Regina George |
| 99716	| Miranda Priestly | 

We have three main suspicions

```sql
SELECT FB.person_id, person.name, COUNT(1) AS times FROM facebook_event_checkin AS FB
JOIN person
ON FB.person_id = person.id
WHERE event_name = 'SQL Symphony Concert' AND date LIKE '201712%'
GROUP BY FB.person_id
HAVING times = 3
```

| person_id | name | times |
| :-------: | :--------- | :--------- |
| 24556 | Bryan Pardo | 3 |
| 99716 | Miranda Priestly | 3 |

It is concluded that the mastermind of the murder was Miranda Priestly

```sql
INSERT INTO solution VALUES (1, 'Miranda Priestly');
SELECT value FROM solution;
```

> Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne!
