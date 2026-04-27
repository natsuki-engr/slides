---
theme: light-icons
background: https://cover.sli.dev
title: pnpmのtrust-policyはどう動いている？
class: text-left
drawings:
  persist: false
transition: fade-out
mdc: true
overviewSnapshots: true
layout: center
twoslash: true
colorSchema: dark
---

# pnpmのtrust-policyはどう動いている？

ソースコードから no-downgrade の仕組みを追う

<div class="text-right">
Natsuki
</div>

---

# axiosへのサプライチェーン攻撃

2026年3月、axios がサプライチェーン攻撃を受けた

- 攻撃者がメンテナーの npm アカウントを乗っ取り
- 悪意のある v1.14.1 と v0.30.4 を公開

https://www.elastic.co/security-labs/how-we-caught-the-axios-supply-chain-attack

<div v-click class="mt-8 text-xl">

pnpm 10系の `trust-policy: no-downgrade` はこうした攻撃を検出できる

</div>

<div v-click class="mt-4 text-xl">

この仕組みをソースコードから読み解いていく

</div>

---

# 前提知識：npmの「信頼性エビデンス」

npm が提供する2つの信頼性の仕組み

<div v-click>

### Provenance Attestation

パッケージがどのリポジトリのどのコミットから、どの CI 環境でビルドされたかを SLSA provenance として署名・添付する仕組み

https://docs.npmjs.com/generating-provenance-statements

</div>

<div v-click>

### Trusted Publisher（Trusted Publishing）

GitHub Actions 等から OIDC を使って**直接** npm に公開する仕組み
人間が npm トークンを保持する必要がなく、トークン漏洩のリスクがない

https://docs.npmjs.com/about-trusted-publishing

</div>

---

# pnpmのランク付け

pnpm はこの2つにランクを付けている

```typescript
type TrustEvidence = 'provenance' | 'trustedPublisher'

const TRUST_RANK = {
  trustedPublisher: 2,  // 最も信頼度が高い
  provenance: 1,
} as const satisfies Record<TrustEvidence, number>
```

<div v-click>

| ランク | エビデンス | 意味 |
|---|---|---|
| 2 | `trustedPublisher` | OIDC で直接公開（トークン不要） |
| 1 | `provenance` | SLSA provenance 署名あり |
| なし | - | 手動公開（トークンベース） |

</div>

<p class="text-sm mt-4">

https://github.com/pnpm/pnpm/blob/v10.33.0/resolving/npm-resolver/src/trustChecks.ts

</p>

---

# エビデンスの判定

各バージョンのメタデータからエビデンスを判定する関数

```typescript
export function getTrustEvidence(
  manifest: PackageInRegistry
): TrustEvidence | undefined {
  if (manifest._npmUser?.trustedPublisher) {
    return 'trustedPublisher'
  }
  if (manifest.dist?.attestations?.provenance) {
    return 'provenance'
  }
  return undefined
}
```

<div v-click class="mt-4">

npm レジストリのメタデータ上の公開情報**だけ**を見ている

</div>

---

# no-downgrade の全体フロー

```
インストール対象のバージョンの公開日時を取得
    ↓
それより前に公開された全バージョンを走査
（メジャーバージョンを跨いですべて）
    ↓
その中で最も強い信頼性エビデンスを取得
    ↓
インストール対象のエビデンスと比較
    ↓
ダウングレードしていたらエラー 🚨
```

<div v-click class="mt-8 text-xl text-center">

⚠️ 比較基準は <strong>semver（バージョン番号）ではなく公開日時</strong>

</div>

---

# Step 1：公開日時の取得

```typescript
const versionPublishedAt = meta.time[version]
const versionDate = new Date(versionPublishedAt)
```

<div class="mt-8">

npm レジストリのメタデータには各バージョンの公開日時が `time` フィールドに入っている

</div>

<div v-click class="mt-4">

```json
{
  "time": {
    "1.13.4": "2026-01-27T18:18:38.613Z",
    "1.14.0": "2026-03-27T09:50:42.218Z",
    "1.14.1": "2026-03-31T00:21:58.168Z"
  }
}
```

</div>

---

# Step 2：過去の最強エビデンスを探す

```typescript{all|5|8|9|11-14}
function detectStrongestTrustEvidenceBeforeDate(
  meta: PackageMetaWithTime,
  beforeDate: Date,
  options: { excludePrerelease: boolean }
): TrustEvidence | undefined {
  let best: TrustEvidence | undefined

  for (const [version, manifest] of Object.entries(meta.versions)) {
    const publishedAt = new Date(meta.time[version])
    if (!(publishedAt < beforeDate)) continue

    const trustEvidence = getTrustEvidence(manifest)
    if (trustEvidence === 'trustedPublisher') {
      return 'trustedPublisher'  // 最高ランクなので即リターン
    }
    best ||= 'provenance'
  }

  return best
}
```

---

# Step 2 のポイント

<div class="text-xl mt-8">

**全バージョンを走査している**

`meta.versions` の全エントリをイテレーション

→ v0.x, v1.x, v2.x のすべてが対象

</div>

<div v-click class="text-xl mt-8">

**比較基準は公開日時**

v2.0.0 が v1.5.0 より前に公開されていれば、
v1.5.0 をインストールする際に v2.0.0 のエビデンスも比較対象になる

</div>

<div v-click class="mt-8 p-4 border border-gray-500 rounded">

> Trust checks are based solely on publish date, not semver. A package cannot be installed if any earlier-published version had stronger trust evidence.

