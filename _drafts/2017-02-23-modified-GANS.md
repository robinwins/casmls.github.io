---
layout: post
title: "Modified GANs"
categories: general
author: Robin Winstanley
excerpt_separator: <!--more-->
comments: true
---

In this week's session, we read and discussed two papers relating to GANs: Wasserstein GAN (Arjovsky et al. 2017 [1]) 
and Adversarial Variatonal Bayes: Unifying Variational Autoencoders and Generative Adversarial Networks 
(Mescheder et al. 2017 [4]). The first paper introduces the use of the Wasserstein distance rather than KL divergence 
for optimization in order to counter some of the problems faced in original GANs. The second paper synthesizes GANs with 
VAEs in an effort to allow arbitrarily complex inference models.

<!--more-->

_The figures and algorithms below are taken from the aforementioned papers._

# Review of GANs

GANs were introduced by Goodfellow et al. in 2014 [2] as an alternative to the maximum likelihood approach to generative models. 
In a maximum likelihood framework, we define a distribution $$x_i \sim p_\theta$$ over our data $$\{x_i\}_{i=1}^n$$ and choose 
parameters $$\theta$$ such that the likelihood of the training data is maximized: 
$$\theta^*=\underset{\theta}{\argmax} \prod_{i=1}^n p_\theta(x_i)$$.

In a GAN framework, the problem is formulated as a minimax game between a generator function G, and a discriminator function D. 
The classic analogy for this is that of a counterfeiter and a policeman. The policeman (D) tries to maximize his ability to 
distinguish between counterfeit products and real products, while the counterfeiter (G) simultaneously tries to produce material 
that is as close to the real deal as possible. 

Define the following notation:
* $$p_\mathcal{D}$$ - the true data generating distribution
* $$p_z$$ - prior on the latent variables $$z$$
* $$p_g$$ - the distribution over fake generated data, $$D(G(z))$$
Setting this up as a minimax game, D is trained to maximize the probability of assigning x to the real data rather than the 
generated data $$p_g$$, while G is trained to minimize the discrepancy between the real data and the generated data. The 
value function is given by:

$$
\underset{G}{\min}\underset{D}{\max} V(G, D) = \E_{x\sim p_\mathcal{D}} \log D(x) + \Prob_{z\sim p_z} \log(1-D(G(z)))
$$

To optimize this, the following updates are used:
* (D) $$\underset{D}{\max} \E_{x\sim p_\mathcal{D}} \log D(x) + \Prob_{z\sim p_z} \log(1-D(G(z)))$$
* (G) $$\underset{G}{\min} \E_{x\sim p_g}[-\log D(x)]$$
The optimal discriminator for any generator can be shown to be: $$D^* = \frac{p_\mathcal{D}}{p_\mathcal{D}+p_g}$$ [2]. 
Plugging this in to the value function for the game, 

$$
C(G) = \underset{D}{\max} V(G, D^*) = -\log(4)+2\cdot JS(p_\mathcal{D} \mid\mid p_g)
$$
where JS is the Jenson-Shannon divergence.

## Problems with GANs
* Unstable training - It is hard to balance between optimizing G and D
* Collapse of the mode, also called mode dropping
* High dependency on the architecture of the NN
* Hard to quantify how well a model does outside of ``these generated pictures look good"

# Wasserstein GAN

This paper overcomes some of the problems with the original GANs through the introduction of an alternative metric to 
KL divergence. When looking at distributions over low dimensional manifolds, the chance of their having overlapping support 
is low, resulting in an undefined or infinite KL divergence. The Earth-Mover, or Wasserstein-1, distance is defined as the 
cost of the optimal transport plan: 

$$
W(p_r, p_g) = \underset{\gamma \in \Pi(p_r, p_g)}{\inf} \E_{(x,y) \sim \gamma}\| x-y \|
$$

The name "Earth-Mover" can be understood by thinking of each distribution as a pile of dirt, with the EM distance defined 
as the amount of dirt that needs to be moved times the distance it needs to be moved.  

