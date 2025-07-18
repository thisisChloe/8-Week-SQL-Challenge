# 🎥 Case Study #1: Cinema

<img src="https://github.com/user-attachments/assets/1d601968-921c-422c-9e4d-d2021065a426" alt="Cinema" width="400"/>

## 📚 Table of Contents
- [Business Task](#Business-Task)
- [Entity Relationship Diagram](#Entity-Relationship-Diagram)
- [Questions and Solutions](#Questions-and-Solutions)

## Business Task

The cinema management team wants to use the database to answer key operational questions about customer behaviour and film performance. They're particularly interested in identifying which films are most popular, how often customers attend, which screenings are underbooked, and how efficiently cinema rooms are being utilised. These insights will help improve scheduling, maximise occupancy, and enhance the overall customer experience.

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

### 3. Show what seat that customer "Dung Nguyen" booked

```sql
SELECT 
  s.row AS Seat_row, 
  s.number AS Seat_number
FROM customer c
JOIN booking b ON c.id = b.customer_id
JOIN reserved_seat rs ON b.id = rs.booking_id
JOIN seat s ON rs.seat_id = s.id
WHERE c.first_name = 'Dung' AND c.last_name = 'Nguyen'
ORDER BY s.row, s.number;
```

**Steps:**

- **JOIN** `customer` and `booking` to get all bookings made by `Dung Nguyen`.
- **JOIN** `reserved_seat` to find which seats were reserved in those bookings.
- **JOIN** `seat` to get seat details like `row` and `number`.
- Filter using **WHERE** `c.first_name = 'Dung' AND c.last_name = 'Nguyen'`.
- **ORDER BY** seat row and number for neat display.
  
**Answer:**

<img width="150" alt="Screenshot 2025-06-30 at 1 44 11 am" src="https://github.com/user-attachments/assets/16df95c8-f251-4e05-a019-c9843db67fce" />

### 4. Which film was screened in the most number of unique rooms?

```sql
WITH room_counts AS (
    SELECT 
        f.name AS Film_name,
        COUNT(DISTINCT s.room_id) AS Unique_rooms
    FROM film f
    JOIN screening s ON f.id = s.film_id
    GROUP BY f.id, f.name
)
SELECT *
FROM room_counts
WHERE unique_rooms = (
    SELECT MAX(unique_rooms) 
    FROM room_counts
);
```

**Steps:**

- Use a **CTE (room_counts)** to calculate how many distinct rooms each film was screened in:
  - **Join** `film` with `screening` to associate each film with its screening rooms.
  - Use **COUNT(DISTINCT s.room_id)** to count unique rooms per film.
  - **Group by** `film ID` and `name` to get one row per film.
- In the main query, use a subquery in the **WHERE** clause to select only those rows where the number of rooms equals the maximum number of rooms from the CTE. This ensures that films with tied maximum values are all returned.
  
**Answer:**

<img width="191" alt="Screenshot 2025-06-30 at 1 21 41 am" src="https://github.com/user-attachments/assets/c3866ee3-06c5-4093-8195-c4db0275157f" />


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

### 6. List customers who spent the least total time in the cinema?

```sql
WITH customer_times AS (
    SELECT 
        c.first_name,
        c.last_name,
        SUM(f.length_min) AS total_watch_time
    FROM customer c
    JOIN booking b ON c.id = b.customer_id
    JOIN screening s ON b.screening_id = s.id
    JOIN film f ON s.film_id = f.id
    GROUP BY c.id, c.first_name, c.last_name
)
SELECT first_name, last_name, total_watch_time
FROM customer_times
WHERE total_watch_time = (
    SELECT MIN(total_watch_time) 
    FROM customer_times
)
ORDER BY first_name, last_name;
```

**Steps:**

- Use a **Common Table Expression (CTE)** named customer_times to:
  - **Join** the `customer`, `booking`, `screening`, and `film` tables to relate each customer to the films they watched.
  - Use **SUM(f.length_min)** to calculate the total number of minutes each customer spent watching films.
  - **Group by** `customer ID` and `name` to calculate total time per customer.
- In the main query, select customers whose `total_watch_time` is equal to the minimum total time in the CTE. This ensures that all customers with the least time spent are included, even in the case of a tie.
- Use **ORDER BY** `first_name`, `last_name` for a clean, alphabetical result.

**Answer:**

<img width="252" alt="Screenshot 2025-06-30 at 1 28 19 am" src="https://github.com/user-attachments/assets/87ef1eab-2606-41d7-9353-70caf2ce0640" />


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

```sql
WITH room_seat_counts AS (
    SELECT 
        r.id AS room_id,
        r.name AS room_name,
        COUNT(s.id) AS total_seats
    FROM room r
    JOIN seat s ON r.id = s.room_id
    GROUP BY r.id, r.name
)
SELECT *
FROM room_seat_counts
WHERE total_seats = (
    SELECT MIN(total_seats) FROM room_seat_counts
)
   OR total_seats = (
    SELECT MAX(total_seats) FROM room_seat_counts
)
ORDER BY total_seats;
```

**Steps:**

- Use a **CTE (room_seat_counts)** to:
  - Join the `room` and `seat` tables on `room_id`
  - Count the number of seats in each room using **COUNT(s.id)**
  - Group by room to calculate the total seats per room
- In the main query:
  - Filter for rooms **where** `total_seats` is either:
  - The minimum number of seats among all rooms, or
  - The maximum number of seats among all rooms
- Use **ORDER BY** `total_seats` to show the room with the fewest seats first
  
**Answer:**

<img width="194" alt="Screenshot 2025-06-30 at 1 39 42 am" src="https://github.com/user-attachments/assets/25c6482c-7dba-49f3-98d5-2fc457ba05dc" />


### 10. Which films were screened more than 10 times?

```sql
SELECT 
  f.name AS Film_name,
  COUNT(s.id) AS Total_screenings
FROM film f
JOIN screening s ON f.id = s.film_id
GROUP BY f.id, f.name
HAVING COUNT(s.id) > 10
ORDER BY total_screenings DESC;
```

**Steps:**

- Use **JOIN** to connect the `film` table with the `screening` table using `film.id = screening.film_id`.
- Use **COUNT(s.id)** to count how many screenings each film had.
- Use **GROUP BY** `f.id`, `f.name` to count screenings per film.
- Use **HAVING COUNT(s.id) > 10** to filter only films that were screened more than 10 times.
- Use **ORDER BY** `total_screenings` **DESC** to sort the films from most to fewest screenings.
  
**Answer:**

<img width="214" alt="Screenshot 2025-06-30 at 1 33 54 am" src="https://github.com/user-attachments/assets/cf58a88d-0f7a-4b2a-b91e-0e9d8bce2662" />
