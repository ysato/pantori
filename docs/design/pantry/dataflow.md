# データフロー図

## ユーザーインタラクションフロー

### メイン認証フロー
```mermaid
flowchart TD
    A[ユーザー] --> B[ログインページ]
    B --> C{認証成功？}
    C -->|Yes| D[アイテム管理画面]
    C -->|No| E[エラーメッセージ表示]
    E --> B
```

### アイテム管理フロー
```mermaid
flowchart TD
    A[アイテム一覧] --> B{操作選択}
    B -->|新規登録| C[アイテム登録フォーム]
    B -->|編集| D[アイテム編集フォーム]
    B -->|削除| E[削除確認]
    B -->|在庫更新| F[在庫更新フォーム]
    
    C --> G[入力検証]
    D --> G
    F --> H[数値検証]
    
    G -->|Valid| I[アイテム保存]
    G -->|Invalid| J[エラー表示]
    H -->|Valid| K[在庫更新]
    H -->|Invalid| L[数値エラー表示]
    
    J --> C
    L --> F
    I --> A
    K --> A
    
    E -->|確認| M[アイテム削除]
    E -->|キャンセル| A
    M --> A
```

## データ処理フロー

### 認証・認可フロー
```mermaid
sequenceDiagram
    participant U as ユーザー
    participant C as コントローラー
    participant S as セッション
    participant M as ミドルウェア
    participant D as データベース
    
    U->>C: ログインリクエスト
    C->>D: 認証情報照会
    D-->>C: ユーザー情報
    C->>S: セッション作成
    C-->>U: リダイレクト（成功時）
    
    Note over U,D: 以後のリクエスト
    U->>M: リクエスト
    M->>S: セッション確認
    S-->>M: 認証状態
    M-->>C: 認証済みリクエスト
    C-->>U: レスポンス
```

### CRUD 処理フロー
```mermaid
sequenceDiagram
    participant U as ユーザー
    participant C as コントローラー
    participant R as FormRequest
    participant M as Eloquentモデル
    participant D as データベース
    
    U->>C: フォーム送信
    C->>R: バリデーション
    R-->>C: 検証結果
    
    alt 検証成功
        C->>M: モデル操作
        M->>D: SQL実行
        D-->>M: 結果
        M-->>C: 処理結果
        C-->>U: 成功レスポンス
    else 検証失敗
        C-->>U: エラーレスポンス
    end
```

### 補充必要アイテム判定フロー
```mermaid
flowchart TD
    A[補充一覧画面アクセス] --> B[パントリー権限確認]
    B -->|権限あり| C[全アイテム取得]
    B -->|権限なし| D[403エラー]
    
    C --> E[在庫情報取得]
    E --> F{補充必要？}
    F -->|Yes| G[補充対象に追加]
    F -->|No| H[次のアイテム]
    
    G --> I{他にアイテム？}
    H --> I
    I -->|Yes| F
    I -->|No| J[補充一覧表示]
    
    J --> K[視覚的強調表示]
```

## エラーハンドリングフロー

### 入力検証エラー
```mermaid
flowchart TD
    A[フォーム入力] --> B[FormRequest検証]
    B -->|数値形式エラー| C[数値エラーメッセージ]
    B -->|必須項目エラー| D[必須エラーメッセージ]
    B -->|権限エラー| E[アクセス拒否エラー]
    B -->|範囲外エラー| F[範囲エラーメッセージ]
    
    C --> G[フォーム再表示]
    D --> G
    E --> H[403ページ]
    F --> G
```

### システムエラー処理
```mermaid
sequenceDiagram
    participant U as ユーザー
    participant C as コントローラー
    participant D as データベース
    participant L as ログ
    
    U->>C: リクエスト
    C->>D: データベース操作
    D-->>C: エラー発生
    C->>L: エラーログ記録
    C-->>U: ユーザーフレンドリーエラー
```

## データ整合性保証フロー

### カスケード削除
```mermaid
flowchart TD
    A[パントリー削除要求] --> B[権限確認]
    B -->|権限あり| C[関連アイテム確認]
    B -->|権限なし| D[403エラー]
    
    C --> E[アイテム削除実行]
    E --> F[パントリー削除実行]
    F --> G[削除完了]
    
    G --> H[成功メッセージ]
    H --> I[アイテム一覧へリダイレクト]
```

### トランザクション管理
```mermaid
sequenceDiagram
    participant C as コントローラー
    participant T as トランザクション
    participant M as モデル
    participant D as データベース
    
    C->>T: トランザクション開始
    C->>M: 操作1実行
    M->>D: SQL1
    C->>M: 操作2実行  
    M->>D: SQL2
    
    alt 全て成功
        C->>T: コミット
        T->>D: コミット実行
    else いずれか失敗
        C->>T: ロールバック
        T->>D: ロールバック実行
    end
```