In Example 1 of [1], the authors present two distributions with a negligible intersection in order to highlight the problems 
that arise when using KL and JS divergence. Take $$Z \sim U[0,1]$$ and define $$P_0$$ to be the distribution of 
$$(0,Z)\in\mathbb{R}^2$$. Let $$P_\theta$$ be the distribution of $$(\theta, Z)$$. Some common metrics in this case are then:
* $$W(\Prob_0, \Prob_\theta) = |\theta|$$
* $$JS(\Prob_0, \Prob_\theta) = \log 2$$ if $$\theta \neq 0$$, $$0$$ otherwise
* $$KL(\Prob_\theta \| \Prob_0) = + \infty$$ if $$\theta \neq 0$$, $$0$$ otherwise

As $$\theta \to 0$$, $$P_\theta$ converges to $P_0$ under the EM distance, but not under the JS or KL distances. The JS and 
KL distances are not even continuous in this instance. The authors formalize this idea in in Theorem 1 [1], providing 
continuity results for the EM distance that motivate their choice in using this as their loss function.

To optimize over $$W(p_\mathcal{D}, p_g)$$ instead of $$JS(p_\mathcal{D}, p_g)$$, something must be done about the intractable 
infimum present in the definition of Wasserstein distance. Enter the handy Kantorovich-Rubinstein duality. Using this duality, 
the EM distance can be reformulated as 

$$
W(p_\mathcal{D}, p_g) = \underset{\|f\|_L \leq 1}{\sup} \E_{x\sim p_\mathcal{D}}[f(x)]-\E_{x\sim p_g}[f(x)]
$$
where the supremum is over all 1-Lipschitz functions.

The Wasserstein GAN (WGAN) is then optimized through the following updates:
* (D) $$\underset{w \in \mathcal{W}}{\max} \E_{x\sim p_\mathcal{D}} f_w(x)-\E_{z\sim p(z)}f_w(g_\theta(z))$$
* (G) $$\underset{\theta}{\min} \E_{z \sim p_z} f_w(g_\theta(z))$$
where $$w \in \mathcal{W}$$ are weights that parameterize a neural network. To ensure $$w$$ lie in a compact space, the weights 
are clamped to $$\mathcal{W} = [-c,c]^l$$ [For comparison to the original GAN setup $$D(x) \approx f_w(x)$$, 
$$G(z) \approx g_\theta(z)$$].

Solving this optimization problem is done through the algorithm described below:

![WGANalgorithm]({{site.base_url}}/img/WGANalgorithm.jpg)

The weight constraint prevents the discriminator from saturating by limiting D to grow at most linearly. The continuity and 
differentiability of the EM distance allows D to be trained to optimality, preventing the modes from collapsing as before.
This contrast between GAN and WGAN is highlighted in Figure 2:

![WGANfig2]({{site.base_url}}/img/WGANfig2.jpg)

In the experiments section, the authors confirm their assertions, even bringing out the bold font in Section 4.3 to emphasize 
their circumvention of mode collapsing. It is also important to note that the nonstationarity of D results in a worse 
performance for momentum based methods like Adam, causing them to use RMSProp instead. 

# Adversarial Variational Bayes

This paper presents an adversarial procedure for training the Variational Autoencoders of Kingma et. al [3] that uses the 
flexibility of neural networks to allow arbitrarily complex inference models.

## Review of VAEs

Consider data $$\{ x_i\}_{i=1}^n$$ generated from some distribution $$p_\theta(x | z)$$ over latent continuous variables 
$$z \sim p(z)$$. We are interested in the true posterior density $$p_\theta(z |x)$$. This distribution is often intractable, 
and so we approximate it with an approximate inference model, $$q_\phi(z\mid x)$$. We wish to choose the model $$q$$ that 
minimizes the KL divergence between the approximate posterior and the true posterior: 
$$q = \underset{q}{\argmin} KL(q(z|x) \| p(z |x))$$.

A common result in variational inference bounds the marginal likelihood of $$x$$ by the variational lower bound (or ELBO):
$$
\log p(x) \geq KL(q_\phi(z\mid x) \| p(z)) + \E_{q}\log p_\theta(x \mid z)
$$

