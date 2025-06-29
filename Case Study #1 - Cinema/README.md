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
SELECT
  f.name AS Film_name,
  ROUND(COUNT(DISTINCT rs.id) * 100.0 / COUNT(DISTINCT s2.id), 2) AS Booking_rate
FROM film f
JOIN screening sc ON f.id = sc.film_id
JOIN room r ON sc.room_id = r.id
JOIN seat s2 ON r.id = s2.room_id
LEFT JOIN booking b ON sc.id = b.screening_id
LEFT JOIN reserved_seat rs ON b.id = rs.booking_id
GROUP BY f.id, f.name;
```

**Steps:**

- Use **INNER JOIN** to connect the `film`, `screening`, `room`, and `seat` tables. This helps identify all available seats for each film across all its screenings.
- Use **LEFT JOIN** to connect `booking` and `reserved_seat` so that we still include `screenings` (and `films`) even if no bookings or seat reservations exist.
- Use **COUNT(DISTINCT rs.id)** to count how many seats were actually booked for each film (each row in reserved_seat = 1 seat booked).
- Use **COUNT(DISTINCT s2.id)** to count the total number of available seats across all rooms used in the film’s screenings.
- Multiply the ratio by `100.0` and use **ROUND(..., 2)** to get the booking rate as a percentage, rounded to 2 decimal places.
- Use **GROUP BY** on `f.id`, `f.name` to return one row per film with its calculated booking rate.

**Answer:**

<img width="190" alt="Screenshot 2025-06-30 at 12 38 03 am" src="https://github.com/user-attachments/assets/7f43ca9e-9f99-4b08-aba2-9e86f5f0b68f" />


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

```sql
SELECT COUNT(*) AS Total_seat
FROM film
JOIN screening ON film.id = screening.film_id
JOIN booking ON screening.id = booking.screening_id
JOIN reserved_seat ON booking.id = reserved_seat.booking_id
WHERE film.name = 'Tom&Jerry';
```

**Steps:**

- Use **JOIN** to link each film to its screenings (i.e. showings) using `film.id`.
- Use **JOIN** to connect each screening to bookings made for that screening.
- Use **JOIN** to connect bookings to their individual seat reservations.
- Use **WHERE** to filter the entire result to include only data for the film titled "Tom&Jerry".

**Answer:**

<img width="80" alt="Screenshot 2025-06-30 at 12 13 59 am" src="https://github.com/user-attachments/assets/517fabd8-ce3b-44ae-a0cc-3561e9428f09" />

### 8. Which films were screened on all 7 days of the week?

```sql
SELECT f.name as Name
FROM film f
JOIN screening s ON f.id = s.film_id
GROUP BY f.id, f.name
HAVING COUNT(DISTINCT DAYNAME(s.start_time)) = 7;
```

**Steps:**

- Use **JOIN** to combine the `film` and `screening` tables, allowing us to access each film’s screening dates.
- Use **DAYNAME(s.start_time)** to extract the day of the week (e.g., Monday, Tuesday) from the `screening`’s start_time.
- Use **COUNT(DISTINCT DAYNAME(...))** in the **HAVING** clause to count how many unique days of the week each film was screened on.
- Filter using **HAVING = 7** to return only films that were shown on all 7 days of the week (Monday through Sunday).
- Use **GROUP BY** `f.id`, `f.name` to group the screenings by each film so that the day-counting is done per film.
  
**Answer:**

<img width="87" alt="Screenshot 2025-06-30 at 12 33 20 am" src="https://github.com/user-attachments/assets/a18f15d5-6dcb-4964-86a1-f7a628a819b9" />


### 9. Which rooms had the fewest and most total seats?
### 10. Which films were screened more than 10 times?

