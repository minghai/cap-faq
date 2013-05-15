# The CAP FAQ

    Version 1.0, May 9th 2013
    By: Henry Robinson / henry.robinson@gmail.com / @henryr
<pre><a href="http:://the-paper-trail.org/">http://the-paper-trail.org/</a></pre>

## 0. What is this document?
## 0. このドキュメントは何か？

No subject appears to be more controversial to distributed systems
engineers than the oft-quoted, oft-misunderstood CAP theorem. The
purpose of this FAQ is to explain what is known about CAP, so as to
help those new to the theorem get up to speed quickly, and to settle
some common misconceptions or points of disagreement.

分散システムエンジニアにとってCAP定理ほどよく引き合いに出され、良く間違って理解されているものはない。このFAQの目的はCAPについて知られていることについて説明し、この定理をまだ知らない者の理解を早め、いくつかの共通な誤解や議論の不一致を解決することだ。

Of course, there's every possibility I've made superficial or
completely thorough mistakes here. Corrections and comments are
welcome: <a href="mailto:henry.robinson+cap@gmail.com">let me have
them</a>.

もちろん私が表面上、または完全な間違いをここで犯している可能性は常にある。訂正やコメントを歓迎するので<a href="mailto:henry.robinson+cap@gmail.com">聞かせて欲しい</a>。

There are some questions I still intend to answer. For example

まだ答えていない質問がいくつか存在する。例えば

* *What's the relationship between CAP and performance?*
* *What does CAP mean to me as an engineer?*
* *What's the relationship between CAP and ACID?*

* *CAPとパフォーマンスの間の関連は何か？*
* *CAPはエンジニアである私に何を意味するのか？*
* *CAPとACIDとの関連は何か？*

Please suggest more.

他にあれば提案して欲しい。

## 1. Where did the CAP Theorem come from?

## 1. CAP定理はどこから来たのか？

Dr. Eric Brewer gave a keynote speech at the Principles of Distributed
Computing conference in 2000 called 'Towards Robust Distributed
Systems' [1]. In it he posed his 'CAP Theorem' - at the time unproven - which
illustrated the tensions between being correct and being always
available in distributed systems.

Eric Brewer博士は2000年に分散コンピューティング原理の会議にて'堅牢な分散システム'と題するキーノートスピーチを行った[1]。その中で彼は彼自身の'CAP定理' - その当時には未証明 - を提示した。それは分散システムにおいて正しい状態と常に利用可能な状態の間の緊張関係を描いた。

Two years later, Seth Gilbert and Professor Nancy Lynch - researchers
in distributed systems at MIT - formalised and proved the conjecture
in their paper “Brewer's conjecture and the feasibility of consistent,
available, partition-tolerant web services” [2].

二年後にSeth GilbertとNancy Lynch教授 - MITの分散システム研究者 - は彼等の論文"Brewerの予想と安定した、利用可能で、ネットワーク分割耐性の有るWebサービスの実現可能性"においてその予想を形式化し、証明した。

[1] http://www.cs.berkeley.edu/~brewer/cs262b-2004/PODC-keynote.pdf
[2] http://lpd.epfl.ch/sgilbert/pubs/BrewersConjecture-SigAct.pdf

## 2. What does the CAP Theorem actually say?
## 2. CAP定理は実際に何を定めているのか？

The CAP Theorem (henceforth 'CAP') says that it is impossible to build
an implementation of read-write storage in an asynchronous network that
satisfies all of the following three properties:

CAP定理（以下'CAP'）は非同期ネットワークにおけるread-writeストレージ実装において以下の3つの条件を全て満たす実装を構築することは不可能であると宣言している。

* Availability - will a request made to the data store always eventually complete?
* Consistency - will all executions of reads and writes seen by all nodes be _atomic_ or _linearizably_ consistent?
* Partition tolerance - the network is allowed to drop any messages.

* Availability(可用性) - データストアに対し作成されたリクエストは常にいつかは完了する
* Consistency(一貫性) - 全てのreadとwriteの実行は全てのノードからアトミックに、または線形に一貫性を持つ
* Partition tolerance(ネットワーク分割耐性) - ネットワークは任意のメッセージを無くしても良い

The next few items define any unfamiliar terms.

次のいくつかの項目では耳慣れない用語を定義する。

