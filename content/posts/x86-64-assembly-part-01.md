+++
title       = 'x86-64 Assembly, Part 1: 逆アセンブルを読む'
date        = 2026-11-08T02:54:11+09:00
draft       = false
description = 'コンパイルした C 関数を objdump で逆アセンブルし、出力の各列が何を表しているのか、Intel 記法の命令をどう読むのかを確認します。'
categories  = ['assembly']
tags        = ['x86-64', 'assembly', 'objdump', 'gcc', 'c']
series      = ['x86-64-assembly']
numbered    = true
+++

コンパイルされたプログラムは機械語になりますが、`objdump` を使えば読める形に戻せます。

この記事では、小さな C言語 の関数をコンパイルして、その逆アセンブル出力を読めるようにします。出力の各列が何を表しているのか、Intel 記法の命令をどう読むのか、そして学習中はなぜ `-O0` でビルドするのかを扱います。

## 検証環境

出力はこの環境で取ったものです。アドレスや命令列は環境によって変わります。手元で実行して確かめてください。

| | |
|---|---|
| CPU | Intel Core i7-8565U（x86-64） |
| OS | Debian GNU/Linux 13 (trixie) / WSL2 |
| カーネル | 6.18.54.1-microsoft-standard-WSL2 |
| コンパイラ | gcc 14.2.0 |
| binutils | 2.44（`objdump`） |
| デバッガ | gdb 16.3 |
| libc | glibc 2.41 |

CPU の型番を書きましたが、この記事で効くのは **x86-64 であること**だけです。AMD の CPU でも同じ結果になります。

## 題材

引数に1を足して返すだけの関数です。

```c
int add_one(int x) {
    int y = x + 1;
    return y;
}
```

`main` を書いていないので、このままでは実行ファイルになりません。今回は実行しないので、`-c` を付けてオブジェクトファイルまでで止めます。リンクしないぶん、libc の中身が混ざらず見たい関数だけが出てきます。

```bash
gcc -O0 -c -o add1.o add1.c
```

## 逆アセンブルする

```console
$ objdump -d -M intel add1.o

add1.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <add_one>:
   0:	55                   	push   rbp
   1:	48 89 e5             	mov    rbp,rsp
   4:	89 7d ec             	mov    DWORD PTR [rbp-0x14],edi
   7:	8b 45 ec             	mov    eax,DWORD PTR [rbp-0x14]
   a:	83 c0 01             	add    eax,0x1
   d:	89 45 fc             	mov    DWORD PTR [rbp-0x4],eax
  10:	8b 45 fc             	mov    eax,DWORD PTR [rbp-0x4]
  13:	5d                   	pop    rbp
  14:	c3                   	ret
```

`-M intel` が Intel 記法を指定するオプションです。付けないと AT&T 記法になり、オペランドの順序が逆になります。

### 3つの列

1行は3つの部分でできています。

```text
   4:   89 7d ec             mov    DWORD PTR [rbp-0x14],edi
   |    |                    |
   |    |                    +-- ニーモニック（命令を人間向けに書いたもの）
   |    +-- 機械語のバイト列（この命令は3バイト）
   +-- 関数の先頭からのオフセット
```

（桁を揃えるため、実際の出力に含まれるタブを空白に置き換えています）

左端の数字は**関数の先頭からのオフセット**です。絶対アドレスではありません。まだリンクしていないので、このコードがメモリのどこに置かれるかが決まっていないためです。

2列目のバイト列が、CPU が実際に読む機械語です。`89 7d ec` の3バイトが `mov DWORD PTR [rbp-0x14],edi` という1つの命令にあたります。長さは命令によって違い、`push rbp` は `55` の1バイト、`ret` は `c3` の1バイトです。

バイト列が邪魔なときは消せます。

```bash
objdump -d -M intel --no-show-raw-insn add1.o
```

```console
0000000000000000 <add_one>:
   0:	push   rbp
   1:	mov    rbp,rsp
   4:	mov    DWORD PTR [rbp-0x14],edi
   7:	mov    eax,DWORD PTR [rbp-0x14]
   a:	add    eax,0x1
   d:	mov    DWORD PTR [rbp-0x4],eax
  10:	mov    eax,DWORD PTR [rbp-0x4]
  13:	pop    rbp
  14:	ret
```

