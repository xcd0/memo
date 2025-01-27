# ソフトウェアへの組み込み言語

前提: C++へ組み込んで使用するインタープリタ言語
現状: luajit
ほしいもの: 資料が多い、処理系が小さい、高速で動作する、他言語に組み込みやすい、一般的な文法の、静的型付けな、インタープリタ言語。(ない)


## 概要

現状luajit一択と言わざるを得ない。
しかし、luaには問題が多い。割と致命的。 -> [luaの批評](#luaの批評)  
luajit以外の選択肢は採用が難しい。 -> [lua以外の選択肢](#lua以外の選択肢)  

luajitを代替する良い選択肢が欲しい。

## 方針

- 現状メジャーな言語をこの用途で使用できるようにする。
	- 静的型付けのインタープリタ言語でメジャーな言語はTypeScript一択と思われる。
- 処理系を小さくするために使用できる範囲を狭める。この用途では十分な場合が多い。



## ざっくり仕様書

ChatGPT製

> # 組み込み向けTypeScriptサブセットランタイム仕様書 - 初期実装版
> 
> ## 1. 目的
> TypeScriptのサブセットをサポートする小型で高速なランタイムを開発し、Windows環境のC++アプリケーションおよびRaspberry Pi Picoに組み込めるようにする。必要最低限のTypeScript文法でスクリプトを記述・実行できる環境を提供し、将来的な拡張性を考慮する。
> 
> ## 2. 初期実装の対象環境
> 1. Windows環境
>    - Visual Studio 2019
>    - C++アプリケーションへの組み込み
> 2. Raspberry Pi Pico
>    - ARM Cortex-M0+プロセッサ
>    - 制限されたメモリ環境（264KB RAM, 2MB フラッシュメモリ）
> 
> ## 3. 要件
> 
> ### 3.1 小型のランタイムサイズ
> - Windows環境: DLLサイズを1MB以内に抑える。
> - Raspberry Pi Pico: フラッシュメモリ使用量を500KB以内に抑える。
> - 不要な機能を除外し、コア機能に集中する。
> 
> ### 3.2 実行性能
> - Windows環境: インタプリタ方式を採用し、必要に応じて簡易的なJITコンパイルを検討。
> - Raspberry Pi Pico: 純粋なインタプリタ方式を採用し、メモリ使用量を最小限に抑える。
> 
> ### 3.3 TypeScriptのサブセット対応
> - 基本的なデータ型（number, string, boolean, array, object）
> - 関数定義と呼び出し
> - 制御構造（if, for, while, switch）
> - 簡易的なクラスとオブジェクト指向プログラミング
> - 基本的な型注釈
> - モジュールシステム（import/export）の簡略版
> 
> ### 3.4 C++への組み込み (Windows環境)
> - Visual Studio 2019でビルド可能なDLLとして提供。
> - C++ APIを提供し、ホストアプリケーションとの簡単な連携を可能にする。
> - スクリプトの実行、変数の受け渡し、関数呼び出しなどの基本的な操作をサポート。
> 
> ### 3.5 Raspberry Pi Pico対応
> - Pico SDKを使用したビルドをサポート。
> - 制限されたメモリ環境での動作を最適化。
> - Picoのハードウェア機能（GPIO、I2C、SPI等）へのアクセスを提供するAPIを実装。
> 
> ### 3.6 最小限の依存関係
> - Windows環境: 標準C++ライブラリ以外の依存を最小限に抑える。
> - Raspberry Pi Pico: Pico SDKのみを使用し、追加の外部ライブラリは使用しない。
> 
> ### 3.7 メモリ管理
> - Windows環境: 簡易的なガベージコレクションを実装。
> - Raspberry Pi Pico: 静的メモリ割り当てとメモリプールを使用し、動的メモリ確保を最小限に抑える。
> 
> ## 4. 実装手法
> 
> ### 4.1 言語と開発環境
> - Windows環境: 
>   - C++17を使用
>   - Visual Studio 2019でのビルドをサポート
> - Raspberry Pi Pico:
>   - C11を使用
>   - Pico SDKとCMakeを使用したビルド環境を構築
> 
> ### 4.2 TypeScriptサブセットのパーサー実装
> - 手書きの再帰下降パーサーを実装し、外部依存を避ける。
> - 字句解析器と構文解析器を最小限の機能で実装。
> 
> ### 4.3 インタプリタの実装
> - スタックベースの仮想マシンを設計・実装。
> - 命令セットを最小限に抑え、メモリ使用量を削減。
> 
> ### 4.4 プラットフォーム固有の最適化
> - Windows環境:
>   - マルチスレッドサポートを考慮した設計。
>   - Windowsのメモリ管理APIを利用した効率的なメモリ割り当て。
> - Raspberry Pi Pico:
>   - シングルスレッド動作を前提とした簡素な設計。
>   - ハードウェア固有の最適化（例：フラッシュメモリアクセスの最適化）。
> 
> ### 4.5 APIの設計と実装
> - 共通のコア機能を抽象化し、プラットフォーム固有の実装を分離。
> - Windows環境用のC++ APIとRaspberry Pi Pico用のC APIを提供。
> 
> ### 4.6 ビルドシステム
> - Windows環境: Visual Studio 2019のプロジェクト設定を提供。
> - Raspberry Pi Pico: CMakeを使用したクロスコンパイル環境を構築。
> 
> ### 4.7 テストと検証
> - 単体テストフレームワークを導入し、コア機能のテストを自動化。
> - Windows環境とRaspberry Pi Pico両方で動作確認できるサンプルアプリケーションを作成。
> 
> ## 5. 将来の拡張性考慮事項
> - モジュール構造を採用し、将来的な機能追加を容易にする。
> - プラットフォーム抽象化レイヤーを設計し、他のプラットフォームへの移植を考慮。
> - コンパイル時オプションにより、機能の有効/無効を切り替えられるようにする。
> 
> ## 6. ドキュメンテーション
> - APIリファレンス、開発者ガイド、サンプルコードを作成。
> - Windows環境とRaspberry Pi Pico向けのセットアップガイドを提供。




---

## 付録
### luaの批評
- 利点
	- 組み込みが簡単。
		- 他の言語は組み込みがかなり煩雑。
			- Luaとmruby以外は選択肢にならないレベル。その他のメジャーな言語はほぼ組み込みが大変。
			- mrubyの場合、ほぼ情報がないと思って間違いない。可能ではある。
				- mrubyはソフトウェアへの組み込みではなく組み込みソフトウェアを主目的としている。
				- mrubyは比較的新しいため資料が少ない。加えて、組み込みソフトウェアとソフトウェアへの組み込みで、同じ「組み込み」「embed」という単語を使うため検索が難しい。
				- https://qiita.com/drednote/items/5bbd13f150e37a771ac7
				- https://github.com/carsonmcdonald/mruby-c-example
				- https://github.com/kotaweav/tutorials/tree/master/mruby_cmake_integration
		- luaはlib/dll一つあれば組み込める。  
			- luaはソフトウェアへの組み込みを主目的としている。簡単に使用できるようになっている。
	- 小さい。
		- 多機能ではないので処理系が小さい。
		- ほかの言語の処理系は100MB以上必要。mrubyのみ比較対象になる。mrubyも数MB程度になるが、使用する標準ライブラリ毎に処理系をコンパイルする仕様の為使いにくい。
		- Luaは数MB程度。小さくすることもできる。
	- 速い。
		- JITコンパイラによって高速に動作する。
		- C/C++より速い場合もある。  
		https://scrapbox.io/eeeeeeeeeeeeeeee/%E4%B8%8B%E6%89%8B%E3%81%AAdll%E3%82%88%E3%82%8A%E3%82%82luajit%E3%81%AE%E6%96%B9%E3%81%8C%E9%80%9F%E3%81%84%E3%81%A8%E3%81%84%E3%81%86%E8%A9%B1  
- 欠点
	- 書式が独特。癖強い。
		- 配列のindexは **1** スタート。
			- これはかなり問題になる。
			- そもそも配列はない。全てテーブル(なんでも入るmap)。
				- ただし`nil`が入らない。
		- コメントの書式は`--`、`--[[`,`--]]`
	- 多機能ではない。
		- これはそれほど問題にならない。
	- luajitの開発が不透明。
		- この用途では実質lua = luajitになる。

### lua以外の選択肢

結論から言えば、現状ない。

luaはかなり使われてきた歴史があるため資料が多い。  
luaの不満も多く、同様の目的の新規言語はそこそこあるものの、使用者が少なく、資料が少ない。  
新規言語は採用が難しい。  
mrubyの方がまだ資料が多い。  

lua,mruby以外だと、2パターンのアプローチがある。
- 新規言語([AngelScript](https://angelcode.com/angelscript/sdk/docs/manual/index.html)(githubなし), [Squirrel](https://github.com/albertodemichelis/squirrel)(☆912))
	- 採用が難しいものしかない。
		- 資料が少ない。多少使われているものもある。
		- AngelScriptはgithubにない。そこそこ採用されているみたいだが個人開発？。いつまでメンテされるのか不明。
		- Squirrelは最新リリースが2022年。
- トランスコンパイルするもの([MoonScript](https://github.com/leafo/moonscript)(☆3.2k), [Terra](https://github.com/terralang/terra)(☆2.7k) )
	- ソフトウェアへの組み込みが主目的ではない。
		- 速度の文脈ではトランスコンパイルの時間を無視する場合が多い。
		- 要は用途が違う。
	- メジャーな言語を変換して実行するもの。
		- luaにコンパイルするもの
		- それ以外の言語にコンパイルするもの

https://www.reddit.com/r/cpp/comments/dcs52f/embeddable_languages_for_c/

