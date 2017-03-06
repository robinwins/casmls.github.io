---
layout: post
title: "Prediction with a Short Term Memory"
categories: general
author: Gabriel Loaiza
excerpt_separator: <!--more-->
comments: true
---

Last Thursday, Andrew presented a paper by Kakade et al. in which the problem of predicting the next observation given a sequence of past observations is studied. In particular, they study how far off a Markov model is from the optimal predictor. For a long time, simple Markov models were the state of the art for this task and have now been beat by Long Short-Term Memory neural networks. The paper tries to figure out why it took so long to beat the Markov Models. An interesting comment that was made pointed out that a Markov model of order $$l$$ has exponentially many parameters while LSTM networks don't have that many parameters.

<!--more-->

Their results can be summarized as follows:
(i) The best $$l^{th}$$ order Markov Model achieves average $$\mathrm{KL}$$ error of at most $$I(M)/l$$, where $$I(M)$$ is the mutual information between the past and future observations.
(ii) Under a hardness condition, $$d^{\Theta(I(M)/\epsilon)}$$ samples are required to obtain average error $$\epsilon$$ for any polynomial time algorithm, where $$d$$ is the size of the alphabet where the observations take values.
(iii) Result (i) implies that if $$l$$ is larger $$I(M)/\epsilon$$, an average $$\mathrm{KL}$$ error of $$\epsilon$$ is achieved. By Pinsker's inequality (which states that the total variation distance is bounded by the square root of 1/2 times the KL), this means that if $$l$$ is larger than $$I(M)/\epsilon^2$$, then an average $$l_1$$ error of $$\epsilon$$ is achieved. This third result shows that there exists a model for which $$l$$ has to be $$O(I(M)/\epsilon^2)$$ in order to achieve average $$l_1$$ error of $$\epsilon$$, meaning that it's impossible do better than $$I(M)/\epsilon^2$$.

### Notation:

Recall that the mutual information $$I(X;Y)$$ between the random variables X and Y is given by $$I(X;Y) = H(Y) - H(Y\mid X)$$ where $$H(Y)=-\sum_x P(x)\log P(x)$$ is the entropy of $$X$$ and $$H(Y\mid X) = H(X,Y) - H(X)$$ is the conditional entropy of $$Y$$ given $$X$$, where $$H(X,Y)$$ is the (joint) entropy of $$(X,Y)$$.

Let $$x_{i}^{j}$$ denote the vector $$(x_i,...,x_j)$$, where $$i$$ and $$j$$ are allowed to be negative or positive infinity. It is assumed that the infinite sequence $$\{x_t\}$$ is generated by some model $$M$$. The optimal predictor $$P_\infty$$ has distribution $$P(x_t\mid x_{-\infty}^{t-1})$$, the conditional distribution of the observation at time $$t$$ given all the previous observations. The distribution of an arbitrary predictor $$A_l$$ which only looks at the past $$(l-1)$$ outputs is denoted $$Q_{A_l}(x_t\mid x_{t-l+1}^{t-1})$$. The optimal predictor $$P_l$$ which only looks at the past $$(l-1)$$ observations is the true conditional distribution $$P(x_t\mid x_{t-l+1}^{t-1})$$. The mutual information $$I(M)$$ between the past and future is defined as:

$$I(M)=\dfrac{1}{T}\displaystyle\sum_{t=0}^{T-1} I(x_{t}^{\infty};x_{-\infty}^{t-1})$$

Note that if $$\{x_t\}$$ is stationary (that is, the distribution of any subset of variables is invariant with respect to time shifts), then $$I(M) = I(x_{0}^{\infty};x_{-\infty}^{-1})$$.

Let $$F(P,Q)$$ be any measure of distance between predictive distributions. The expected loss  $$\delta_F(A_l)$$ of $$A_l$$ with respect to the optimal predictor is defined as:

$$\delta_F(A_l) = \dfrac{1}{T}\displaystyle \sum_{t=0}^{T-1}\delta_{F}^{(t)}(A_l)$$, where:

$$\delta_{F}^{(t)} = \mathbb{E}_{x_{-\infty}^{t-1}}\Big[F\big(P(x_t\mid x_{-\infty}^{t-1}),Q_{A_l}(x_t\mid x_{t-l+1}^{t-1})\big)\Big]$$

Analogously, the expected loss $$\hat{\delta}_F(A_l)$$ of $$A_l$$ with respect to the optimal predictor which only looks $$(l-1)$$ steps into the past is:

$$\hat{\delta}_F(A_l) = \dfrac{1}{T}\displaystyle \sum_{t=0}^{T-1}\hat{\delta}_{F}^{(t)}(A_l)$$, where:

$$\hat{\delta}_{F}^{(t)} = \mathbb{E}_{x_{t-l+1}^{t-1}}\Big[F\big(P(x_t\mid x_{t-l+1}^{t-1}),Q_{A_l}(x_t\mid x_{t-l+1}^{t-1})\big)\Big]$$

### Results:

(i) The first result is the following (Proposition 1 on page 6):

$$\delta_{\mathrm{KL}}(P_l) \leq I(M)/l \text{ for any } M.$$

Furthermore,

$$\delta_{\mathrm{KL}}(A_l) \leq I(M)/l + \hat{\delta}_{\mathrm{KL}}(A_l) \text{ for any }A_l.$$