More informally, the CAP theorem tells us that we can't build a
database that both responds to every request and returns the results
that you would expect every time. It's an _impossibility_ result - it
tells us that something we might want to do is actually provably out
of reach. It's important now because it is directly applicable to the
many, many distributed systems which have been and are being built in
the last few years, but it is not a death knell: it does _not_ mean
that we cannot build useful systems while working within these
constraints.

正確さを犠牲にすれば、CAP定理は全てのリクエストに応答し、かつ期待する結果を毎回返すデータベースを構築することはできないと言っている。それは不確実性の結論であり、我々が行いたいだろう何かは実際には手が届かないことが証明可能だ。それが今重要なのはそのことがここ数年で構築された、そして今も構築されている多くの、多くの分散システムにも直接適用できるからだ。しかしそれは全ての終焉を意味しない。そのことは我々がこれらの制約の範疇で働く中で実用的なシステムを構築することができないことを意味**しない**。

The devil is in the details however. Before you start crying 'yes, but
what about...', make sure you understand the following about exactly
what the CAP theorem does and does not allow.

しかし神は細部に宿る。あなたが”はい、でももし”と叫ぶ前に、以下のCAP定理が何を許し、何を許さないかを確実に理解したかを確認して欲しい。

## 3. What is 'read-write storage'?
## 3. 'read-writeストレージ'とは何か？

CAP specifically concerns itself with a theoretical
construct called a _register_. A register is a data structure with two
operations:

CAPは具体的に理論的な構成概念であるレジスタについて考える。レジスタとは2つの命令を持つデータ構造である。

* set(X) sets the value of the register to X
* get() returns the last value set in the register

* set(X) はレジスタにXという値を設定する
* get() はレジスタに設定された最新の値を返す

A key-value store can be modelled as a collection of registers. Even
though registers appear very simple, they capture the essence of what
many distributed systems want to do - write data and read it back.

キーバリューストアはレジスタの集合であるとモデル化できる。レジスタはとてもシンプルに見えるが多くの分散システムが行いたい事 - データを記録し、それを読み返す- のエッセンスを含んでいる。

## 4. What does _atomic_ (or _linearizable_) mean?
## 4. _atomic_（または_linearizable_)とは何を意味するのか？

Atomic, or linearizable, consistency is a guarantee about what values
it's ok to return when a client performs get() operations. The idea is
that the register appears to all clients as though it ran on just one
machine, and responded to operations in the order they arrive.

Atomic、またはlinearizable consistencyとはクライアントがget()命令を実行した時にどの値が返還可能であるかの保証である。この考えはレジスタが全てのクライアントに対しそれがたった1つの機械上にて動作しており、命令が現われた順に返信するかのように見えるということである。

Consider an execution consisting the total set of operations performed
by all clients, potentially concurrently. The results of those
operations must, under atomic consistency, be equivalent to a single
serial (in order, one after the other) execution of all operations.

命令の集合から成る実行が全てのクライアントにより恐らく並列に実行されたと想定する。それらの命令の結果は、atomic consistencyの下において、全ての命令が単一の連続した（順において、次々に）実行である場合と等価でなければならない。

This guarantee is very strong. It rules out, amongst other guarantees,
_eventual consistency_, which allows a delay before a write becomes
visible. So under EC, you might have:

この保証はとても強い。これは、他の保証の中でも、writeがvisibleになる前に遅延を認める_eventual consistency_を排他する。ECの下では以下が起こり得る

    set(10), set(5), get() = 10

But this execution is invalid under atomic consistency.

しかしこの実行はatomic consistencyの下では無効である。

Atomic consistency also ensures that external communication about the
value of a register is respected. That is, if I read X and tell you
about it, you can go to the register and read X for yourself. It's
possible under slightly weaker guarantees (_serializability_ for
example) for that not to be true. In the following we write A:<set or
get> to mean that client A executes the following operation.

atomic consistencyはまたレジスタの値に関する外部コミュニケーションが順守されることを保証する。それは、もし私がXをreadし、あなたにそれを伝えた時、あなたはレジスタに向かいあなた自身のためにXをreadしても良いことを意味する。わずかに弱い保証の下では（例えば_serializability_）それが真にならないことが起こり得る。以下ではA:<set または get>はクライアントAが続く命令を実行することを意味する。

    B:set(5), A:set(10), A:get() = 10, B:get() = 10

This is an atomic history. But the following is not:

これはatomicなヒストリである。しかし次はそうではない。

    B:set(5), A:set(10), A:get() = 10, B:get() = 5

even though it is equivalent to the following serial history:

例え次の順歴に等価でもそうではない。

    B:set(5), B:get() = 5, A:set(10), B:get() = 10

