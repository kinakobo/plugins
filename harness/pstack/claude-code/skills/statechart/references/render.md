# チャートを描画する（Render a chart）

見せる `.mmd` ファイルは、それぞれ npm パッケージ `pele@1.9.0` で描画し、ユーザーには SVG を渡す。Pele は、リポジトリの外に一度だけインストールする。各 SVG は、対応する `.mmd` ファイルの隣に同じ名前で書く。API は https://pele.run/api にある。`render` のオプションとして `{ limit: 1_000_000, outputLimit: 8_000_000, maxEdges: 10_000 }` を渡す。Pele の既定値は、500 状態のチャートがなりうる大きさより小さい。Pele が `PeleError` を投げたら、ユーザーに Mermaid のテキストとエラーを見せる。

## 色（Colors）

Pele は、Mermaid の `theme` と `themeVariables` を無視する。Pele の `--pele-*` の CSS 変数を `.pele` に設定する。変数の一覧は https://pele.run/theming にある。コマンドは、`class sN teal` のように、各状態にそのグループの色のクラスを付ける。また、問題のある各状態に `class sN breaks` か `class sN mayBreak` を、バグへの最短経路上の各状態に `class sN bugPath` を、PROPERTY の反例が永遠に繰り返す各状態に `class sN repeats` を付けて印にする。Pele は、これらのクラスを状態のノードに付ける。クラスの色は CSS で付ける。

どの Mermaid のビューアーでもバグの印が見えるように、`.mmd` ファイルには `breaks` と `mayBreak` の `classDef` の行がある。グループの色、`bugPath`、`repeats` の `classDef` はない。Pele は各 `classDef` の色をノードの `style` 属性として書き、`style` 属性は通常の CSS のルールより優先される。そのため、下の印のルールは `!important` を使う。これがないと、ダークテーマでも印がライトテーマの色のままになる。バグの印は塗りと文字の色を設定し、`bugPath` と `repeats` は枠線を設定する。印がグループの色の上に表示されるよう、印のルールはグループの色のあとに置く。

これらの色は提案である。以前のスキルで使っていた色である。各 SVG の先頭にある `<style>` 要素にこれらを入れる。そうすると、各 SVG は閲覧者のライト・ダークの設定に従う。

```css
.pele {
  --pele-bg: #ffffff;
  --pele-fg: #1c1c1c;
  --pele-muted: #656565;
  --pele-line: #808080;
  --pele-surface: #fcfcfc;
  --pele-surface-alt: #f1f1f1;
  --pele-border: #c9c9c9;
}
.pele .blue rect { fill: #c3e0ff; }
.pele .orange rect { fill: #f0d3bf; }
.pele .purple rect { fill: #e3d5ff; }
.pele .yellow rect { fill: #f0dcbf; }
.pele .teal rect { fill: #bfe3df; }
.pele .tan rect { fill: #e5d9ce; }
.pele .pink rect { fill: #ffcbe6; }
.pele .breaks rect {
  fill: #ffecee !important;
  stroke: #e72e3b !important;
  stroke-width: 2.5px !important;
}
.pele .breaks text {
  fill: #cd2834 !important;
}
.pele .mayBreak rect {
  fill: #fff3e6 !important;
  stroke: #e8730c !important;
  stroke-width: 2.5px !important;
}
.pele .mayBreak text {
  fill: #b85a06 !important;
}
.pele .bugPath rect {
  stroke: #1680f3 !important;
  stroke-width: 2.5px !important;
}
.pele .repeats rect {
  stroke: #9b35d6 !important;
  stroke-width: 2.5px !important;
  stroke-dasharray: 6 3;
}
@media (prefers-color-scheme: dark) {
  .pele {
    --pele-bg: #1f1f1f;
    --pele-fg: #f5f5f5;
    --pele-muted: #acacac;
    --pele-line: #939393;
    --pele-surface: #121212;
    --pele-surface-alt: #2c2c2c;
    --pele-border: #4e4e4e;
  }
  .pele .blue rect { fill: #1b3857; }
  .pele .orange rect { fill: #573117; }
  .pele .purple rect { fill: #3c2e57; }
  .pele .yellow rect { fill: #573d17; }
  .pele .teal rect { fill: #174641; }
  .pele .tan rect { fill: #3d3127; }
  .pele .pink rect { fill: #57233e; }
  .pele .breaks rect {
    fill: #2e2021 !important;
    stroke: #cf2b35 !important;
  }
  .pele .breaks text {
    fill: #ff6463 !important;
  }
  .pele .mayBreak rect {
    fill: #2e2619 !important;
    stroke: #d0700f !important;
  }
  .pele .mayBreak text {
    fill: #ffa257 !important;
  }
  .pele .bugPath rect {
    stroke: #116dd2 !important;
  }
  .pele .repeats rect {
    stroke: #9a3fcf !important;
  }
}
```