The authors emphasize that the first part of this result can be interpreted as "to obtain accurate prediction on average it suffices to have short term memory", since the Markov model uses only the last $$(l-1)$$ observations. The authors provide some intuition on this: when predicting $$x_t$$, the prediction is either correct or not. If it is incorrect, then $$x_t$$ somehow contains information about the past which will help when predicting $$x_{t+1}$$. Finally, the authors also argue that this result should be interpreted as saying that average prediction error is not a good metric because a Markov model, which does not incorporate long term dependency, should not be able to accurately predict in tasks such as understanding natural language, where this long term dependency is expected. We discussed this point and mentioned that arguably, as $$M$$ becomes very complex, $$I(M)$$ could potentially become very large thus requiring a very large $$l$$ to actually achieve accurate predictions on average, effectively using long term memory.

I won't write the proof of the result since it can be seen on the paper, but Andrew made two comments that don't appear on the paper. The first one is that the second equality of the proof (on top of page 7) is due to the fact that for any distributions $$P$$ and $$\hat{P}$$:

$$\mathbb{E}_{y,z}\Big[D_{\mathrm{KL}}\big(P(x\mid y,z)\mid \mid \hat{P}(x \mid y)\big)\Big]$$

$$= \mathbb{E}_{y,z}\Big[D_{\mathrm{KL}}\big(P(x\mid y,z)\mid\mid P(x\mid y)\big)\Big] + \mathbb{E}_{y}\Big[D_{\mathrm{KL}}\big(P(x\mid y)\mid\mid\hat{P}(x,y)\big)\Big]$$

Applying this with $$P:=P(x_t\mid x_{-\infty}^{t-1})$$ and $$\hat{P}:=Q_{A_l}(x_t\mid x_{t-l+1}^{t-1})$$ yields the equality. The second is that the last equality of the proof (again on page 7) requires more justification if $$\{x_t\}$$ is not stationary.

As a corollary of this result (Corollary 2 on page 7), bounds can also be obtained when $$F$$ is the $$l_1$$ loss instead of the $$\mathrm{KL}$$ divergence:

$$\delta_{l_1}(P_l)\leq \sqrt{I(M)/(2l)}$$ for any $$M$$ and $$\delta_{l_1}(A_l)\leq \sqrt{I(M)/(2l)} + \hat{\delta}_{l_1}(A_l)$$ for any $$A_l$$.

(ii) We did not discuss the second result (Section 3) as most of us were not familiar enough with computational complexity theory.

(iii) By result (i), choosing $$l$$ to be at least $$I(M)/\epsilon$$ (or $$I(M)/\epsilon^2$$) guarantees an average $$\mathrm{KL}$$ (or $$l_1$$) error of less than $$\epsilon$$. It is known that if $$\{x_t\}$$ is generated by a HMM, then $$I(M) \leq \log(n)$$. The authors construct a HMM which requires $$l=O(\log(n)/\epsilon)$$ (or $$O(\log(n)/\epsilon^2)$$) to achive an average $$\mathrm{KL}$$ (or $$l_1$$) error of less than $$\epsilon$$ (Proposition 3 on page 15). This means that in general, the rate of the bound from (i) can't be improved. The question of what happens in practice remains unanswered though: How often are HMMs as "bad" as the one the authors construct?

Again, I won't go through the proof since it can be seen in the paper, but we did have an interesting discussion about equation 5.1 (on page 17). Andrew justified why the authors say that the inequality follows directly from Hoeffding's inequality with the following theorem:

Let $$(X_k)_{k=0}^{n}$$ be a martingale with respect to a filtration $$(F_k)_{k=0}^{n}$$ on a finite probability space $$(\Omega,F,P)$$ with mean $$0$$. Consider $$Y_k = X_k - X_{k-1}$$, and define the spread of $$Y_k$$ as the smallest $$s_k$$ such that on each atom of $$F_{k-1}$$, $$Y_k$$ has range $$s_k$$. Then:

$$P(X_n \geq \mu)\leq \exp\Big(-\dfrac{2\mu^2}{s_n}\Big)$$

Proof sketch:
It can be seen that $$E\big[\exp(hY_k)\mid F_{k-1}\big]\leq \exp(h^2s_k^2/8)$$. From this, it follows that $$E\big[\exp(hX_n)\big]\leq \exp(h^2s_n/8)$$. Then, by Markov's inequality:

$$P(X_n\geq \mu) = P\big(\exp(hX_n)\geq \exp(h\mu)\big)$$

$$\leq \dfrac{E\big[\exp(hX_n)\big]}{\exp(h\mu)}\leq \dfrac{\exp(h^2s_n/8)}{\exp(h\mu)}\leq \exp\Big(-\dfrac{2\mu^2}{s_n}\Big)$$

where the last inequality follows by finding the value of $$h$$ which minimizes the bound.

Now, on $$\{0,1\}^l$$ a filtration can be constructed where $$F_k$$ looks only at the first $$k$$ coordinates. That is, $$F_0$$ is the trivial $$\sigma$$-algebra; $$F_1$$ is the $$\sigma$$-algebra generated by two sets: the set of sequences that start with 0 and the set of sequences that start with 1. $$F_2$$ is the $$\sigma$$-algebra generated by four sets: the sets of sequences starting with 00, 01, 10 and 11. And so on. By applying the previous theorem with this filtration, the bound on the Hamming distance of equation 5.1 follows.

### References

Kakade, S., Liang, P., Sharan, V., & Valiant, G. (2016). Prediction with a Short Memory. arXiv preprint arXiv:1612.02526.

