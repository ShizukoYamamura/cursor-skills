# Code Review — 出力例

C# / Unity コードに対するレビュー出力の具体例。`SKILL.md` の出力フォーマットに従う。

## 入力例（ユーザーから渡されるコード）

```csharp
public class Enemy : MonoBehaviour
{
    public int hp;

    void Update()
    {
        var player = GameObject.Find("Player");
        if (Vector3.Distance(transform.position, player.transform.position) < 5)
        {
            hp = hp - 10;
        }
    }
}
```

## 出力例

## コードレビュー結果

### 🟢 Good
- 敵の挙動が `MonoBehaviour` に素直にまとまっており、距離判定のロジック自体は読みやすいです。

### 🟡 Suggestion（任意）
- `5`（攻撃範囲）や `10`（ダメージ量）がマジックナンバーになっています。`[SerializeField] private float _attackRange = 5f;` のように定数化・インスペクタ公開するとバランス調整が容易になります。
- `hp` が `public` フィールドで外部から直接書き換え可能です。`[SerializeField] private int _hp;` + 公開用プロパティ/メソッド経由のアクセスに変更すると、カプセル化が保てます。

### 🔴 Critical（必須修正）
- **パフォーマンス**: `Update()` 内で毎フレーム `GameObject.Find("Player")` を呼んでおり、非常に高コストかつGC負荷の原因になります。`Start()` でキャッシュすべきです。
- **堅牢性（null参照）**: `player` が見つからない場合 `player.transform` で `NullReferenceException` が発生しクラッシュします。null チェックが必須です。

### 💡 修正コード例

```csharp
public class Enemy : MonoBehaviour
{
    [SerializeField] private int _hp = 100;
    [SerializeField] private float _attackRange = 5f;
    [SerializeField] private int _attackDamage = 10;

    private Transform _player;

    private void Start()
    {
        var playerObject = GameObject.FindWithTag("Player");
        if (playerObject != null)
        {
            _player = playerObject.transform;
        }
    }

    private void Update()
    {
        if (_player == null) return;

        if (Vector3.SqrMagnitude(transform.position - _player.position) < _attackRange * _attackRange)
        {
            _hp -= _attackDamage;
        }
    }
}
```

修正のポイント:
- `Find` を `Start()` でキャッシュし、毎フレームの探索を排除。
- null チェックでクラッシュを防止。
- マジックナンバーを `[SerializeField]` 化。
- `Distance`（平方根計算）を `SqrMagnitude` 比較に置き換えて軽量化。