In the second example, if A tells B about the value of the register
(10) after it does its get(), B will falsely believe that some
third-party has written 5 between A:get() and B:get(). If external
communication isn't allowed, B cannot know about A:set, and so sees a
consistent view of the register state; it's as if B:get really did
happen before A:set.

2つ目の例ではもしAがBにregisterの値（10）についてget()の後に伝えたとすると、Bは誰か第三者がA:get()とB:get()の間に5を書いたのだと間違って信じてしまうだろう。もし外部コミュニケーションが許可されない場合、BはA:setについて知ることができず、従ってレジスタの状態が一定であると見てしまう。それはまるでB:getが本当にA:setの前に起ったのかであるようだ。

Wikipedia [1] has more information. Maurice Herlihy's original paper
from 1990 is available at [2].

Wikipedia[1]により多くの情報が存在する。Maurice Herlihyの1990年のオリジナル論文は[2]に存在する。

[1] http://en.wikipedia.org/wiki/Linearizability
[2] http://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf

## 5. What does _asynchronous_ mean?
## 5. _asynchronous_ (非同期)とは何を意味するのか？

An _asynchronous_ network is one in which there is no bound on how
long messages may take to be delivered by the network or processed by
a machine. The important consequence of this property is that there's
no way to distinguish between a machine that has failed, and one whose
messages are getting delayed.

非同期ネットワークとはその中でメッセージがネットワークにより配送されるのに、または機械により処理されるのにどれくらい時間がかかるかについて制約が無い物を言う。この属性の重要な結論は機械の故障とメッセージの遅配との間で区別をつける方法が無いことである。

## 6. What does _available_ mean?
## 6. _available_とは何を意味するのか？

A data store is available if and only if all get and set requests
eventually return a response that's part of their specification. This
does _not_ permit error responses, since a system could be trivially
available by always returning an error.

データストアは全てのgetとsetの要求がいつかはレスポンスを返す場合、その場合のみavailableである。これは仕様の一部である。エラーレスポンスは許されない。なぜならいつもエラーを返すことでシステムは意味なくともavailableになってしまうためである。

There is no requirement for a fixed time bound on the response, so the
system can take as long as it likes to process a request. But the
system must eventually respond.

レスポンスには一定時間の要件は存在しないのでシステムはリクエストの処理において好きなだけ時間をかけることが可能である。しかしシステムはいつかは返信しなければならない。

Notice how this is both a strong and a weak requirement. It's strong
because 100% of the requests must return a response (there's no
'degree of availability' here), but weak because the response can take
an unbounded (but finite) amount of time.

これが如何に強力であり、かつ弱い要件であるか注意すること。これはリクエストの100%が返信を必ず行う（ここにはavailabilityの度合いは存在しない）意味で強力であり、しかし返信が際限ない（しかし有限である）量の時間を消費可能な意味で弱い。

## 7. What is a _partition_?
## 7. _partition_とは何か？

A partition is when the network fails to deliver some messages to one
or more nodes by losing them (not by delaying them - eventual delivery
is not a partition).

paritionとはネットワークが1つ、またはより多くのノードへのメッセージ配信をメッセージを失なうことで失敗する場合である。（遅配ではなく - eventualy deliveryはpartitionではない。）

The term is sometimes used to refer to a period during which _no_
messages are delivered between two sets of nodes. This is a more
restrictive failure model. We'll call these kinds of partitions _total
partitions_.

この用語は時々2つのノードの集合の間で_1つも_メッセージが配信されない間の時間を参照するのに用いられる。これはより制限された故障のモデルである。我々はこのような種類のpartitionを_total partitions_と呼ぶ。

The proof of CAP relied on a total partition. In practice,
these are arguably the most likely since all messages may flow through
one component; if that fails then message loss is usually total
between two nodes.

CAPの証明はtotal partitionに依存する。実際にはこれらが恐らく間違いなく多くの場合を占めるであろう。なぜなら全てのメッセージは1つのコンポーネントを通して流れるだろうからである。もしそれが故障すればメッセージロスは通常2つのノード間においてtotalである。

## 8. Why is CAP true?
## 8. なぜCAPは正しいのか？

The basic idea is that if a client writes to one side of a partition,
any reads that go to the other side of that partition can't possibly
know about the most recent write. Now you're faced with a choice: do
you respond to the reads with potentially stale information, or do you
wait (potentially forever) to hear from the other side of the
partition and compromise availability?

