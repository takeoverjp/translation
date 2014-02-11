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

