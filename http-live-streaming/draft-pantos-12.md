# HTTP Live Streaming
## draft-pantos-http-live-streaming-12

### 概要

この仕様書では、終端のないマルチメディアストリームを転送するためのプロトコルについて述べる。このプロトコルは、転送するファイルの形式と、ストリームのサーバ（送信者）とクライアント（受信者）の振る舞いについて規定する。この仕様書では**バージョン６**のプロトコルについて解説する。


### このメモの状態

このインターネット草案はBCP78とBCP79の規定に完全に準拠して提出されている。この仕様書は更新されないかもしれないし、派生物が作られないかもしれないし、インターネット草案として以外公開されないかもしれない。

インターネット草案は Internet Engineering Task Force (IETF) の作業文書である。他の団体もインターネット草案として作業文書を配布するかもしれない。最新のインターネット草案は http://datatracker.ietf.org/drafts/current/ にある。

インターネット草案は最大で6ヶ月間有効な草案で、いついかなるときでも、更新や置き換え、他の文書により廃止することがある。また、参考資料としてインターネット草案を使用したり、「作業途中」としてではない引用をすることは不適切である。

このインターネット草案は、2014/4/17に無効となる。


### 著作権表示
Copyright (c) 2013 IETF Trust and the persons identified as the document authors.  All rights reserved.

This document is subject to BCP 78 and the IETF Trust's Legal Provisions Relating to IETF Documents (http://trustee.ietf.org/license-info) in effect on the date of publication of this document.  Please review these documents carefully, as they describe your rights and restrictions with respect to this document.

This Informational Internet Draft is submitted as an RFC Editor Contribution and/or non-IETF Document (not as a Contribution, IETF Contribution, nor IETF Document) in accordance with BCP 78 and BCP 79.


### 目次

1. 導入  
2. 概要  
3. プレイリストファイル  
  3.1. 導入  
  3.2. 属性リスト  
  3.3. 標準タグ  
    3.3.1. EXTM3U  
    3.3.2. EXTINF  
  3.4. 新規タグ  
    3.4.1. EXT-X-BYTERANGE  
    3.4.2. EXT-X-TARGETDURATION  
    3.4.3. EXT-X-MEDIA-SEQUENCE  
    3.4.4. EXT-X-KEY  
    3.4.5. EXT-X-PROGRAM-DATE-TIME  
    3.4.6. EXT-X-ALLOW-CACHE  
    3.4.7. EXT-X-PLAYLIST-TYPE  
    3.4.8. EXT-X-ENDLIST  
    3.4.9. EXT-X-MEDIA  
      3.4.9.1. Renditionグループ  
    3.4.10. EXT-X-STREAM-INF  
      3.4.10.1. 代替Renditions  
    3.4.11. EXT-X-DISCONTINUITY  
    3.4.12. EXT-X-DISCONTINUITY-SEQUENCE  
    3.4.13. EXT-X-I-FRAMES-ONLY  
    3.4.14. EXT-X-MAP  
    3.4.15. EXT-X-I-FRAME-STREAM-INF  
    3.4.16. EXT-X-START  
    3.4.17. EXT-X-VERSION  
4. メディアセグメント  
5. 鍵  
  5.1. 導入  
  5.2. AES128のIV  
6. クライアント/サーバのふるまい  
  6.1. 導入  
  6.2. サーバの動作  
    6.2.1. 導入  
    6.2.2. ライブ配信するプレイリスト  
    6.2.3. メディアセグメントの暗号化  
    6.2.4. 異なる複数ストリームの配信  
  6.3. クライアントの動作  
    6.3.1. 導入  
    6.3.2. プレイリストの読み込み  
    6.3.3. プレイリストの再生  
    6.3.4. プレイリストの再読み込み  
    6.3.5. 次に読み込むセグメントの決定  
    6.3.6. 暗号化されたメディアセグメントの復号  
7. プロトコルバージョンの互換性  
8. 例  
  8.1. 導入  
  8.2. 単純なプレイリスト  
  8.3. HTTPSを使ったライブ配信のプレイリスト  
  8.4. 暗号化されたメディアセグメントを含むプレイリスト  
  8.5. マスタープレイリスト  
  8.6. Iフレームを含むマスタープレイリスト  
  8.7. 代替音声を含むマスタープレイリスト  
  8.8. 代替音声を含むマスタープレイリスト  
9. 貢献者  
10. IANAについての考察  
11. セキュリティについての考察  
12. 参考  
  12.1. 引用規格  
  12.2. 参考規格  

