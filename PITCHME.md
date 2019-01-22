## Rustのエラーハンドリングについての話

---

### About

name: nasa(近藤アサン)

福岡生まれ福岡育ち

Q.英語喋れんの？ A.無理だ

ONE OK ROCKを聞きながら奇声を上げながらコード書いてる

Rustでコマンドラインツールを作って遊んでる

Twitter: @nasa_desu

GitHub: k-nasa

Note:
- 母に歌を奇声と間違えた話
- GithubID載せてるのでよかったらコマンドラインツールを使ってみて

---

### 今日の注意
- 間違ったことを話してたらマサカリをぶん投げてください
- 初心者向けのお話になります
- 超温かい目で見てください

![温かい目](assets/atatakaime.gif)

Note:
- Rustやったことない人、やりたての人向けの話
- こんな事知ってるわ！って人は,,,
- 自分自身も初心者です温かい目を忘れずに

---

### Result型について

Result型はエラーになる可能性を示すジェネリクス列挙型

- 成功時はOkで
- 失敗時はErrでくるんであげる

たしかこんな定義

```
enum Result<T, E> {
  Ok(t),
  Err(e)
}
```

---

### エラーハンドリングどうしたらいいの？

- unwrap
- unwrap_or_else
- expect
- matchで丁寧に

一個ずつ見ていく

Note:
- これらはよく使うやつ

---
### unwrap
無理矢理脱がす
スタックトレースでちゃう

expectはunwrapとほぼ一緒(unwrap_failedにわたすメッセージを指定できる)

```
pub fn unwrap(self) -> T {
  match self {
    Ok(t) => t,
    Err(e) => unwrap_failed("called `Result::unwrap()` on an `Err` value", e),
  }
}

fn unwrap_failed<E: fmt::Debug>(msg: &str, error: E) -> ! {
    panic!("{}: {:?}", msg, error)
}
```

Note:
- 値を無理やり取りたい時に使える
- 普通にpanicするので、あんまよくない

---

### unwrap_or_else

関数を受け取ってそれをeに適用する

```rust
pub fn unwrap_or_else<F: FnOnce(E) -> T>(self, op: F) -> T {
  match self {
    Ok(t) => t,
    Err(e) => op(e)
  }
}
```

---

### match
一番丁寧なやつ

Resultに定義されたメソッドじゃやりきれないときはこれ

```rust
match File::open("hoge") {
  Ok(_) => (),
  Err(e) => eprintln!("{}", e),
}

```

---

自分のしてしまったミスはもしかしたら初心者あるあるかもしれないので紹介

---
### 初心者あるある(私だけかも？)

全部Resultを返すやつ

え。全部成功するの前提？

```
fn crate(path: &Path) {
    create_dir(path);
    create_dir(path.join("contents"));
    create_dir(path.join("layouts"));
    create_dir(path.join("public"));
    create_dir(path.join("assets"));

    File::create(path.join("config.toml"));
}
```

---

### 自信過剰すぎ
せめてunwrapしようよ

![私失敗しないので](assets/dr_x.jpg)

Note:
某大門未知子のような自信プリだが、失敗に築けない

---

### イケてない

```
fn crate(path: &Path) {
    create_dir(path).unwrap_or_else(|e| eprintln!("{}", e));
    create_dir(path.join("contents")).unwrap_or_else(|e| eprintln!("{}", e));
    create_dir(path.join("layouts")).unwrap_or_else(|e| eprintln!("{}", e));
    create_dir(path.join("contents")).unwrap_or_else(|e| eprintln!("{}", e));
    create_dir(path.join("assets")).unwrap_or_else(|e| eprintln!("{}", e));

    File::create(path.join("config.toml"));
}
```

Note:
- 昔の私、「エラーハンドリング大事や！」

---

### 素直にResultでよくね？

```
fn create_project_dir(path: &Path) -> Result<(), Error> {
    create_dir(path)?;
    create_dir(path.join("contents"))?;
    create_dir(path.join("layouts"))?;
    create_dir(path.join("public"))?;
    create_dir(path.join("assets"))?;

    File::create(path.join("config.toml"))?;

    Ok(())
}
```

?はこんな定義
```
match result {
  Ok(t) => t,
  Err(e) => return e,
}
```

Note:
正確にはerror時に返り値への暗黙的な変換が行われる

---

## ?使えばいい感じになる！

---

### そうも行かない時がある
Error型にもいろいろある

```
fn create_project_dir(path: &Path) -> Result<(), ここを迷う> {
    create_dir(path)?;
    create_dir(path.join("contents"))?;
    create_dir(path.join("layouts"))?;
    create_dir(path.join("public"))?;
    create_dir(path.join("assets"))?;

    File::create(path.join("config.toml"))?;

    Ok(())
}
```
@[1]

---

### そこでFailureですよ
Failureとは?

実験的な新しいエラー処理ライブラリ

- failure::Error型とfailuer::Fail traitを提供しているやつ

---

failure::Errorを使うとfailure::Fail traitを実装してるやつをいい感じにさばける。
```
fn create_project_dir(path: &Path) -> Result<(), failuer::Error> {
    create_dir(path)?;
    create_dir(path.join("contents"))?;
    create_dir(path.join("layouts"))?;
    create_dir(path.join("public"))?;
    create_dir(path.join("assets"))?;

    File::create(path.join("config.toml"))?;

    Ok(())
}
```
---

### 独自エラーも簡単に

```rust
#[derive(Debug, Fail)]
enum CliError {
  #[fail(display = "{:?}", error)]
  Hoge { error: std::io::Error }

  #[fail(display = "invalid input")]
  InvalidInput,
  };
}

impl From for CliError {
  fn from(hoge) {
    hoge
  }
```

---
### まとめ
Failureですよ！
