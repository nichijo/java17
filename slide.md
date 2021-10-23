---
theme: "moon"
title: java17
---

# java17

---

## プロフィール
- にちじょうです
- Java, C# バックエンド周りがメイン

---

今日はこのあたりを中心に話します
- Java11からの変化
- Java17新機能

---

## Java17

Java11 に続き 久方ぶりのLTS  
覚えておいて損はないバージョン

--

## Java11からの変化

※ 言語機能に限定

```
- 306: 	Restore Always-Strict Floating-Point Semantics
- 361: 	Switch Expressions
- 378: 	Text Blocks
- 394: 	Pattern Matching for instanceof
- 395: 	Records
- 409: 	Sealed Classes
```

レコード、シールクラス、スイッチ式周りが大きい

---

### 306: 	Restore Always-Strict Floating-Point Semantics

昔はいろいろあって、CPUによっては浮動小数計算をするときにコストがかかったようだ。それを避けるためにJavaでは厳密ではない方法で処理をしていたらしい。
厳密に計算するときには `strictfp` という修飾子を使うことで解決できた。

現代ではなんというか、そんなの気にしなくてよくなったんで、デフォルトの挙動を `strictfp` を使うようにしました。　という話らしい。

---

###  361: 	Switch Expressions

スイッチ式、C# でもおなじみのやつ
今まであったのは「スイッチ**文**」そこ注意。

RustやScalaだと **パターンマッチング** と言われています。
C++23でも提案が出ていたり、結構いろいろな言語で使える（ようになる）のではないかと思います。

--

<aside class="notes">
とりあえずコードを見比べてみる
上が今までの「switch文」　下が「switch式」です
まず、目に見えるところでいえば、書き方が結構違っていますね
アロー演算子を使っている、ブレイクがなくなっている あたりでしょうか。
</aside>

switch文
```java 
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        System.out.println(6);
        break;
    case TUESDAY:
        System.out.println(7);
        break;
    case THURSDAY:
    case SATURDAY:
        System.out.println(8);
        break;
    case WEDNESDAY:
        System.out.println(9);
        break;
}
```
switch式
```java
switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> System.out.println(6);
    case TUESDAY                -> System.out.println(7);
    case THURSDAY, SATURDAY     -> System.out.println(8);
    case WEDNESDAY              -> System.out.println(9);
}
```

--

<aside class="notes">
うん、それだけだとあんまり恩恵ないよね。
プログラミングにおける 文/式 は大きく意味合いが違います。

その解説は今回の範囲を大きく逸脱するので省きますが、
文は値を返さず…（if文、for文とかね）
式は評価できる（値を返す、変数に代入できる）ものです。

ということで、switch式も代入可能ですので…
この例みたいな使い方ができるわけです。
</aside>


だけじゃない switch式

```java
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    case THURSDAY, SATURDAY     -> 8;
    case WEDNESDAY              -> 9;
};
```
```java
static void howMany(int k) {
    System.out.println(
        switch (k) {
            case  1 -> "one";
            case  2 -> "two";
            default -> "many";
        }
    );
}
```

--

<aside class="notes">
個人的には結構好きな機能です。かなり簡潔に書けるなと感じます。
特に、switch使うときは冗長で長ったらしくなるのが嫌で、メソッド化したりして解決するとか…
ラムダにするとかね…　なんか一工夫必要だったのが嫌だったのです。

もちろん、きちんとインターフェースや継承を扱えるコードなら、スイッチ文なんかそもそも不要なんですが、
現実そうなっていない場合のほうが多くて…。解決にかけるなら、そのほうが良いです。
</aside>

```java
final String hoge; // そもそもなんでこんな未初期化変数用意しないといけない？
switch (fuga) {
    case 1:
        hoge = "one"; // final あるならその行で解決したくない？
        break; // これいる？
    case 2:
        hoge = "two";
        break;
    default:
        hoge = "default";
        break;
}
```
```java
// しょうがないからメソッドで解決… （詳細設計のメソッド一覧修正しなきゃ…オエー）
String retHoge(int type) {
    switch (type) {
        case 1: return "one";
        case 2: return "two";
        default: return "default";
    }
}
```
```java
// 簡潔でよい
final String hoge = switch (fuga) {
    case 1 -> "one";
    case 2 -> "two";
    default -> "default";
}
```