基本となるアイデアはもしクライアントがpartitionのあるサイドに書き込み、任意のreadがpartitionのもう一方のサイドに行く場合、もしかすると最新のwriteについて知ることができないだろう。今あなたは1つの選択に直面した：あなたは古いかもしれない情報を返しますか？それとも（可能性としては永遠に）partitionのもう一方のサイドからの連絡を待つことで可用性について妥協しますか？

This is a proof by _construction_ - we demonstrate a single situation
where a system cannot be consistent and available. One reason that CAP
gets some press is that this constructed scenario is not completely
unrealistic. It is not uncommon for a total partition to occur if
networking equipment should fail.

この証明は_構造_によるものだ。我々はシステムがconsistentでなく、かつavaiableでもない単一のシチュエーションを実演する。CAPがいくらかの圧力を受けるのはこの構成されたシナリオが完全には非現実的ではないためだ。ネットワーク装置が故障した時total partitionが発生することは珍しいことではない。

## 9. When does a system have to give up C or A?
## 9. どんな場合にシステムはCまたはAを諦めるのか？

CAP only guarantees that there is _some_ circumstance in which a
system must give up either C or A. Let's call that circumstance a
_critical condition_.  The theorem doesn't say anything about how
likely that critical condition is. Both C and A are strong guarantees:
they hold only if 100% of operations meet their requirements. A single
inconsistent read, or unavailable write, invalidates either C or
A. But until that critical condition is met, a system can be happily
consistent _and_ available and not contravene any known laws.

CAPは_いくらかの_状況においてシステムがCまたはAのどちらかを諦めなければいけないことだけを保証する。その状況を_critical condition_と呼ぼう。定理は_critical condition_がどの程度発生するのかについては一切説明しない。CとAの両者が強く保証するのは命令の100%が要件に合うかどうかのみである。単一の不安定なread、またはavailableでないwriteがCまたはAを無効にする。しかしそのcritical conditionが発生するまでは、システムは幸福の内にconsistent、_かつ_、availableであり、どんな既知の法にも矛盾しない。

Since most distributed systems are long running, and may see millions
of requests in their lifetime, CAP tells us to be
cautious: there's a good chance that you'll realistically hit one of
these critical conditions, and it's prudent to understand how your
system will fail to meet either C or A.

多くの分散システムは長時間実行され、そのライフタイムの間に幾百万ものリクエストを処理するため、CAPは我々に慎重になれと告げる。現実に我々が1つのcritical conditionに出会う可能性は存在し、そしてあなたのシステムがCまたはAを満たすのにどのように失敗するのかを知ることこそが慎重であるということだ。

## 10. Why do some people get annoyed when I characterise my system as CA?
## 10. なぜある人々は私が私のシステムをCAであると見做すとイライラするのか？

Brewer's keynote, the Gilbert paper, and many other treatments, places
C, A and P on an equal footing as desirable properties of an
implementation and effectively say 'choose two!'. However, this is
often considered to be a misleading presentation, since you cannot
build - or choose! - 'partition tolerance': your system either might experience
partitions or it won't.

Brewerのキーノート、Gilbertの論文、そして多くの他の文献がC、A、Pを実装における理想的属性として等価な基盤として扱い、事実上'2つを選べ！'と言っているようだ。しかしこれはしばしばミスリーディングな表現だと考えられている。なぜならあなたは'partition tolerance'を構築することも選択することも（！）できないからだ。あなたのシステムはpartitionを経験するかしないかだ。

CAP is better understood as describing the tradeoffs you
have to make when you are building a system that may suffer
partitions. In practice, this is every distributed system: there is no
100% reliable network. So (at least in the distributed context) there
is no realistic CA system. You will potentially suffer partitions,
therefore you must at some point compromise C or A.

