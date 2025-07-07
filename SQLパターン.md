SQLのパターンを30種類、具体的なコード例と共に記述。
サンプルとして、以下のようなテーブルを作成。

*   **employees**: 従業員テーブル (employee_id, name, department_id, salary, hire_date, manager_id)
*   **departments**: 部署テーブル (department_id, department_name)
*   **products**: 商品テーブル (product_id, product_name, price)
*   **orders**: 注文テーブル (order_id, customer_id, order_date)
*   **order_details**: 注文明細テーブル (order_id, product_id, quantity)

---

### 【基礎編：まずはここから】

#### 1. 基本的なデータ取得 (SELECT, FROM, WHERE)
全ての基本です。テーブルから条件に合うデータを取得します。
- `SELECT`: どの列を取得するか
- `FROM`: どのテーブルから取得するか
- `WHERE`: どのような条件の行を取得するか

```sql
-- 従業員テーブルから、部署IDが1の従業員の全情報を取得
SELECT *
FROM employees
WHERE department_id = 1;
```
**ポイント**: `*` は全ての列を意味しますが、実務では**必要な列だけを明示的に指定する**ことがパフォーマンス上推奨されます。

#### 2. 特定の列の取得
必要なデータ（列）だけを絞って取得します。通信量や処理負荷の削減につながります。

```sql
-- 従業員の名前と給与だけを取得
SELECT name, salary
FROM employees;
```

#### 3. 別名をつける (AS)
列名やテーブル名に別名（エイリアス）を付けます。クエリが読みやすくなり、複数のテーブルを扱う際に特に役立ちます。

```sql
-- name列を「従業員名」、salary列を「給与」として表示
SELECT
  name AS "従業員名",
  salary AS "給与"
FROM employees;
```
**ポイント**: `AS` は省略可能です。`SELECT name "従業員名"` のように書けますが、明示的に`AS`を書いた方が分かりやすいです。

#### 4. 並び替え (ORDER BY)
取得した結果を指定した列の昇順または降順で並び替えます。
- `ASC`: 昇順（小さい順、省略可能）
- `DESC`: 降順（大きい順）

```sql
-- 給与が高い順に従業員を並び替え
SELECT name, salary
FROM employees
ORDER BY salary DESC;
```

#### 5. 件数制限 (LIMIT)
取得する行数を制限します。大量のデータから一部だけを確認したい場合に便利です。

```sql
-- 入社日が新しい従業員を5人だけ取得
SELECT name, hire_date
FROM employees
ORDER BY hire_date DESC
LIMIT 5;
```
**ポイント**: `LIMIT`はMySQLやPostgreSQLで使われます。SQL Serverでは `TOP 5`、Oracleでは `FETCH FIRST 5 ROWS ONLY` を使います。

#### 6. 重複の排除 (DISTINCT)
指定した列で値が重複している行を1つにまとめて表示します。

```sql
-- 従業員が所属している部署IDの一覧を重複なく取得
SELECT DISTINCT department_id
FROM employees;
```

#### 7. 条件分岐 (CASE式)
値に応じて表示内容を動的に変更できます。ExcelのIF関数に似ています。

```sql
-- 給与に応じて「高給」「標準」「低給」のラベルを付ける
SELECT
  name,
  salary,
  CASE
    WHEN salary >= 8000000 THEN '高給'
    WHEN salary >= 5000000 THEN '標準'
    ELSE '低給'
  END AS salary_rank
FROM employees;
```

---

### 【集計・分析編：データからインサイトを得る】

#### 8. 件数カウント (COUNT)
行数を数える集計関数です。条件に合うデータが何件あるかを調べます。

```sql
-- 全従業員数をカウント
SELECT COUNT(*) FROM employees;

-- NULLではない給与データを持つ従業員数をカウント
SELECT COUNT(salary) FROM employees;
```
**ポイント**: `COUNT(*)` は全行を数え、`COUNT(列名)` はその列の値がNULLでない行を数えます。

#### 9. 合計値の計算 (SUM)
数値列の合計値を計算します。売上合計などの計算で頻繁に使います。

