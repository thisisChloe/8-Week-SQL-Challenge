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



### 2. Show films that never had any bookings

```sql
SELECT name
FROM film
LEFT JOIN screening ON film.id=screening.film_id
WHERE screening.id is NULL;
```

**Steps:**
- Use **LEFT JOIN** to return all records from the `film` table, and matching records from the `screening` table where `film.id = screening.film_id`.
If a film doesn't have a corresponding screening, the `screening` columns will be NULL.
- Use **WHERE** to filter the results to only include films that do not have any screenings. In other words, it shows films that have never been scheduled for a screening.

**Answer:**

<img width="153" alt="Screenshot 2025-06-29 at 11 56 16 pm" src="https://github.com/user-attachments/assets/b2475036-421e-473c-abb2-b2a95d778c71" />

- Film 'New' doesn't have any bookings.

### 3. Top 3 weekdays based on total bookings
### 4. Which film was screened in the most number of unique rooms?
### 5. Which customers booked more than one seat in one booking?

```sql
SELECT customer.first_name, customer.last_name, booking.id, COUNT(*)
FROM reserved_seat
JOIN booking ON reserved_seat.booking_id = booking.id
JOIN customer ON booking.customer_id = customer.id
GROUP BY booking.id, customer.first_name, customer.last_name
HAVING COUNT(*) > 1;
```

**Steps:**

- Use **SELECT** to retrieve the `customer’s name`, the `booking ID`, and the number of seats reserved in that booking by `COUNT(*)` to count how many seat records are linked to the booking (i.e. how many seats were reserved).
- Use **JOIN** to connect each seat reservation to its related booking record and link the booking to the customer who made it.
- Use **GROUP BY** to group results by each individual booking and customer. This ensures the count of seats is calculated per booking per customer.
- Use **HAVING COUNT(*) > 1** to filter only those groups (bookings) where more than one seat was reserved. So it only shows customers who booked multiple seats in a single booking.


**Answer:**

<img width="264" alt="Screenshot 2025-06-30 at 12 06 07 am" src="https://github.com/user-attachments/assets/05b1ce15-57fc-417d-ae75-df7559001505" />

### 6. Who are the top 2 customers who spent the least total time in the cinema?
### 7. How many seats were booked for the film "Tom&Jerry"?
### 8. Which films were screened on all 7 days of the week?
### 9. Which rooms had the fewest and most total seats?
### 10. Which films were screened more than 10 times?

