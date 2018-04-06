##########################################
Memo on Block Chain Study Group meeting #2
##########################################

(ブロックチェーン勉強会２)

- [Thursday, May 26](https://www.facebook.com/events/607357016084911/)
- AltCoin といわれるものから、その合意プロトコル的なものについていくつか紹介
- TL;DR いっぱいありすぎてわかんねえ！


## ブロックチェーンとは

ある事実なり履歴を改竄できない形で複製して保持する技術の総称

- 高可用性、耐故障性
- （ものによっては）匿名性
- ？？？ 性能

過去の履歴もMerkle Treeでまとめることによってデータの改竄を検出しやすくしている


Block generation::

 H(k) = Hash(T(k-1), PublicKey(k)), 
 T(k) = (PublicKey(k), H(k), Sign(H(k), PrivateKey(k)))
 Block(n) = (T(k-i), ..., T(k), Hash(Block(n-1), Nonce)


Nonce::

 Hash(Block(n), Nonce(n)) ~ 0000001xxxxxx
                            ^^^^^^ first x bits of the hash


**電子署名をどんどん追加するタイプの追記型データ構造** があればブロックチェーンなのでは？？→昔からあった

電子署名の問題点は、署名そのものが正しいかどうかが自明でない::


 $ gpg --key-gen
 ...
 $ gpg -u kuenishi@gmail.com --clearsign hello.py
 $ gpg -d hello.py.asc
 print("Hello Python!")
 gpg: Signature made Sun May 22 11:53:10 2016 JST using RSA key ID 1EB41214
 gpg: Good signature from "Kota UENISHI (nope) <kuenishi@gmail.com>"
 $ echo $?
 0
 $ gpg -d hello.py.asc.cracked
 print("Hello Ruby!")
 gpg: Signature made Sun May 22 11:53:10 2016 JST using RSA key ID 1EB41214
 gpg: BAD signature from "Kota UENISHI (nope) <kuenishi@gmail.com>"
 $ echo $?
 1


改竄してから署名してしまえばよい::

 $ gpg -d hello.py.cracked.asc
 print("Hello Ruby!")
 gpg: Signature made Sun May 22 11:57:01 2016 JST using RSA key ID 1EB41214
 gpg: Good signature from "Kota UENISHI (nope) <kuenishi@gmail.com>"
 $ echo $?
 0


BitCoinにおける合意（といわれているやつ）
--------------------------------------------------

TL;DR 分散システムでいうところの合意ではない

``Block(n)`` が合意されるか

- Termination (Liveness) ... あとからいつでも覆される
- Validity ... 全てのプロセスがvを提案したら全員がvに決定する
- Integrity ... 全てのプロセスは１つだけブロックを選ぶ
- Agreement ... 全てのプロセスは同じvを選ぶ(????)

反例 ... あるチェーンAが伸びてから、あとからより長いBを渡されたケース::

 -o-o-o-o-o-o-o-o-A

           b
 -o-o-o-o-o+o-o-o-A
           |
           +o-o-o-o-o-o-o-o-o-o-o-o-o-o-B


TODO: O(N) じゃない、Nonceを求める計算量？

あとから長いBを生成するためのクラシックな理論でいうと、あるNonceを計算
する計算量を O(N) とすると、ブランチした点 b から A までの計算にかかっ
たコストはおよそ ``O(N * (A-b))`` で、 B の計算にかかるコストは
``O(N * (B-b))`` なので、履歴の改竄にかかるコストは、改竄対象のチェー
ンを作ったコストの ``O(N * (B-b)/(A-b))`` 倍程度になる。

その程度のコストを支払える計算機があれば合意（？）された値を変更できる。

AltCoins
--------

バリエーションは沢山ある、暗号化や署名のアルゴリズム、通貨としての利用
や交換形態、通貨じゃないやつ、Public or Private, 運営や開発の主体、分
岐したチェーンの破棄方法、実装やパラメータなど様々な分類軸がある

ここでは分岐したチェーンの破棄方法を主な観点にする::

 PoW => PoS => DPoS => QDPoS
         +=> Stellar


メジャーな攻撃方法

- 51% attack - 計算能力の51% をとって常にRewardを独占
- Sybil attack - 悪意のあるノードを大量に追加して善意のあるクライアントをネットワークからきりはなす
- Network partition attack

Proof of Work
-------------

BitCoinと同様、Nonceの計算を競う早い者勝ちのプロトコル。より早くNonce
を見つけるためには、より多くのコンピュータがあった方がよい。

- Pros: フェア
- Cons: エコじゃない, network attack で分岐する, 計算力の集中化が進んでいる, higher latency

Proof of Stake
--------------

投票によりより多くのコインを集めたBlockが正しいものになる。

- Pros: PoWよりはエコ, attacker は過半数に近いコインを集める必要がある, lower latency
- Cons: Richer get richer, "The Monopoly problem"

Monopoly => Double spending, denying service

- Nothing at Stake: 全てのForkに投票する（PoWと違って投票はチープだか
  ら）
- Stake Grinding: まずいくらかStakeを確保して、過去の歴史を遡って自分
  が過半数をとれるブロックを探す。見つかったら、そこから歴史を今まで改
  竄する。

Stake Grinding::

 "Both benevolent and malevolent monopoly are potentially profitable,
 so there are reasons to suspect that an entrepreneurial miner might
 attempt to become a monopolist at some point. Due to the Tragedy of
 the Commons effect, attempts at monopoly become increasingly likely
 over time."


現実的には需要と供給の関係によって、独占するには $20M必要と見積もられ
ている（PoWなら$10M)。独占すると通貨価値がなくなるので独占したがる奴は
いない、らしい etc, etc

バリエーションがいくつかある

- Randomized block selection (Nxt, BlackCoin)
- Coin age based selection (PPCoin)
- Velocity based selection (PoSV, Reddcoin)
- Voting based selection

他
-----

- [Proof of Activity Proposal](https://bitcointalk.org/index.php?topic=102355.0)
- [Proof of burn](https://en.bitcoin.it/wiki/Proof_of_burn) - Coinを "burn" する=利用不可能なWalletに送金するとProofになる
- [Proof of Stake Velocity](https://www.reddit.com/r/reddCoin/comments/249dnl/major_announcement_reddcoin_to_implement_new/), [paper](https://www.reddcoin.com/papers/PoSV.pdf)
- [Queued Delegated Proof-of-Stake](https://wiki.qointum.com/design/dev/#queued-delegated-proof-of-stake)
- [Tendermint - The completely decntralized consensus engine](https://news.ycombinator.com/item?id=9138923) - PoS

GHOST Protocol
--------------

[Accelerating Bitcoin’s Transaction Processing - Fast Money Grows on Trees, Not Chains](https://eprint.iacr.org/2013/881.pdf)
 
1. Bitcoinのやつだとブランチしやすい→アタックを受けた時などにスループットが頭打ちになる（らしい
1. Branch selection 戦略を変える
1. **"Greedy Heaviest-Observed Sub-Tree (GHOST)"** protocol を提案

::

   maintains the threshold of hash power needed to successfully pull off
   a 50% attack at 50%, even if the network suffers from extreme delays
   and the attacker does not.


**B の子ブロックのうち、サブツリーに一番沢山ブロックを持っている子ブロックを正当なツリーとして認める** 戦略

他にも、論文中では

* Bitcoin/GHOSTのスループットの理論値を計算 (Bitcoinでは <1.0tps だったのに特定条件下で200を超えるように
* レイテンシの理論値の評価 etc, etc, ...

ところが… => Casper

Major AltCoins
--------------

Ripple: proof of work, TRUST?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Heuristics, to assume some limitations on network latency
* The Ripple protocol can achieve consensus in the face of (n-1)/5 failures +several other desirable features
* Q. is this for a fixed node set? <= given there's "n"

* [Ripple Consensus Algorithm](https://ripple.com/files/ripple_consensus_whitepaper.pdf)

UNL (Unique Node List) を各クライアントでもって、それの 4/5 の合意がとれればOK（えー

### Etherium: PoW (rather a Smart Contracts platform)

Public な open ledger で、 Smart Contracts といわれる、チューリング完
全なプログラミング言語 (Solidity) とその処理系（EVM）がノード上に実装されている。

* "Uncle" 主流にならなかったブランチだけどRewardがもらえる→より多くの
  Minerが現れるのでブロックが進みやすい（レイテンシを短くできる）
* [More uncle statistics](https://blog.ethereum.org/2015/09/25/more-uncle-statistics/)
* [Heavily centralized](https://etherchain.org/statistics/miners)
* [Uncle Mining flaw](https://bitslog.wordpress.com/2016/04/28/uncle-mining-an-ethereum-consensus-protocol-flaw/):
  特定条件下では Uncle だけを掘った方が儲かるケースがあり、全員がそれ
  をやると収束しないという指摘

脆弱性もあり、Uncle Miningなどの設計が難しいので将来的に PoS に移行する => Casper

* [Introducing Casper “the Friendly Ghost”](https://blog.ethereum.org/2015/08/01/introducing-casper-friendly-ghost/): Security debositをベースにしたPoS

  * おかしなことしたvalidatorはそのDepositoを失う
  * *their authentication chain ends in the list of currently-bonded validators.*

* [ETHEREUM: A SECURE DECENTRALISED GENERALISED TRANSACTION LEDGER](http://gavwood.com/paper.pdf)
* [Ethereum入門](http://book.ethereum-jp.net/)

``Casper is a security-deposit based economic consensus protocol.``

* Security-deposit based security and authentication 'bonded validators'
* Gambling on Consensus: *'if on the other hand they do not quickly agree, they re-earn less of their deposit'*
* By-height Consensus: *'Casper forfeits their security deposit.'*
* Transaction Finality: ???
* Recovery from netsplits: ???


* [ETHEREUM: A SECURE DECENTRALISED GENERALISED TRANSACTION LEDGER](http://gavwood.com/paper.pdf)
* [Ethereum入門](http://book.ethereum-jp.net/)

Stellar: Stellar consensus protocol (TRUST)
--------------------------------------------

* Federated Byzantine Agreementというプロトコルを開発し、 PoW, PoS の問題を全て解決したと主張
* Byzantine General Problem を限定的な状況について効率的に解いたとのこと
* Quorum set とかいう不思議な概念で、このset毎に合意をとる（模様）→なぜか全体で上手くいく

- ステラコンセンサスプロトコル
  [EN](https://www.stellar.org/blog/stellar-consensus-protocol-proof-code/)
  [JA](http://stellar1.blog.fc2.com/blog-entry-48.html)

BitShares: DPOS (Delegated PoS)
----------------------------------

* [Delegated Proof-of-Stake](https://bitshares.org/technology/delegated-proof-of-stake-consensus/):

  * Block Production by Elected Witnesses, which produce blocks and get
  * paid; netsplitもするよ
  * Each account is allowed one vote per share per witness, a process
    known as approval voting. The top N witnesses by total approval
    are selected. The number (N) of witnesses is defined such that at
    least 50% of voting stakeholders believe there is sufficient
    decentralization.
  
// * [Announcing BitShares 2.0](https://bitshares.org/blog/2015/06/08/announcing-bitshares-2.0/)
// * [The History of Graphene](https://bitshares.org/blog/2015/06/15/the-history-of-graphene/)

詳しいものを発見できてない

Qointum; QDPoS
-----------------

* [Queued Delegated Proof-of-Stake](https://wiki.qointum.com/design/dev/#queued-delegated-proof-of-stake)

QueueにDelegate (これからサインするブロック候補）を積んで、それを
*blockchain entropy* を使ってランダムに振り分けていく→より広範囲に分散
できてスケールできる

Consensusは [Maximally Vetted Delegate Chain](https://wiki.qointum.com/design/dev/#maximally-vetted-delegate-chain) の長さで決まる

* Nxt: Proof of Stake, w/ leased forging
* NEM: Ripple Consensus
* [BlackCoin's Proof-of-Stake Protocol v2](http://blackcoin.co/blackcoin-pos-protocol-v2-whitepaper.pdf)

References
----------

- [Proof of Stake versus Proof of Work white paper](http://bitfury.com/content/5-white-papers-research/pos-vs-pow-1.0.2.pdf)
- [Infographic: The Blockchain Ecosystem in 2016](https://ibsintelligence.com/ibs-journal/ibs-news/infographic-the-blockchain-ecosystem-in-2016/)
- [Non-PoW consensus protocols (Ethereum, Bitshares, Stellar/Ripple, etc)](https://bitcointalk.org/index.php?topic=1032927.0)
- [JA translation of Ethereum White Paper](https://github.com/ethereum/wiki/wiki/%5BJapanese%5D-White-Paper)
- [Larry Ren, Proof of Stake Velocity: Building the Social Currency of the Digital Age](https://www.reddcoin.com/papers/PoSV.pdf)

- [Introducing Casper "the friendly GHOST"](https://blog.ethereum.org/2015/08/01/introducing-casper-friendly-ghost/)
- [Casper the Friendly Ghost](https://docs.google.com/presentation/d/1bV_vXJBko-DmhAgnOFYg8ZNbAvCZCZrlf0KBFPqwVIw/edit)

- [GnuPGのコマンド](http://www.nina.jp/server/windows/gpg/commands.html)
- [ビットコイン2.0](http://jpbitcoin.com/bitcoin2s)
- [電子署名の仕組み](http://esac.jipdec.or.jp/intro/shikumi.html)
- [電子署名の基礎知識](http://www.c-a-c.jp/about/knowledge.html)
- [OpenSSLコマンドによる公開鍵暗号、電子署名の方法](http://qiita.com/kunichiko/items/3c0b1a2915e9dacbd4c1)
- [電子署名のファイルフォーマット](http://qiita.com/kunichiko/items/2e0a2bd35c8e9492ceb5)
- [図説RSA署名の巻](http://blog.livedoor.jp/k_urushima/archives/979220.html)


# Omake

- [ビットコイン関連詐欺を４つに分類する](http://coinandpeace.hatenablog.com/entry/4_types_of__bitcoin_scam)



手作りの温かみのあるブロックチェーン (ブランチ選択なし）::

  $ gpg -u kuenishi@gmail.com --clearsign hello.py.2

  You need a passphrase to unlock the secret key for
  user: "Kota UENISHI (nope) <kuenishi@gmail.com>"
  4096-bit RSA key, ID 1EB41214, created 2016-05-22

  $ gpg -d hello.py.2.asc
  print("Hello Python, 2")
  print("MD5 (hello.py.asc) = a1b90918efe7ce659b919a76b29e4eca")
  gpg: Signature made Sun May 22 14:22:16 2016 JST using RSA key ID 1EB41214
  gpg: Good signature from "Kota UENISHI (nope) <kuenishi@gmail.com>"


``hello.py`` => ``hello.py.asc`` => ``hello.py.2`` => ``hello.py.2.asc`` => ... => ``hello.py.$n`` => ``hello.py.$n.asc`` => ...


Misc
----


- [Landscape](http://startupmanagement.org/wp-content/uploads/2015/12/Blockchain-in-Financial-Services-Landscape.jpg)
- [Distributed Ledger Technology: beyond block chain](https://www.gov.uk/government/uploads/system/uploads/attachment_data/file/492972/gs-16-1-distributed-ledger-technology.pdf)
- [Blockchain Ecosystem Database](https://www.blockstars.io/ecosystem/)
- [Blockchain Consensus Protocols](http://www.slideshare.net/lablogga/blockchain-consensus-protocols)

[BitShares / ビットシェアズ](http://jpbitcoin.com/bitcoin2/bitshares)

> Bitsharesの基軸通貨はBTSと呼ばれます。このBTSの発行上限は約37億BTSであり、スタート時にはそのうち約25億が配布されました。残りの約12億は、取引の承認者に支払われる報酬金などの用途のため、準備金プール(Reserve Pool)と呼ばれる、誰もコントロールできない資産としてブロックチェーン上に保管されています。Bitsharesにおける取引手数料の20%はこの準備金プールへと送られます。
Bitsharesにおける取引承認のシステムは、DPOS(Delegated Proof of Stake、代表者によるProof of Stake)と呼ばれています。DPOSにおける取引の承認者はwitness(立会人)と呼ばれています。witnessには誰でも立候補することができ、BTS保有者の投票により選ばれ、また、投票によりwitnessの座から下ろすことも可能です。この投票権は、全BTS保有者に保有量に応じて割り当てられており、他人に投票権を渡すこともできます。
選ばれたwitnessは、一定時間ごとにシャッフルされて承認の順番が割り当てられ、順に取引・ブロックを承認していきます。承認をしなかったwitnessは順番が飛ばされ次のwitnessが承認を行います。承認を行うごとに、witnessはBTS保有者の投票により定められた一定の報酬(現在は1ブロック1.5BTS)がもらえます。この報酬は先の準備金プールから支払われます。
かつては、101人がwitnessに選出されていましたが、さらに少ない人数でもセキュリティ的には十分であることや、多すぎるとwitnessに支払う総報酬が多くなりコストがかさむ問題や投票者がwitness一人一人を評価することが困難になる問題などがあるため、現在では20人前後となっています。

- [Graphene](https://bitshares.org/doxygen/index.html)

Others
------

- [20 MINS INTRO TO CONSENSUS](https://devcon.ethereum.org/slides/consensus_williams.pdf)
- A Prunable Blockchain Consensus Protocol Based on Non-Interactive
  Proofs of Past States Retrievability, Alexander Chepurnoy, Mario
  Larangeira, Alexander Ojiganov
  [Arxiv](http://arxiv.org/abs/1603.07926)
- SCP: A Computationally-Scalable Byzantine Consensus Protocol For
  Blockchains [PDF](http://eprint.iacr.org/2015/1168.pdf)
- The Stellar Consensus Protocol: A Federated Model for Internet-level
  Consensus, DAVID MAZIERES , Stellar Development Foundation
  [PDF](https://www.stellar.org/papers/stellar-consensus-protocol.pdf)
- Byzantine Fault Tolerance Algorithmes, Practical Byzantine Fault
  Tolerance, 1999 Castro, Liskov
- Random Oracles in Constantinople: Practical Asynchronous Byzantine
  Agreement using Cryptography, 2000 Cachin, Kursawe, Shoup
