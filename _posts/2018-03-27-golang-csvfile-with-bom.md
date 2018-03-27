---
layout: post
title: 'GolangでBOM付きCSVファイルを読み込む'
tags:
 - go
 - golang
 - csv
 - unicode
---

先行事例・解決策は山ほどあるかと思いますが、はまったのでメモがてら解決方法を。BOM付きUTF16などでエンコードされたテキストを扱うに当たって、ライブラリなどが対応してくれているといいのですが、GolangのCSVReaderは対応していないようです。このため、次のような関数を準備してエンコーディングを変換してやります。

```go
func NewBomAwareCsvReader(r io.Reader) *csv.Reader {
  var (
    bomUtf8    = []byte{0xef, 0xbb, 0xbf}
    bomUtf16BE = []byte{0xfe, 0xff}
    bomUtf16LE = []byte{0xff, 0xfe}
  )
  br := bufio.NewReader(r)
  mark, err := br.Peek(3)
  if err != nil {
    panic(err)
  }

  // decoder
  d := unicode.UTF8.NewDecoder()

  if bytes.HasPrefix(mark, bomUtf8) {
    br.Discard(len(bomUtf8))
  } else if bytes.HasPrefix(mark, bomUtf16BE) {
    d = unicode.UTF16(unicode.BigEndian, 
      unicode.UseBOM).NewDecoder()
  } else if bytes.HasPrefix(mark, bomUtf16LE) {
    d = unicode.UTF16(unicode.LittleEndian,
     unicode.UseBOM).NewDecoder()
  }

  return csv.NewReader(transform.NewReader(br, d))
}
```

あとはファイルを読み込めば変換できます。ひとまず必要が無かったので対応しませんでしたが、[utf32パッケージ](https://godoc.org/golang.org/x/text/encoding/unicode/utf32)を使って同様処理を追加してやればUTF32も対応できます。
