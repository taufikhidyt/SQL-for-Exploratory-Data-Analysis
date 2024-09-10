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

3. MoM revenue
```sql
SELECT
  EXTRACT(MONTH FROM oi.Created_at) AS months,
  ROUND(SUM(oi.sale_price * o.num_of_item), 2) AS total_revenue,
  SUM(o.num_of_item) AS total_sales,
  COUNT(DISTINCT oi.order_id) AS order_count,
  COUNT(DISTINCT oi.user_id) AS customers_purchased
FROM `bigquery-public-data.thelook_ecommerce.order_items`AS oi
INNER JOIN `bigquery-public-data.thelook_ecommerce.orders` AS o 
ON oi.order_id = o.order_id
WHERE oi.status NOT IN ('Cancelled', 'Returned')
GROUP BY 1
ORDER BY 2 DESC
```
### Output:
months | revenue | order_count | customers_purchased
-- | -- | -- | --
8 | 2024225.98 | 12925 | 11838
7 | 1730887.66 | 10690 | 10048
9 | 1465431.58 | 8711 | 7793
6 | 1434075.1 | 9161 | 8754
5 | 1417477.58 | 8699 | 8347
4 | 1305888.76 | 7846 | 7553
3 | 1187877.65 | 7408 | 7142
12 | 1028928.7 | 6155 | 5995
1 | 1014080.3 | 6494 | 6275
2 | 986666.35 | 6246 | 6059
10 | 895719.13 | 5525 | 5377
11 | 881674.83 | 5599 | 5440

### Insights
* Revenue has increased steadily

## Customer Analysis

6. Menghitung keuntungan berdasarkan kelompok usia pelanggan
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

7. Menghitung keuntungan berdasarkan jenis kelamin pelanggan
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

8. Mengidentifikasi negara dengan jumlah pelanggan terbesar
```sql
SELECT 
  country,
  COUNT(DISTINCT id) AS customer_count
FROM `bigquery-public-data.thelook_ecommerce.users` 
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```

9. Mengidentifikasi daftar 9 customer IDs dengan pembelian terbanyak. Pembeli tersebut akan diberi diskon 9.9
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


**Rekomendasi**