```sql
-- 全従業員の給与合計を計算
SELECT SUM(salary) FROM employees;
```

#### 10. 平均値の計算 (AVG)
数値列の平均値を計算します。

```sql
-- 全従業員の平均給与を計算
SELECT AVG(salary) FROM employees;
```

#### 11. 最大値・最小値の取得 (MAX, MIN)
数値や日付などの列から最大値と最小値を取得します。

```sql
-- 全従業員の中で最高の給与と最低の給与を取得
SELECT MAX(salary), MIN(salary) FROM employees;
```

#### 12. グループ化 (GROUP BY)
特定の列の値が同じ行をグループにまとめ、そのグループごとに集計関数を適用します。

```sql
-- 部署ごとに従業員数と平均給与を計算
SELECT
  department_id,
  COUNT(employee_id) AS num_employees,
  AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id;
```

#### 13. グループ化後の条件指定 (HAVING)
`GROUP BY`で集計した結果に対して、さらに条件を指定します。

```sql
-- 従業員が10人以上いる部署のみを表示
SELECT
  department_id,
  COUNT(employee_id) AS num_employees
FROM employees
GROUP BY department_id
HAVING COUNT(employee_id) >= 10;
```
**ポイント**: **`WHERE`はグループ化前の元データに対する条件**、**`HAVING`はグループ化後の集計結果に対する条件**と覚えるのが重要です。

#### 14. ウィンドウ関数 (OVER句)
`GROUP BY`のように行を集約せず、各行に対して集計結果（グループ全体の平均値など）を付与できます。

```sql
-- 各従業員の給与と、その従業員が所属する部署の平均給与を並べて表示
SELECT
  name,
  salary,
  department_id,
  AVG(salary) OVER (PARTITION BY department_id) AS dept_avg_salary
FROM employees;
```
**ポイント**: `PARTITION BY`でグループ分けのキーを指定します。これにより、一行ごとに「自分のデータ」と「自分の属するグループの集計値」を比較できます。

#### 15. ランキング付け (RANK, DENSE_RANK, ROW_NUMBER)
ウィンドウ関数の一種で、順位を付けます。
- `ROW_NUMBER()`: 同順位でも連番 (1, 2, 3, 4)
- `RANK()`: 同順位の場合、次の順位を飛ばす (1, 2, 2, 4)
- `DENSE_RANK()`: 同順位でも、次の順位を飛ばさない (1, 2, 2, 3)

```sql
-- 給与が高い順にランキングを付ける
SELECT
  name,
  salary,
  RANK() OVER (ORDER BY salary DESC) AS rnk,
  DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rnk,
  ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num
FROM employees;
```

#### 16. 移動平均 (AVG OVER with ROWS)
時系列データなどで、直近N日間の平均といった「移動平均」を計算します。株価や売上のトレンド分析に有効です。

```sql
-- 日々の売上データ（daily_salesテーブルを想定）から、当日を含む過去3日間の移動平均を計算
SELECT
  order_date,
  daily_sales,
  AVG(daily_sales) OVER (ORDER BY order_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg_3days
FROM daily_sales;
```

---

### 【テーブル結合編：複数のデータを組み合わせる】

#### 17. 内部結合 (INNER JOIN)
**両方のテーブルに存在するデータのみ**を、指定したキーで結合します。最も基本的な結合方法です。

```sql
-- 従業員名と、その従業員が所属する部署名を結合して取得
SELECT
  e.name,
  d.department_name
FROM employees AS e
INNER JOIN departments AS d ON e.department_id = d.department_id;
```

#### 18. 左外部結合 (LEFT JOIN)
左側のテーブル（`FROM`句の直後）の**全データを基準**に、右側のテーブルから一致するデータを結合します。一致しない場合はNULLが入ります。

```sql
-- 全ての従業員と、所属している部署名を表示（部署未所属の従業員も表示される）
SELECT
  e.name,
  d.department_name
FROM employees AS e
LEFT JOIN departments AS d ON e.department_id = d.department_id;
```
**ポイント**: どちらのテーブルを主軸にしたいかで `INNER JOIN` と `LEFT JOIN` を使い分けます。`RIGHT JOIN`もありますが、`LEFT JOIN`でテーブルの順序を入れ替える方が直感的でよく使われます。

