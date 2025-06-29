# 🎥 Case Study #1: Cinema

<img src="https://github.com/user-attachments/assets/1d601968-921c-422c-9e4d-d2021065a426" alt="Cinema" width="400"/>

## 📚 Table of Contents
- [Business Task](#Business-Task)
- [Entity Relationship Diagram](#Entity-Relationship-Diagram)
- [Questions and Solutions](#Questions-and-Solutions)

## Business Task

## Entity Relationship Diagram
<img width="600" alt="Cinema-Diagram" src="https://github.com/user-attachments/assets/907a1dda-b736-45f3-a766-8c4341b4636d" />

## Questions and Solutions


### 1. What is the booking rate for each film?


```sql
SELECT name
FROM film
LEFT JOIN screening ON film.id=screening.film_id
WHERE screening.id is NULL;


### 2. Show films that never had any bookings
### 3. Top 3 weekdays based on total bookings
### 4. Which film was screened in the most number of unique rooms?
### 5. Which customers booked more than one seat in one booking?
### 6. Who are the top 2 customers who spent the least total time in the cinema?
### 7. How many seats were booked for the film "Tom&Jerry"?
### 8. Which films were screened on all 7 days of the week?
### 9. Which rooms had the fewest and most total seats?
### 10. Which films were screened more than 10 times?

