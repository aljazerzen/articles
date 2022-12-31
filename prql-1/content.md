<div class="row">

<img src="./icon.svg" height="120">

# PRQL

</div>

> a sequel to SQL

notes:
SQL does not feel like a programming language.
Well technically it is not - it's a query langauge - but the question is why?
When I say "feel like a programming language" I mean that you can build with it.
Starting with something small and then iteratively adding and expanding.

Let me show you an exmaple.


---

## Chinook

https://github.com/lerocha/chinook-database

<div class="row" data-id="db-schema">

|album|
|---|
|album_id|
|title|

|track |
|---|
|track_id |
|album_id |
|title |

|invoice_line|
|---|
|invoice_line_id|
|invoice_id|
|track_id|
|unit_price|
|quantity|

|invoice|
|---|
|invoice_id|
|total|
|billing_country|

</div>

<p class="fragment">
task: top 10 albums by revenue generated
</p>

notes:
I'll be using Chinook sample database.

---

## Chinook

<div class="row" data-id="db-schema">

|album|
|---|
|album_id|
|title|

|track |
|---|
|track_id |
|album_id |
|title |

|invoice_line|
|---|
|track_id|
|unit_price|
|quantity|

</div>

task: top 10 albums by revenue generated


notes:
I'll be using Chinook sample database.

---

## SQL

<div class="row" data-id="db-schema">

|album|
|---|
|album_id|
|title|

|track |
|---|
|track_id |
|album_id |
|title |

|invoice_line|
|---|
|track_id|
|unit_price|
|quantity|

</div>

<pre data-id="code"><code data-trim data-line-numbers>
SELECT track_id, quantity, unit_price
FROM invoice_line;
</code></pre>

<div data-id="output" class="fragment">

| track\_id | quantity | unit\_price |
| :--- | :--- | :--- |
| 2 | 1 | 0.99 |
| 4 | 1 | 0.99 |
| 6 | 1 | 0.99 |
| 8 | 1 | 0.99 |



---

## SQL

<div class="row" data-id="db-schema">

|album|
|---|
|album_id|
|title|

|track |
|---|
|track_id |
|album_id |
|title |

|invoice_line|
|---|
|track_id|
|unit_price|
|quantity|

</div>

<pre data-id="code"><code data-trim data-line-numbers>
SELECT track_id, quantity * unit_price AS revenue
FROM invoice_line;
</code></pre>

    
| track\_id | revenue |
| :--- | :--- |
| 2 | 0.99 |
| 4 | 0.99 |
| 6 | 0.99 |
| 6 | 0.99 |


---

## SQL

<div class="row" data-id="db-schema">

|album|
|---|
|album_id|
|title|

|track |
|---|
|track_id |
|album_id |
|title |

|invoice_line|
|---|
|track_id|
|unit_price|
|quantity|

</div>
<pre data-id="code"><code data-trim data-line-numbers>
SELECT track_id, SUM(quantity * unit_price) AS revenue
FROM invoice_line
GROUP BY track_id;
</code></pre>

    
| track\_id | revenue |
| :--- | :--- |
| 2 | 0.99 |
| 4 | 0.99 |
| 6 | 1.98 |

---

<pre><code data-trim data-line-numbers>
SELECT track.track_id, track.name, album_id, revenue
FROM (
    SELECT track_id, SUM(quantity * unit_price) AS revenue
    FROM invoice_line
    GROUP BY track_id
) track_revenue
JOIN track ON track_revenue.track_id = track.track_id;
</code></pre>


| track\_id | name | album\_id | revenue |
| :--- | :--- | :--- | :--- |
| 2 | Balls to the Wall | 2 | 0.99 |
| 4 | Restless and Wild | 3 | 0.99 |
| 6 | Put The Finger On You | 1 | 1.98 |


---

<pre><code data-trim data-line-numbers>
SELECT album_id, SUM(revenue) as revenue
FROM (
    SELECT track_id, SUM(quantity * unit_price) AS revenue
    FROM invoice_line
    GROUP BY track_id
) track_revenue
JOIN track ON track_revenue.track_id = track.track_id
GROUP BY album_id;
</code></pre>


| album\_id | sum |
| :--- | :--- |
| 1 | 9.9 |
| 2 | 1.98 |
| 3 | 2.97 |

---
<pre data-id="code"><code data-trim data-line-numbers>
SELECT album.album_id, title, revenue
FROM (
    SELECT album_id, SUM(revenue) as revenue
    FROM (
        SELECT track_id,
               SUM(quantity * unit_price) AS revenue
        FROM invoice_line
        GROUP BY track_id
    ) track_revenue
    JOIN track ON track_revenue.track_id = track.track_id
    GROUP BY album_id
) album_revenue
JOIN album ON album_revenue.album_id = album.album_id;
</code></pre>