CAPはparitionを被るかもしれないシステムを構築する時にあなたが決定しなければならないトレードオフだと説明することでより良く理解される。実際にはこれは全ての分散システムだ。100%信頼できるネットワークは存在しない。従って(最低でも分散の文脈では）現実ではCAシステムは存在しない。起こり得ることとしてあなたはpartitionの被害をいつか受けるし、従ってある時点においてCまたはAを妥協せざるを得ない。

Therefore it's arguably more instructive to rewrite the theorem as the
following:

従って恐らく定理を次のように書き直すことでより役に立つだろう。

<pre>Possibility of Partitions => Not (C and A)</pre>

<pre>partitionの可能性 => Not (C and A)</pre>

i.e. if your system may experience partitions, you can not always be C
and A.

言い換えれば、もしシステムがpartitionを経験するのであれば、常にCとAが両立はしない。

There are some systems that won't experience partitions - single-site
databases, for example. These systems aren't generally relevant to the
contexts in which CAP is most useful. If you describe your distributed
database as 'CA', you are misunderstanding something.

paritionにならないシステムはいくつか存在する - 単一のサイトのデータベースが一例だ。これらのシステムは通常CAPが最も役立つ文脈には属さない。もしあなたがあなたの分散データベースを'CA'だと説明するならあなたは何か誤解している。

## 11. What about when messages don't get lost?
## 11. メッセージが失なわれない場合はどうか？

A perhaps surprising result from the Gilbert paper is that no
implementation of an atomic register in an asynchronous network can be
available at all times, and consistent only when no messages are lost.

恐らくGilbertの論文の驚くべき結論は非同期ネットワークにおけるどのレジスタの実装も常にavailableであることはできないということだ。そしてメッセージが全く失われない場合のみconsistentである。

This result depends upon the asynchronous network property, the idea
being that it is impossible to tell if a message has been dropped and
therefore a node cannot wait indefinitely for a response while still
maintaining availability, however if it responds too early it might be
inconsistent.

この結論は非同期ネットワークの属性に依存しており、その考えはメッセージが失なわれたのかを確認することは不可能であるがため、ノードはavailabilityを保ったまま無期限に返信を待つことはできない、しかしもしそれの返信が早過ぎる場合にはinconsistentとなるだろうということだ。

## 12. Is my network really asynchronous?
## 12. 私のネットワークは本当に非同期なのか？

Arguably, yes. Different networks have vastly differing characteristics.

恐らく間違いなく、そうだ。異なるネットワークは大きく異なる特性を持つ。

If

もし

* Your nodes do not have clocks (unlikely) or they have clocks that may drift apart (more likely)
* System processes may arbitrarily delay delivery of a message (due to retries, or GC pauses)

* あなたのノード群が時計を持っていない（ありえないだろうが）、または時計を持っているがそれぞれの時間がずれてしまう（最もありえる）
* システムの処理は任意にメッセージの配信が遅れる（リトライやGCによる一時停止のため等）

then your network may be considered _asynchronous_.

ならばあなたのネットワークは_非同期_だと考えられるだろう。

Gilbert and Lynch also proved that in a _partially-synchronous_
system, where nodes have shared but not synchronised clocks and there
is a bound on the processing time of every message, that it is still
impossible to implement available atomic storage.

GilbertとLynchはまた_partially-synchronous_なシステムでは、ノード群が時計を持ってはいるが同期していなく、各メッセージの処理時間に制約が存在する場合、availableなatomicストレージを実装することは不可能であることを証明している。

However, the result from #8 does _not_ hold in the
partially-synchronous model; it is possible to implement atomic
storage that is available all the time, and consistent when all
messages are delivered.

しかし、#8の結果はpartially-synchronous modelでは当たらない。常にavailableであり、かつ全てのメッセージが配信された時consistentなatomic storageを実装することは可能である。

## 13. What, if any, is the relationship between FLP and CAP?
## 13. もし存在するなら、FLPとCAPの間の関係とは何か？

The Fischer, Lynch and Patterson theorem ('FLP') (see [1] for a link
to the paper and a proof explanation) is an extraordinary
impossibility result from nearly thirty years ago, which determined
that the problem of consensus - having all nodes agree on a common
value - is unsolvable in general in asynchronous networks where one
node might fail.

Fischer、Lynch、Pattersonの定理（'FLP')(論文と証明の説明へのリンクは[1]を参照)は尋常ならざる不可能性の結論であり、約30年前に、合意の問題 - 共通の値に対し全てのノードが合意することは非同期ネットワークにおいてあるノードが故障するかもしれない場合には通常、解決不能であると決定した。

The FLP result is not directly related to CAP, although they are
similar in some respects. Both are impossibility results about
problems that may not be solved in distributed systems. The devil is
in the details. Here are some of the ways in which FLP is different
from CAP:

FLPの結論は直接はCAPに関係が無い。しかしある程度の詳細において似通っている。両者共、分散システムにおいて解決不能な問題についての不可能性についての結論である。神は細部に宿る。ここにFLPがCAPとどのように異なるかいくつかを示す。

* FLP permits the possibility of one 'failed' node which is totally
  partitioned from the network and does not have to respond to
  requests.
