---
title: "Diffusion and Flow Matching"
date: 2026-10-08
description: "Understanding both as probability transport methods"
---

**Author's note**

This is a general overview for the author's own use. 

Supplemental details and relevant basic / advanced topics (annealed langevin sampling, SMC, Feynman-Kac Correctors, etc) will be posted later. For readers interested in stochastic interpolants, this study guide ([Ziseok Lee, 2026](https://ziseoklee.github.io/assets/study/2026-GMT-I/)) provides a comprehensive overview.

**Preliminaries and notation**

The objective is straightforward: transport a simple source distribution into the target data distribution.

\(p_{\mathrm{noise}}\): source distribution, usually Gaussian.  
\(p_{\mathrm{data}}\): target data distribution.  
\(p_t\): intermediate density during generation.  
\(v_t(x)\): deterministic velocity field.  
\(s_t(x)=\nabla_x\log p_t(x)\): score of the intermediate density.  
\(W_t\): standard Brownian motion.

Use \(t\in[0,1]\) for generation, running from noise to data. When discussing forward diffusion, use \(\tau\in[0,1]\), running from data to noise, with \(t=1-\tau\).

Exact statements assume sufficient regularity, integrability, appropriate boundary behavior, and well-posed dynamics. Scores require positive differentiable densities. Singular endpoint distributions may require smoothing or a limiting interpretation.

**Generative modeling as probability transport**

A deterministic generative process starts from a random latent variable and evolves through an ODE:

$$
X_0\sim p_{\mathrm{noise}},
\qquad
\frac{dX_t}{dt}=v_t(X_t).
$$

The solution defines a flow map \(\Phi_t\):

$$
X_t=\Phi_t(X_0).
$$

The desired endpoint condition is

$$
(\Phi_1)_{\#} p_{\mathrm{noise}}=p_{\mathrm{data}}.
$$

The pushforward notation means that applying \(\Phi_1\) to source samples produces target-distributed samples.

The velocity field describes instantaneous motion; the flow map is obtained by integrating that field.

Diffusion and flow matching can both be studied through this transport perspective. We focus on their ODE and SDE formulations, leaving aside diffusion’s hierarchical latent-variable and variational-autoencoder interpretation. ([Luo, 2022](https://arxiv.org/abs/2208.11970)) That interpretation is valid, but the differential-equation perspective makes the relationship between the methods especially direct.

**Continuity equation**

If \(p_t(x)\) is a density and \(J_t(x)\) is its probability current, conservation of probability gives

$$
\boxed{
\partial_t p_t+\nabla\cdot J_t=0.
}
$$

For the deterministic dynamics

$$
\frac{dX_t}{dt}=v_t(X_t),
$$

the current is

$$
J_t=p_tv_t.
$$

Therefore,

$$
\boxed{
\partial_t p_t+\nabla\cdot(p_tv_t)=0.
}
$$

The density changes because probability moves between regions. Under suitable boundary conditions, its total integral remains one.

**Fokker–Planck equation**

Consider an Itô SDE with scalar, spatially constant noise amplitude:

$$
dX_t=b_t(X_t)\,dt+h_t\,dW_t.
$$

Its density satisfies

$$
\boxed{
\partial_t p_t=
-\nabla\cdot(p_tb_t)
+\frac{h_t^2}{2}\Delta p_t.
}
$$

This is the Fokker–Planck equation. It also has continuity-equation form:

$$
J_t=
p_tb_t-\frac{h_t^2}{2}\nabla p_t.
$$

Using

$$
s_t=\nabla\log p_t,
\qquad
\nabla p_t=p_ts_t,
$$

we obtain

$$
J_t=
p_t\left(b_t-\frac{h_t^2}{2}s_t\right).
$$

The corresponding probability-transport velocity is therefore

$$
\boxed{
v_t=b_t-\frac{h_t^2}{2}s_t.
}
$$

The SDE drift \(b_t\) and the probability-transport velocity \(v_t\) have different roles. Brownian sample paths generally have no ordinary instantaneous velocity; the expression above describes the evolution of their distribution. ([Song et al., 2021](https://arxiv.org/abs/2011.13456))

**Diffusion: prescribe a forward noising process**

Write the forward diffusion as

$$
dX_\tau=
f_\tau(X_\tau)\,d\tau
+
g_\tau\,dW_\tau,
\qquad
X_0\sim p_{\mathrm{data}}.
$$

Let \(q_\tau\) denote its density. The forward process adds noise; the reverse process generates data.

A canonical example is the variance-preserving, or VP, diffusion:

$$
\boxed{
dX_\tau=
-\frac{\beta(\tau)}{2}X_\tau\,d\tau
+
\sqrt{\beta(\tau)}\,dW_\tau.
}
$$

Its conditional distribution is

$$
X_\tau\mid X_0=x_0
\sim
\mathcal N\left(a_\tau x_0,\sigma_\tau^2I\right),
$$

where

$$
a_\tau=
\exp\left(
-\frac12\int_0^\tau\beta(r)\,dr
\right),
\qquad
\sigma_\tau^2=1-a_\tau^2.
$$

As \(a_\tau\) becomes small, the distribution approaches a standard Gaussian. For a finite schedule, the terminal distribution \(q_1\) is generally only approximately Gaussian.

**Anderson’s reverse-time SDE**

Switch to the increasing generation clock

$$
t=1-\tau,
\qquad
p_t=q_{1-t},
\qquad
s_t=\nabla\log p_t.
$$

Anderson’s reverse-time formula gives

$$
\boxed{
dY_t=
\left[
-f_{1-t}(Y_t)
+
g_{1-t}^2s_t(Y_t)
\right]dt
+
g_{1-t}\,d\overline W_t.
}
$$

Starting from the exact terminal distribution,

$$
Y_0\sim q_1,
$$

the reverse process satisfies

$$
\operatorname{Law}(Y_t)=q_{1-t},
\qquad
Y_1\sim p_{\mathrm{data}}.
$$

The score correction is essential: fresh Brownian noise continues to spread probability during reverse simulation. Merely negating the forward drift does not reverse that spreading. ([Anderson, 1982](https://eva.fcien.udelar.edu.uy/pluginfile.php/166428/mod_resource/content/1/1982_andersonREVERSE-TIME%20DIFFUSION%20EQUATION%20MODELS.pdf))

This recovers the original **distribution**. A fresh reverse simulation does not generally recover the particular clean sample that produced a given noisy observation.

**Connection to DDPM**

DDPM uses discrete forward transitions

$$
q(x_k\mid x_{k-1})=
\mathcal N\left(
\sqrt{1-\beta_k}\,x_{k-1},
\beta_k I
\right).
$$

For small steps with

$$
\beta_k\approx\beta(\tau_k)\Delta\tau,
$$

this process approaches the VP SDE.

DDPM’s learned reverse Gaussian transitions approximate the corresponding reverse dynamics. At finite step sizes, the true reverse conditional is generally not Gaussian. Exact continuous-time reversal therefore does not imply exact recovery by a finite-step DDPM sampler. ([Ho et al., 2020](https://arxiv.org/abs/2006.11239))

**Probability-flow ODE**

The same diffusion marginals can be generated deterministically.

Reversing the forward Fokker–Planck equation gives

$$
\partial_t p_t=
\nabla\cdot(p_tf_{1-t})-
\frac{g_{1-t}^2}{2}\Delta p_t.
$$

Since \(\nabla p_t=p_ts_t\),

$$
\partial_t p_t=
-\nabla\cdot
\left[
p_t
\left(
-f_{1-t}
+
\frac{g_{1-t}^2}{2}s_t
\right)
\right].
$$

This is the continuity equation for

$$
\boxed{
\frac{dY_t}{dt}=
-f_{1-t}(Y_t)
+
\frac{g_{1-t}^2}{2}s_t(Y_t).
}
$$

Thus, in generation time:

$$
\text{Reverse SDE drift}=
-f_{1-t}+g_{1-t}^2s_t,
$$

$$
\text{Probability-flow ODE velocity}=
-f_{1-t}+\frac12g_{1-t}^2s_t.
$$

Removing the stochastic noise also requires reducing the score coefficient from one to one-half.

With exact fields, correct initialization, and well-posed evolution, the SDE and ODE share every time marginal. Their trajectories, transition probabilities, and endpoint pairings generally differ.

**Connection to deterministic DDIM**

For the VP coefficients defined above, a deterministic DDIM step from \(\tau\) to \(\tau-h\) is

$$
\widehat x_0=
\frac{
x_\tau-\sigma_\tau\epsilon_\theta(x_\tau,\tau)
}{
a_\tau
},
$$

$$
x_{\tau-h}=
a_{\tau-h}\widehat x_0
+
\sigma_{\tau-h}\epsilon_\theta(x_\tau,\tau).
$$

Expanding one step to first order yields

$$
\frac{dx_\tau}{d\tau}=\frac{a_\tau'}{a_\tau}x_\tau+\left(\sigma_\tau'-
\frac{a_\tau'}{a_\tau}\sigma_\tau
\right)
\epsilon_\theta(x_\tau,\tau).
$$

For the VP schedule,

$$
\frac{a_\tau'}{a_\tau}=
-\frac{\beta(\tau)}2,
$$

$$
\sigma_\tau'-\frac{a_\tau'}{a_\tau}\sigma_\tau=
\frac{\beta(\tau)}{2\sigma_\tau}.
$$

With independent \(\epsilon\sim\mathcal N(0,I)\), the optimal noise predictor satisfies

$$
\epsilon_*(x,\tau)=
\mathbb E\left[
\epsilon
\mid
a_\tau X_0+\sigma_\tau\epsilon=x
\right]=
-\sigma_\tau\nabla\log q_\tau(x).
$$

Substitution gives

$$
\boxed{
\frac{dx_\tau}{d\tau}=
-\frac{\beta(\tau)}2x_\tau-
\frac{\beta(\tau)}2\nabla\log q_\tau(x_\tau).
}
$$

This is the VP probability-flow ODE, expressed in the noising clock and integrated toward decreasing \(\tau\) for generation.

To clarify, **deterministic DDIM has the corresponding probability-flow ODE as its continuous-time limit**. Finite DDIM steps are numerical approximations, and arbitrary ODE discretizations need not equal DDIM updates. ([Song, Meng, and Ermon, 2020](https://arxiv.org/abs/2010.02502))

Detailed training parameterizations, schedule choices, loss weights, and practical discrete samplers will be discussed in a separate post.

**Flow matching: construct the transport directly**

Instead of prescribing a forward diffusion, choose an interpolation between source and target samples.

Sample an endpoint pair

$$
(Z,D)\sim\pi,
\qquad
Z\sim p_{\mathrm{noise}},
\qquad
D\sim p_{\mathrm{data}}.
$$

The coupling \(\pi\) is a joint distribution with the required endpoint marginals. Independent sampling is one possible choice.

For a linear interpolation, define

$$
I_t=(1-t)Z+tD,
\qquad
U=D-Z.
$$

Then

$$
\dot I_t=U,
\qquad
p_t=\operatorname{Law}(I_t).
$$

The endpoints are correct by construction:

$$
p_0=p_{\mathrm{noise}},
\qquad
p_1=p_{\mathrm{data}}.
$$

Different sampled interpolations can pass through the same location at the same time with different velocities. Their aggregate probability current is represented by

$$
\boxed{
v_t(x)=
\mathbb E[U\mid I_t=x].
}
$$

This is a conditional average of interpolation velocities, weighted by the endpoint pairs compatible with the observed position and time. It is defined \(p_t\)-almost everywhere.

For a general differentiable interpolation, the corresponding formula is

$$
v_t(x)=
\mathbb E[\dot I_t\mid I_t=x].
$$
For more details, read [Lipman et al., 2022](https://arxiv.org/abs/2210.02747).

**Why the conditional mean satisfies the continuity equation**

Let \(\varphi\) be any smooth, compactly supported test function.

Differentiate its expectation along the interpolation:

$$
\frac{d}{dt}\mathbb E[\varphi(I_t)]=
\mathbb E[\nabla\varphi(I_t)\cdot U].
$$

Apply the law of iterated expectations:

$$
\mathbb E[\nabla\varphi(I_t)\cdot U]=
\mathbb E\left[
\mathbb E[
\nabla\varphi(I_t)\cdot U
\mid I_t
]
\right].
$$

Because \(\nabla\varphi(I_t)\) is determined by \(I_t\),

$$
=\mathbb E\left[
\nabla\varphi(I_t)
\cdot
\mathbb E[U\mid I_t]
\right].
$$

Using the definition of \(v_t\),

$$
=\int
\nabla\varphi(x)\cdot v_t(x)p_t(x)\,dx.
$$

Integration by parts gives

$$
=-\int
\varphi(x)\nabla\cdot(p_t(x)v_t(x))\,dx.
$$

Meanwhile,

$$
\frac{d}{dt}\mathbb E[\varphi(I_t)]=
\int
\varphi(x)\partial_t p_t(x)\,dx.
$$

Therefore,

$$
\int
\varphi(x)
\left[
\partial_t p_t(x)+
\nabla\cdot(p_t(x)v_t(x))
\right]dx=
0.
$$

Because this holds for every such test function,

$$
\boxed{
\partial_t p_t+\nabla\cdot(p_tv_t)=0
}
$$

in the weak sense.

With the required uniqueness properties, solving

$$
\frac{dY_t}{dt}=v_t(Y_t),
\qquad
Y_0\sim p_{\mathrm{noise}},
$$

therefore produces

$$
\operatorname{Law}(Y_t)=p_t.
$$

The generated trajectories need not follow the sampled training lines or reproduce the original coupling \(\pi\). The guarantee concerns their time-dependent marginal distributions.

Straight training interpolations do not necessarily produce straight ODE trajectories or an optimal-transport map.

Also, a density path does not uniquely determine its velocity field. Adding any field \(w_t\) satisfying

$$
\nabla\cdot(p_tw_t)=0
$$

preserves the same continuity equation.

**MSE learns the conditional mean**

The transport argument comes first. We can now choose a learning objective that estimates its conditional-mean velocity.

For a neural field \(v_\theta(x,t)\), use

$$
\boxed{
\mathcal L(\theta)=
\mathbb E_{t,(Z,D)}
\left[
\left\|
v_\theta((1-t)Z+tD,t)-
(D-Z)
\right\|^2
\right],
}
$$

where \(t\sim\mathcal U(0,1)\) is independent of \((Z,D)\sim\pi\).

At a fixed time,

$$
\begin{aligned}
\mathbb E\|v_\theta(I_t,t)-U\|^2
&=
\mathbb E\|v_\theta(I_t,t)-v_t(I_t)\|^2\\
&\quad+
\mathbb E\|U-v_t(I_t)\|^2.
\end{aligned}
$$

The second term is independent of \(\theta\). Thus, with sufficient capacity and ideal optimization, squared-error regression recovers

$$
v_t(x)=\mathbb E[U\mid I_t=x].
$$

The reasoning is:

$$
\text{Probability current}
\;\longrightarrow\;
\text{Conditional mean}
\;\longrightarrow\;
\text{Regression objective}.
$$

MSE is a convenient realization of conditional-mean estimation. It is not needed to establish the transport identity. A different objective must preserve the same population target; an arbitrary replacement loss need not do so.

Training requires sampling a time and endpoint pair, constructing \(I_t\) and \(U\), and performing the regression update. No ODE simulation is needed to obtain those training examples.

The minimum loss can remain positive because the conditional velocities can have nonzero variance.

**Randomness can come entirely from the initial sample**

For a fixed latent \(Z=z\), a well-posed ODE gives one output:

$$
Y=\Phi_1(z).
$$

Across random initial samples,

$$
\Pr(\Phi_1(Z)\in A)=
\int
\mathbf 1_{\{\Phi_1(z)\in A\}}
p_{\mathrm{noise}}(z)\,dz.
$$

This distribution can be highly multimodal despite every trajectory being deterministic.

For conditional generation with a condition \(c\),

$$
Z\sim p_{\mathrm{noise}},
\qquad
Y=\Phi_1^c(Z).
$$

The same condition can produce many outputs by changing the latent sample. For a fixed condition and fixed full latent state, the output is deterministic.

A regular ODE does not literally split one initial state into several trajectories.

**Adding stochasticity while preserving the probability path**

Suppose \(v_t\) generates \(p_t\), and its score

$$
s_t=\nabla\log p_t
$$

is available.

Choose a time-dependent scalar noise amplitude \(h_t\). Consider

$$
\boxed{
dY_t=
\left[
v_t(Y_t)
+
\frac{h_t^2}{2}s_t(Y_t)
\right]dt
+
h_t\,dW_t.
}
$$

Substituting \(p_t\) into its Fokker–Planck equation gives

$$
\partial_t p_t=
-\nabla\cdot
\left[
p_t\left(v_t+\frac{h_t^2}{2}s_t\right)
\right]
+
\frac{h_t^2}{2}\Delta p_t.
$$

Using \(p_ts_t=\nabla p_t\),

$$
\partial_t p_t=
-\nabla\cdot(p_tv_t)-
\frac{h_t^2}{2}\Delta p_t
+
\frac{h_t^2}{2}\Delta p_t.
$$

Hence,

$$
\boxed{
\partial_t p_t=-\nabla\cdot(p_tv_t).
}
$$

The score drift cancels the additional diffusion current. With the same initial distribution and well-posed evolution, the stochastic sampler retains the deterministic flow’s marginals.

Simply adding noise without the score correction generally changes those marginals. ([Albergo, Boffi, and Vanden-Eijnden, 2023](https://arxiv.org/abs/2303.08797))

**Recovering diffusion from the stochastic flow**

Insert the generation-direction probability-flow velocity

$$
v_t=-f_{1-t}
+
\frac{g_{1-t}^2}{2}s_t.
$$

Choose

$$
h_t=g_{1-t}.
$$

The stochastic sampler’s drift becomes

$$
v_t+\frac{h_t^2}{2}s_t=
-f_{1-t}+g_{1-t}^2s_t.
$$

Therefore,

$$
\boxed{
dY_t=
\left[
-f_{1-t}(Y_t)
+
g_{1-t}^2s_t(Y_t)
\right]dt
+
g_{1-t}\,dW_t,
}
$$

which is Anderson’s reverse SDE.

Choosing \(h_t=0\) instead recovers the probability-flow ODE.

**Flow matching can learn the diffusion probability-flow velocity**

Take independent samples

$$
D\sim p_{\mathrm{data}},
\qquad
Z\sim\mathcal N(0,I),
$$

and choose the interpolation

$$
I_t=A_tD+B_tZ,
\qquad
A_t=a_{1-t},
\qquad
B_t=\sigma_{1-t}.
$$

This construction has the reversed diffusion marginals. On the interior where the coefficients are nonzero,

$$
s_t(x)=
-\frac{\mathbb E[Z\mid I_t=x]}{B_t}.
$$

The conditional-mean velocity is

$$
v_t(x)=
\dot A_t\,\mathbb E[D\mid I_t=x]
+
\dot B_t\,\mathbb E[Z\mid I_t=x].
$$

Because

$$
x=
A_t\mathbb E[D\mid I_t=x]
+
B_t\mathbb E[Z\mid I_t=x],
$$

we obtain

$$
v_t(x)=
\frac{\dot A_t}{A_t}x
+
\left(
\frac{\dot A_t}{A_t}B_t^2-
B_t\dot B_t
\right)s_t(x).
$$

For the reversed VP schedule,

$$
\boxed{
v_t(x)=
\frac{\beta(1-t)}2
\left(x+s_t(x)\right).
}
$$

This is exactly the generation-direction VP probability-flow velocity.

Thus, selecting the diffusion interpolation lets flow matching learn the corresponding deterministic transport. Selecting another interpolation generally changes the intermediate distributions.

For a finite VP schedule, the initial marginal of this particular interpolation is the exact diffusion terminal distribution \(q_1\), which is generally only approximately standard Gaussian.

**Recovering a score from a linear Gaussian flow**

Obtaining the score is an additional requirement for stochastic sampling. Some paths provide an explicit conversion.

For the independent linear Gaussian interpolation

$$
I_t=(1-t)Z+tD,
\qquad
Z\sim\mathcal N(0,I),
\qquad
Z\perp D,
$$

the conditional density is

$$
I_t\mid D=d
\sim
\mathcal N\left(td,(1-t)^2I\right).
$$

Differentiating the marginal density and applying Bayes’ rule gives

$$
s_t(x)=
\mathbb E\left[
-\frac{x-tD}{(1-t)^2}
\;\middle|\;
I_t=x
\right].
$$

Since \(I_t-tD=(1-t)Z\),

$$
s_t(x)=
-\frac{\mathbb E[Z\mid I_t=x]}{1-t}.
$$

Also,

$$
I_t=Z+t(D-Z).
$$

Taking conditional expectations,

$$
x=
\mathbb E[Z\mid I_t=x]
+
t\,v_t(x).
$$

Therefore,

$$
\boxed{
s_t(x)=
\frac{t\,v_t(x)-x}{1-t},
\qquad
0\le t<1.
}
$$

This conversion relies on the independent Gaussian endpoint noise and the chosen linear interpolation. It does not apply unchanged to arbitrary couplings or paths.

With learned fields and numerical solvers, marginal equality is approximate. In particular, converting an imperfect \(v_\theta\) using this formula does not automatically produce the exact score of the density generated by its ODE.

Exact preservation of a learned ODE’s distribution requires that distribution’s own score. The endpoint \(t=1\) also requires separate limiting or numerical treatment.