| album\_id | title | revenue |
| :--- | :--- | :--- |
| 1 | For Those About To Rock We Salute You | 9.9 |
| 2 | Balls to the Wall | 1.98 |
| 3 | Restless and Wild | 2.97 |

---
<pre><code data-trim data-line-numbers>
SELECT album.album_id, title, revenue
FROM (
    SELECT album_id, SUM(revenue) as revenue
    FROM (
        SELECT track_id,
               SUM(quantity * unit_price) AS revenue
        FROM invoice_line
        GROUP BY track_id
    ) track_revenue
    JOIN track ON track_revenue.track_id = track.track_id
    GROUP BY album_id
) album_revenue
JOIN album ON album_revenue.album_id = album.album_id
ORDER BY revenue DESC
LIMIT 10;
</code></pre>

| album\_id | title | revenue |
| :--- | :--- | :--- |
| 253 | Battlestar Galactica \(Classic\), Season 1 | 35.82 |
| 251 | The Office, Season 3 | 31.84 |
| 23 | Minha Historia | 26.73 |


---
<pre data-id="code"><code data-trim data-line-numbers>
WITH track_revenue AS (
    SELECT track_id, SUM(quantity * unit_price) AS revenue
    FROM invoice_line
    GROUP BY track_id
),
album_revenue AS (
    SELECT album_id, SUM(revenue) AS revenue
    FROM track_revenue
    JOIN track ON track_revenue.track_id = track.track_id
    GROUP BY album_id
)
SELECT album.album_id, title, revenue
FROM album_revenue
JOIN album ON album_revenue.album_id = album.album_id
ORDER BY revenue DESC
LIMIT 10;
</code></pre>
---

## PRQL

<pre data-id="prql"><code data-trim data-line-numbers class="language-prql">
from invoice_line
select [invoice_id, track_id, quantity, unit_price]
</code></pre>

<div data-id="output" class="fragment">

| invoice\_id | track\_id | quantity | unit\_price |
| :--- | :--- | :--- | :--- |
| 1 | 2 | 1 | 0.99 |
| 1 | 4 | 1 | 0.99 |
| 2 | 6 | 1 | 0.99 |
| 7 | 6 | 1 | 0.99 | 0.99 |
</div>

---

<pre data-id="prql"><code data-trim data-line-numbers class="language-prql">
from invoice_line
select [invoice_id, track_id, quantity, unit_price]
derive [revenue = quantity * unit_price]
</code></pre>

<div data-id="output">

| invoice\_id | track\_id | quantity | unit\_price | revenue |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 2 | 1 | 0.99 | 0.99 |
| 1 | 4 | 1 | 0.99 | 0.99 |
| 2 | 6 | 1 | 0.99 | 0.99 |
| 7 | 6 | 1 | 0.99 | 0.99 |
</div>
---

<pre data-id="prql"><code data-trim data-line-numbers class="language-prql">
from invoice_line
select [invoice_id, track_id, quantity, unit_price]
derive revenue = quantity * unit_price
</code></pre>

<div>

| invoice\_id | track\_id | quantity | unit\_price | revenue |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 2 | 1 | 0.99 | 0.99 |
| 1 | 4 | 1 | 0.99 | 0.99 |
| 2 | 6 | 1 | 0.99 | 0.99 |
| 7 | 6 | 1 | 0.99 | 0.99 |
</div>

---

<pre data-id="prql"><code data-trim data-line-numbers class="language-prql">
from invoice_line
select [invoice_id, track_id, quantity, unit_price]
derive revenue = quantity * unit_price
aggregate [revenue = sum revenue]
</code></pre>

| revenue |
| :--- |
| 2328.6 |

---
<pre data-id="prql"><code data-trim data-line-numbers class="language-prql">
from invoice_line
select [invoice_id, track_id, quantity, unit_price]
derive revenue = quantity * unit_price
group [track_id] (
	aggregate [revenue = sum revenue]
)
</code></pre>

| track\_id | revenue |
| :--- | :--- |
| 2 | 0.99 |
| 4 | 0.99 |
| 6 | 1.98 |

---
<pre data-id="prql"><code data-trim data-line-numbers class="language-prql">
from invoice_line
select [invoice_id, track_id, quantity, unit_price]
derive revenue = quantity * unit_price
group [track_id] (
	aggregate [revenue = sum revenue]
)
join track [invoice_line.track_id == track.track_id]
</code></pre>
---
<pre data-id="prql"><code data-trim data-line-numbers class="language-prql">
from invoice_line
select [invoice_id, track_id, quantity, unit_price]
derive revenue = quantity * unit_price
group [track_id] (
	aggregate [revenue = sum revenue]
)
join track [==track_id]
</code></pre>
---
<pre data-id="prql"><code data-trim data-line-numbers class="language-prql">
from invoice_line
select [invoice_id, track_id, quantity, unit_price]
derive revenue = quantity * unit_price
group [track_id] (
	aggregate [revenue = sum revenue]
)
join track side:inner [==track_id]
</code></pre>
---
<pre data-id="prql"><code data-trim data-line-numbers class="language-prql">
from invoice_line
select [invoice_id, track_id, quantity, unit_price]
derive revenue = quantity * unit_price
group [track_id] (
	aggregate [revenue = sum revenue]
)
join track side:inner [==track_id]
</code></pre>

