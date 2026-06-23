# yad_reservation_count_crawling

`yad_reservation_count_crawling.py` を PyInstaller で exe 化するためのメモです。

## 前提

- カレントディレクトリを本フォルダに移動して実行すること
  - `yad_crawling/yad_reservation_counts_crawling`
- 実行時に `config.yaml` は exe と同じディレクトリに配置すること

## Bash コマンド（onedir 推奨）

```bash
pyinstaller \
  --clean \
  --noconfirm \
  --onedir \
  --console \
  --name yad_reservation_count_crawling \
  --collect-all bs4 \
  --collect-all soupsieve \
  --collect-all charset_normalizer \
  --collect-all certifi \
  yad_reservation_count_crawling.py
```

生成物:

- `dist/yad_reservation_count_crawling/`

## Bash コマンド（onefile）

```bash
pyinstaller \
  --clean \
  --noconfirm \
  --onefile \
  --console \
  --name yad_reservation_count_crawling \
  --collect-all bs4 \
  --collect-all soupsieve \
  --collect-all charset_normalizer \
  --collect-all certifi \
  yad_reservation_count_crawling.py
```

生成物:

- `dist/yad_reservation_count_crawling`

## 実行例

onedir の場合:

```bash
cd dist/yad_reservation_count_crawling
./yad_reservation_count_crawling
```

onefile の場合:

```bash
cd dist
./yad_reservation_count_crawling
```


## アクセス間隔とブラウザ起動

検索結果ページと予約件数ページは、短時間の連続アクセスを避けるため、各 URL 取得前に `settings.access_interval_seconds` 秒だけ待機します。
また、取得ごとに Selenium の Chrome ウインドウを新規起動し、ページ取得後に終了します。

```yaml
settings:
  access_interval_seconds: 5
  page_load_wait_seconds: 3
```

画面を表示して挙動を確認したい場合は、`settings.headless` を `false` にしてください。

## 診断ログの取得

EC2 やタスクスケジューラー実行時に取得件数が不安定な場合は、`config.yaml` の `settings.diagnostics.enabled` を `true` にすると、exe と同じディレクトリ配下（既定は `diagnostics/`）に以下を保存できます。

- `crawler.log`: 標準出力/標準エラーに出た実行ログ
- `*.html`: 宿件数が取得できないページ、アクセス制限が疑われるページ、予約数が取得できない遷移先ページ
- `*.png`: Selenium で予約数が取得できない遷移先画面のスクリーンショット

設定例:

```yaml
settings:
  diagnostics:
    enabled: true
    output_dir: diagnostics
    save_html: true
    save_screenshot: true
    save_on_success: false
```

`save_on_success: true` にすると成功時の HTML/スクリーンショットも保存しますが、対象宿数に応じてファイル数が多くなります。通常は `false` のまま、失敗時のみ保存する運用を推奨します。
