# SQL for Data Exploratory: TheLook Ecommerce

**Transforming Data into Stories**

Repository ini memuat eksplorasi data theLook Ecommerce menggunakan Google Big Query.

Dataset: [theLook Ecommerce Dataset (BigQuery Public Data)](https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=thelook_ecommerce&page=dataset&project=my-gcp-data-projects&ws=!1m9!1m4!4m3!1sbigquery-public-data!2sthelook_ecommerce!3sorder_items!1m3!3m2!1sbigquery-public-data!2sthelook_ecommerce)

## Product and Brand Analysis

**1. Menghitung total penjualan dan pendapatan berdasarkan kategori produk**
```sql
WITH product_sales AS(
  SELECT 
    p.category as product_category,
    ROUND(SUM(oi.sale_price * o.num_of_item), 2) AS total_revenue,
    SUM(o.num_of_item) AS total_sales
  FROM `bigquery-public-data.thelook_ecommerce.orders` AS o
  INNER JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS oi
  ON o.order_id = oi.order_id
  INNER JOIN `bigquery-public-data.thelook_ecommerce.products` AS p 
  ON oi.product_id = p.id
  WHERE oi.status NOT IN ('cancelled', 'Returned')
  GROUP BY 1
)
SELECT * FROM product_sales
ORDER BY 2 DESC
```

### Output:

***TOP 5 kategori produk yang menghasilkan pendapatan terbesar***

product_category| total_revenue | total_sales
-- | -- | --
Outwear & Coats | 2192420.18 | 15250
Jeans | 2071195.11 | 21082
Sweaters | 1453760.76 | 19044
Suits & Sport Coats | 1112227.37 | 8929
Swim | 1087861.11 | 19075

***TOP 5 kategori produk yang menghasilkan pendapatan terkecil***

product_category| total_revenue | total_sales
-- | -- | --
Skirts | 175456.17 | 3394
Leggings | 149220.95 | 5265
Socks & Hosiery | 108852.92 | 6494
Jumpsuits & Rompers | 69276.87 | 1475
Clothing Sets | 23908.6 | 285

#### Insights:
* Kategori produk yang menghasilkan pendapatan tertinggi adalah adalah Outwear & Coats, lalu diikuti by Jeans and Sweaters.

* Clothing Sets dan Jumpsuits & Rompers adalah kategori produk yang menghasilkan pendapatan terendah.

**2. Menghitung total penjualan dan pendapatan berdasarkan brand**
```sql
WITH brand_sales AS(
  SELECT 
    p.brand as product_brand,
    ROUND(SUM(oi.sale_price * o.num_of_item), 2) AS total_revenue,
    SUM(o.num_of_item) AS total_sales
  FROM `bigquery-public-data.thelook_ecommerce.orders` AS o
  INNER JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS oi
  ON o.order_id = oi.order_id
  INNER JOIN `bigquery-public-data.thelook_ecommerce.products` AS p 
  ON oi.product_id = p.id
  WHERE oi.status NOT IN ('cancelled', 'Returned')
  GROUP BY 1
)
SELECT * FROM brand_sales
ORDER BY 2 DESC
```

### Output:

***TOP 5 brand produk yang menghasilkan pendapatan terbesar***

product_brand| total_revenue | total_sales
-- | -- | --
Calvin Klein | 342566.72 | 5577
Diesel | 341386.33 | 2464
True Religion | 312022.21 | 1518
7 For All Mankind | 300046.5 | 1904
Carhartt | 278229.74 | 4252

#### Insights:
* Brand yang menghasilkan pendapatan tertinggi adalah adalah Calvin Klein.

**3. MoM revenue**
```sql
SELECT
  EXTRACT(MONTH FROM oi.Created_at) AS months,
  ROUND(SUM(oi.sale_price * o.num_of_item), 2) AS total_revenue
FROM `bigquery-public-data.thelook_ecommerce.order_items`AS oi
INNER JOIN `bigquery-public-data.thelook_ecommerce.orders` AS o 
ON oi.order_id = o.order_id
WHERE oi.status NOT IN ('Cancelled', 'Returned')
GROUP BY 1
ORDER BY 2 DESC
```
### Output:
months | revenue 
-- | -- 
10 | 2024225.98 
9 | 1658986.66
8 | 1507239.76
7 | 1395609.97
6 | 1239216.78
5 | 1226179.71
3 | 1146874.68
4 | 1086385.7
1 | 1005730.67
2 | 988035.57
12 | 954615.49
11 | 901517.32

### Insights
* Revenue has increased steadily. The month with the highest revenue is October.

###------------------------------------------------------------------------------------------

## Customer Analysis