<p class="text-sm text-right">— pnpm のエラーメッセージ hint より</p>

</div>

---

# Step 3：比較してダウングレードなら中断

```typescript
const currentTrustEvidence = getTrustEvidence(manifest)

if (
  currentTrustEvidence == null ||
  TRUST_RANK[strongestEvidencePriorToRequestedVersion]
    > TRUST_RANK[currentTrustEvidence]
) {
  throw new PnpmError(
    'TRUST_DOWNGRADE',
    `High-risk trust downgrade for "${meta.name}@${version}"
     (possible package takeover)`
  )
}
```

<div v-click class="mt-8">

過去のエビデンスより弱ければ `ERR_PNPM_TRUST_DOWNGRADE` で中断

</div>

<div v-click class="mt-4">

過去にエビデンスを持つバージョンが一つもなければチェックは通過する

</div>

---

# axiosの攻撃を振り返る

npm レジストリ上の axios の信頼性エビデンスの変遷

```
2026-01-27  v1.13.4 公開 (trustedPublisher)
                     ↑ 1.x で初めて Trusted Publishing を導入
    :
2026-02-18  v0.30.3 公開 (エビデンスなし / 手動公開)
    :
2026-03-27  v1.14.0 公開 (trustedPublisher)
2026-03-31  v1.14.1 公開 (エビデンスなし) ← 🚨 攻撃者による公開
2026-03-31  v0.30.4 公開 (エビデンスなし) ← 🚨 攻撃者による公開
    :
2026-04-08  v1.15.0 公開 (trustedPublisher)
2026-04-12  v0.31.0 公開 (trustedPublisher) ← 0.x にも導入
```

---

# v1.14.1（攻撃バージョン）の場合

<div class="text-xl mt-4">

v1.14.1 の公開日（2026-03-31）**より前**に公開された v1.13.4 ~ v1.14.0 が `trustedPublisher` を持っている

</div>

<div v-click class="mt-8">

```
v1.13.4 (trustedPublisher) ──→ v1.14.1 (エビデンスなし)
        ランク 2                       ランク 0
```

</div>

<div v-click class="mt-8 text-2xl text-center">

🚫 ダウングレード検出 → <strong>インストールがブロックされる</strong>

</div>

<div v-click class="mt-4 text-lg">

1.x 系は v1.13.4 で Trusted Publishing を導入していたおかげで防御が機能した

</div>

---

# v0.30.4（攻撃バージョン）の場合

<div class="text-lg">

v0.x 自体は一度も Trusted Publishing を使ったことがなかった

</div>

<div v-click class="mt-4">

v0.x だけ見れば「エビデンスなし → エビデンスなし」でダウングレードなし

</div>

<div v-click class="mt-8 text-lg">

しかし、pnpm は**メジャーバージョンを跨いで全バージョンを公開日時順に走査**する

</div>

<div v-click class="mt-4">

```
v1.13.4 (2026-01-27, trustedPublisher) ──→ v0.30.4 (2026-03-31, エビデンスなし)
```

v0.30.4 の公開日より前に v1.13.4（trustedPublisher）が存在

→ **ダウングレードとして検出される** 🚫

</div>

<div v-click class="mt-8 text-sm">

※ ただし正規の v0.30.3 もブロックされてしまうので、`trust-policy-exclude` で除外するか v0.x 自体に Trusted Publishing を適用する必要がある

</div>

---

# axiosの対応

攻撃後、v0.x にも OIDC ベースの公開ワークフローが追加された

https://github.com/axios/axios/pull/10639

```
2026-04-12  v0.31.0 公開 (trustedPublisher) ← 修正後
```

<div v-click class="mt-8">

### trust-policy に引っかかった場合の対処方法

**バージョンを上げる**：trustedPublisher で公開されたバージョンを使う

**trust-policy-exclude で除外する**：

```ini
# .npmrc
trust-policy=no-downgrade
trust-policy-exclude=パッケージ名@バージョン
```

</div>

---

# まとめ

<div class="text-lg">

pnpm の `trust-policy: no-downgrade` は

- npm レジストリのメタデータから `trustedPublisher`（ランク2）と `provenance`（ランク1）を読み取る
- **公開日時順に**信頼度が下がっていないかを検証

</div>

<div v-click class="mt-8 text-lg">

**強み**：攻撃者が古いバージョン番号で公開しても検出できる

</div>

<div v-click class="mt-4 text-lg">

**注意点**：複数メジャーバージョンを並行メンテナンスしているパッケージでは、Trusted Publishing の導入タイミング次第で意図しないブロックが発生することも

</div>

---

# おまけ：vite で no-downgrade を確認する

vite v6.4.1 をインストールしようとすると trust-policy に引っかかる

```
2024-12-02  v6.0.2  公開 (provenance)
    :  v6.0.3 ~ v6.3.5 は provenance のみ
2025-08-19  v7.1.3  公開 (trustedPublisher) ← 初の trustedPublisher
    :  v7.x は trustedPublisher を継続
2025-10-20  v6.4.1  公開 (provenance)
```

<div v-click>

| | v6.0.2 | v6.4.1 |
|---|---|---|
| 公開日 | 2024-12-02 | 2025-10-20 |
| エビデンス | provenance | provenance |
| 公開日より前の最強 | provenance | **trustedPublisher** (v7.1.3~) |
| 結果 | インストール可 | **ブロック** |

</div>

<div v-click class="mt-4">

別のメジャーバージョン（v7.x）が先に trustedPublisher を導入したことが原因。日時順の判定が表れているケース

</div>

---
layout: center
---

# おしまい
