# SQL Project: Music Store Data Analysis 🎶

This project explores an SQL database for a fictional Music Store, analyzing customer behavior, sales patterns, and identifying key business insights. Queries are categorized by difficulty: **Easy**, **Moderate**, and **Advanced**.

---

## 📊 Project Outline

The project is organized into **three question sets**:

1. **Easy Questions:** Basic data analysis queries to understand employees, invoices, and top customers.
2. **Moderate Questions:** Intermediate queries exploring customer music preferences, top artists, and song durations.
3. **Advanced Questions:** Complex queries providing detailed insights into customer spending, genre popularity, and customer behavior across countries.

## 📄 Database Requirements

To run these SQL queries, a relational database (such as MySQL, PostgreSQL, or SQL Server) with tables for `employee`, `customer`, `invoice`, `invoice_line`, `track`, `album`, `artist`, and `genre` is needed. Each table contains data for employees, customers, invoices, tracks, and other related information for the Music Store.

---

### 🚀 Question Sets and Queries

#### **Question Set 1: Easy**

1. **Senior-most Employee:**
   - Find the senior-most employee by job title, age, or hire date.

   ```sql
   SELECT employee_id, levels FROM employee
   ORDER BY levels DESC
   LIMIT 1;
   ```
   <img src="https://github.com/lmno3418/sql_project/blob/main/Responses/1.1.JPG" alt="1.1">

2. **Top Countries by Invoice Count:**
   - Identify countries with the highest number of invoices.

   ```sql
   SELECT COUNT(billing_country) AS total, billing_country
   FROM invoice
   GROUP BY billing_country
   ORDER BY total DESC;
   ```
   <img src="https://github.com/lmno3418/sql_project/blob/main/Responses/1.2.JPG" alt="1.2">

3. **Top 3 Invoice Totals:**
   - List the top 3 invoices by total amount.

   ```sql
   SELECT total AS total_sum
   FROM invoice
   ORDER BY total_sum DESC
   LIMIT 3;
   ```
   <img src="https://github.com/lmno3418/sql_project/blob/main/Responses/1.3.JPG" alt="1.3">

4. **Best City for Customers:**
   - Determine the city generating the highest revenue to target for a promotional Music Festival.

   ```sql
   SELECT SUM(total) AS orders, billing_city
   FROM invoice
   GROUP BY billing_city
   ORDER BY orders DESC
   LIMIT 1;
   ```
   <img src="https://github.com/lmno3418/sql_project/blob/main/Responses/1.4.JPG" alt="1.4">

5. **Best Customer:**
   - Find the customer who has spent the most money.

   ```sql
   SELECT customer.customer_id, customer.first_name, customer.last_name, SUM(invoice.total) AS total
   FROM customer
   JOIN invoice ON customer.customer_id = invoice.customer_id
   GROUP BY customer.customer_id
   ORDER BY total DESC
   LIMIT 1;
   ```
   <img src="https://github.com/lmno3418/sql_project/blob/main/Responses/1.5.JPG" alt="1.5">

#### **Question Set 2: Moderate**

1. **Rock Music Listeners:**
   - Return email, first name, last name, and genre for all rock music listeners.

   ```sql
   SELECT DISTINCT email, first_name, last_name
   FROM customer
   JOIN invoice ON customer.customer_id = invoice.customer_id
   JOIN invoice_line ON invoice.invoice_id = invoice_line.invoice_id
   JOIN track ON track.track_id = invoice_line.track_id
   JOIN genre ON genre.genre_id = track.genre_id
   WHERE genre.name LIKE 'Rock'
   ORDER BY email;
   ```
   <img src="https://github.com/lmno3418/sql_project/blob/main/Responses/2.1.JPG" alt="2.1">

