# データベース設計

## データベース設計方針

### 設計原則
- **正規化**: 第3正規形までの正規化を基本とする
- **整合性**: 外部キー制約による参照整合性保証
- **パフォーマンス**: 適切なインデックス設計
- **拡張性**: 将来的な機能追加に対応可能な構造

### RDBMS選択
- **推奨**: PostgreSQL（数値精度・制約機能が充実）
- **代替**: MySQL 8.0+（DECIMAL精度対応）

### タイムスタンプ設計方針
- **保存形式**: TIMESTAMP型（UTC基準）
- **出力形式**: ISO8601準拠（`2024-03-15T14:30:45.000Z`）
- **Laravel設定**: `config/app.php` でタイムゾーン='UTC'
- **フロントエンド表示**: JavaScriptで地域時刻に変換

## テーブル設計

### users テーブル

**責務**: ユーザー認証・基本情報管理

```sql
-- ユーザー基本情報
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    pantry_id BIGINT NOT NULL,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (pantry_id) REFERENCES pantries(id) ON DELETE CASCADE,
    UNIQUE (pantry_id, email)
);
```


### pantries テーブル

**責務**: パントリー管理・オーナーシップ

```sql
-- パントリー情報
CREATE TABLE pantries (
    id BIGSERIAL PRIMARY KEY,
    slug VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


### items テーブル

**責務**: アイテム情報管理・在庫数量管理

```sql
-- アイテム基本情報・在庫管理
CREATE TABLE items (
    id BIGSERIAL PRIMARY KEY,
    pantry_id BIGINT NOT NULL,
    name VARCHAR(255) NOT NULL,
    current_quantity DECIMAL(9,2) NOT NULL DEFAULT 0.00,
    threshold_quantity DECIMAL(9,2) NOT NULL DEFAULT 1.00,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (pantry_id) REFERENCES pantries(id) ON DELETE CASCADE
);
```



## 制約設計

### データ整合性制約

**外部キー制約**:
- users.pantry_id → pantries.id (CASCADE)
- items.pantry_id → pantries.id (CASCADE)

**数値制約**:
- 数値範囲: DECIMAL(9,2) = -9999999.99 to 9999999.99
- バリデーション: アプリケーションレベル（Laravel）で実装

**一意性制約**:
- users.(pantry_id, email): パントリー内でメールアドレス一意
- pantries.slug: グローバル一意

### ビジネスルール制約

**カスケード削除**:
```
Pantries削除 → Users削除 → Items削除
```

**データ精度制約**:
- DECIMAL(9,2): 整数部7桁、小数部2桁
- タイムスタンプ: ISO8601形式・UTC基準での管理（TIMESTAMP型使用）


## マイグレーション戦略

### 段階的テーブル作成順序

1. **pantries テーブル**: パントリー基本情報
2. **users テーブル**: ユーザー情報（パントリーとの関係性）
3. **items テーブル**: アイテム・在庫情報管理

### データ初期化

**必須マスターデータ**:
- なし（ユーザー登録から開始）

**テストデータ**:
- サンプルユーザー（開発環境のみ）
- サンプルパントリー・アイテム（開発環境のみ）

