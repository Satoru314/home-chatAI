# home-chatAI

マイクに向かって日本語を話すと、リアルタイムで文字起こしされるプログラム。
オフライン音声認識エンジン [Vosk](https://alphacephei.com/vosk/) を使用。


---

## 前提環境（別マシンでゼロから動かす場合）

以下がOS側に必要。無ければインストールする。

| 前提 | 用途 | 確認コマンド | 無い場合（Debian/Ubuntu 系）|
|---|---|---|---|
| Python 3 | 実行本体 | `python3 --version` | `sudo apt install python3` |
| venv | プロジェクト専用の環境（箱）を作る | `python3 -m venv --help` | `sudo apt install python3-venv` |
| pip | Python パッケージ管理 | `pip --version` | venv 内に同梱される |
| PortAudio | マイク入力（sounddevice が内部で使う OS ライブラリ）| — | `sudo apt install portaudio19-dev` |

### WSL2 でマイクを使う場合（追加）

WSL2 は標準ではマイクが見えない。WSLg 経由で PulseAudio に橋渡しするため、以下も入れる。

```bash
sudo apt install pulseaudio-utils libasound2-plugins
```

マイクが Vosk から見えているかの確認:

```bash
pactl list sources short      # RDPSource 系の行が出れば Windows のマイクが届いている
python test_microphone.py -l  # sounddevice が認識するデバイス一覧
```

---

## セットアップ手順

プロジェクトのディレクトリに移動してから、以下を上から順に実行する。

### 1. venv（専用環境の箱）を作る

```bash
python3 -m venv .venv
```

### 2. 箱を有効化（activate）する

```bash
source .venv/bin/activate
```

成功するとプロンプト先頭に `(.venv)` が付く。
※ このプロジェクトを使うときは、毎回この 1 行を最初に実行する。

### 3. 依存ライブラリを入れる

```bash
pip install -r requirements.txt
```

`requirements.txt` の中身（外部ライブラリ）:

- `vosk` … 音声認識エンジン
- `sounddevice` … マイク入力

---

## 実行

```bash
python test_microphone.py -m ja
```

- `-m ja` … 言語コード。初回はモデルが自動ダウンロードされる（`~/.cache/vosk/` に保存）。
- マイクに日本語を話すと、途中経過（partial）と確定文（result）が表示される。
- `Ctrl + C` で終了。

### その他のオプション

```bash
python test_microphone.py -l          # 音声デバイス一覧を表示して終了
python test_microphone.py -d <番号>    # 使用する入力デバイスを指定
python test_microphone.py -m ja -f out.wav   # 録音を out.wav に保存
```

---

## 補足

- **モデルの置き場所**: `-m ja` で落ちるモデルはプロジェクト外の `~/.cache/vosk/` に入る。プロジェクトをコピーしてもモデルは付いてこないが、`-m ja` 実行時に再ダウンロードされる（ネット接続が必要）。
- **精度について**: `-m ja` は軽量モデル（small, 約 48MB）。誤変換が多い場合は高精度な大モデル（`vosk-model-ja-0.22`, 約 1GB）への差し替えを検討する。

---

## 高精度モデル（大モデル）を使う

軽量モデルより誤変換が少ない大モデル `vosk-model-ja-0.22`（約 1GB）を使う場合。

### 1. ダウンロードと展開（プロジェクト直下に置く）

```bash
curl -SL -O https://alphacephei.com/vosk/models/vosk-model-ja-0.22.zip
unzip vosk-model-ja-0.22.zip
rm vosk-model-ja-0.22.zip
```

`unzip` が無ければ先に `sudo apt install unzip`。

### 2. 実行（`-m` にモデルのフォルダを指定）

```bash
python test_microphone.py -m ./vosk-model-ja-0.22
```

`-m` にフォルダを渡すとそのモデルを直接読み込む。従来どおり `-m ja`（言語コード = 小モデル自動DL）も引き続き使える。