#### 19. 自己結合 (SELF JOIN)
同じテーブル同士を別名（エイリアス）を付けて結合します。従業員とその上司、親子カテゴリなど、テーブル内で階層関係を持つデータを扱う際に使います。

```sql
-- 従業員名と、その上司の名前を一覧表示
SELECT
  e1.name AS employee_name,
  e2.name AS manager_name
FROM employees AS e1
LEFT JOIN employees AS e2 ON e1.manager_id = e2.employee_id;
```

#### 20. 複数テーブルの結合
3つ以上のテーブルを連続して`JOIN`でつなげることができます。

```sql
-- どの注文で、どの商品が、何個売れたかを取得
SELECT
  o.order_date,
  p.product_name,
  od.quantity
FROM orders AS o
JOIN order_details AS od ON o.order_id = od.order_id
JOIN products AS p ON od.product_id = p.product_id;
```

#### 21. 集合演算 (UNION / UNION ALL)
複数の`SELECT`文の結果を縦に結合します。
- `UNION`: 重複行を排除して結合します。
- `UNION ALL`: 重複行もそのまま全て結合します。**パフォーマンスはこちらの方が良い**です。

```sql
-- 東京支社の顧客リストと大阪支社の顧客リストを結合（customers_tokyo, customers_osakaテーブルを想定）
SELECT customer_id, customer_name FROM customers_tokyo
UNION ALL
SELECT customer_id, customer_name FROM customers_osaka;
```
**ポイント**: 結合する`SELECT`文の**列の数とデータ型を一致させる**必要があります。

---

### 【サブクエリ・応用編：複雑なロジックを実装する】

#### 22. サブクエリ (WHERE句内)
`WHERE`句の中に別の`SELECT`文（サブクエリ）を埋め込み、その結果を条件として利用します。

```sql
-- 「営業部」に所属する従業員を取得（部署IDが不明な場合）
SELECT name, salary
FROM employees
WHERE department_id = (SELECT department_id FROM departments WHERE department_name = '営業部');
```

#### 23. 相関サブクエリ
サブクエリ内で外側のクエリの列を参照する、より高度なサブクエリです。外側のクエリの行ごとにサブクエリが実行されるイメージです。

```sql
-- 自分が所属する部署の平均給与よりも高い給与をもらっている従業員を検索
SELECT
  name,
  salary
FROM employees AS e1
WHERE salary > (
  SELECT AVG(salary)
  FROM employees AS e2
  WHERE e2.department_id = e1.department_id
);
```

#### 24. インラインビュー (FROM句内のサブクエリ)
`FROM`句の中にサブクエリを記述し、その結果を一時的なテーブル（インラインビュー）として扱います。

```sql
-- 各部署の平均給与を計算し、その中で平均給与が600万以上の部署のみを表示
SELECT
  dept_summary.department_id,
  dept_summary.avg_salary
FROM (
  SELECT
    department_id,
    AVG(salary) AS avg_salary
  FROM employees
  GROUP BY department_id
) AS dept_summary
WHERE dept_summary.avg_salary >= 6000000;
```

#### 25. 共通テーブル式 (WITH句 / CTE)
`WITH`句を使って、クエリの先頭で一時的な名前付きの結果セット（CTE: Common Table Expression）を定義します。複雑なクエリを分割して可読性を高めたり、再帰的なクエリを書く際に役立ちます。

```sql
-- 上記のインラインビューの例をCTEで書き換える
WITH dept_summary AS (
  SELECT
    department_id,
    AVG(salary) AS avg_salary
  FROM employees
  GROUP BY department_id
)
SELECT
  d.department_name,
  ds.avg_salary
FROM dept_summary AS ds
JOIN departments AS d ON ds.department_id = d.department_id
WHERE ds.avg_salary >= 6000000;
```
**ポイント**: **複雑な中間集計を行う場合は、サブクエリよりもCTEを使う**方がクエリが構造化され、非常に読みやすくなります。

