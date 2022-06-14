{% raw %}
【第5回】 2022年6月9日


- [第4回の補足](day4.md#day4suppl)
- テンプレートにおけるプログラミング



#  テンプレートにおけるプログラミング

-[ テンプレートファイルがブラウザに表示される仕組み](https://otree.readthedocs.io/en/latest/templates.html#how-templates-work-an-example)



## 変数とフィルター

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/K9BagHTRH3Y?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>

- [詳しくはこちら](otree_ref/templatefile.md#変数の展開)



## `if` と `for`

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/6jRM-p5tJ2s?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>

- [詳しくはこちら](otree_ref/templatefile.md#テンプレートタグ)



## その他のテンプレートタグ

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/FnpInHMIe1M?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>

- [詳しくはこちら](otree_ref/templatefile.md#include)



## oTree 3 との違い

- oTree 3 はDjangoがベースであったため，Djangoの組み込みフィルターが使えたが， oTree 5 では，Djangoの組み込みフィルターが（ほとんど）使えなくなった．oTree 3 から oTree 5 へ移行するときは注意．多用されていたと思われるフィルターは以下:
    - `floatformat:0`: 整数への（銀行家の）丸め． oTree 5 では `to0` に変更されたが，切り下げとなることに注意．
    - `add`: テンプレートで変数の足し算ができた（たとえば `{{ x|add:"1" }}` とすれば xに1を足した数値が表示されていた）．
    - `date`: 変数が `datetime` オブジェクトであれば，日付が人間が読める書式に変換されて表示されていた．
- テンプレートタグのデリミタが `{% %}` から `{{ }}` に変更され，テンプレートタグと変数の記述方法が縮退してしまった．
    - oTree 5 でも互換性のために `{% %}` は使える．ただし，`{%` と `{{`， `%}` と `}}` をそれぞれ区別しないという実装のようなので，`"` と `'` の関係のようにネストさせて使えるというわけではない．


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/3eVb6rKUtPw?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>




{% endraw %}
