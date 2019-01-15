## RustのResult型についての話

---

### About

name: nasa

福岡生まれ福岡育ち

Q.英語喋れんの？ A.無理だ

Rustでコマンドラインツールを作って遊んでる

Twitter: @nasa_desu

Github: k-nasa

---

### 今日の注意
- 大した話はできませんので注意
- 間違ったことを話してたらマサカリをぶん投げてください

---

### 一旦Result型について

Result型はエラーになる可能性を示す列挙型
(HaskellでいうEither)

たしかこんな定義

```
enum Result<T, E> {
  Ok(t),
  Err(e)
}
```

---

### 一旦Result型について
成功時はOkで

失敗時はErrでくるんであげる

---

### エラーハンドリングどうしたらいいの？

- unwrap
- unwrap_or_else
- unwrap_or_default
- expect
- matchで丁寧に

一個ずつ見ていく

---
### unwrap
無理矢理脱がす
スタックトレースでちゃう

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

---

### unwrap_or_else

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

```rust
match File::open("hoge") {
  Ok(_) => (),
  Err(e) => eprintln!("{}", e),
}

```

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

![私失敗しないので] (assets/dr_x.jpg)

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

---

### そうも行かない時がある
Error型にもいろいろある

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
@[1]
