# ビルドシステムについて

## 必須ソフト類

- Visual Studio 2022 Professional Version 17.14.15 (September 2025) またはそれ以降
- CMake 3.22.2
  - `cmake.exe` へのパスが通っていること
- Python 3.13.15 (x64)
  - `.py` の関連付けができていること
- 7-Zip 25.00 (x64)
  - `7z.exe` へのパスが通っていること

## ビルド手順

スタートから `Developer Command Prompt for VS 2022` を起動

実行:

```bat
prepareBuildFiles.py
```

出力:

```txt
Writing PG96_x86_v143/build.cmd
Writing PG96_x64_v143/build.cmd
Writing PG100_x86_v143/build.cmd
Writing PG100_x64_v143/build.cmd
Writing PG110_x64_v143/build.cmd
Writing PG120_x64_v143/build.cmd
Writing PG130_x64_v143/build.cmd
Writing PG140_x64_v143/build.cmd
Writing PG150_x64_v143/build.cmd
Writing PG160_x64_v143/build.cmd
Writing PG170_x64_v143/build.cmd
Writing buildAll.cmd
Writing packer.json
```

実行:

```bat
buildAll.cmd
```

出力は省略

実行:

```bat
packer.py
```

出力は省略

## ビルド成果物

`pack` 以下にビルド成果をデプロイします。

圧縮したものを `packs/20250925.7z` のようなファイル名で作成します。