This lower bound is used to define the objective function we wish to optimize: 
$$
\underset{\theta}{\max}\underset{\phi}{\max}\E_{p_D(x)}[-KL(q_\phi(z|x),p(z)) + \E_{q_\phi}\log p_\theta(x|z)]
$$

## AVB

This paper starts by rewriting the objective function in the following form:
$$
\underset{\theta}{\max}\underset{\phi}{\max}\E_{p_D(x)}\E_{q_\phi}[\log p(z) - \log q_\phi(z|x)+\log p_\theta(x|z)]
$$

Ideally, we want $$q_\phi(z|x)$$ to be arbitrarily complex. But if $$q_\phi(z|x)$$ is defined through a black-box procedure, 
it can no longer be optimized using the reparameterization trick and stochastic gradient descent, as in VAE [3]. To get around 
this, the authors define a discriminative network $$T(x,z) = \log p(z) - \log q_\phi(z | x)$$ that is optimized simultaneously.

The objective function for $$T$$ is as follows: 
$$
\underset{T}{\max} \E_{p_D(x)}\E_{q_\phi(z|x)}\log \sigma(T) +  \E_{p_D(x)}\E_{p(z)}\log(1-\sigma(T))
$$

where $$\sigma(T)$$ is the sigmoid function. In this way $$T$$ is optimized to distinguish between pairs $$(x,z)$$ sampled 
independently from $$p_D(x)p(z)$$ and those sampled from the current model $$p_D(x)q_\phi(z|x)$$. The optimal discriminator 
for this objective is shown to be $$T^* = \log q_\phi(z|x)-\log p(z)$$. The new objective function is then,
$$
\underset{\theta}{\max}\underset{\phi}{\max}\E_{p_D(x)}\E_{q_\phi}[-T^*(x,z)+\log p_\theta(x|z)]
$$

In Proposition 2, the authors show that the gradient of $$T^*$$ with respect to $$\phi$$ satisfies 
$$\E_{q_\phi} \nabla_\phi T^*(x,z) = 0$$. Therefore, taking the gradients with respect to $$\theta$$ and $$\phi$$ is 
straightforward. Using the reparameterization trick of [3], the AVB algorithm alternates between optimizing the objective 
function and the discriminative network $$T$$:

![AVBalgorithm]({{site.base_url}}/img/AVBalgorithm.jpg)

As in the original GAN setup, the optimal discriminator can be shown to be: 
$$\sigma(T^*) = \frac{q_\phi(z|x)}{q_\phi(z|x) + p(z)}$$. By allowing $$T$$ and $$q_\phi(z|x)$$ to be arbitrarily complex, 
Corollary 4 proves that the game will recover the ML estimate $$\theta^*$$. Additionally, Corollary 4 proves that 
$$T^*(x,z) = \log \frac{p_{\theta^*}(x,z)}{p_{\theta^*}(x)p(z)}$$ is the pointwise mutual information between $$x$$ and $$z$$.

The experiments indicate an improvement between VAE and AVB on a variety of generative tasks. Their performance on a toy 
example is shown below:

![AVBtoyexample]({{site.base_url}}/img/AVBtoyexample.jpg)

For a nice implementation of this example, see [link](https://gist.github.com/poolio/b71eb943d6537d01f46e7b20e9225149).




### References
[1] Arjovsky, et al. "Wasserstein GAN." _arXiv preprint arXiv:1701.07875_ (2017). [link](https://arxiv.org/abs/1701.07875)
[2] Goodfellow, et al. "Generative Adversarial Networks." _Advances in Neural Information Processing Systems_ (2014). [link](https://arxiv.org/abs/1406.2661)
[3] Kingma, Welling. "Auto-enconding Variational Bayes." _The International Conference on Learning Representations_ (2014). [link](https://arxiv.org/abs/1312.6114)
[4] Mescheder, et al. "Adversarial Variational Bayes: Unifying Variational Autoencoders and Generative Adversarial Networks." _arXiv preprint arXiv:1701.04722_ (2017). [link](https://arxiv.org/abs/1701.04722)




