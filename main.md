# 型推論器と現実

型の恩恵は欲しい、  
でも型は書きたくないって……おかしいですか？

人は型を書くために生きるのにあらず…

### TAPLにのってる実装

数学的なアプローチ

- Constraint Typingをやって制約を集めてくる
  ![成約型付け](./const_typing.jpg)
- HindleyとMilnerのUnificationアルゴリズムで制約を解く

### 実際にやってみよう

![成約型付け](./pic1.png)
![成約型付け](./pic2.png)
![成約型付け](./pic3.png)
![成約型付け](./pic4.png)
![成約型付け](./pic5.png)

### 現実は厳しい

- このアルゴリズムめっちゃ遅い  
  - 現実のプログラムじゃ下から上に遡りたくなんてない
  - いちいち成約を集めて保持しておくなんて面倒ですよね？
- `X = Y`がわかった時に代入しちゃえばいいじゃん  
  - `min-caml`の実装とか
  - これ多相性ないけど残った型変数を多相型にしちゃえばいい

### 実装

`min-caml`でやっていることの焼きまし簡略版だよ〜

``` ocaml
type t = Int | Arrow of t * t | Var of t option ref;;

let gentyp () = Var (ref None);;

let rec unify a b = match a, b with
  | Int, Int -> ()
  | Arrow (s, t), Arrow (u, v) -> begin unify s u; unify t v end
  | Var ({contents = None} as r), _ | _, Var ({contents = None} as r) -> begin
      r := Some b
    end
  | Var ({contents = Some c}), _ -> unify c b
  | _, Var ({contents = Some c}) -> unify a c
  | _ -> failwith "unify"

(*
    utop[16]> let t = gentyp ();;
    val t : t = Var {contents = None}
    utop[17]> unify t (Arrow (gentyp (), Int));;
    - : unit = ()
    utop[18]> t;;
    - : t = Var {contents = Some (Arrow (Var {contents = None}, Int))}
    utop[19]> unify t (Arrow (Int, Int));;
    - : unit = ()
    utop[20]> t;;
    - : t = Var {contents = Some (Arrow (Var {contents = Some Int}, Int))}
*)
```

### min-camlの実装を読みましょう

- [type.ml](https://github.com/esumii/min-caml/blob/master/type.ml)
- [typing.ml](https://github.com/esumii/min-caml/blob/master/typing.ml)

### さて現実

- 現実のプログラミング言語にはもっと豊かな型が必要
  - レコード、タプルとか
  - オブジェクト
  - 第一級モジュール
  - GADT
  - `-rectypes`のrecursive types (!)


``` ocaml
[~/] ocaml -rectypes
        OCaml version 4.02.3
# let rec f x = f f;;`

```


### 現実の実装

- [types.ml](http://github.com/ocaml/ocaml/blob/typing/types.mli)
- `wc -l *.ml *.mli`の合計が31208行、えぐい

### OCamlの実装を読むよ

- let式の型付けをまずは読む  
  - [typecore.ml](http://github.com/ocaml/ocaml/blob/typing/typecore.ml)
  - `begin_def`とかいう副作用を起こす不穏な関数  
    `level`とか不思議なものが出てきてるぞ？

### Level-basedな型推論アルゴリズム
  - Remmyさんが考えてOlegさんがドキュメント化している  
  [How OCaml type checker works --  
   or what polymorphism and garbage collection have in common](http://okmij.org/ftp/ML/generalization.html)  
    型変数のGeneralizationはコストがかかるからこれを高速化する  
  




