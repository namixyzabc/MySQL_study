# MySQLサーバーの導入とMySQL Workbenchの使用方法

## 1. MySQLサーバーの導入

### Windows環境での導入

**1. MySQL Community Serverのダウンロード**
- MySQL公式サイト（https://dev.mysql.com/downloads/mysql/）にアクセス
- 「MySQL Community Server」をダウンロード
- インストーラー形式（mysql-installer-web-community-x.x.x.msi）を選択

**2. インストール手順**
```
1. インストーラーを管理者権限で実行
2. Setup Type: "Developer Default"を選択
3. Check Requirements: 必要に応じて追加コンポーネントをインストール
4. Installation: "Execute"をクリック
5. Product Configuration: 以下の設定を行う
   - Config Type: Development Computer
   - Connectivity: Port 3306（デフォルト）
   - Authentication Method: 推奨設定を使用
   - Accounts and Roles: rootパスワードを設定
```

### Linux（Ubuntu）環境での導入

```bash
# パッケージリストを更新
sudo apt update

# MySQLサーバーをインストール
sudo apt install mysql-server

# MySQLサービスを開始
sudo systemctl start mysql

# 自動起動を有効化
sudo systemctl enable mysql

# セキュリティ設定
sudo mysql_secure_installation
```


```

## 2. MySQL Workbenchの導入

### インストール方法

**Windows/macOS:**
- MySQL公式サイトからMySQL Workbenchをダウンロード
- インストーラーを実行して標準インストール

**Linux:**
```bash
# Ubuntu/Debian
sudo apt install mysql-workbench

# CentOS/RHEL
sudo yum install mysql-workbench-community
```

## 3. MySQL Workbenchの基本的な使用方法

### 接続設定

**1. 新しい接続の作成**
```
1. MySQL Workbench起動
2. "MySQL Connections"の横の「+」をクリック
3. 接続情報を入力:
   - Connection Name: 任意の名前（例：Local MySQL）
   - Hostname: 127.0.0.1 または localhost
   - Port: 3306
   - Username: root
   - Password: Store in Vault...をクリックして設定
4. "Test Connection"で接続確認
5. "OK"をクリック
```

**2. 接続の実行**
- 作成した接続をダブルクリック
- パスワードを入力してサーバーに接続

### 基本的なデータベース操作

**1. データベースの作成**
```sql
-- 新しいデータベースを作成
CREATE DATABASE sample_db;

-- データベースを使用
USE sample_db;

-- 作成されたデータベースを確認
SHOW DATABASES;
```

**2. テーブルの作成**
```sql
-- ユーザーテーブルの作成
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    age INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 商品テーブルの作成
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(200) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    stock_quantity INT DEFAULT 0,
    category VARCHAR(50)
);
```

**3. データの挿入**
```sql
-- ユーザーデータの挿入
INSERT INTO users (name, email, age) VALUES
('田中太郎', 'tanaka@example.com', 25),
('佐藤花子', 'sato@example.com', 30),
('鈴木一郎', 'suzuki@example.com', 28);

-- 商品データの挿入
INSERT INTO products (product_name, price, stock_quantity, category) VALUES
('ノートPC', 89800.00, 15, 'Electronics'),
('マウス', 2500.00, 50, 'Electronics'),
('コーヒーマグ', 800.00, 100, 'Kitchen');
```

## 4. MySQL Workbenchの主要機能

### スキーマエディタ

**1. 視覚的なテーブル設計**
```
1. 左側のNavigatorで「Schemas」タブを選択
2. データベースを右クリック→「Create Table」
3. Table Editorで以下を設定：
   - Column Name: カラム名
   - Datatype: データ型
   - PK: Primary Key
   - NN: Not Null
   - AI: Auto Increment
```

**2. EER図の作成**
```
1. メニューから「Database」→「Reverse Engineer」
2. 既存のデータベースからER図を生成
3. 「Model」→「Add Diagram」で新しいEER図を作成
```

### クエリエディタの活用

**1. 基本的なクエリの実行**
```sql
-- データの検索
SELECT * FROM users WHERE age > 25;

