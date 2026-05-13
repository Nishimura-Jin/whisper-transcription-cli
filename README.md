# Whisper 文字起こしツール

OpenAIの音声認識モデル「Whisper」を使用し、**文字起こし・フィラー除去・多形式保存**をまとめて行えるCLIツールです。

> このCLIツールを発展させたWebアプリ版はこちら →  
> **[whisper-transcription-app](https://github.com/Nishimura-Jin/whisper-transcription-app)**

---

## 開発の背景

フリーランスでYouTube動画を制作する中で、字幕を付けたいと思ったのがきっかけです。
有料の動画編集ソフトには字幕機能がありましたが、無料で実現できないか調べたところ、
Pythonで実装できることがわかりました。ちょうどPythonを独学で勉強していたこともあり、
練習も兼ねて作りました。

---

## このツールの特徴

- **高精度な文字起こし**: turbo モデルを使用し、高速かつ高精度な文字起こしを実現。
- **フィラー除去**: 文字起こし結果から「えー」「あのー」といった言葉を正規表現で自動除去。
- **マルチフォーマット対応**: SRT, VTT, TXT, TSV, JSONの全形式をコマンド一つで一括出力可能。
- **無限ループ防止**: `condition_on_previous_text=False` を設定し、Whisper特有の誤認の連鎖を防止。
- **進捗表示**: tqdm による進捗バーと、処理内容のリアルタイムログ出力。

---

## 必要環境

- Python 3.8以上
- FFmpeg（音声・動画のデコードに必要）
- ライブラリ: `openai-whisper`, `torch`, `tqdm`

---

## 開発のこだわり

**コマンドラインから柔軟に操作できる設計**  
`argparse` を使い、ファイルパス・出力形式・フィラー除去の有無をコマンドライン引数で切り替えられるようにしました。コードを書き換えなくても使い方を変えられるので、実用的なツールとして使いやすくなっています。

**Whisperの出力データを直接加工してフィラー除去を実装**  
Whisperは文字起こし結果をセグメント（時間ごとの文章）のリストとして返します。最初はテキスト全体に正規表現をかけていましたが、字幕ファイル（SRT）に出力する際は各セグメントのテキストを個別に書き換える必要があることに気づき、`result["segments"]` をループして1件ずつ処理する方法に修正しました。

**GPU/CPU自動切り替え**  
`torch.cuda.is_available()` でCUDAが使えるか確認し、使える場合はGPU、ない場合はCPUで動作するようにしています。自分の環境（RTX3070）では問題なく動いていましたが、GPUがない環境でも使えるように対応しました。

---

## 使い方

準備
```bash
pip install openai-whisper torch tqdm
```

基本的な実行
```bash
python main.py "your_audio_file.mp4"
```

フィラー除去して全形式で保存する場合
```bash
python main.py "your_audio_file.mp4" --clean --format all
```

---

## 対応フォーマット

- `.srt`（字幕ファイル）
- `.vtt`（Web字幕）
- `.txt`（プレーンテキスト）
- `.tsv` / `.json`（データ解析用）

---

## 関連プロジェクト

このCLIツールをベースに、話者分離・Webインターフェース・Docker対応などを追加したWebアプリ版を開発しました。

| リポジトリ | 概要 |
|---|---|
| whisper-transcription-cli（本リポジトリ） | CLIで動くシンプルな文字起こしツール |
| [whisper-transcription-app](https://github.com/Nishimura-Jin/whisper-transcription-app) | FastAPI + React + Dockerで作ったWebアプリ版 |