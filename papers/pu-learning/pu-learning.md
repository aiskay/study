# Positive and Unlabeled (PU) Learning

## Abstract

教師あり学習を正例と負例ではなく正例と未知のデータ (unlabeled data) として行う手法として positive and unlabeled (PU) learning というものが存在する。
ここでは、PU learning を行う際の一般的な仮定をまとめた後、labeling が正例の内からランダムに行われるという最も単純な仮定を用いた場合の学習の行い方を説明する。

## Introduction

一般の二値分類の学習では、データは全て正例と負例に分類されている。
しかし、世の中には一部の正例のみが与えられており、その他は正例か負例かわからないというデータも多く存在する。
例えば、

- ページや広告への興味関心 : 広告やページへの興味関心はある程度 click で計ることができるが、click していないことは必ずしも興味がないことを表さない
- 病人の特定 : 有症者だけでなく無症者からも病気かどうかを判定しなければならない
- Knowledge Base (KB) の作成 : 既知の分類を用いて新たなデータを分類しなければならない

といった例では一部の正例のみを用いて正例か負例かわからないデータを分類することが求められる。
このような問題設定の枠組みは Positive-Unlabeld (PU) learning と呼ばれており、2000 年以降積極的に議論が行われている[^1]。

ここでは参考文献

- [Learning from positive and unlabeled data: a survey](https://paperpile.com/app/p/1c3c03e4-b516-033e-87f9-9ff071d33711)
- [Learning classifiers from only positive and unlabeled data](https://paperpile.com/app/p/a19eb2e0-21c3-0343-bfd7-8b4f5ae0053f)

に基づいて最も簡単な PU learning の学習手法についてまとめる。

なお、Notation は以下で統一する。

$$
\begin{cases}
    x & :{\rm \ example\ (vector\ of\ attributes)},\\
    y \in \{0,1\} & :{\rm \ binary\ label},\\
    s \in \{0,1\} & :{\rm \ labeled\ data\ or\ not}.
\end{cases}
$$

## Preliminaries on PU learning

正例と unlabeled なデータからなるデータセットの生成方法には主に以下の 2 つの種類がある。

- single-training-set scinario : 一つの母集団から i.i.d. にサンプルされたデータの一部が正例であると仮定する
- case-control scenario : 正例と unlabeled なデータが異なるデータセットから抽出され、unlabeled なデータが母集団から i.i.d にサンプルされたと仮定する  

正例まで込みの分布を含んでいるという点で single-training-set scenario の方がより多くの情報を持っており、盛んに研究がされている。
そこで、ここでは主に single-training-set scenario についてまとめる。


## Assumptions to enable PU learning

### Labeling mechanism assumptions

PU learning では正例にラベルが付く機構が学習方法の定式化に大きな影響を与える。
そこで、一般的な labeling mechanism の仮定を以下にまとめる。

1. Selected Completely At Random (SCAR)  
    Labeled examples are selected completely at random, independent from their attributes, from the positive distribution. The **propensity score $e \left( x \right)$**, which is the probability for selecting a positive example, is constant and equal to the **label frequency $c$**:

$$
e(x) \equiv p \left( s=1|x,y=1 \right) = p \left(s=1|y=1\right) = c.
$$

2. Selected At Random (SAR)  
    Labeled examples are a biased sample from the positive distribution, where the bias completely depends on the attributes and is defned by the propensity score $e(x)$:  
$$
e(x) \equiv p(s = 1|x,y = 1).
$$

3. Probabilistic gap  
    Labeled examples are a biased sample from the positive distribution, where examples with a smaller probabilistic gap $\Delta p(x)$ are less likely to be labeled. The propensity score is a non-negative, monotone increasing function f of the probabilistic gap $\Delta p (x)$:  

$$
e (x) = f (∆p (x)) = f (p (y = 1|x) − p (y = 0|x)), \quad \frac{d}{dt}f(t) > 0.
$$

### Data assumptions

データの特性についても一般的によく用いられる仮定をまとめる。

1. Negativity  
To assume that the unlabeled examples all belong to the negative class. Despite the fact that this assumption obviously does not hold, it is often used in practice.
2. Separability  
正例と負例が分割されている (つまり正例と負例を完全に区別できる分類器が存在する)という仮定。
3. Smootheness  
If two instances $x_1$ and $x_2$ are similar, then the probabilities $p\left(y=1|x_{1}\right)$ and $p\left(y=1|x_{2}\right)$ will also be similar.


## PU learning methods

ここでは、先に紹介した仮定のうち最も単純な以下の仮定

- single-training-set scinario
- (Selected Completely At Random) SCAR 

を採用した場合のPU learning の手法を 2 つ紹介する。

###  Learning a traditinal classifier from non-traditional input

一つ目の方法は、シンプルに代数的な仮定を用いて $p(y=1|x)$ を直接求めようというものである。
言い換えると、正例のみラベルが与えられている $p(s=1|x,y=0)=0$ とき、traditinal classifier の予測 $f(x) \equiv p(y=1|x)$ を non-traditinal classifier   $g(x) \equiv p(s=1|x)$  を用いて直接求めようということである。

SCAR 仮定 $p(s = 1|x,y = 1) = p(s = 1|y = 1)$ を用いると、$f(x)$ は以下のように求まることが分かる。

$$
\begin{align}
    p \left(s=1|x\right) 
    & = p \left(y=1 \land s=1|x \right),\\
    & = p \left(y=1|x\right) p\left(s=1|x,y=1\right),\\
    & = p \left(y=1|x\right) p\left(s=1|y=1\right).
\end{align}
$$
$$
\begin{equation}
    \therefore \quad 
    f(x) = p(y=1|x) = \frac{p(s=1|x)}{p(s=1|y=1)} = \frac{g(x)}{c}.
\end{equation}
$$

従って、求めたい予測 traditinal classifier は non-traditinal classifier の予測 $g(x)$ を $c = p(s=1|y=1)$ で割ることで求めることができる。
式を見て分かる通り、これは予測結果の順序を変更しないので ROC-AUC などの順位にしか依らない metrics には影響を与えない。

$c$ の求め方の一つは、ラベルの付いているもの $s=1$ のみに着目して以下の関係を用いることである。

$$
\begin{align}
    g(x) 
    & = p (s=1 |x),\\
    & = p(s=1|x, y=1) p(y=1 |x) + p(s=1|x, y=0) p(y=0 |x), \\
    & = p (y=1|x) p\left(s=1|y=1\right). \quad ({\rm when}\ x \in P)
\end{align}
$$

where $P$ is the subset of examples that are labeled.

従って、実用的にはデータを適当な training set と validation set に分け、training set で $g(x)$ を学習した後に validation set で近似的に $c$ を求め、$g(x)$ をその $c$ で割るといった手法で結果を計算することができる。

以上が最もシンプルな traditinal classifier の予測の求め方である。

### Weighting unlabeled examples

2 つ目の方法は、学習時に $s=1$ のものと $s=0$ のものに適切な重みを与えることで方法である。

任意の関数 $h(x, y)$ について、

$$
\begin{align}
    \mathbb{E}[h] 
    & =\sum_{s,y} \int h(x,y) p(x,y,s) dx,\\
    & = \int p(x) \sum_{s=0}^{1} p(s|x) \sum_{y=0}^{1} p(y|x,s) h(x,y) dx,\\
    & = \int p(x) \left[
        p(s=1|x) h(x,1) + p(s=0|x) \left(p(y=1|x,s=0) h(x,1) + p(y=0|x,s=0) h(x,0)\right)
    \right] dx.
\end{align}
$$

積分を ensemble average の関係

$$
\begin{align}
    \int p(x,s) h(x,y) dx 
    & = \int \left[p(x,s=0) + p(x,s=1)\right] h(x,y) dx,\\
    & = \lim_{m \rightarrow \infty} \frac{1}{m} \sum_{i=1}^{m}h\left(x_{i},y\right), \\
    & = \lim_{m \rightarrow \infty} \frac{1}{m} \left[
            \sum_{i,s=0} h\left(x_{i},y\right) + \sum_{i,s=1} h\left(x_{i},y\right)
        \right].
\end{align}
$$

を用いて積分を和で置き換えると、

$$
\begin{equation}
    \mathbb{E}[h] \simeq
    \frac{1}{m} \left[
        \sum_{<x,s=1>} h(x,1) + \sum_{<x,s=0>} \left(
        w (x) h(x,1) + \left(1-w(x)\right) h(x,0)\right)
    \right],
\end{equation}
$$

where 

$$
\begin{align}
    w(x)
    & \equiv p(y=1| x,s=0), \\
    & = \frac{p(s=0| x,y=1) p(y=1| x)}{p(s=0| x)}, \\
    & = \frac{(1-p(s=1| x,y=1))p(y=1|x)}{1 - p(s=1|x)}, \\
    & = \frac{1-c}{c} \frac{p(s=1| x)}{1-p(s=1|x)},\\
\end{align}
$$

and $m$ is the cardinality of the training set.

従って、$c$ と $g(x) = p(s=1|x)$ を前章と同様に求めて $w(x)$ を算出し、ラベルありデータに重み 1 を、ラベルなしデータを複製してそれぞれに重み $w(x)$ と $1 − w(x)$ を与えて重み付き学習をしてあれば良いことがわかる。


[^1]: といっても機械学習界隈ではかなりマイナーな分野であり、かつ予測結果の妥当性を確かめる方法があまりないので実際に使われている例は少ない