| track\_id | name | album\_id | track\_id | revenue |
| :--- | :--- | :--- | :--- | :--- |
| 1 | For Those About To Rock | 1 | 1 | 0.99 |
| 2 | Balls to the Wall | 2 | 2 | 1.98 |
| 3 | Fast As a Shark | 2 | 3 | 0.99 |
---
<pre data-id="prql"><code data-trim data-line-numbers class="language-prql">
from invoice_line
select [invoice_id, track_id, quantity, unit_price]
derive revenue = quantity * unit_price
group [track_id] (
	aggregate [revenue = sum revenue]
)
join track side:inner [==track_id]
group [album_id] (
	aggregate [revenue = sum revenue]
)
join a = album [==album_id]
sort [-revenue]
take 10
select [a.album_id, a.title, revenue]
</code></pre>

<div class="fragment">

| album\_id | title | revenue |
| :--- | :--- | :--- |
| 253 | Battlestar Galactica \(Classic\), Season 1 | 35.82 |
| 251 | The Office, Season 3 | 31.84 |
| 23 | Minha Historia | 26.73 |
</div>

notes:
PRQL compiles to SQL

---

Generated SQL

<pre><code data-trim data-line-numbers>
WITH table_1 AS (
    SELECT SUM(quantity * unit_price) AS revenue,
           track_id
    FROM invoice_line
    GROUP BY track_id
),
table_2 AS (
    SELECT SUM(table_1.revenue) AS revenue,
           track.album_id
    FROM table_1
    JOIN track ON table_1.track_id = track.track_id
    GROUP BY track.album_id
)
SELECT a.album_id, a.title, table_2.revenue
FROM table_2
JOIN album AS a ON table_2.album_id = a.album_id
ORDER BY table_2.revenue DESC
LIMIT 10
</code></pre>

<small>(formatted manually)</small>

---

Generated SQL

<pre><code data-trim data-line-numbers="2-3">
WITH table_1 AS (
    SELECT SUM(quantity * unit_price) AS revenue,
           track_id
    FROM invoice_line
    GROUP BY track_id
),
table_2 AS (
    SELECT SUM(table_1.revenue) AS revenue,
           track.album_id
    FROM table_1
    JOIN track ON table_1.track_id = track.track_id
    GROUP BY track.album_id
)
SELECT a.album_id, a.title, table_2.revenue
FROM table_2
JOIN album AS a ON table_2.album_id = a.album_id
ORDER BY table_2.revenue DESC
LIMIT 10
</code></pre>

<small>(formatted manually)</small>

---

Generated SQL

<pre><code data-trim data-line-numbers="11">
WITH table_1 AS (
    SELECT SUM(quantity * unit_price) AS revenue,
           track_id
    FROM invoice_line
    GROUP BY track_id
),
table_2 AS (
    SELECT SUM(table_1.revenue) AS revenue,
           track.album_id
    FROM table_1
    JOIN track ON table_1.track_id = track.track_id
    GROUP BY track.album_id
)
SELECT a.album_id, a.title, table_2.revenue
FROM table_2
JOIN album AS a ON table_2.album_id = a.album_id
ORDER BY table_2.revenue DESC
LIMIT 10
</code></pre>

<small>(formatted manually)</small>

---

Generated SQL

<pre><code data-trim data-line-numbers="17">
WITH table_1 AS (
    SELECT SUM(quantity * unit_price) AS revenue,
           track_id
    FROM invoice_line
    GROUP BY track_id
),
table_2 AS (
    SELECT SUM(table_1.revenue) AS revenue,
           track.album_id
    FROM table_1
    JOIN track ON table_1.track_id = track.track_id
    GROUP BY track.album_id
)
SELECT a.album_id, a.title, table_2.revenue
FROM table_2
JOIN album AS a ON table_2.album_id = a.album_id
ORDER BY table_2.revenue DESC
LIMIT 10
</code></pre>

<small>(formatted manually)</small>

---

<div class="row">

<img src="./icon.svg" height="120">

# PRQL

</div>

- https://prql-lang.org/
  - documentation
  - playground

- https://github.com/prql/prql
  - develop compiler & bindings
  - discuss language design