-- 結合クエリ
SELECT u.name, u.email, p.product_name, p.price
FROM users u
CROSS JOIN products p
WHERE p.price < 10000;

-- 集計クエリ
SELECT category, COUNT(*) as product_count, AVG(price) as avg_price
FROM products
GROUP BY category;
```

**2. クエリの最適化**
```sql
-- インデックスの作成
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_products_category ON products(category);

-- クエリの実行計画を確認
EXPLAIN SELECT * FROM users WHERE email = 'tanaka@example.com';
```

### データのインポート・エクスポート

**1. CSVファイルのインポート**
```
1. テーブルを右クリック→「Table Data Import Wizard」
2. CSVファイルを選択
3. カラムマッピングを確認
4. インポートを実行
```

**2. データベースのエクスポート**
```
1. メニューから「Server」→「Data Export」
2. エクスポートするスキーマを選択
3. エクスポートオプションを設定：
   - Export to Dump Project Folder
   - Export to Self-Contained File
4. 「Start Export」を実行
```

## 5. 実践的な使用例

### ユーザー管理システムの構築

**1. データベース設計**
```sql
-- データベース作成
CREATE DATABASE user_management;
USE user_management;

-- ユーザーテーブル
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- ロールテーブル
CREATE TABLE roles (
    role_id INT AUTO_INCREMENT PRIMARY KEY,
    role_name VARCHAR(50) UNIQUE NOT NULL,
    description TEXT
);

-- ユーザーロール関連テーブル
CREATE TABLE user_roles (
    user_id INT,
    role_id INT,
    assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(role_id) ON DELETE CASCADE
);
```

**2. サンプルデータの投入**
```sql
-- ロールデータ
INSERT INTO roles (role_name, description) VALUES
('admin', 'システム管理者'),
('user', '一般ユーザー'),
('moderator', 'モデレーター');

-- ユーザーデータ
INSERT INTO users (username, password_hash, email, first_name, last_name) VALUES
('admin_user', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', 'admin@example.com', '管理', '太郎'),
('john_doe', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', 'john@example.com', 'John', 'Doe'),
('jane_smith', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', 'jane@example.com', 'Jane', 'Smith');

-- ユーザーロール関連
INSERT INTO user_roles (user_id, role_id) VALUES
(1, 1), -- admin_user に admin ロール
(2, 2), -- john_doe に user ロール
(3, 3); -- jane_smith に moderator ロール
```

**3. 実用的なクエリ例**
```sql
-- アクティブユーザー一覧（ロール付き）
SELECT 
    u.username,
    u.email,
    CONCAT(u.first_name, ' ', u.last_name) as full_name,
    GROUP_CONCAT(r.role_name) as roles,
    u.created_at
FROM users u
LEFT JOIN user_roles ur ON u.user_id = ur.user_id
LEFT JOIN roles r ON ur.role_id = r.role_id
WHERE u.is_active = TRUE
GROUP BY u.user_id;

-- 特定ロールのユーザー検索
SELECT u.username, u.email, u.first_name, u.last_name
FROM users u
JOIN user_roles ur ON u.user_id = ur.user_id
JOIN roles r ON ur.role_id = r.role_id
WHERE r.role_name = 'admin' AND u.is_active = TRUE;
```

## 6. トラブルシューティング

### よくある問題と解決方法

**1. 接続エラー**
```sql
-- MySQLサービスの状態確認
-- Windows
net start mysql80

-- Linux
sudo systemctl status mysql
sudo systemctl start mysql
```

**2. パスワードリセット**
```sql
-- rootパスワードをリセット
-- 安全モードでMySQLを起動後
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
FLUSH PRIVILEGES;
```

**3. 文字化け対策**
```sql
-- 文字セットの確認
SHOW VARIABLES LIKE 'character_set%';

-- データベースの文字セット設定
CREATE DATABASE sample_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

MySQL WorkbenchのGUIを活用することで、データベースの設計から運用まで効率的に行えます。特に初心者の場合は、視覚的なツールを使いながら徐々にSQLクエリに慣れていくことをお勧めします。
