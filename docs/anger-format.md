# SDGs怒りダッシュボード

`index.html` をブラウザで開くと表示されます。同じフォルダに `anger.json` があれば読み込み（http(s)で配信時。`file://` では内蔵サンプルデータを表示）、なければ内蔵のサンプルデータを使います。
ローカル配信: `python3 -m http.server` → http://localhost:8000/

## anger.json の形式（各SDGsボットはこれを出力）
```json
{
  "news": {"title": "ニュースの見出し", "url": "https://..."},
  "goals": [
    {"goal": 8, "anger": 10, "comment": "ひと言の怒りコメント", "targets": ["8.7"]}
  ]
}
```
- `goal`: 1〜17の目標番号
- `anger`: 0〜10。0より大きいと赤い通知バッジ＋震えるアニメーション（値が大きいほど激しい）
- `comment`: ホバー/クリックで表示する怒りのひと言
- `targets`: 関連ターゲット番号の配列
- 載っていない目標は anger 0 扱い。最も怒りが大きい目標が「今日の怒り代表」
- `"sample": true` を付けると「サンプルデータ」表示が出ます
- 各ボットは `goals` の要素1つを出し、パイプライン側で配列に集約してください。

## ニュースの蓄積（history.json / merge.py）
- `history.json`: 過去の anger.json の配列（新しい順）。各要素は `news`, `goals`, `addedAt`（ISO時刻）。
- `anger.json` を更新したら `python3 merge.py` を実行すると、同じ見出しが未登録の場合だけ history.json の先頭に追記されます（何度実行しても重複しません）。
- `index.html` は最新ニュース（anger.json）をメイン表示し、下に history.json の全ニュースをアーカイブ一覧で表示。クリックするとその時の怒り状態を表示します。history.json がなければ anger.json のみ表示。
- 公開時のアップロード対象: `index.html`, `anger.json`, `history.json`

## 読み上げ（audio）
- anger.json のトップレベルと history.json の各要素に任意で `"audio": "audio/xxx.mp3"`（相対パス）を付けられます。
- ページの「▶ 読み上げ」ボタン：audio があればその音声を再生（読み上げ係の音声）、なければブラウザの音声合成（ja-JP）で見出し→怒り代表→コメントを読み上げます。ニュースを切り替えると停止。アーカイブで音声付きのカードには「音声」バッジ。
- merge.py は audio も引き継ぎ、既存の見出しに後から audio が付いた場合はその audio を更新します。

## 公開準備（publish_prep.sh / public/）
- `./publish_prep.sh` で merge.py を実行し、`public/` をデプロイ対象だけで作り直します：index.html, anger.json, history.json, audio/（参照されている mp3 のみ）。
- デプロイは `public/` の中身をアップロードしてください。
