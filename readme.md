# MacでのPython環境構築ガイド

> 目的：**Homebrew → pyenv → Python → venv** の順に、はじめての方でも迷わず環境構築できるようにします。  
> Macの標準シェル **zsh** を前提にしています（macOS Catalina以降の標準）。
> Windowsの方はごめんなさい。無理です。
> 
---

セットアップが完了したあとは、以下でPythonの練習ができます。   
    - [基礎練習](https://github.com/PM2951/Python-practice)    
    - [グラフ作成練習](https://github.com/PM2951/Python-graph)  
    - [DNA解析練習](https://github.com/PM2951/Python-dna)
    - [機械学習AI練習](https://github.com/PM2951/Python-AI)


---

## 0. いまのMacの状態を確認する

1. **AppleシリコンかIntelか**

   ```bash
   uname -m
   ```

   * `arm64` → Appleシリコン（M1/M2/M3…）
   * `x86_64` → Intel Mac

2. **シェルの確認**（基本は zsh）

   ```bash
   echo $SHELL
   ```

   → （zshを使っている場合の例） /bin/zsh

   zshとbashでややコードが異なるため、注意してください。

4. **開発者ツール（Xcode Command Line Tools）**

   * Pythonのビルドに必要です。未導入ならインストール：

   ```bash
   xcode-select --install
   ```

5. **VS Codeアプリをインストールする**

   これはプログラミングのコードを書くときの便利ツールです。

   色つけたり自動修正してくれたり、AIでコード生成してくれたりとめちゃくちゃ便利なので、入れることをお勧めします。

   <https://code.visualstudio.com/>

   入れたのちに、拡張パッケージでよりレベルアップできます。

   <https://pythonsoba.tech/vscode-extension/>
   これを参考に入れてみると良いです。

   最低でも "Pylance"と"Jupyter"を入れとくと良いです。
   

---

## 1. Homebrew をインストールする（Macの定番パッケージ管理）

1. **インストールコマンド**（公式推奨）　https://brew.sh/

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. **PATHの設定（必須）**
   使っている **シェルごと** に設定ファイルが異なります。**テキストエディタ（TextEdit）で開いて追記**する方法にします。

   **(a) zsh の場合（macOS標準）**

   1. 設定ファイルを開く（なければ新規作成されます）

      ```bash
      open -e ~/.zprofile
      ```

   2. **Appleシリコン (arm64)** の方は、開いたウィンドウの末尾に次の1行を貼り付けて保存：

      ```bash
      eval "$(/opt/homebrew/bin/brew shellenv)"
      ```
   
      **Intel (x86\_64)** の方は次の1行を貼り付けて保存：

      ```bash
      eval "$(/usr/local/bin/brew shellenv)"
      ```

   3. 反映：

      ```bash
      source ~/.zprofile
      ```

   **(b) bash の場合**

   1. 設定ファイルを開く：

      ```bash
      open -e ~/.bash_profile
      ```

   2. **Appleシリコン (arm64)** の方は次を貼り付けて保存：

      ```bash
      eval "$(/opt/homebrew/bin/brew shellenv)"
      ```

      **Intel (x86\_64)** の方は次を貼り付けて保存：

      ```bash
      eval "$(/usr/local/bin/brew shellenv)"
      ```

   3. 反映：

      ```bash
      source ~/.bash_profile
      ```

   > `open -e` は mac の TextEdit を開きます。VS Code を使いたい場合は `code ~/.zprofile` などでもOKです（`code` コマンドが有効な場合）。

3. **確認**

   ```bash
   brew -v
   ```

> もし `brew` コマンドが見つからない場合は、**ターミナルを一度閉じて開き直してください**。

---

## 2. pyenv をインストールする（複数のPythonを切替）

1. **pyenv の導入**

   ```bash
   brew install pyenv
   ```

2. **シェル初期化設定を追加**（zsh / bash でファイルが異なります）

   **(a) zsh の場合（`.zshrc` に追記）**

   1. ファイルを開く：

      ```bash
      open -e ~/.zshrc
      ```

   2. 下記を末尾へ貼り付けて保存：

      ```bash
      export PYENV_ROOT="$HOME/.pyenv"
      export PATH="$PYENV_ROOT/bin:$PATH"
      eval "$(pyenv init --path)"
      eval "$(pyenv init -)"
      ```

   3. 反映：

      ```bash
      source ~/.zshrc
      ```

   **(b) bash の場合（`.bashrc` に追記し、`.bash_profile` から読み込む）**

   1. `.bashrc` を開く：

      ```bash
      open -e ~/.bashrc
      ```

   2. 下記を貼り付けて保存：
   
      ```bash
      export PYENV_ROOT="$HOME/.pyenv"
      export PATH="$PYENV_ROOT/bin:$PATH"
      eval "$(pyenv init --path)"
      eval "$(pyenv init -)"
      ```

   3. `.bash_profile` から `.bashrc` を読み込む設定（未設定なら追加）：
  
      bash_profileを開く

         ```bash
         open -e ~/.bash_profile
         ```

      追加して保存：
   
         ```bash
         [[ -r ~/.bashrc ]] && . ~/.bashrc
         ```

   4. 反映：

      ```bash
      source ~/.bash_profile
      ```

3. **依存ライブラリ**（任意：Pythonをビルドするため、必要に応じて）

   ```bash
   brew install openssl readline sqlite3 xz zlib tcl-tk
   ```

4. **確認**

   ```bash
   pyenv -v
   ```

> これで、複数のPythonバージョンを並行してインストール・切替できるようになります。

---

## 3. Python をインストールする（pyenv経由）

1. **インストール可能なバージョン一覧を確認**

   ```bash
   pyenv install --list | grep " 3\."
   ```

   例：3.12.x や 3.13.x など、安定版の**最新マイナー**を選ぶのがおすすめです。

2. **Pythonのインストール**（例：3.12.1 ）

   ！ここではpython 3.12.1をインストールしてみます。状況に応じてインストールするバージョンは変更してください。

   ```bash
   pyenv install 3.12.1
   ```


4. **グローバル（全体デフォルト）を設定**

   Macだとデフォルトでpython 2.10 などが入っているかもしれないので、バージョンを先ほどインストールしたものに変更する。

   ！ここではpython 3.12.1を設定してみます。状況に応じてバージョンを変更してください。

   ```bash
   pyenv global 3.12.1
   ```

   ターミナルを一度閉じて開き直す。


   ```bash
   python --version
   which python
   ```

   * `python --version` が `Python 3.12.1` を指していればOK。

   * `which python` が `~/.pyenv/shims/python` または、`<自分のMACの名前>/.pyenv/shims/python`を指していればOK。

> **プロジェクトごと**にバージョンを固定したい場合は、そのフォルダで `pyenv local 3.12.x` を使います（`.python-version` が作成されます）。

5. 動作確認

   ```bash
   python
   ```
   pythonのコンソールが有効化されます。
   
   
4. 以下のコードを実行：

   ```python
   print("Hello, Python on Mac!")
   ```

   Hello, Python on Mac!と出ると思います

---

## 4. pip の基本設定

1. **pip を最新化**

   ```bash
   python -m pip install --upgrade pip setuptools wheel
   ```

2. 動作確認
   
   ```bash
   pip --version
   ```


> **禁止事項**：`sudo pip install` は環境破壊の主因になりがちなので避けましょう。

---

## （以下任意） 5. 仮想環境　venv を使ってみる？

* **venv** はプロジェクト専用のPythonとライブラリの「箱」。
* プロジェクトごとに独立してライブラリを管理でき、他のプロジェクトへ影響しません。
* 典型的な流れ：

  1. `pyenv` で **Python本体**（バージョン）を切り替える。
  2. その上で `venv` を **プロジェクトごと**に作る。


## venv の作り方（作成 → 有効化 → 無効化）

以下は、任意の新規プロジェクト `myproj` を例にした手順です。

1. **作業用フォルダを作る**

   ```bash
   mkdir -p ~/projects/myproj
   cd ~/projects/myproj
   ```

2. **（任意）このプロジェクトに使う Python を固定**

   ```bash
   pyenv local 3.12.1
   ```

3. **venv を作成**（フォルダ名は `.venv` が慣例）

   ```bash
   python -m venv .venv
   ```

4. **有効化（activate）**

   ```bash
   source .venv/bin/activate
   ```

   * プロンプトの先頭に `(.venv)` が付きます。これが**仮想環境が有効**の印。
  
   * 別の名前の仮想環境を作りたいとき（new_venvという環境を作る場合）

     ```bash
     python -m venv new_venv
     source new_venv/bin/activate
     ```

     となります。名前の部分（new_venv）を入れ替えることで独自の仮想環境を構築できます。

5. **パッケージのインストール例**

   ```bash
   pip install requests numpy pandas
   ```

6. **無効化（deactivate）**

   ```bash
   deactivate
   ```


---

## 7. よくあるトラブルと対処

* **`python` の場所が期待と違う**

  * `which python` が `~/.pyenv/shims/python` になっているか確認。違う場合は `exec $SHELL -l` で再読み込み、または `source ~/.zshrc`。

* **ビルドに失敗する**（`pyenv install` 中のエラー）

  * 依存パッケージが必要：`brew install openssl readline sqlite3 xz zlib tcl-tk`
  * 古いビルドキャッシュが邪魔する場合：`rm -rf ~/.cache/pyenv` して再試行。

* **Homebrew のPATHが通っていない**

  * `echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile && source ~/.zprofile`（Appleシリコン例）

* **仮想環境で pip が使えない**

  * `source .venv/bin/activate` を忘れていないか確認。

* **片付け・アンインストール**

  ```bash
  pyenv uninstall 3.12.x
  ```

* **仮想環境を削除**

  ```bash
  rm -rf .venv
  ```

---


## `~/.zshrc` と `~/.zprofile` の違い

**結論（TL;DR）**

* `~/.zprofile`：**ログイン**時に1回だけ読み込まれる設定。**PATHなど一度だけ設定すればよい環境変数**を置く。
* `~/.zshrc`：**対話シェル**を開くたびに読み込まれる設定。**エイリアス／補完／プロンプト／pyenvの初期化**など、対話操作向け設定を置く。

**読み込み順序（代表例）**

1. `~/.zshenv`（常に）
2. `~/.zprofile`（ログイン時）
3. `~/.zshrc`（対話時）
4. `~/.zlogin`（ログイン時、最後）

> macOS の Terminal.app は既定で **ログインシェル**として zsh を起動するため、通常は **`.zprofile` も `.zshrc` も両方**読み込まれます。iTerm2 や VS Code は設定次第で挙動が変わる場合があります。

**どっちに書く？（典型例）**

* Homebrew の PATH（zsh）：**`.zprofile`**

  ```bash
  # Appleシリコン
  eval "$(/opt/homebrew/bin/brew shellenv)"
  # Intel の場合はこちらを使用
  # eval "$(/usr/local/bin/brew shellenv)"
  ```
* Homebrew の PATH（bash）：**`.bash_profile`**

  ````bash
  # Appleシリコン
  eval "$(/opt/homebrew/bin/brew shellenv)"
  # Intel の場合はこちらを使用
  # eval "$(/usr/local/bin/brew shellenv)"
  ```bash
  # ~/.zprofile
  eval "$($(brew --prefix)/bin/brew shellenv)"
  ````
* 一度だけ通せばよい PATH や環境変数（例：`PATH`, `LANG`）：**`.zprofile`**
* pyenv の初期化や対話用の設定（エイリアス・プロンプト・補完）：**`.zshrc`**

  ```bash
  # ~/.zshrc
  export PYENV_ROOT="$HOME/.pyenv"
  command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"
  eval "$(pyenv init -)"
  # pip を仮想環境外で使えなくする保護（任意）
  export PIP_REQUIRE_VIRTUALENV=true
  ```

**反映方法**

* ファイルを編集したら、当該ファイルを再読み込み：

  ```bash
  source ~/.zprofile   # zprofile を編集した直後
  source ~/.zshrc      # zshrc を編集した直後
  ```
* うまく反映しない場合は **ターミナルを再起動**。