* Otherwise, FLP does not allow message loss; the network
  is only asynchronous but not lossy.
* FLP deals with _consensus_, which is a similar but different problem
  to _atomic storage_.

* FLPはある'故障した'ノードがネットワークから分離されリクエストに対し返信しない状況を認めている
* 他には、FLPはメッセージロスを許可しない。ネットワークは非同期であるが損失はない。
* FLPは_合意_について取り扱い、それは_atomic storage_に似ているが異なる問題である

For a bit more on this topic, consult the blog post at [2].

このトピックについてのより詳細は[2]のブログ記事より得られる。

[1] http://the-paper-trail.org/blog/a-brief-tour-of-flp-impossibility/
[2] http://the-paper-trail.org/blog/flp-and-cap-arent-the-same-thing/

## 14. Are C and A 'spectrums'?
## 14. CとAは'スペクトラム'なのか？

It is possible to relax both consistency and availability guarantees
from the strong requirements that CAP imposes and get useful
systems. In fact, the whole point of CAP is that you
_must_ do this, and any system you have designed and built relaxes one
or both of these guarantees. The onus is on you to figure out when,
and how, this occurs.

consistencyとavailabilityの両者の保証をCAPが提示する強い保証から緩めた上で実用的なシステムを得ることは可能である。実際にCAPの確信とは_絶対に_このことを行わなければならないということであり、設計・構築された任意のシステムはこれらの保証の内、1つ、または両方を緩めている。いつ、どうやってこれを行うのかの責任はあなたにある。

Real systems choose to relax availability - in the case of systems for
whom consistency is of the utmost importance, like ZooKeeper. Other
systems, like Amazon's Dynamo, relax consistency in order to maintain
high degrees of availability.

現実のシステムはavailabilityを緩める選択する - ZooKeeperのようにconsistencyが最も重要なシステムの場合には。他のシステム、例えばAmazonのDynamo等は、高度のavailabilityを保持するためにconsistencyを緩める。

Once you weaken any of the assumptions made in the statement or proof
of CAP, you have to start again when it comes to proving an
impossibility result.

一度、CAPの証明、または記述にて定められた想定を緩めた場合、システムが不可能性についての結論についての証明に辿りついた時、再開しなければならない。

## 15. Is a failed machine the same as a partitioned one?
## 15. 故障した機械とはpartition状態と同じなのか？

No. A 'failed' machine is usually excused the burden of having to
respond to client requests. CAP does not allow any machines to fail
(in that sense it is a strong result, since it shows impossibility
without having any machines fail).

いいえ。'故障した'機械はクライアントのリクエストに対し返信する責務を負い起動されている。CAPはどの機械に対しても故障することを許さない。（その意味では、これは全く故障した機械が存在しないのに不可能性を定義するのであるから強い結論である）

It is possible to prove a similar result about the impossibility of
atomic storage in an asynchronous network when there are up to N-1
failures. This result has ramifications about the tradeoff between how
many nodes you write to (which is a performance concern) and how fault
tolerant you are (which is a reliability concern).

非同期ネットワークにおけるatomic storageの不可能性について、N-1台に渡る故障が存在する時に同様の結論を示すことが可能である。この結論はいくつのノードにwriteを行うのか（パフォーマンス上の考慮点）とどれだけ障害に耐えられるのか（信頼性上の考慮点）の間のトレードオフに関する悪影響について示す。

## 16. Is a slow machine the same as a partitioned one?
## 16. 遅い機械はpartition状態と同じか？

No: messages eventually get delivered to a slow machine, but they
never get delivered to a totally partitioned one. However, slow
machines play a significant role in making it very hard to distinguish
between lost messages (or failed machines) and a slow machine. This
difficulty is right at the heart of why CAP, FLP and other results are
true.

いいえ：メッセージはいつかは遅い機械に配信される。しかしメッセージはpartition状態の機械には絶対に配信されない。しかし遅い機械はメッセージの欠落（または故障した機械）と遅い機械との識別をとても難しくする重要な役割を行う。この困難さはなぜCAP、FLPと他の結論が正しいのかの核心にまさに存在する。

## 17. Have I 'got around' or 'beaten' the CAP theorem?
## 17. CAP定理を迂回したり、否定することは可能か？

No. You might have designed a system that is not heavily affected by
it. That's good.

いいえ。あなたはその影響を大きくは受けないシステムを設計したかもしれない。それは良いことだ。
