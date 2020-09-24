# クラス LazySegTreeFast

[モノイド](https://ja.wikipedia.org/wiki/%E3%83%A2%E3%83%8E%E3%82%A4%E3%83%89) `(S,⋅:S×S→S,e∈S)` と、`S` から `S` への写像の集合 `F` であって、以下の条件を満たすようなものについて使用できるデータ構造です。

- `F` は恒等写像 `id` を含む。つまり、任意の $x∈S$ に対し `id(x) = x` をみたす。
- `F` は写像の合成について閉じている。つまり、任意の `f,g∈F` に対し `f∘g∈F` である。
- 任意の `f∈F,x,y∈S` に対し `f(x⋅y)=f(x)⋅f(y)` をみたす。
- <b>`F`, `S` はそれぞれ long[`F`の次元]、long[`S`の次元] で表現される。</b>

長さ `N` の `S` の配列に対し、

- 区間の要素に一括で `F` の要素 `f` を作用 (`x = f(x)`)
- 区間の要素の総積の取得

を $O(\log N)$ で行うことが出来ます。全てをlong[]表現で扱うことでオブジェクトの生成やGCを極力抑えた構造になっているため、若干速いです。一方でモノイドや写像の積の定義の引数がimmutableではないため実装に少し気を使います。このライブラリはオラクルとして `op`, `mapping`, `composition` をLazySegTreeと違い、Abstractメソッドとして実装します。以下全て`S`及び`F`はlong[]表現で考えます。

## コンストラクタ

LazySegTreeと違い、型引数は持たず全てlong[]表現で持ちます。引数の意味は以下の通りです。

- `e` : モノイドの単位元のlong[]表現
- `id` : 恒等写像のlong[]表現

```java
public LazySegTree(int n, long[] e, long[] id)
```

長さ `n` の配列 `a[0], a[1], ..., a[n - 1]` を作ります. 初期値はすべて $e$ です.

計算量: $O(n)$

```java
public LazySegTree(long[] dat, long[] e, long[] id)
```

長さ `n` の配列 `a[0], a[1], ..., a[n - 1]` を `dat` により初期化します.

計算量: $O(n)$

## Abstractメソッド
モノイド及び写像のlong[]表現を受け取り、結果のlong[]表現をretに出力するコードを書きます。
高速化のために<b>引数のオ入力用のブジェクトs1, s2, f1, f2,..と出力用のオブジェクトretは同じオブジェクトになることがあります。</b>そのため、途中結果は一旦ローカル変数に保持して最後に一気にretに代入してください。具体的には実装例を見てください。

```java
public abstract void op(long[] s1, long[] s2, long[] ret);
```
モノイドの積 `S×S→S` を定義します。

```java
public abstract void composite(long[] f1, long[] f2, long[] ret);
```
写像の積 `F×F→F` を定義します。

```java
public abstract void mapping(long[] f, long[] s, long[] ret);
```

実装例はこちら。
```java
public void op(long[] s1, long[] s2, long[] ret) {
    // s2 == ret だったりするので、 ret[0] = s1[0] + s2[0] はNG! 
    long a = s1[0] + s2[0];
    if (a >= mod)
        a -= mod;
    long b = s1[1] + s2[1];
    ret[0] = a;
    ret[1] = b;
}

public void composite(long[] f1, long[] f2, long[] ret) {
    long a = f1[0] * f2[0] % mod;
    long b = (f1[0] * f2[1] + f1[1]) % mod;
    ret[0] = a;
    ret[1] = b;
}

public void mapping(long[] f, long[] s, long[] ret) {
    long a = (f[0] * s[0] + f[1] * s[1]) % mod;
    long b = s[1];
    ret[0] = a;
    ret[1] = b;
}
```


## メソッド

### set

```java
public void set(int p, long[] x)
```

`a[p]=x` とします．

計算量: $O(\log n)$

制約: `0 <= p < n`

### get

```java
public long[] get(int p)
```

`a[p]` を取得します．

計算量: $O(\log n)$

制約: `0 <= p < n`

### prod

```java
public long[] prod(int l, int r)
```

`op(a[l], ..., a[r - 1])` を、モノイドの性質を満たしていると仮定して計算します。`l = r` のときは単位元 `e` を返します。

計算量: $O(n)$

制約: `0 <= l <= r <= n`

### allProd

```java
public long[] allProd()
```

`op(a[0], ..., a[n - 1])` を、モノイドの性質を満たしていると仮定して計算します。`n = 0` のときは単位元 `e` を返します。

計算量: $O(1)$

### apply

```java
// (1)
public void apply(int p, long[] f)
// (2)
public void apply(int l, int r, long[] f)
```

- (1): `a[p]` に作用素 `f` を作用させます
- (2): `i∈[l, r)` に対して `a[i]` に作用素 `f` を作用させます

制約

- (1): `0 <= p < n`
- (2): `0 <= l <= r <= n`

計算量

$O(\log n)$

### maxRight

```java
public int maxRight(int l, java.util.function.Predicate<long[]> f)
```

`S` を引数にとり `boolean` を返す関数を渡して使用します。
以下の条件を両方満たす `r` を (いずれか一つ) 返します。

- `r = l` もしくは `f(op(a[l], a[l + 1], ..., a[r - 1])) = true`
- `r = n` もしくは `f(op(a[l], a[l + 1], ..., a[r])) = false`

`f` が単調だとすれば、`f(op(a[l], a[l + 1], ..., a[r - 1])) = true` となる最大の `r`、と解釈することが可能です。

制約

- `f` を同じ引数で呼んだ時、返り値は等しい(=副作用はない)
- __`f(e) = true`__
- `0 <= l <= n`
計算量

$O(\log n)$

### minLeft

```java
public int minLeft(int r, java.util.function.Predicate<long[]> f)
```

`S` を引数にとり `boolean` を返す関数オブジェクトを渡して使用します。
以下の条件を両方満たす `l` を (いずれか一つ) 返します。

- `l = r` もしくは `f(op(a[l], a[l + 1], ..., a[r - 1])) = true`
- `l = 0` もしくは `f(op(a[l - 1], a[l + 1], ..., a[r - 1])) = false`

fが単調だとすれば、`f(op(a[l], a[l + 1], ..., a[r - 1])) = true` となる最小の `l`、と解釈することが可能です。

制約

- `f` を同じ引数で呼んだ時、返り値は等しい(=副作用はない)
- `f(e) = true`
- `0 <= r <= n`

計算量

$O(\log n)$

## 使用例

[AtCoder Library Practice Contest K - Range Affine Range Sum](https://atcoder.jp/contests/practice2/submissions/16982533)
