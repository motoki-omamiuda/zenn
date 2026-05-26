---
title: "Juliaでプロジェクトを作る方法"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [julia]
published: false
---
## Juliaの基礎知識
### Pkgモード
Juliaではライブラリ等の情報を管理しやすくするためPkgモードという環境が用意されています。  
Juliaを起動して `]` キーを押すことでこのモードに入ることができます。
```shell
$ julia
julia> ]
pkg>
```

また、Pkgモード内で `add` コマンドを使用することで、任意のパッケージをインストールすることができます。
```shell
pkg> add Plots  # 例として Plots をインストール
```

[参考資料：Julia docs (Pkg mode)](https://docs.julialang.org/en/v1/stdlib/Pkg/)

### Shellモード
Juliaでは、内部からOSのシェルを使う機能が提供されており、これをShellモードと言います。  
Juliaを起動して `;` キーを押すことでモードに入ることができます。
```shell
$ julia
julia> ;
shell>
```

[参考資料：Julia docs (Shell mode)](https://docs.julialang.org/en/v1/stdlib/REPL/#man-shell-mode)

# Juliaでプロジェクトを作る方法
## プロジェクトの作成
Pkgモードに入って `generate` コマンドを使用することで、任意の名前が付いたプロジェクトを作成することができます。
```shell
pkg> generate MyProject
```
注意：Juliaのプロジェクト名にはケバブケースを使用することができません。

```shell
MyProject/
├── Project.toml
└── src
    └── MyProject.jl
```

## プロジェクト環境を起動
作成したプロジェクトのディレクトリに移動し `activate` コマンドを使用することで環境を起動することができます。  
環境を利用することで、各プロジェクトごとに独立したパッケージを管理することができます。
```shell
$ cd MyProject
$ julia
julia> ]
pkg> activate .
```

## コードの実行
シェルモードに移動してコードを実行すれば、各環境を使ってコードを実行することができます。
```shell
shell> julia src/MyProject.jl
```
