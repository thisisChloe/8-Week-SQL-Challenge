# 🏥 Case Study #2: Hospital

<img src="https://github.com/user-attachments/assets/78fc2b2b-b949-47ab-831b-2866386ce587" alt="ChatGPT Image" width="400"/>


## 📚 Table of Contents
- [Business Task](#Business-Task)
- [Entity Relationship Diagram](#Entity-Relationship-Diagram)
- [Questions and Solutions](#Questions-and-Solutions)

## Business Task

This SQL portfolio showcases a range of analytical queries developed to extract actionable insights from a healthcare and service-oriented database. The queries address key operational areas such as patient activity, doctor assignments, department performance, appointment trends, and payment behaviours. These insights support better decision-making around scheduling, resource allocation, revenue forecasting, and overall service efficiency. The portfolio reflects a data analyst’s ability to connect technical querying skills with real-world business needs to drive continuous improvement.

## Entity Relationship Diagram

<img src="https://github.com/user-attachments/assets/f846a887-5a52-4d83-92c3-a13593924f6c" alt="ERD Diagram" width="600"/>

## Questions and Solutions

### 1. List all customers who have a registered bank card, including both active and inactive cards.

```sql
SELECT *
FROM customer c
JOIN card ca ON ca.customer_id = c.id;
```

**Steps:**

- Use **JOIN** to find all card records where `card.customer_id` matches `customer.id`.

**Answer:**

<img width="725" alt="Screenshot 2025-07-02 at 5 13 59 pm" src="https://github.com/user-attachments/assets/ae187df5-291e-4027-9b7f-1cccaebcdf21" />


### 2. Calculate the percentage of ACTIVE and INACTIVE accounts across the system.

```sql
SELECT 'ACTIVE' AS `status`, count(*)*100/(select count(*) 
					from account) as percentage
FROM account 
WHERE status = 'ACTIVE'

UNION 

SELECT 'INACTIVE' as `status`, count(*)*100/(select count(*) 
					from account) as percentage
FROM account 
WHERE status = 'INACTIVE';
```

**Steps:**

- Use **SELECT** `'ACTIVE'` AS status and filter **WHERE** `status = 'ACTIVE'` to calculate the percentage of active accounts.
- Use **SELECT** `'INACTIVE'` AS status and filter **WHERE** `status = 'INACTIVE'` to calculate the percentage of inactive accounts.
- Use **UNION** to combine both rows into one result set.

**Answer:**

<img width="144" alt="Screenshot 2025-07-02 at 5 21 14 pm" src="https://github.com/user-attachments/assets/ad01219b-354a-419e-a4a8-977407ee5d2d" />


### 3. List all doctors along with their assigned skills/departments.

```sql
SELECT d.*, de.*
FROM doctor d
LEFT JOIN doctor_in_department did ON d.id = did.doctor_id
LEFT JOIN department de ON de.id = did.department_id;
```

**Steps:**

- Use **FROM** doctor as the base table to list all doctors.
- Use **LEFT JOIN** to connect `doctor_in_department` so that doctors are included even if they aren't assigned to a department.
- Use **LEFT JOIN** to connect `department` so we can display department details (if any) alongside each doctor.

**Answer:**

<img width="743" alt="Screenshot 2025-07-02 at 5 28 04 pm" src="https://github.com/user-attachments/assets/2462abfb-9ef0-434a-93c1-205a14303051" />


### 4. Identify departments that currently do not have any assigned doctors.

```sql
SELECT de.*, COUNT(d.id) AS total_doctors
FROM department de
LEFT JOIN doctor_in_department did ON de.id = did.department_id
LEFT JOIN doctor d ON d.id = did.doctor_id
GROUP BY de.id
HAVING total_doctors = 0;
```

**Steps:**

- Use **FROM** `department` as the main table to list all departments.
- Use **LEFT JOIN** to connect `doctor_in_department` so that departments are included even if they have no assigned doctors.
- Use **LEFT JOIN** to bring in doctor details (if any), allowing counting of doctors per department.
- Use **COUNT(d.id)** to count doctors per department, and alias as `total_doctors`.
- Use **GROUP BY** de.id to group results by department.
- Use **HAVING** `total_doctors = 0` to filter and show only departments that have no doctors.

**Answer:**

<img width="272" alt="Screenshot 2025-07-02 at 5 29 21 pm" src="https://github.com/user-attachments/assets/28df8146-216b-4ba9-b50b-73c1901dc831" />

### 5. Determine which department has the most doctors, and which has the fewest, including the doctor count for each.

```sql
SELECT  d.*, COUNT(department_id) AS skills
FROM doctor d
LEFT JOIN doctor_in_department did ON d.id = did.doctor_id
LEFT JOIN department de ON de.id = did.department_id
GROUP BY d.id
ORDER BY skills DESC;
```

**Steps:**

- Use **FROM** `doctor` as the main table to list all doctors.
- Use **LEFT JOIN** to connect `doctor_in_department` so we include doctors even if they are not assigned to any department.
- Use **LEFT JOIN** to bring in the department table, if you want to access department info (optional in this case).
- Use **COUNT(department_id)** to count how many departments (skills) each doctor is linked to.
- Use **GROUP BY d.id** to group the results by each doctor.
- Use **ORDER BY** skills **DESC** to show doctors with the most skills (departments) first.

**Answer:**

<img width="482" alt="Screenshot 2025-07-02 at 5 32 37 pm" src="https://github.com/user-attachments/assets/0a1cf7ff-8e6e-4ab3-828f-9cd17a9ab0d7" />

### 6. Calculate the total appointment income earned per year.

```sql
SELECT YEAR(date_time), sum(price) 
FROM booking
WHERE status = 'PAID'
GROUP BY YEAR(date_time);
```

**Steps:**

- Use **FROM** `booking` to access all booking records.
- Use **WHERE** `status = 'PAID'` to filter only the bookings that were successfully paid.
- Use **YEAR(date_time)** to extract the year from the booking date.
- Use **SUM(price)** to calculate total revenue for each year.
- Use **GROUP BY** `YEAR(date_time)` to group the revenue totals by year.

**Answer:**

<img width="177" alt="Screenshot 2025-07-02 at 5 36 19 pm" src="https://github.com/user-attachments/assets/87fbcf31-3ac8-4f39-8a0a-363cac144141" />

### 7. Find the department with the highest average income per appointment, excluding appointments marked as REARRANGED.

```sql
SELECT department_id, AVG(price) AS Average_price
FROM booking
GROUP BY department_id
HAVING average_price= (
	select max(average_price)
    from (select department_id, avg(price) as average_price
    from booking
    group by department_id) as temp_table);
```

**Steps:**

- Use **FROM** `booking` as the main table to analyse booking prices.
- Use **GROUP BY** `department_id` to calculate average price per department.
- Use `AVG(price) AS average_price` to calculate average booking price for each department.
- Use a subquery to get the maximum average price across all departments.
- Use **HAVING** to filter and return only the department(s) with the highest average price.

**Answer:**

<img width="189" alt="Screenshot 2025-07-02 at 5 42 23 pm" src="https://github.com/user-attachments/assets/70c56fe8-7a6f-47e2-a9de-752596f53860" />


### 8. Identify the most frequently used payment method in the past 35 months, and show its percentage of total payment count.

```sql
SELECT method, 
	count(*) as total_time, 
    count(*)*100/(
		select count(*) 
		from booking b 
        join payment p on b.id = p.booking_id
        where status = 'PAID' AND TIMESTAMPDIFF(MONTH, date_time, now()) > 0 AND TIMESTAMPDIFF(MONTH, date_time, now()) <35) as percentage 
FROM booking b
JOIN payment p on b.id = p.booking_id
WHERE status = 'PAID'  AND TIMESTAMPDIFF(MONTH, date_time, now()) > 0 AND TIMESTAMPDIFF(MONTH, date_time, now()) <35
GROUP BY method
HAVING total_time = (
	select max(total_time) 
    from (select method, count(*) as total_time
    from booking b
    join payment p on b.id = p.booking_id
    where status = 'PAID'   AND TIMESTAMPDIFF(MONTH, date_time, now()) > 0 AND TIMESTAMPDIFF(MONTH, date_time, now()) <35
    group by method) as temp_table);
```

**Steps:**

- Use FROM booking and JOIN payment to access payment methods associated with bookings.
- Use WHERE status = 'PAID' to focus only on completed bookings.
- Use TIMESTAMPDIFF(MONTH, date_time, NOW()) > 0 AND < 35 to include bookings in the past 1–34 months.
- Use GROUP BY method to calculate stats per payment method.
- Use COUNT(*) AS total_time to get the total number of bookings for each method.
- Use COUNT(*) * 100 / (SELECT COUNT(...)...) to calculate the percentage of total for each method.
- Use HAVING total_time = (...) to filter only the payment method with the highest usage count.

**Answer:**

<img width="189" alt="Screenshot 2025-07-02 at 5 42 23 pm" src="https://github.com/user-attachments/assets/ea26ac59-7328-495c-a6eb-e850c1b29ad6" />


### 9. Display full appointment details for all customers, including doctor, department, status, and datetime.
```sql
SELECT * 
FROM customer 
JOIN appointment ON customer.id = appointment.customer_id;
```

**Steps:**

- Use **FROM** customer as the main table to start with customer data.
- Use **JOIN **appointment ON customer.id = appointment.customer_id to link each customer to their appointment(s).
- Use **SELECT *** to display all columns from both tables (customer + appointment).

**Answer:**

<img width="1000" alt="Screenshot 2025-07-02 at 6 08 11 pm" src="https://github.com/user-attachments/assets/d5f4866f-4e48-46c7-94bc-ac0897515759" />


### 10. Display the total payment amount made through each bank card, showing 0 for cards with no payments.

```sql
SELECT c.*, SUM(IF(b.status = 'PAID' AND b.price is not null, price, 0)) AS total
FROM card c
LEFT JOIN payment p ON p.card_id = c.id
LEFT JOIN booking b ON p.booking_id = b.id
GROUP BY c.id;
```

**Steps:**

- Use FROM card as the base table to list all cardholders.
- Use LEFT JOIN payment to connect each card to its related payments.
- Use LEFT JOIN booking to bring in booking details tied to those payments.
- Use SUM(IF(...)) to calculate the total value of bookings paid with each card
- Use GROUP BY c.id to calculate one row per card.

**Answer:**

<img width="404" alt="Screenshot 2025-07-02 at 6 09 57 pm" src="https://github.com/user-attachments/assets/21878bc5-ec34-4b2c-974b-4fe87c7e8bd2" />

