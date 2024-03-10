# Diffusion-faster-conv

In the paper "Denoising Diffusion Probabilistic Models", the authors obtain a variational bound for the training loss which they optimize through gradient descent in order to train a neural network. After a series of tricks and rearrangements, the authors obtain the following variational bound:

```math
\mathbb{E}[\log p_{\theta}(x_{0})] \leq \mathbb{E}\left[ \frac{\beta_{t}^{2}}{2\sigma^{2}_{t}\alpha_{t}(1-\overline{\alpha}_{t})} \|\varepsilon - \varepsilon_{\theta}(\sqrt{\overline{\alpha}_{t}}x_{0} + \sqrt{1-\overline{\alpha}_{t}}\varepsilon, t)\|^{2} \right]
```

where $\(\varepsilon\)$ is the Gaussian noise added in the forward process and $\(\varepsilon_{\theta}\)$ the predicted noise generated by the neural network. They argue that it is possible to drop the weight on the objective since this leads to a weighted variational bound that allows the network to focus on the more difficult denoising tasks at larger timesteps \(t\), so they obtain the simplified objective

```math
\mathbb{E}_{t,x_{0},\varepsilon}\left[ \|\varepsilon - \varepsilon_{\theta}(\sqrt{\overline{\alpha}_{t}}x_{0} + \sqrt{1 - \overline{\alpha}_{t}}\varepsilon, t)\|^{2} \right].
```

The goal for this project is to follow the ideas in the paper "Towards Faster Non-Asymptotic convergence for Diffusion-Based Generative Models" and adapt it to DDPM in order to then implement their proposed new objective in code, which (to my knowledge) has not been done in the paper or has been done in other papers. The main difficulty of the implementation is to decide which factors on the extra objective (see below) to keep in order to allow the training to learn efficiently.

----- Theoretical basis

As shown in the paper, we see that the sampling objective admits the form:

```math
\mathbb{E}_{t,x_{0},\varepsilon}\left[ \|\frac{1}{\sqrt{1-\overline{\alpha_{t}}}}\varepsilon - \varepsilon_{\theta}(\sqrt{\overline{\alpha}_{t}}x_{0} + \sqrt{1 - \overline{\alpha}_{t}}\varepsilon, t)\|^{2} \right],
```

and this is the \(L^{2}\) estimator for \(-\frac{1}{\sqrt{1-\overline{\alpha}_{t}}}\varepsilon\). In this case, the updates follow

```math
\Phi_{t}(x) = \frac{1}{\sqrt{\alpha_{t}}}\left( x+\frac{1-\alpha_{t}}{2}\varepsilon_{t}^{\ast}(x) \right).
```

Now, if we knew the estimates to the minimizer(s) \(y_{t}^{\ast}\) of

```math
\mathbb{E}\left[ \|\varepsilon\|_{2}^{2}\left( \frac{1}{\sqrt{1-\overline{\alpha}_{t}}}\varepsilon + \varepsilon_{t}^{\ast}(x_{t}) + \frac{1}{1-\overline{\alpha}}\varepsilon\varepsilon^{T}\varepsilon_{t}^{\ast}(x_{t}) - y_{t}(x_{t})\|^{2}_{2} \right) \right],
```

then we can introduce the update rule

```math
\Phi_{t}(x) = \frac{1}{\sqrt{\alpha_{t}}}\left( x+\left( \frac{1-\alpha_{t}}{2} + \frac{(1-\alpha_{t})^{2}}{8(1-\overline{\alpha}_{t})} -\frac{(1-\alpha_{t})^{2}}{8}\|\varepsilon_{t}^{\ast}\|_{2}^{2} \right)\varepsilon_{t}^{\ast}(x) + \frac{(1-\alpha_{t})^{2}}{8}y^{\ast}_{t}(x) \right),
```

which the paper shows allows us to only $\(O\left( \frac{d^{3}}{\sqrt{\delta}} \right)\)$ in order to have the total variation between the predicted distribution and the data distribution to have $\(TV(q,p)\leq \delta\)$ for sufficiently small $\(\delta\)$, which improves the convergence rate from the previous $\(O\left( \frac{1}{\delta} \right)\)$. This is the improvement we attempt to implement in order to improve the accuracy with respect to the number of epochs.

Our current approach consists of training the extra objective through another network and passing the results onto the original network, which is not entirely efficient unless the networks can be trained parallel to each other. Currently working on finding a more efficient way to implement the results.

--------------------------
Try running ddpm_conditional.py
