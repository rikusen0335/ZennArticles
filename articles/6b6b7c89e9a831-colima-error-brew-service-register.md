---
title: "Homebrewでcolimaのスタートアップ起動しようとしたらエラーが出た時"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["colima", "homebrew"]
published: false
---

`brew services start colima`したら以下のエラーがでて、スタートアップ時に自動でcolimaをスタートできない人向け

```
Bootstrap failed: 5: Input/output error
Try re-running the command as root for richer errors.
Error: Failure while executing; `/bin/launchctl bootstrap gui/501 /Users/soshi/Library/LaunchAgents/homebrew.mxcl.colima.plist` exited with 5.
```

## TL;DR

`brew install docker`を実行する
これで治らなかったらブラウザバックか、下記を参照


## 詳細に

エラーに出ているplistのファイルを見る
`vim ~/Library/LaunchAgents/homebrew.mxcl.colima.plist`

そこにあるであろう`StandardErrorPath`を探して、そのログファイルを見る
今回は`/opt/homebrew/var/log/colima.log`だった

こういう感じのエラーがあればおk
```
time="2024-06-10T11:20:15+09:00" level=info msg="starting colima"
time="2024-06-10T11:20:15+09:00" level=info msg="runtime: docker"
time="2024-06-10T11:20:15+09:00" level=fatal msg="dependency check failed for docker: docker not found, run 'brew install docker' to install"
```

今回はdockerをいれていないことが原因だったので、`brew install docker`をしてから`brew services restart colima`したら上手くいった。