2. **Top 10 Rock Artists:**
   - List the top 10 artists with the most rock tracks.

   ```sql
   SELECT artist.name, COUNT(track.track_id) AS number_of_songs
   FROM track
   JOIN album ON track.album_id = album.album_id
   JOIN artist ON artist.artist_id = album.artist_id
   JOIN genre ON genre.genre_id = track.genre_id
   WHERE genre.name = 'Rock'
   GROUP BY artist.name
   ORDER BY number_of_songs DESC
   LIMIT 10;
   ```
   <img src="https://github.com/lmno3418/sql_project/blob/main/Responses/2.2.JPG" alt="2.2">

3. **Longest Songs by Track Length:**
   - Retrieve all track names longer than the average song length.

   ```sql
   SELECT name, milliseconds
   FROM track
   WHERE milliseconds > (SELECT AVG(milliseconds) FROM track)
   ORDER BY milliseconds DESC;
   ```
   <img src="https://github.com/lmno3418/sql_project/blob/main/Responses/2.3.JPG" alt="2.3">

#### **Question Set 3: Advanced**

1. **Customer Spending by Artist:**
   - Calculate the total amount spent by each customer on specific artists.

   ```sql
   WITH best_selling_artist AS (
       SELECT artist.artist_id, artist.name, SUM(invoice_line.unit_price * invoice_line.quantity) AS total_sales
       FROM invoice_line
       JOIN track ON track.track_id = invoice_line.track_id
       JOIN album ON album.album_id = track.album_id
       JOIN artist ON artist.artist_id = album.artist_id
       GROUP BY artist.artist_id
       ORDER BY total_sales DESC
       LIMIT 1
   )
   SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name, SUM(il.unit_price * il.quantity) AS amount_spent
   FROM invoice i
   JOIN customer c ON i.customer_id = c.customer_id
   JOIN invoice_line il ON il.invoice_id = i.invoice_id
   JOIN track t ON il.track_id = t.track_id
   JOIN album alb ON alb.album_id = t.album_id
   JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
   GROUP BY c.customer_id, c.first_name, c.last_name, bsa.artist_name
   ORDER BY amount_spent DESC;
   ```
   <img src="https://github.com/lmno3418/sql_project/blob/main/Responses/3.1.JPG" alt="3.1">

2. **Most Popular Genre by Country:**
   - Identify the most popular music genre for each country.

   ```sql
   WITH genre_purchases AS (
       SELECT customer.country, genre.name, COUNT(*) AS genre_count
       FROM customer
       JOIN invoice ON customer.customer_id = invoice.customer_id
       JOIN invoice_line ON invoice.invoice_id = invoice_line.invoice_id
       JOIN track ON invoice_line.track_id = track.track_id
       JOIN genre ON track.genre_id = genre.genre_id
       GROUP BY customer.country, genre.name
   )
   SELECT country, name AS genre
   FROM genre_purchases
   WHERE genre_count = (SELECT MAX(genre_count) FROM genre_purchases WHERE country = genre_purchases.country);
   ```
   <img src="https://github.com/lmno3418/sql_project/blob/main/Responses/3.2.JPG" alt="3.2">

3. **Top Customer by Country:**
   - Find the top-spending customer for each country.

   ```sql
   WITH customer_spending AS (
       SELECT customer.country, customer.customer_id, customer.first_name, customer.last_name, SUM(invoice.total) AS total_spent
       FROM customer
       JOIN invoice ON customer.customer_id = invoice.customer_id
       GROUP BY customer.country, customer.customer_id, customer.first_name, customer.last_name
   )
   SELECT country, first_name, last_name, total_spent
   FROM customer_spending
   WHERE total_spent = (SELECT MAX(total_spent) FROM customer_spending WHERE country = customer_spending.country);
   ```
   <img src="https://github.com/lmno3418/sql_project/blob/main/Responses/3.3.JPG" alt="3.3">

---

### 💾 Getting Started

1. Clone the repository to your local machine.
2. Set up the SQL database and load the data.
3. Run each query using your SQL interface or command-line tool to view results.

---

### ✨ Contributions & Feedback

Contributions and feedback are welcome. Feel free to open an issue or submit a pull request for improvements.

---