## Intel 記法の読み方

オペランドが2つある命令は、**左が宛先、右が元**です。

```asm
mov rbp,rsp
```

これは `rsp` の値を `rbp` に入れる命令です。`mov A, B` を A に B を移すと英語の語順どおりに読めば間違えません。AT&T 記法では `mov %rsp,%rbp` と左右が逆になるので、資料を読むときは記法を確認する必要があります。

オペランドに出てくる書き方は4種類です。

| 書き方 | 意味 |
|---|---|
| `rbp`, `eax`, `edi` | レジスタ |
| `0x1` | 即値（命令に直接埋め込まれた定数） |
| `[rbp-0x14]` | `rbp` の値から `0x14` 引いた番地の**メモリ** |
| `DWORD PTR` | 直後のメモリを4バイトとして扱う指定 |

角かっこの有無が重要です。`rbp` はレジスタそのものですが、`[rbp]` は `rbp` に入っている値を番地として、そこにあるメモリを指します。

`DWORD PTR` が必要なのは、メモリ参照だけでは何バイト読み書きするのかが決まらないからです。幅の指定は4種類あります。

| 指定 | 幅 |
|---|---|
| `BYTE PTR` | 1バイト |
| `WORD PTR` | 2バイト |
| `DWORD PTR` | 4バイト |
| `QWORD PTR` | 8バイト |

`int` は4バイトなので `DWORD PTR` が出てきます。

これで `mov DWORD PTR [rbp-0x14],edi` が読めます。`edi` レジスタの値を、`rbp-0x14` 番地から4バイトのメモリに書き込む命令です。

## C言語とアセンブリのコードを並べて見る

`-g` を付けてコンパイルしておくと、`objdump -S` で元の C と並べて表示できます。

```bash
gcc -O0 -g -c -o add1g.o add1.c
objdump -d -S -M intel --no-show-raw-insn add1g.o
```

```asm
0000000000000000 <add_one>:
int add_one(int x) {
   0:	push   rbp
   1:	mov    rbp,rsp
   4:	mov    DWORD PTR [rbp-0x14],edi
    int y = x + 1;
   7:	mov    eax,DWORD PTR [rbp-0x14]
   a:	add    eax,0x1
   d:	mov    DWORD PTR [rbp-0x4],eax
    return y;
  10:	mov    eax,DWORD PTR [rbp-0x4]
}
  13:	pop    rbp
  14:	ret
```

C言語 の1行がどの命令になったのかが見えます。`int y = x + 1;` は3命令です。`rbp-0x14` から読んで、1を足して、`rbp-0x4` に書く。読んだ場所と書いた場所が違うので、`rbp-0x14` が引数 `x`、`rbp-0x4` が変数 `y` だと分かります。

`-S` は読み方を覚える段階では強力ですが、現実に解析するバイナリにデバッグ情報が付いていることはまずありません。あくまで学習用の補助輪です。

## なぜ `-O0` なのか

ここまで `-O0` を付けてきました。最適化を有効にすると結果が変わります。

```bash
gcc -O2 -c -o add1_O2.o add1.c
```

```asm
0000000000000000 <add_one>:
   0:	lea    eax,[rdi+0x1]
   3:	ret
```

9命令が2命令になりました。`rbp` を触る処理が消え、メモリへの読み書きも消え、変数 `y` に至っては存在しません。

`lea` は本来アドレス計算の命令ですが、ここでは `rdi+1` を計算して `eax` に入れる**ただの足し算**として使われています。`lea` が算術に流用されるのは x86-64 では日常的で、最適化されたコードを読むときによく出てきます。

C の行と命令が1対1で対応しなくなるので、読み方を覚えている間は `-O0` が向いています。

## 参考文献

- [`objdump(1)`](https://man7.org/linux/man-pages/man1/objdump.1.html), Linux manual page
- [Intel 64 and IA-32 Architectures Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdk-manuals.html), Intel
- [System V ABI, AMD64 Architecture Processor Supplement](https://gitlab.com/x86-psABIs/x86-64-ABI)
- [GCC: Options That Control Optimization](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)
