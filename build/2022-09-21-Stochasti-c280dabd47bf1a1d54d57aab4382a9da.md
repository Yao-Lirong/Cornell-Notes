---
title: Stochastic Gradient Descent Improved
date: 2022-09-21
tags:
  - CS4787
---

## Minibatch SGD Running Time

Recall from last time, we have

$$
\mathbf{E}\left[f(w_T) - f^* \right]
\le exp(-\alpha \mu T) \; (f(w_0) - f^*) + \frac{\alpha \sigma^2 L} {2 \mu B}
$$

Similar to our proof of GD converges linearly on PL-condition functions, set our goal as **for a given margin $\epsilon \gt 0$, by running minibatch SGD $T$ times, we can output a prediction $\hat w$ s.t. $\mathbf E [f(\hat w) - f^*] \le \epsilon$**, so we can write

$$
exp(-\alpha \mu T) \; (f(w_0) - f^*) + \frac{\alpha \sigma^2 L} {2 \mu B} \le \epsilon
$$

To make it easier for ourselves, we instead find at least how many $T$ we need to run so that

$$
exp(-\alpha \mu T) \; (f(w_0) - f^*) \le \frac \epsilon 2 \\
\frac{\alpha \sigma^2 L} {2 \mu B} \le \frac \epsilon 2
$$

The first expression gives

$$
T \ge \frac 1 {\alpha \mu} \log(\frac {2(f(w_0) - f^*)} {\epsilon})
$$

The second expression gives

$$
\alpha \le \frac {B \mu \epsilon} {L \sigma^2}
$$

Recall we have another constraint on $\alpha$ that $\alpha L \le 1$, so

$$
\alpha \le \frac 1 L \min(\frac {B \mu \epsilon} {\sigma^2}, 1)
$$

This is actually a pretty interesting inequality. It says if we have a big batch size and low variance, so the batch gradient is representative of the global gradient, we can set $\alpha$ to be big. Replace the $\alpha$ in $T$ expression here

$$
T \ge \max(\frac {\sigma^2} {B \mu \epsilon}, 1) \; \frac L \mu \; \log(\frac {2(f(w_0) - f^*)} {\epsilon})
$$

Note $\frac L \mu = \kappa$ is the condition number. All the $T$ satisfies this condition will produce a small enough $\epsilon$, so the first / smallest one that satisfices this condition will simply be the floor. Therefore, the **number of step for minibatch SGD to converge** is

$$
T = \left\lceil \max(\frac {\sigma^2} {B \mu \epsilon}, 1) \; \kappa\log(\frac {2(f(w_0) - f^*)} {\epsilon}) \right\rceil
$$

