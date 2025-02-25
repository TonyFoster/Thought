# MySQL 分割區快速指南

## 基本概念
分割區將大型資料表分成多個實體片段，提高查詢效能和維護性，同時保持單一邏輯資料表的操作介面。

## 必要條件
- 分割區鍵必須是主鍵的一部分
- 所有唯一鍵必須包含分割區鍵

## 分割區類型

### 1. RANGE 分割
按值範圍分配資料列。適用於時間序列資料。

```sql
CREATE TABLE sales_by_year (
    sale_id INT NOT NULL,
    sale_date DATE NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (sale_id, sale_date)
)
PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p_2022 VALUES LESS THAN (2023),
    PARTITION p_2023 VALUES LESS THAN (2024),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

### 2. LIST 分割
按離散值列表分配資料列。適用於分類資料。

```sql
CREATE TABLE sales_by_region (
    sale_id INT NOT NULL,
    region VARCHAR(20) NOT NULL,
    PRIMARY KEY (sale_id, region)
)
PARTITION BY LIST COLUMNS(region) (
    PARTITION p_north VALUES IN ('NORTH', 'NORTHWEST'),
    PARTITION p_south VALUES IN ('SOUTH', 'SOUTHWEST')
);
```

### 3. HASH 分割
按雜湊值均勻分配資料列。適用於均等分佈需求。

```sql
CREATE TABLE customer_orders (
    order_id INT NOT NULL,
    customer_id INT NOT NULL,
    PRIMARY KEY (order_id, customer_id)
)
PARTITION BY HASH(customer_id)
PARTITIONS 6;
```

### 4. 進階：子分割區
將分割區進一步細分。

```sql
CREATE TABLE sales_range_hash (
    sale_id INT NOT NULL,
    sale_date DATE NOT NULL,
    PRIMARY KEY (sale_id, sale_date)
)
PARTITION BY RANGE (YEAR(sale_date))
SUBPARTITION BY HASH(sale_id)
SUBPARTITIONS 4 (
    PARTITION p_2023 VALUES LESS THAN (2024),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

## 常用維護操作

```sql
-- 查看分割區信息
EXPLAIN SELECT * FROM sales_by_year;

-- 刪除分割區
ALTER TABLE sales_by_year DROP PARTITION p_2022;

-- 新增分割區
ALTER TABLE sales_by_year ADD PARTITION (
    PARTITION p_2024 VALUES LESS THAN (2025)
);

-- 重組分割區
ALTER TABLE sales_by_year REORGANIZE PARTITION p_future INTO (
    PARTITION p_2024 VALUES LESS THAN (2025),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

## 最佳實踐
- 選擇在查詢中常用的欄位作為分割區鍵
- 避免過度分割
- 使用 EXPLAIN 驗證分割區修剪效果
- 分割區最適合：大型資料表、頻繁基於特定欄位查詢、定期歸檔資料

是的，這是正確的：當對資料進行更新並將其移動到另一個分割區時，它實際上是執行了插入 + 刪除操作。更新跨分割區的資料時，MySQL 會在目標分割區新增一筆資料，然後從原始分割區刪除該資料。