#### 26. 存在チェック (EXISTS)
サブクエリが1行でも結果を返せば`TRUE`、返さなければ`FALSE`を返します。条件に合うデータが「存在するかどうか」だけを判定したい場合に効率的です。

```sql
-- 1つでも商品を注文したことがある顧客のリストを取得
SELECT
  c.customer_name
FROM customers AS c -- (customersテーブルを想定)
WHERE EXISTS (
  SELECT 1 FROM orders AS o WHERE o.customer_id = c.customer_id
);
```
**ポイント**: `IN`句でも同じことができますが、一般的にサブクエリの結果が大きい場合は`EXISTS`の方が高速に動作します。

---

### 【データ操作・その他編】

#### 27. NULLの扱い (IS NULL, COALESCE, NULLIF)
- `IS NULL / IS NOT NULL`: 値がNULLか否かを判定します。
- `COALESCE(A, B, C)`: AがNULLならB、BもNULLならCを返します。最初に現れたNULLでない値を返します。
- `NULLIF(A, B)`: AとBが等しければNULL、等しくなければAを返します。

```sql
-- 上司がいない（manager_idがNULL）従業員を検索
SELECT name FROM employees WHERE manager_id IS NULL;

-- manager_idがNULLの場合は「-」（ハイフン）と表示する
SELECT name, COALESCE(CAST(manager_id AS VARCHAR), '-') AS manager
FROM employees;

-- '該当なし'という文字列をNULLに変換する
SELECT NULLIF(status, '該当なし') FROM tasks;
```

#### 28. 日付・時刻関数の活用
日付での絞り込みや期間計算は実務で非常に多用されます。
`CURRENT_DATE` (今日の日付), `INTERVAL` (期間の加減算) などが代表的です。

```sql
-- 今月注文されたデータを取得 (PostgreSQLの例)
SELECT *
FROM orders
WHERE order_date >= date_trunc('month', CURRENT_DATE)
  AND order_date < date_trunc('month', CURRENT_DATE) + interval '1 month';

-- 入社してから10年以上経過した従業員
SELECT name, hire_date
FROM employees
WHERE hire_date <= CURRENT_DATE - interval '10 year';
```

#### 29. 文字列操作 (LIKE, CONCAT, SUBSTRING)
- `LIKE`: 部分一致検索。`%`は任意の文字列、`_`は任意の1文字。
- `CONCAT`: 文字列を連結します。
- `SUBSTRING`: 文字列の一部を切り出します。

```sql
-- 商品名に「PC」が含まれる商品を検索
SELECT * FROM products WHERE product_name LIKE '%PC%';

-- 姓と名を連結してフルネームを作成（first_name, last_name列を想定）
SELECT CONCAT(last_name, ' ', first_name) AS full_name FROM employees;

-- 従業員IDの先頭3文字を取得
SELECT SUBSTRING(employee_id, 1, 3) FROM employees;
```

#### 30. トランザクション制御 (BEGIN, COMMIT, ROLLBACK)
一連の処理を一つのまとまり（トランザクション）として扱い、データの整合性を保ちます。
- `BEGIN` (or `START TRANSACTION`): トランザクションを開始します。
- `COMMIT`: 全ての処理を確定させ、データベースに永続的に反映します。
- `ROLLBACK`: 全ての処理を取り消し、トランザクション開始前の状態に戻します。

```sql
-- 銀行振込の例：A口座からB口座へ1万円送金
BEGIN;

-- A口座の残高を1万円減らす
UPDATE accounts SET balance = balance - 10000 WHERE account_id = 'A';

-- B口座の残高を1万円増やす
UPDATE accounts SET balance = balance + 10000 WHERE account_id = 'B';

-- 両方の処理が成功したら確定
COMMIT;

-- もし途中でエラーが起きたら、全て取り消す
-- ROLLBACK;
```
**ポイント**: `UPDATE`, `INSERT`, `DELETE`など、データを変更する処理を複数まとめて実行する際は、**必ずトランザクションを使う**ことで、中途半端な状態でデータが更新されるのを防ぎます。
