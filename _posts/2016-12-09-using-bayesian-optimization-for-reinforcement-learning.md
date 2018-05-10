---
layout: post
current: post
cover: assets/images/bayesian-optimization/artificial-intelligence.jpg
navigation: true
title: Using Bayesian Optimization for Reinforcement Learning
date: 2016-12-09 8:00:00
tags: [tech]
class: post-template
subclass: 'post'
author: eric
permalink: /:categories/:year/:month/:day/:title/
---

In this post, we will show you how [Bayesian optimization](https://static.sigopt.com/2d66b84dcdbbd7fffad087f58b67a585eb89444c/pdf/SigOpt_Bayesian_Optimization_Primer.pdf) was able to dramatically improve the performance of a reinforcement learning algorithm in an AI challenge. We’ll provide background information, detailed examples, code, and references.

### Background

Reinforcement learning is a field of machine learning in which a software agent is taught to maximize its acquisition of rewards in a given environment. Observations of the state of the environment are used by the agent to make decisions about which action it should perform in order to maximize its reward.

Reinforcement learning has recently garnered significant news coverage as a result of innovations in deep Q-networks (DQNs) by DeepMind Technologies. Through deep reinforcement learning, DeepMind was able to teach computers to [play Atari games better than humans](http://www.nature.com/nature/journal/v518/n7540/full/nature14236.html), as well as [defeat one of the top Go players in the world](https://en.wikipedia.org/wiki/AlphaGo).

As noted in [DeepMind’s paper](https://storage.googleapis.com/deepmind-data/assets/papers/DeepMindNature14236Paper.pdf#page=10), an “informal search” for hyperparameter values was conducted in order to avoid the high computational cost of performing grid search. Because the complexity of grid search grows exponentially with the number of parameters being tuned, experts often spend considerable time and resources performing these “informal searches.” This may lead to [suboptimal performance](https://www.nervanasys.com/sigopt/), or can lead to the systems not being tuned at all. Bayesian optimization represents a way to efficiently optimize these high dimensional, time consuming, and expensive problems.

We will demonstrate the power of hyperparameter optimization by using [SigOpt](https://start.sigopt.com/)’s ensemble of state-of-the-art Bayesian optimization techniques to tune a DQN. We’ll show how this approach finds better hyperparameter values much faster than traditional methods such as grid and random search, without requiring expert time spent doing “informal” hand tuning of parameters. The DQN under consideration will be used to solve a classic learning control problem called the Cart-Pole problem <sup class="footnote-ref"><a href="#fn1" id="fnref1">[1]</a></sup>. In this problem, a pole must be balanced upright on a cart for as long as possible. To simulate this environment, we will use [OpenAI’s Gym library](https://gym.openai.com/).

<img src="/assets/images/bayesian-optimization/cart-pole.gif" alt="Figure 1" style="max-height: 500px;"/>
_‍**Figure 1:** A rendered episode from the OpenAI Gym’s Cart-Pole environment_

The OpenAI Gym provides a common interface to various reinforcement learning environments; the code written for this post ([available on Github](https://github.com/sigopt/sigopt-examples/tree/master/reinforcement-learning)) can be easily modified to solve other learning control problems from the Gym’s environments.

### The Environment

In OpenAI’s simulation of the cart-pole problem, the software agent controls the movement of the cart, earning a **reward** of +1 for each timestep until the terminating step. A terminating step occurs when the pole is more than 15 degrees from vertical or if the cart has moved more than 2.4 units from the center. The agent receives 4 continuous values that make up the **state** of the environment at each timestep: the position of the cart on the track, the angle of the pole, the cart velocity, and the rate of change of the angle. The agent’s only possible **actions** at each timestep are to push the cart to the left or right by applying a force of either -1 or +1, respectively. A series of states and actions, ending in a terminating state, is known as an **episode**. The agent will have no prior concept about the meaning of the values that represent these states and actions.

### Q-learning

Q-learning is a reinforcement learning technique that develops an action-value function (also known as the Q-function) that returns an expected utility of an action given a current state. Thus, the policy of the agent is to take the action with the highest expected utility.

Assume there exists an all-knowing Q-function that always selects the best action for a given state. Through Q-learning, we construct an approximation of this all-knowing function by continually updating the approximation using the results of previously attempted actions. The Q-learning algorithm updates the Q-function iteratively, as is explained below; initial Q-values are arbitrarily selected. An existing expected utility is updated when given new information using the following algorithm<sup class="footnote-ref"><a href="#fn2" id="fnref2">[2]</a></sup>

$$ Q_{t+1}(s_t, a_t) = Q_t(s_t, a_t) + \alpha(r_{t+1} + \gamma \max_a( Q_t(s_{t+1}, a)) - Q_t(s_t, a_t)). $$

* $a_t$ is the action executed in the state $s_t$.
* $s_{t+1}$ is the new state observed. In a deterministic environment<sup class="footnote-ref"><a href="#fn3" id="fnref3">[3]</a></sup>, it is a function of $s_t$ and $a_t$.
* $r_{t+1}$ is the immediate reward gained. It is a function of $s_t$, $a_t$, and $s_{t+1}$.
* $\alpha$ is the constant learning rate; how much the new information is weighted relative to the old information.
* $\gamma$ is the constant discount factor that determines how much long-term rewards should be valued.

In its simplest form, the Q-function can be implemented as a table mapping all possible combinations of states and actions to expected utility values. Since this is infeasible in environments with large or continuous action and observation spaces, we use a neural net to approximate this lookup table. As the agent continues act within the environment, the estimated Q-function is updated to better approximate the true Q-function via [backpropagation](https://en.wikipedia.org/wiki/Backpropagation).

### The Objective Metric

To properly tune the hyperparameters of our DQN, we have to select an appropriate objective metric value for SigOpt to optimize. While we are primarily concerned with maximizing the agent’s reward acquisition, we must also consider the DQN’s stability and efficiency. To ensure our agent’s training is efficient, we will train the DQN over the course of only 350 episodes and record the total reward accumulated for each episode. We use a rolling average of the reward for each set of 100 consecutive episodes (episodes 1 to 100, 2 to 101, etc.) and take the maximum for our objective metric. This helps stabilize the agent’s learning while also giving a robust metric for the overall quality of the agent with respect to the reward.

### Tunable Parameters of Reinforcement Learning Via Deep Q-Networks

While there are many tunable hyperparameters in the realm of reinforcement learning and deep Q-networks<sup class="footnote-ref"><a href="#fn4" id="fnref4">[4]</a></sup>, for this blog post the following 7 parameters<sup class="footnote-ref"><a href="#fn5" id="fnref5">[5]</a></sup> were selected:

**minibatch_size:** The number of training cases used to update the Q-network at each training step. These training cases, or minibatches, are randomly selected from the agent’s replay memory. In our implementation, the replay memory contains the last 1,000,000 transitions in the environment.

**epsilon_decay_steps:** The number of episodes required for the initial $\epsilon$ value to linearly decay until it reaches its end value. $\epsilon$ is the probability that our agent takes a random action, which decreases over time to balance exploration and exploitation. The upper bound for this parameter depends on the total number of episodes run. Initially, $\epsilon$ is 1, and it will decrease until it is 0.1, as suggested in DeepMind’s paper.

**hidden_multiplier:** Determines the number of nodes in the hidden layers of the Q-network. We set the number of nodes by multiplying this value by the size of the observation space. We formulated this parameter in this way to make it easier to switch to environments with different observation spaces.

**initial_weight_stddev** and **intial_bias_stddev:** Both the Q-network’s weights and biases are randomly initialized from normal distributions with a mean of 0. The standard deviations of these distributions affect the rate of convergence of the network.

**learning_rate:** Regulates the speed and accuracy of the Q-network by controlling the rate at which the weights of the network are updated. We look at this parameter on the logarithmic scale. This is equivalent to $\alpha$ in the Q-learning formula.

**discount_factor:** Determines the importance of future rewards to the agent. A value closer to zero will place more importance on short-term rewards, and a value closer to 1 will place more importance on long-term rewards. This is equivalent to $\gamma$ in the Q-learning formula.

Other good hyperparameters to consider tuning are the minimum epsilon value, the replay memory size, and the number of episodes of pure exploration (**\_final_epsilon**, **\_replay_memory_size**, and **\_episodes_pure_exploration** in the Agent class).

### The Code

[The code is available on Github](https://github.com/sigopt/sigopt-examples/tree/master/reinforcement-learning). The only dependencies required to run this example are NumPy, Gym, TensorFlow, and SigOpt. If you don’t have a SigOpt account, you can [sign up for a free SigOpt trial](https://sigopt.com/signup). SigOpt also has [a free plan available for academic users](https://start.sigopt.com/edu).

Running this code can be computationally intensive. If possible, try running this example on a CPU optimized machine. On a c4.4xlarge AWS instance, the entire example can take up to 5 hours to run. If you are running the code on an AWS instance, you can try using the SigOpt Community AMI that includes several pre-installed machine learning libraries.

### Results

We compared the results of SigOpt’s Bayesian optimization to two standard hyperparameter tuning methods: grid search and random search<sup class="footnote-ref"><a href="#fn6" id="fnref6">[6]</a></sup>. 128 objective evaluations for each optimization method were run, and we took the median of 5 runs.

|            | SigOpt | Random Search | Grid Search |
|------------|--------|---------------|-------------|
| Best Found | 847.19 | 569.93        | 194.62      |

_**Table 1:** SigOpt outperforms random search and grid search._

<img src="/assets/images/bayesian-optimization/best-trace.png" alt="Figure 2" style="max-height: 500px;"/>
_**Figure 2:** The best seen trace of hyperparameter tuning methods over the course of 128 objective evaluations._

SigOpt does dramatically better than random search and grid search! For fun, let’s look at the performance of the DQN with the best configuration found by SigOpt. Below are snapshots showing the progress of the sample network’s evolution over the 350 episodes.

<img src="/assets/images/bayesian-optimization/ep64.gif" alt="Figure 3" style="max-height: 500px;"/>
_**Figure 3:** Episode 64 of 350. The pole tilts too far, ending the episode._

<img src="/assets/images/bayesian-optimization/ep125.gif" alt="Figure 4" style="max-height: 500px;"/>
_**Figure 4:** Episode 125 of 350. The cart goes too far in one direction, ending the episode._

<img src="/assets/images/bayesian-optimization/ep216.gif" alt="Figure 5" style="max-height: 500px;"/>
_**Figure 5:** Episode 216 of 350. The agent performs well. The video cuts off before the agent fails._

As shown above, the agent initially has trouble keeping the pole balanced. Eventually, it learns that it can go in the direction that the pole is angled in order to prevent it from falling over immediately. However, the cart would then travel too far in one direction, another failure condition. Finally, the agent learns to move just enough to swing the pole the opposite way so that it is not constantly travelling in a single direction. This is the power of tuning **discount_factor** effectively!

### Closing Remarks

Through hyperparameter tuning with Bayesian optimization, we were able to achieve better performance than otherwise possible with standard search methods. The example code presented in this post is easily adaptable to explore more computationally intensive tasks. We encourage you to try:

* Implementing more sophisticated DQN features to improve performance
* Tuning a greater number of hyperparameters
* Attempting more complicated games from the OpenAI Gym, such as Acrobot-v1 and LunarLander-v0. Our code currently supports games with a discrete action space and a 1-D array of continuous states for the observation space
* Tuning a DQN to maximize general performance in multiple environments

Let us know what you try!

_This blog post was originally published on the [SigOpt blog](https://blog.sigopt.com/posts/using-bayesian-optimization-for-reinforcement-learning#footnotes). It is co-written by Olivia Kim._

<hr class="footnotes-sep">
<section class="footnotes">
<ol class="footnotes-list">
  <li id="fn1" class="footnote-item"><p>We use the version of the cart-pole problem as described by Barto, Sutton, and Anderson. <a href="#fnref1" class="footnote-backref">↩︎</a></p>
  </li>
  <li id="fn2" class="footnote-item"><p>For further insight into the Q-function, as well as reinforcement learning in general, check out <a href="https://www.nervanasys.com/demystifying-deep-reinforcement-learning/">this blog post from Nervana</a> <a href="#fnref2" class="footnote-backref">↩︎</a></p>
  </li>
  <li id="fn3" class="footnote-item"><p>The environment does not need to be deterministic for Q-learning to work. The <a href="http://users.isr.ist.utl.pt/~mtjspaan/readingGroup/ProofQlearning.pdf">proof that Q-learning converges</a> takes into account stochastic environments. <a href="#fnref3" class="footnote-backref">↩︎</a></p>
  </li>
  <li id="fn4" class="footnote-item"><p>DeepMind lists <a href="https://storage.googleapis.com/deepmind-data/assets/papers/DeepMindNature14236Paper.pdf#page=10">the hyperparameters used in their algorithm</a> to train an agent to play Atari games <a href="#fnref4" class="footnote-backref">↩︎</a></p>
  </li>
   <li id="fn5" class="footnote-item"><p>The types and ranges of the hyperparameters used in this example:
     <ul><li><strong>minibatch_size</strong>: integer [10, 500]</li><li><strong>epsilon_decay_steps</strong>: integer [nn/10,&nbsp;nn] where&nbsp;nn&nbsp;is the number of episodes (for the cart-pole problem, this is set to 350)</li><li><strong>hidden_multiplier</strong>: integer [5, 100]</li><li><strong>initial_weight_stddev</strong>: double [0.01, 0.5]</li><li><strong>initial_bias_stddev</strong>: double [0.0, 0.5]</li><li><strong>log(learning_rate)</strong>: double [log(0.0001), log(1)]</li><li><strong>discount_factor</strong>: double [0.5, 0.9999]</li></ul>
     <a href="#fnref5" class="footnote-backref">↩︎</a></p>
  </li>
  <li id="fn6" class="footnote-item"><p>We used uniform random search and the vertices of the hypercube for grid search. <a href="#fnref6" class="footnote-backref">↩︎</a></p>
  </li>
</ol>
