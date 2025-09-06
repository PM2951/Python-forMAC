# MacでのPython環境構築ガイド

> 目的：**Homebrew → pyenv → Python → venv** の順に、はじめての方でも迷わず環境構築できるようにします。Macの標準シェル **zsh** を前提にしています（macOS向け）。

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
   例） /usr/local/bin/zsh # <- zsh使ってる

3. **開発者ツール（Xcode Command Line Tools）**

   * Pythonのビルドに必要です。未導入ならインストール：

   ```bash
   xcode-select --install
   ```

---

## 1. Homebrew をインストールする（Macの定番パッケージ管理）

1. **インストールコマンド**（公式推奨）

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. **PATHの設定（必須）**

   * Appleシリコン（多くの最新Mac）：

     ```bash
     echo 'eval "$($(brew --prefix)/bin/brew shellenv)"' >> ~/.zprofile
     eval "$($(brew --prefix)/bin/brew shellenv)"
     ```

     ※ `$(brew --prefix)` は通常 `/opt/homebrew` です。
   * Intel Mac：

     ```bash
     echo 'eval "$($(brew --prefix)/bin/brew shellenv)"' >> ~/.zprofile
     eval "$($(brew --prefix)/bin/brew shellenv)"
     ```

     ※ Intel では多くの場合 `/usr/local` が prefix です。

3. **確認**

   ```bash
   brew -v
   ```

> もし `brew` コマンドが見つからない場合は、**ターミナルを一度閉じて開き直す**か、`source ~/.zprofile` を実行してください。

---

## 2. pyenv をインストールする（複数のPythonを切替）

1. **pyenv の導入**

   ```bash
   brew install pyenv
   ```

2. **シェル初期化設定を追加**（`~/.zshrc` に追記）

   ```bash
   echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
   echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
   echo 'eval "$(pyenv init -)"' >> ~/.zshrc
   source ~/.zshrc
   ```

3. **依存ライブラリ**（Pythonをビルドするため、必要に応じて）

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

2. **インストール**（例：3.12.**x** の最新）

   ```bash
   pyenv install 3.12.\
   ```

   ※ `Tab` 補完や `--list` で実際の最新番号を確認して埋めてください（例：`3.12.6`）。

3. **グローバル（全体デフォルト）を設定**

   ```bash
   pyenv global 3.12.x
   exec $SHELL -l   # シェル再読み込み（またはターミナル再起動）
   python --version
   which python
   ```

   * `which python` が `~/.pyenv/shims/python` を指していればOK。

> **プロジェクトごと**にバージョンを固定したい場合は、そのフォルダで `pyenv local 3.12.x` を使います（`.python-version` が作成されます）。

---

## 4. pip の基本設定（安全＆快適に使う）

1. **pip を最新化**

   ```bash
   python -m pip install --upgrade pip setuptools wheel
   pip --version
   ```

2. **グローバルへ直接インストールしない保護**（推奨）

   * `~/.zshrc` に以下を追記：

     ```bash
     echo 'export PIP_REQUIRE_VIRTUALENV=true' >> ~/.zshrc
     source ~/.zshrc
     ```

     これで、**仮想環境外での `pip install` を防止**でき、環境が壊れにくくなります。

3. **pip の設定ファイル**（必要な人向け）

   * macOSのユーザ設定ファイル：`~/.config/pip/pip.conf`
   * 例：インデックスURLを明示して安定運用（通常は既定のままでOK）

     ```ini
     # ~/.config/pip/pip.conf
     [global]
     index-url = https://pypi.org/simple
     ```

> **禁止事項**：`sudo pip install` は環境破壊の主因になりがちなので避けましょう。

---

## 5. venv とは？（仮想環境の基礎）

* **venv** はプロジェクト専用のPythonとライブラリの「箱」。
* プロジェクトごとに独立してライブラリを管理でき、他のプロジェクトへ影響しません。
* 典型的な流れ：

  1. `pyenv` で **Python本体**（バージョン）を切り替える。
  2. その上で `venv` を **プロジェクトごと**に作る。

---

## 6. venv の作り方（作成 → 有効化 → 無効化）

以下は、任意の新規プロジェクト `myproj` を例にした手順です。

1. **作業用フォルダを作る**

   ```bash
   mkdir -p ~/projects/myproj
   cd ~/projects/myproj
   ```

2. **（任意）このプロジェクトに使う Python を固定**

   ```bash
   pyenv local 3.12.x
   python --version   # 期待のバージョンか確認
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

5. **パッケージのインストール例**

   ```bash
   pip install requests numpy pandas
   ```

6. **依存関係の保存**（再現性のために）

   ```bash
   pip freeze > requirements.txt
   ```

7. **無効化（deactivate）**

   ```bash
   deactivate
   ```

> VS Code を使う場合：ワークスペースを `myproj` に開き、コマンドパレット（⇧⌘P）→「Python: Select Interpreter」で `.venv` を選択すると、自動でその環境を使えます。

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

---

## 8. 片付け・アンインストール

* **特定のPythonバージョンを削除**

  ```bash
  pyenv uninstall 3.12.x
  ```

* **仮想環境を削除**

  ```bash
  rm -rf .venv
  ```

---

## 9. 動作確認（Hello, world!）

1. 仮想環境を有効化：

   ```bash
   cd ~/projects/myproj
   source .venv/bin/activate
   ```
2. スクリプト `hello.py` を作成：

   ```python
   print("Hello, Python on Mac!")
   ```
3. 実行：

   ```bash
   python hello.py
   ```

---

## 10. まとめ（最小セット）

1. Homebrew を入れて PATH を通す。
2. `brew install pyenv`。
3. `~/.zshrc` に pyenv 初期化を追記して `source ~/.zshrc`。
4. `pyenv install <最新版>` → `pyenv global <同じ>`。
5. プロジェクトごとに `python -m venv .venv` → `source .venv/bin/activate`。
6. `pip install ...`（**sudo は使わない**）。

---

### 参考キーワード（公式ドキュメント）

* Homebrew Documentation（brewのインストール・shellenv）
* pyenv README（インストールと `pyenv init -`）
* Python公式ドキュメント（`venv`, `pip`）

> この手順は、公式ドキュメントの一般的な推奨事項に沿った**保守的で安全な構成**です。これで研究・開発の多くの用途に十分対応できます。