Each step, we need $B$ time to compute the batch gradient (assuming $\mathcal O(1)$ time to calculate each sample's gradient. This time is usually $\mathcal O(d)$ but it depends, so we just ignore it for now). The **running time of minibatch SGD** to reach $\epsilon$ precision is

$$
\mathcal O \left(\max(\frac {\sigma^2} {\mu \epsilon}, B) \; \kappa \log(\frac {2(f(w_0) - f^*)} {\epsilon})\right)
$$

For most of the time, $\frac {\sigma^2} {\mu \epsilon}$ is the bigger part, and we ignore the $\log$ value, so running time of SGD is in short

$$
\mathcal O \left(\frac {\sigma^2} {\mu \epsilon} \kappa \right)
$$

## Comparing to GD

Compare the full version of minibatch SGD running time with GD, we see that the $n$ is basically replaced with the $\max(\frac {\sigma^2} {\mu \epsilon}, B)$

$$
\mathcal O \left(n \kappa \log(\frac {2(f(w_0) - f^*)} {\epsilon})\right)
$$

So there are some situations more suitable to minibatch SGD we can immediately see:

- huge $n$: GD runs too slowly to calculate gradient on the whole dataset
- not too big precision $\epsilon$ needed

In what situation does GD runs better? Maybe when we have too large $\sigma^2$, so the average batch gradient is not representative of the whole dataset? Is this true though? Think when we have a dataset of a very large variance (maybe all the data points are uniformly distributed on some space), so randomly sample a data point $i$, $\nabla f_i$ will have almost nothing to do with $\mathbf E [\nabla f_i]$. If this is the case, even GD cannot learn much and this learning / optimization problem just doesn't hold itself.

## Another Advantage of SGD - Hardware

Minibatch SGD runs significantly faster than GD not only because of the decrease in computation cost, but also because when we have too large an $n$, we will blow up the memory and have to do a lot of memory swap and other overheads.

Minibatch SGD is faster than vanilla SGD because minibatch can utilize parallelism of the hardware.

## Minibatch SGD in Practice

When analyzing convergence and running time, we drew the batch with replacement. However, in practice, drawing the batch without replacement gives us better result.

The best practice is **random reshuffle**, where we **randomly shuffle the whole dataset and divide it into several minibatches**. So there is no duplicate elements across all the batches. This method will make our previous convergence proof absolutely not hold, because everything the previous proof built on was we assumed each batch is drawn independently, so all those expectations only depend on $w_t$. However, now the expectation will depend on a whole lot of stuff - what was the previous batch, the order they were drawn, ...

The term related to random shuffling is **epoch**, which means 1 pass through the whole dataset.

We have other methods for doing minibatch SGD:

1. without replacement within batch level, so there can be duplicates across different batches
2. shuffle-once: only shuffle the dataset once before start training, so each epoch has the same order

## Improve: Non-Constant Learning Rate on PL Function

Now instead of having a constant learning rate $\alpha$, we will have an adaptive (usually diminishing) learning rate specific for each step: $\alpha_t$

$$
\rho_{t+1}
\le (1 - \alpha_t \mu)   \rho_t + \frac{\alpha_t^2 \sigma^2 L}{2B}
$$

Think of $\rho_{t+1}$ as a function of $\alpha_t$

$$
\rho_{t+1} = g(\alpha_t)
= (1 - \alpha_t \mu)   \rho_t + \frac{\alpha_t^2 \sigma^2 L}{2B}
$$

Since we want $\rho_{t+1}$ to be as small as possible, we take derivative of $g$ with respect to $\alpha_t$, we solve $g'(\alpha_t) = 0$, so see **value of $\alpha$ when $\rho_{t+1}$ reaches its minimum**.

$$
\alpha_t = \frac {\rho_t \mu B} {\sigma^2 L}
$$

Substitute the $\alpha_t$ into the above inequality and take its inverse

$$
\begin{align}
\rho_{t+1} &\le \rho_t - \frac {\mu^2 B} {2 \sigma^2 L} \rho_t^2 \\
\frac 1 {\rho_{t+1}} & \ge \frac 1 {\rho_t - \frac {\mu^2 B} {2 \sigma^2 L} \rho_t^2} \\
&= \frac 1 {\rho_t} (1  - \frac {\mu^2 B} {2 \sigma^2 L} \rho_t)^{-1} \\
&\ge \frac 1 {\rho_t} (1  + \frac {\mu^2 B} {2 \sigma^2 L} \rho_t) \\
&= \frac 1 {\rho_t} + \frac {\mu^2 B} {2 \sigma^2 L} \\
\end{align}
$$

Then we use the fact $\forall x \lt 1, \; (1-x)^{-1} \ge 1+x$. We can say $\frac {\mu^2 B} {2 \sigma^2 L} \rho_t \lt 1$ because we can make $\rho_t$ really small. To do this, at the beginning of the training stage, we first run minibatch SGD at a constant step size for several epochs. In these constant $\alpha$ runs, we don't care about this property, so nothing is violated. At the same time, we can make $\rho_t$ relatively small so we are ready to switch to a diminishing step size while making sure $\frac {\mu^2 B} {2 \sigma^2 L} \rho_t \lt 1$ holds.

$$
\begin{align}
\frac 1 {\rho_{t+1}} & \ge \frac 1 {\rho_t} (1  - \frac {\mu^2 B} {2 \sigma^2 L} \rho_t)^{-1} \\
&\ge \frac 1 {\rho_t} (1  + \frac {\mu^2 B} {2 \sigma^2 L} \rho_t) \\
&= \frac 1 {\rho_t} + \frac {\mu^2 B} {2 \sigma^2 L} \\
\end{align}
$$

Now look at a $T$ iterations:

$$
\begin{align}
\frac 1 {\rho_{T}}
&\ge \frac 1 {\rho_0} + T \frac {\mu^2 B} {2 \sigma^2 L} \\
\rho_T = \mathbf{E}\left[f(w_t) - f^* \right] &\le (\frac 1 {f(w_0) - f^*} + T \frac {\mu^2 B} {2 \sigma^2 L})^{-1}\\
&= \mathcal O(\frac {\sigma^2 L} {T \mu^2 B})\\
&= \mathcal O(\frac 1 T)
\end{align}
$$

Now we have sort of a value of $\rho_t$, or at least we now know how it changes. Recall we solved what value of $\alpha$ can make the next iteration as good as possible.

$$
\alpha_t = \frac {\rho_t \mu B} {\sigma^2 L} \propto \rho_t \propto \frac 1 T
$$

Therefore, we conclude that **we should make $\alpha_t$ changes proportional to step's inverse**.