**4. Menghitung keuntungan berdasarkan kelompok usia pelanggan**
```sql
SELECT 
  CASE 
    WHEN u.age < 12 THEN 'Anak-anak'
    WHEN u.age BETWEEN 12 AND 17 THEN 'Remaja Awal'
    WHEN u.age BETWEEN 17 AND 25 THEN 'Remaja Akhir'
    WHEN u.age BETWEEN 25 AND 35 THEN 'Dewasa Awal'
    WHEN u.age BETWEEN 35 AND 55 THEN 'Dewasa Akhir'
    WHEN u.age > 55 THEN 'Lansia'
  END AS age_group,
  COUNT(DISTINCT oi.user_id) AS total_customer,
  ROUND(SUM(oi.sale_price * o.num_of_item), 2) AS total_revenue
FROM `bigquery-public-data.thelook_ecommerce.users` AS u
INNER JOIN `bigquery-public-data.thelook_ecommerce.orders` AS o 
ON u.id = o.user_id
INNER JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS oi
ON o.order_id = oi.order_id
WHERE oi.status NOT IN ('Cancelled', 'Returned') 
GROUP BY 1
ORDER BY 3 DESC
```

### Output:
age_group | total_customer | total_revenue
-- | -- | --
Dewasa Akhir | 22563 | 5184256.56
Lansia | 16950 | 3848659.9
Dewasa Awal | 11135 | 2567048.03
Remaja Akhir | 8976 | 2089841.59
Remaja Awal | 6695 | 1505111.38

#### Insights:
* Pelanggan pada kelompok usia `Dewasa Akhir` memberikan kontribusi pendapatan terbesar dan merupakan kelompok usia dengan jumlah pelanggan terbanyak dibanding kelompok usia lainnya.

**5. Menghitung keuntungan berdasarkan jenis kelamin pelanggan**
```sql
SELECT 
  u.gender,
  ROUND(SUM(oi.sale_price * o.num_of_item), 2) AS total_revenue,
  SUM(o.num_of_item) AS total_sales
FROM `bigquery-public-data.thelook_ecommerce.users` AS u
INNER JOIN `bigquery-public-data.thelook_ecommerce.orders` AS o 
ON u.id = o.user_id
INNER JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS oi
ON o.order_id = oi.order_id
WHERE oi.status NOT IN ('Cancelled', 'Returned')
GROUP BY 1
ORDER BY 2 DESC
```
### Output:
gender | total_revenue | total_sales 
-- | -- | --
M | 8116630.85 | 129318
F | 7078286.63 | 127876

#### Insights:
* Pelanggan laki-laki memberikan kontribusi revenue dan penjualan lebih besar dibandingkan dengan pelanggan perempuan.

**6. Mengidentifikasi TOP 5 negara dengan jumlah pelanggan terbesar**
```sql
SELECT 
  country,
  COUNT(DISTINCT id) AS customer_count
FROM `bigquery-public-data.thelook_ecommerce.users` 
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```

### Output:
country | customer_count
-- | -- 
China | 33894
United States | 22498
Brasil | 14595
South Korea | 5366
France | 4833

#### Insights:
* TOP 5 negara dengan jumlah pelanggan terbesar berturut-turut adalah China, United States, Brasil, Sounth Korea, dan France.

**7. Mengidentifikasi daftar 9 customer IDs dengan pembelian terbanyak. Pembeli tersebut akan diberi diskon 9.9**
```sql
SELECT
  u.id AS customer_id,
  CONCAT(u.first_name, ' ', u.last_name) AS full_name,
  u.email AS  email,
  ROUND(SUM(oi.sale_price * o.num_of_item), 2) AS total_revenue
FROM bigquery-public-data.thelook_ecommerce.order_items AS oi
INNER JOIN bigquery-public-data.thelook_ecommerce.orders AS o
ON oi.order_id = o.order_id
INNER JOIN bigquery-public-data.thelook_ecommerce.users AS u
ON o.user_id = u.id
GROUP BY 1, 2, 3
ORDER BY 4 DESC
LIMIT 9
```

### Output:
customer_id | fullname | email | total_revenue
-- | -- | -- | -- 
58143 | Thomas Torres | thomastorres@example.org | 5647.71
93116 | Raymond Smith | raymondsmith@example.net | 5307.76
78686 | Arthur Hansen | arthurhansen@example.com | 5204.16
98120 | Phillip Miller | phillipmiller@example.com | 4897.08
89749 | Richard Benton | richardbenton@example.net | 4698.75
84990 | Erica Davis | ericadavis@example.net | 4675.58
52576 | Kurt Daniels | kurtdaniels@example.org | 4669.2
92094 | Russell Durham | russelldurham@example.com | 4666.91
20346 | John Miller| johnmiller@example.com | 4588.46

#### Insights:
* Tabel di atas memberikan daftar pelanggan dengan total pembelian terbanyak, pelanggan nomor satu dengan total pembelian terbanyak adalah Thomas Torres.

## Rekomendasi
