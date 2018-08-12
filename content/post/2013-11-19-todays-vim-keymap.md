---
date: "2013-11-19T00:00:00Z"
title: 今日のVimキーマップ
aliases:
  - /2013/11/19/todays-vim-keymap.html
---

MacVimで長い文章を書く際に劇的に役立つキーマップ。これを入れておく事で1行なのにGUI上改行されている改行も、通常のj,kで移動できる！
@seizans の.vimrcから拝借しました。

{{< highlight vim >}}
nnoremap j gj
nnoremap k gk 
{{< / highlight >}}