## 1. 導入
この仕様書では、終端のないマルチメディアストリームを転送するためのプロトコルについて述べる。このプロトコルはメディアデータの暗号化をサポートし、クライアントが複数のエンコードの中から再生するエンコードを選択することを可能にする。メディアデータは作成された直後から転送されることが可能であり、そのためリアルタイムに近いタイミングで再生することが可能となる。データは一般的にHTTP [[RFC2616](http://tools.ietf.org/html/rfc2616)]で配信される。

HTTPのような関連した標準の参考文献は第11章に挙げる。

## 2. 概要
１つのマルチメディアpresentationはプレイリストファイルを指す１つのURIによって特定される。そのプレイリストはメディアのURIとメディアに対する情報を表すためのタグからなるリストである。このメディアのURIと関連するタグは、メディアセグメントの流れを特定する。

presentationを再生するために、クライアントは初めにプレイリストファイルを取得し、それからプレイリストに記載された各メディアセグメントを取得・再生する。クライアントはこの仕様書に記載されているように、セグメントの追加を検出するためにプレイリストファイルを再取得する。

この文書のキーワード "MUST"、"MUST NOT"、"REQUIRED"、"SHALL"、"SHALL NOT"、"SHOULD"、"SHOULD NOT"、"RECOMMENDED"、"MAY"、そして "OPTIONAL" は RFC 2119 [[RFC2119](http://tools.ietf.org/html/rfc2119)] で記述されたとおりに解釈されるべきである。

## 3. プレイリストファイル
### 3.1. 導入
プレイリストはM3Uプレイリスト[[M3U](http://tools.ietf.org/html/draft-pantos-http-live-streaming-12#ref-M3U)]を拡張したもので**なければならない**。この仕様書では追加のタグを定義することによって、M3Uファイルフォーマットを拡張する。

M3Uプレイリストは独立した複数の行からなるテキストファイルである。各行の終端はLFもしくはCRLFである。各行は、URIを表す行か、空行か、もしくは'#'で始まる。空行は無視される。空白はそれが明示的に指定されている要素以外で現れては**ならない**。
URI行に書かれたURIは、メディアセグメントかプレイリストファイル（3.4.10）を指す。各メディアセグメントはメディアURIとそれにかかるタグによって記述される。

すべてのURI行がメディアセグメントを指している場合、そのプレイリストをメディアプレイリストと呼ぶ。すべてのURI行がメディアプレイリストを指している場合、そのプレイリストをマスタープレイリストと呼ぶ。

'#'で始まる行はコメントかタグである。

タグは#EXTで始まる。それ以外の'#'で始まるすべての行はコメントであり、無視される**べきである**。

プレイリストにあるURIは、URI行であってもタグの一部分であっても、相対URI **かもしれない**。相対URIは、そのURIを記載したプレイリストファイルのURIに対して解決されなければ**ならない**。

メディアプレイリストのdurationは、プレイリスト中に含まれるメディアセグメントのdurationの合計である。

ファイル名が.m3u8で終わる、かつ/もしくは、HTTPのContent-Typeが"application/vnd.apple.mpegurl"であるプレイリストはUTF-8 [[RFC3629](http://tools.ietf.org/html/rfc3629)]でエンコードされている。ファイル名が.m3uで終わる、かつ/もしくは、HTTPのContent-Type [[RFC2616](http://tools.ietf.org/html/rfc2616)]が"audio/mpegurl"であるプレイリストはUS-ASCII[[US ASCII](http://tools.ietf.org/html/draft-pantos-http-live-streaming-12#ref-US_ASCII)]でエンコードされている。

プレイリストファイルは下記のいずれかを満たさなければ**ならない**。
* ファイル名が.m3u8で終わる、かつ/もしくは、(もしHTTPで転送される場合は) Content-Typeが"application/vnd.apple.mpegurl"である。
* ファイル名が.m3uで終わる、かつ/もしくは、Content-Typeが"audio/mpegurl"である（互換性のため）。


### 3.2. 属性リスト
いくつかの拡張M3Uタグは属性リスト形式の値を取る。属性リストとは、属性名と値のペアの空白を含まないカンマ区切りのリストである。

属性名と値のペアは下記の書式に則る。

*属性名*=*属性値*

*属性名*は[A..Z]と'-'からなる単純文字列である。

*属性値*は下記のいずれかである。

* 10進整数: 10進数の整数を表す、[0..9]からなる単純文字列。
* 16進整数：16進数の整数を表す、[0..9]と[A..F]からなる'0x'もしくは'0X'で始まる単純文字列。
* 10進浮動小数：10進数の浮動小数を表す、[0..9]と'.'からなる単純文字列。
* 引用文字列：二重引用符(")のペアに囲まれた、Uniform Type Identifier [[UTI](http://tools.ietf.org/html/draft-pantos-http-live-streaming-12#ref-UTI)]を含む文字列。使用できる文字や特殊記号のエスケープ方法は属性の定義によって指定されるが、引用文字列はラインフィード(0xA)とキャリッジリターン(0xD)と二重引用符(0x22)のいずれも含んでは**ならない**。これらの文字を使用しなければならない場合は、URIに対するパーセントエンコードのように、エスケープしなければならない。
* 列挙文字列：その属性に対して明示的に定義されている候補の中の単純文字列。列挙文字列は決して二重引用符(")や、カンマ(,)や、空白を含まない。
* 10進解像度："x"によって区切られる２つの10進整数。１つ目の整数は水平方向のピクセル数（幅）を表し、２つ目の整数は垂直方向のピクセル数（高さ）を表す。

ある*属性名*に対応する*属性値*の型は、その属性の定義によって指定される。

*属性名*は１つの属性リストの中に１回以上現れては**ならない**。

認識されない*属性名*をもつ属性名と値のペアは、クライアントによって無視されなければ**ならない**。

*属性名*は認識できるが*属性値*を認識できない列挙文字列型の属性をもつあらゆるタグは、クライアントによって無視されなければ**ならない**。


### 3.3. 標準タグ
#### 3.3.1. EXTM3U
拡張M3Uファイルは、一行目で従来のM3Uファイルと区別される。その一行目はEXTM3Uタグでなければ**ならない**。このタグはメディアプレイリストとマスタープレイリストのどちらの場合でも含まれなければ**ならない**。タグの書式は下記の通りである。

    #EXTM3U

#### 3.3.2. EXTINF
EXTINFタグは１つのメディアセグメントのdurationを表す。このタグは直後のメディアセグメントにのみ適用され、直後にはメディアセグメントのURIが続かなければ**ならない**。各メディアセグメントは直前にEXTINFタグがなければ**ならない**。タグの書式は下記の通りである。

    #EXTINF:<duration>,<title>

`<duration>`は、メディアセグメントのdurationを秒単位で表す、10進整数もしくは10進浮動小数である。`<duration>`が整数で表される場合は、実際のdurationにもっとも近い整数に丸められる**べきである**。プロトコルバージョンが3より小さい場合、`<duration>`は整数でなければ**ならない**。プロトコルバージョンが3以上の場合、`<duration>`は浮動小数である**べきである**。カンマに続く残りの部分`<title>`は、メディアセグメントのタイトルが可読形式で記載される。

### 3.4. 新規タグ
この仕様書では、以下の新規タグを定義する。
EXT-X-BYTERANGE, EXT-X-TARGETDURATION, EXT-X-MEDIA-SEQUENCE, EXT-X-KEY, EXT-X-PROGRAM-DATE-TIME, EXT-X-ALLOW-CACHE, EXT-X-PLAYLIST-TYPE, EXT-X-STREAM-INF, EXT-X-I-FRAME-STREAM-INF, EXT-X-I-FRAMES-ONLY, EXT-X-MEDIA, EXT-X-ENDLIST, EXT-X-DISCONTINUITY, EXT-X-DISCONTINUITY-SEQUENCE, EXT-X-START, EXT-X-VERSION

#### 3.4.1. EXT-X-BYTERANGE
EXT-X-BYTERANGEタグはメディアセグメントがメディアURIで指定されるリソースの一区間であることを表す。このタグはプレイリストにおける次のメディアURIにのみ適用される。タグの書式は下記の通りである。

    #EXT-X-BYTERANGE:<n>[@<o>]

ここで`<n>`は、区間の長さのバイト数を表す10進整数である。もし`<o>`が存在するとき、それは区間の開始点をリソースの先頭からのバイト数で表す10進整数である。もし`<o>`が存在しなければ、区間の開始点は１つ前のメディアセグメントの区間の次のバイトである。

もし`<o>`が存在しなければ、１つ前のメディアセグメントがプレイリスト中に存在しなければ**ならない**し、１つ前のメディアセグメントは同じメディアリソースの移築館でなければ**ならない**。

EXT-X-BYTERANGEタグの適用されないメディアURIは、リソース全体がメディアセグメントに相当する。

EXT-X-BYTERANGEタグはプロトコルバージョン４から導入された。このタグはマスタープレイリストに現れては**ならない**。

#### 3.4.2. EXT-X-TARGETDURATION
EXT-X-TARGETDURATIONタグはメディアセグメントの最大のdurationを表す。プレイリスト中に現れる各メディアセグメントのEXTINFのdurationは、もっとも近い整数に丸められているとき、このタグに書かれているduration以下の値でなければ**ならない**。このタグはメディアプレイリストに必ず一度だけ現れなければ**ならない**。このタグはプレイリストファイル全体に対して適用される。タグの書式は以下の通りである。

    #EXT-X-TARGETDURATION:<s>

ここで`<s>`は最大のdurationを秒単位で表す10進整数である。

EXT-X-TARGETDURATIONタグはマスタープレイリストに現れては**ならない**。

#### 3.4.3. EXT-X-MEDIA-SEQUENCE
プレイリスト中に現れる各メディアセグメントは、ユニークな整数のシーケンス番号を持っている。セグメントのシーケンス番号は、一行前に書かれたセグメントのシーケンス番号に１を加えたものに等しい。EXT-X-MEDIA-SEQUENCEタグはプレイリストの一番最初に現れるセグメントのシーケンス番号の数値を表す。タグの書式は下記の通りである。

    #EXT-X-MEDIA-SEQUENCE:<number>

ここで`<number>`は10進整数である。シーケンス番号は減少しては**ならない**。

メディアプレイリストは１つより多くのEXT-X-MEDIA-SEQUENCEタグを含んでは**ならない**。もしメディアプレイリストがEXT-X-MEDIA-SEQUENCEタグを含まない場合、最初のセグメントのシーケンス番号は0と見なされる**ことになる**。クライアントは異なるメディアプレイリストにおいて同一のシーケンス番号をもつメディアセグメントが同一の内容であると仮定しては**ならない**。

メディアURIはシーケンス番号を含む必要はない。

EXT-X-MEDIA-SEQUENCEタグの取り回しに関する情報は、6.2.1章と6.3.2章と6.3.5章を参照すること。

EXT-X-MEDIA-SEQUENCEタグはマスタープレイリストに現れては**ならない**。

#### 3.4.4. EXT-X-KEY
メディアセグメントは暗号化されている**かもしれない**。EXT-X-KEYタグはそれらの暗号化されたメディアセグメントの復号化方法を指定するためのタグである。このタグは、同じKEYFORMAT属性を持つ次のEXT-X-KEYタグまで（もしくはプレイリストの終端まで）の間の各メディアセグメントに適用される。異なるKEYFORMAT属性を持つ２個以上のEXT-X-KEYタグが同一のメディアセグメントに対して適用される**かもしれない**。その場合、それらのEXT-X-KEYタグは同一の鍵を表さなければ**ならない**。タグの書式は下記のとおりである。

    #EXT-X-KEY:<attribute-list>

`<attribute-list>`は下記の属性を取りうる。

METHOD

この属性に対する値は、暗号化方式を示す列挙文字列型である。この属性は**必須である**。

定義されている暗号化方式は、NONE, AES-128, SAMPLE-AESである。

暗号化方式NONEはメディアセグメントが暗号化されていないことを意味する。もし暗号化方式がNONEの場合、属性URI, IV, KEYFORMAT, KEYFORMATVERSIONSは現れては**ならない**。

暗号化方式AES-128はAdvanced Encryption Standard[[AES 128](http://tools.ietf.org/html/draft-pantos-http-live-streaming-12#ref-AES_128)]によってメディアセグメント全体を暗号化していることを意味する。このとき鍵長は128ビットでパディング方式はPKCS7[[RFC5652](http://tools.ietf.org/html/rfc5652)]である。もし暗号化方式がAES-128の場合、URI属性は必ず現れなければ**ならない**。IV属性は現れる**かもしれない**。IV属性に関する詳細は、5.2章を参考のこと。

暗号化方式SAMPLE-AESはメディアセグメントがAdvanced Encryption Standard[[AES 128](http://tools.ietf.org/html/draft-pantos-http-live-streaming-12#ref-AES_128)]を用いて暗号化された音楽か映像かその他のサンプルのエレメンタリストリームを含むことを意味する。エレメンタリストリームがどのように暗号化されているかはメディアのエンコード方式によって決まる。H.264 [[H 264](http://tools.ietf.org/html/draft-pantos-http-live-streaming-12#ref-H_264)]かAAC [[ISO 14496](http://tools.ietf.org/html/draft-pantos-http-live-streaming-12#ref-ISO_14496)]かAC-3 [[AC 3](http://tools.ietf.org/html/draft-pantos-http-live-streaming-12#ref-AC_3)]のエレメンタリストリームに対する暗号化方式は、[[SampleEnc](http://tools.ietf.org/html/draft-pantos-http-live-streaming-12#ref-SampleEnc)]に記載されている。IV属性は現れる**かもしれない**。IV属性に関する詳細は、5.2章を参照のこと。

クライアントは認識できないMETHOD属性を持つEXT-X-KEYタグのセグメントを復号しようと試みては**ならない**。

URI

この属性に対する値は鍵を得る方法を特定するURI[[RFC3986](http://tools.ietf.org/html/rfc3986)]を含む引用文字列である。この属性はMETHODがNONEでない場合、**必須である**。

IV

この属性に対する値は、鍵とともに使われる初期化ベクタを表す16進整数である。IV属性はプロトコルバージョン2から追加された。IV属性が使われるときについての詳細は、5.2章を参照のこと。