---

<aside class="notes">
あい、皆さんお待ちかねの TextBlocks です。
複数行対応。まぁもうこれは見てもらったほうが早いです。
</aside>

## 378: 	Text Blocks
複数行の文字列サポート

…個人的には「ようやくか」

--

Javaは複数行文字の扱いが…だった

```java
// なんだその末尾の\nと+はよぉ！
String html = "<html>\n" +
              "    <body>\n" +
              "        <p>Hello, world</p>\n" +
              "    </body>\n" +
              "</html>\n";
```
TextBlocks
```java
String html = """
              <html>
                  <body>
                      <p>Hello, world</p>
                  </body>
              </html>
              """;
```

--

エスケープも…だった
```java
String query = "SELECT \"EMP_ID\", \"LAST_NAME\" FROM \"EMPLOYEE_TB\"\n" +
               "WHERE \"CITY\" = 'INDIANAPOLIS'\n" +
               "ORDER BY \"EMP_ID\", \"LAST_NAME\";\n";
```
TextBlocks
```java
String query = """
               SELECT "EMP_ID", "LAST_NAME" FROM "EMPLOYEE_TB"
               WHERE "CITY" = 'INDIANAPOLIS'
               ORDER BY "EMP_ID", "LAST_NAME";
               """;
```

…　よし！

---

<aside class="notes">
instanceof の型パターン対応です。
とりえあず、まずはコードを見てもらうといいでしょう
</aside>

## 394: Pattern Matching for instanceof

instanceof のパターンマッチング対応

～型パターン対応～

--

<aside class="notes">
うん、このコードは、いかにもJavaなコードです。
下のコードは、型パターンを使って、古臭いキャストがなくなっています。いいですねぇ。

この書き方は、パターンマッチングの一つで、型パターンというものです。
ある型に一致したとき、その処理に入るパターンです。
この時、 `String s` に 文字列型の obj の値が入ります。

はい、この機能はこれだけです。
正直これだけだと、あーこれだけかーって気持ちになるんですが、
Java17ではプレビューですが、これ、switch式でも使えます。

</aside>


いかにもJavaコード
```java
if (obj instanceof String) {
    String s = (String) obj;    // grr...
    ...
}
```

これを型パターンを使っていい感じに扱えるようにしたのが `Pattern Matching for instanceof` です。
```java
if (obj instanceof String s) {
    // Let pattern matching do the work!
    ...
}
```

--

<aside class="notes">
おまけです、Java17では、プレビューではありますが
Switch式で型パターンが扱えます。いいですねぇ。
現状はプリミティブ型は使えないみたいですが、将来的には追加する予定らしい…
</aside>

### Pattern Matching for switch (Preview)

おまけです

```java
Object o = 123L;
String formatted = switch (o) {
    case Integer i -> String.format("int %d", i);
    case Long l    -> String.format("long %d", l);
    case Double d  -> String.format("double %f", d);
    case String s  -> String.format("String %s", s);
    default        -> o.toString();
};
```

---

## 395: 	Records
lombok的な

--

Javaは冗長…、高々二次元座標をあらわすのに必要な情報量が多い
みんなLombokを使うやろそら。
```java
class Point {
    private final int x;
    private final int y;

    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    int x() { return x; }
    int y() { return y; }

    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        Point other = (Point) o;
        return other.x == x && other.y == y;
    }

    public int hashCode() {
        return Objects.hash(x, y);
    }

    public String toString() {
        return String.format("Point[x=%d, y=%d]", x, y);
    }
}
```
Recordを使うと
```java
record Point(int x, int y) { }
```
--

Recordは equals, hashCode, toString などの Object メソッドの実装を自動的に提供する。
また、検証も以下のようにすると行える。

```java
// lo, hi はメンバ変数に勝手に入るらしい
public record Range(int lo, int hi) {
    public Range {
        if (lo > hi)
            throw new IllegalArgumentException(String.format("%d, %d", lo, hi));
    }
}
```

また、こういうこともできるみたい。
```java
record SmallPoint(int x, int y) {
  public int x() { return this.x < 100 ? this.x : 100; }
  public int y() { return this.y < 100 ? this.y : 100; }
}
```

---

## 409: 	Sealed Classes

WIP...