---
layout: post
title: "DRL (Part 2): DQN implementation with only numpy"
date: 2020-09-28
categories: reinforcement-learning deep-learning numpy dqn
---

<p>&nbsp;In my last post, I talked about starting reinforcement learning with a small implementation of the deeplizard
    videos on RL in Python with just numpy. This post is about that.</p>
<p>For the sake of clarity, I defined the following goals before beginning:</p>
<p></p>
<ol style="text-align: left;">
    <li>No use of any standard machine learning library for the computation and network definition. Can use a library
        for the environment.</li>
    <li>Little to no functions/abstractions at all in the first iteration. This will make debugging easier and help any
        potential readers in understanding the flow of control better.</li>
    <li>No focus on speed/efficiency/complexity for now. Only numpy will be used along with cpu</li>
</ol>The conditions might seem absurd given that DRL and ML in general require tensors and inevitably the GPU for fast
computation. But like I said in my last post, I was doing this purely for learning and slowly building towards a larger
project. I struggled in finding an implementation that does the same so&nbsp; I will be publishing my code at my github
repo for newbies like me to learn easier.<p></p>
<p>Okay onto the code.</p>
<p>I'm writing this post on backlog so I might not be able to recall my full thought process but bear with me.</p>
<p>I knew a bit of theory from David Silver's videos and reading the book. To start off with the implementation I
    followed along with deeplizard's videos and wrote the code for the first game for the Frozen lake environment which
    implements the famous q-update rule given below. Doing this was critical to not get lost as the layers add on in the
    future.</p>
<p></p>
<div class="separator" style="clear: both; text-align: center;"><a
        href="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/bellman_update.png"
        style="margin-left: 1em; margin-right: 1em;"><img alt="" data-original-height="163" data-original-width="800"
            height="111"
            src="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/bellman_update.png"
            width="546" /></a></div>Next up was the Deep Q network implementation for the cartpole environment. I always
start off with writing pseudocode and I came up with the following for the same. (I haven't removed any mistakes
purposely to let the reader know what I was doing wrong initially). I will add all the code as I did the first time and
discuss the mistakes later on. Try to spot them as we go on.<br />
<p></p>
<p style="background-color: white; color: #080808; font-size: 9.8pt;"><span style="font-family: times;"><i><span
                style="color: teal;"><b>psedocode</b></span></i></span><span style="font-family: times;"><i><span
                style="color: teal;"><br /></span></i></span><span style="font-family: times;"><i><span
                style="color: teal;">-constants</span></i></span><span style="font-family: times;"><i><span
                style="color: teal;"><br /></span></i></span><span style="font-family: times;"><i><span
                style="color: teal;">gym.make<br /></span></i></span><span style="font-family: times;"><i><span
                style="color: teal;">reset environment<br /></span></i></span><span style="font-family: times;"><i><span
                style="color: teal;">get current screen<br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;">process screen: crop &gt; contigous &gt; resize &gt;
                add batch dim<br /></span></i></span><span style="font-family: times;"><i><span style="color: teal;">get
                black screen of same size as processed screenshot<br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;">create network<br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;">initialise random theta weights</span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"><br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"><b>for</b> each episode<br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp; &nbsp;</span>reset
                environment<br /></span></i></span><span style="font-family: times;"><i><span style="color: teal;">
                <span>&nbsp;&nbsp; &nbsp;</span>current_state = black screen</span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"><br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"> <b> for</b> each time
                step<br /></span></i></span><span style="font-family: times;"><i><span style="color: teal;">
                <span>&nbsp;&nbsp; &nbsp;</span>forward pass network with current_state(1x10800) to get current q(s)
                values (1x2)<br /></span></i></span><span style="font-family: times;"><i><span style="color: teal;">
                <span>&nbsp;&nbsp; &nbsp;</span>if random num &gt; epsilon:<br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp;
                    &nbsp;</span>select action which has larger q value<br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp;
                    &nbsp;</span>else;<br /></span></i></span><span style="font-family: times;"><i><span
                style="color: teal;"> <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span>select random
                action<br /></span></i></span><span style="font-family: times;"><i><span style="color: teal;">
                <span>&nbsp;&nbsp; &nbsp;</span>reward, done = take above action in environment<br />
            </span></i></span><span style="font-family: times;"><i><span style="color: teal;"> </span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp; &nbsp;</span>new_screen = take
                screenshot and process it like above<br /></span></i></span><span style="font-family: times;"><i><span
                style="color: teal;"> <span>&nbsp;&nbsp; &nbsp;</span>new_state =
                current_state-new_screen<br /></span></i></span><span style="font-family: times;"><i><span
                style="color: teal;"> <span>&nbsp;&nbsp; &nbsp;</span>store (current_state, action, reward, new_state)
                in replay memory<br /></span></i></span><span style="font-family: times;"><i><span style="color: teal;">
            </span></i></span><span style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp;
                    &nbsp;</span>if replay memory is bigger than the batch size:<br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp;
                    &nbsp;</span>sample batchsize from replay memory<br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp;
                    &nbsp;</span>X = batch sized input for policy<br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp;
                    &nbsp;</span>X_t = batch input for target<br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"> </span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp;
                    &nbsp;</span>q_current = forward pass X into policy net<br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp;
                    &nbsp;</span>q_target = forward pass X_t into target net<br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp;
                    &nbsp;</span>check if any target vals are not finishing states<br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp;
                    &nbsp;</span>q_star = reward + gamma*np.argmax(q_target) # bellman
                equation<br /></span></i></span><span style="font-family: times;"><i><span style="color: teal;">
            </span></i></span><span style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp;
                    &nbsp;<span>&nbsp;&nbsp; &nbsp;</span></span>cost = mean squared error of q_star and
                q_current<br /></span></i></span><span style="font-family: times;"><i><span style="color: teal;">
                <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span>grad = backpropagate the cost over
                policy net<br /></span></i></span><span style="font-family: times;"><i><span style="color: teal;">
                <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span>check gradient using finite
                differences<br /></span></i></span><span style="font-family: times;"><i><span style="color: teal;">
            </span></i></span><span style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp;
                    &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span>theta1 = theta1 -
                alpha*grad<br /></span></i></span><span style="font-family: times;"><i><span style="color: teal;">
                <span>&nbsp;&nbsp; &nbsp;<span>&nbsp;&nbsp; &nbsp;</span></span>theta2 = theta2 -
                alpha*grad<br /></span></i></span><span style="font-family: times;"><i><span style="color: teal;">
                <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span>theta3 = theta3 -
                alpha*grad<br /></span></i></span><span style="font-family: times;"><i><span style="color: teal;">
            </span></i></span><span style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp;
                    &nbsp;</span>current_state = new_state<br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp; &nbsp;</span>if step reaches
                multiple of 10<br /></span></i></span><span style="font-family: times;"><i><span style="color: teal;">
                <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span>change target network weights to policy
                weights<br /> </span></i></span><span style="font-family: times;"><i><span style="color: teal;">
                <span>&nbsp;&nbsp; &nbsp;</span>if done:<br /></span></i></span><span
        style="font-family: times;"><i><span style="color: teal;"> <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp;
                    &nbsp;</span>break<br /> </span></i></span><span style="font-family: times;"><i><span
                style="color: teal;"> decay epsilon for each episode</span></i></span></p>
<p></p><span><!--more--></span>
<p style="background-color: white; font-size: 9.8pt;"><span style="font-family: times;">Once this was done, I started
        with getting the processed screen from the environment for each time step and checking this with the deeplizard
        blog post&nbsp;</span></p>
<p style="background-color: white; font-size: 9.8pt;"><span style="font-family: times;">In summary, to get the processed
        screen one must do the following:</span></p>
<p style="background-color: white; font-size: 9.8pt;"><span style="font-family: times;">RGB array from render &gt; crop
        &gt; make it contiguous &gt; resize as 90x40 array &gt; add another dimension for the batch size.<br /></span>
</p>
<p style="background-color: white; font-size: 9.8pt;"><span style="font-family: times;">All of the above can be done
        using numpy except the resize part for which I used PIL(Pillow). The expand dims part just adds another
        dimension at the start of the 3 dimensional array. This is just a measure taken to concatenate states in the
        future. The black screen at the end is just the starting state for each episode.</span></p>
<div
    style="background: rgb(240, 240, 240) none repeat scroll 0% 0%; overflow: auto; padding: 0.5em 0.6em; width: auto;">
    <pre style="line-height: 125%; margin: 0px;">current_screen = env.render('rgb_array')[160:480, :, :]  # crop
current_screen = np.array(PILimg.fromarray(np.ascontiguousarray(current_screen)).resize((90, 40))).astype(np.float32)/255 
current_screen = np.expand_dims(current_screen.transpose((2,0,1)), axis=0) # make CHW&gt;add batch size dimension(helps in concatenation)
black_screen = np.zeros_like(current_screen).astype(np.float32) # initial state is all black
</pre>
</div>
<p>Let's start building the network next. We need to create a four layer network just like in the videos with 24 and 32
    units for the hidden layers.</p>
<div
    style="background: rgb(240, 240, 240) none repeat scroll 0% 0%; overflow: auto; padding: 0.5em 0.6em; width: auto;">
    <pre style="line-height: 125%; margin: 0px;"># Neural Network
# create  and initialise the network
h = black_screen.shape[2]
w = black_screen.shape[3]
ch = black_screen.shape[1]</pre>
    <pre style="line-height: 125%; margin: 0px;"><br /></pre>
    <pre style="line-height: 125%; margin: 0px;">inputLayer_size = h*w*ch # for three channels of each image
hidden1_size = 24
hidden2_size = 32
outputLayer_size = env.action_space.n

#initialisation
theta1 = theta1_t = np.random.uniform(0, 1, size=(hidden1_size, inputLayer_size+1))   # 24 x 10801
theta2 = theta1_t = np.random.uniform(0, 1, size=(hidden2_size, hidden1_size+1))      # 32 x 25
theta3 = theta1_t = np.random.uniform(0, 1, size=(outputLayer_size, hidden2_size+1)   # |actions| x 33</pre>
</div>
<p><i>Can you spot the mistakes above?</i></p>
<p>We can start our episode and time loops now. However, I'll go through the loop element wise to avoid confusion. I'll
    start with the inner time step loop first. Lets discuss the e-greedy policy first as that will make the first
    decision.&nbsp;</p>
<p>The e-greedy policy says that with a certain probability e we will choose a random exploratory action and choose the
    action with the highest q value the rest of the time. What I was confused about initially was that how do I know
    what the q values are at the start of the iteration? What we can do is make a&nbsp; forward pass of the current
    state through the network to calculate its expected future reward (q value)</p>
<div
    style="background: rgb(240, 240, 240) none repeat scroll 0% 0%; overflow: auto; padding: 0.5em 0.6em; width: auto;">
    <pre style="line-height: 125%; margin: 0px;">a1 = np.insert(current_state.reshape(1, inputLayer_size), 0, [1], axis=1) # 1
a2 = np.insert(sigmoid(a1 @ theta1.T), 0, [1], axis=1) # 1 x (24+1)
a3 = np.insert(sigmoid(a2 @ theta2.T), 0, [1], axis=1) # 1 x (32+1)
q_current = a3 @ theta3.T # 1 x 2

epsilon = epsilon_min + (epsilon_max - epsilon_min) * np.exp(-cum_step * epsilon_decay)
<br /></pre>
    <pre style="line-height: 125%; margin: 0px;">if np.random.rand() &gt; epsilon:
    action = np.argmax(q_current)
else:
    action = env.action_space.sample()

_, reward, done, _ = env.step(action)
</pre>
    <div><br /></div>
</div>
<p><i>Can you spot what's wrong above?</i></p>
<p>The first four lines make a forward pass through our network. The @ symbol is a matrix multiplication. Now, I know I
    said this is a function-less implementation but you weren't really expecting me to not write the activation
    functions! :)<br /></p>
<p>A note about the activation of the last layer. In our case and the deeplizard case we have kept the last layer as a
    linear activation "(which means no activation). This really depends on the type of targets that you will calculate
    your cost from. You can experiment with different functions.</p>
<div
    style="background: rgb(240, 240, 240) none repeat scroll 0% 0%; overflow: auto; padding: 0.5em 0.6em; width: auto;">
    <pre style="line-height: 125%; margin: 0px;">def sigmoid(x):  # i didn't want to do this okay..
    return 1/(1+np.exp(-x))

def sigmoid(x):</pre>
    <pre style="line-height: 125%; margin: 0px;"><span>&nbsp;&nbsp; &nbsp;return x*(1-x)</span><br /></pre>
    <pre style="line-height: 125%; margin: 0px;"><span><br /></span></pre>
    <pre style="line-height: 125%; margin: 0px;">def RELU(x):
    a = np.copy(x)
    a[a &lt;= 0] = 0
    return a

def dRELU(x):
    a=np.copy(x)
    a[a &lt;= 0] = 0
    a[a &gt; 0] = 1
    return a</pre>
</div>
<p>Now that our faithful agent has taken a step in some direction, Its time to calculate the new sate/new screen. As
    explained very clearly in the course, we need to take difference of two&nbsp;successive&nbsp;screens to get the new
    state. We'll use the same process above to get the new screen.</p>
<div
    style="background: rgb(240, 240, 240) none repeat scroll 0% 0%; overflow: auto; padding: 0.5em 0.6em; width: auto;">
    <pre style="line-height: 125%; margin: 0px;">new_screen = env.render('rgb_array')[160:480, :, :]  # crop
new_screen = np.array(PILimg.fromarray(np.ascontiguousarray(new_screen)).resize((90, 40))).astype(np.float32)/255  # contigous&gt;tensor&gt;resize
new_screen = np.expand_dims(new_screen.transpose((2, 0, 1)), axis=0)  # make CHW&gt;batch size
new_state = new_screen - current_screen
current_screen = new_screen</pre>
</div>
<p><i>Can you spot what's wrong above?</i></p>
<p>Next we fill the replay memory. This is fairly straightforward. After that we can update our current state to new
    state.</p>
<div
    style="background: rgb(240, 240, 240) none repeat scroll 0% 0%; overflow: auto; padding: 0.5em 0.6em; width: auto;">
    <pre style="line-height: 125%; margin: 0px;">if len(replay_mem) &lt; N:
    replay_mem.append([current_state, action, reward, done, new_state])
else:
    # push newest experience to tail if full
    replay_mem = replay_mem[1:]
    replay_mem.append([current_state, action, reward, done, new_state])l</pre>
    <pre style="line-height: 125%; margin: 0px;"><br /></pre>
    <pre style="line-height: 125%; margin: 0px;">#iteration update
current_state = new_state</pre>
</div>
<p>All of the above happens in the inner loop and the agent gains some experience until the replay memory is big enough
    to get sampled from.&nbsp;</p>
<p>Next is the part where we actually sample stated from the memory. The biggest problem that I was facing here wasn't a
    conceptual one but it was the implementation part in numpy. I have done it in a different way than in the videos.
    Parallel to RL I was also doing the ML course of stanford. So naturally, I was thinking of neural network in the
    terminaology that was taught there.</p>
<p>All this to say, I wanted The network to take in 'X' as the design matrix of training samples and input layer size.
    To create this input matrix I would need to sample an entire matrix from the memory. I have implemented this by
    storing stated in memory as rows where each of the columns are shown below</p>
<p></p>
<div class="separator" style="clear: both; text-align: center;"><a
        href="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/train_state.png"
        style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="346"
            data-original-width="945" height="244"
            src="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/train_state.png"
            width="667" /></a></div>
<div class="separator" style="clear: both; text-align: left;"><br /></div>
<div class="separator" style="clear: both; text-align: left;">The tough part was getting X. As I was storing sub arrays
    inside each element. I simply could not just index the first column and get X. This wasnt possbible as each action
    and reward was an integer so by default numpy stored the memory as an object array. After some search I got it
    working by using the concatenate method of numpy to get the first column. The snippet is shown below. All the things
    below happen when the replay memory is bigger than batch size</div>
<p></p>
<div
    style="background: rgb(240, 240, 240) none repeat scroll 0% 0%; overflow: auto; padding: 0.5em 0.6em; width: auto;">
    <pre style="line-height: 125%; margin: 0px;">if len(replay_mem) &gt;= n:
   memory_sample = np.array(replay_mem)[np.random.choice(len(replay_mem), n, replace= False), :]
   X = np.concatenate(memory_sample[:, 0]).reshape(n, inputLayer_size)
   actions = memory_sample[:, 1]
   rewards = memory_sample[:, 2]
   done_all = memory_sample[:, 3]
   X_t = np.concatenate(memory_sample[:, 4]).reshape(n, inputLayer_size)
    </pre>
</div>
<p>The next parts are not new(except some areas). Just some standard forward and backward passes. No?&nbsp;</p>
<p style="text-align: center;">NO.</p>
<p style="text-align: left;">To be honest the math stays the same but the concept has shifted a bit. I remember
    struggling with this part for some time. Since almost all good ML libraries use autograd for their backpropagation,
    things are very much abstracted like in the deeplizard videos. Its good to understand what is happening at the back
    even if you don't end up using it.&nbsp;</p>
<p style="text-align: left;">The way backprop + optimization is done in PyTorch usually is:&nbsp;</p>
<div
    style="background: rgb(240, 240, 240) none repeat scroll 0% 0%; overflow: auto; padding: 0.5em 0.6em; width: auto;">
    <pre style="line-height: 125%; margin: 0px;">current_q_values = QValues.get_current(policy_net, states, actions)
next_q_values = QValues.get_next(target_net, next_states)
target_q_values = (next_q_values * gamma) + rewards

loss = F.mse_loss(current_q_values, target_q_values.unsqueeze(1))
optimizer.zero_grad()
loss.backward()
optimizer.step()</pre>
</div>
<p>What autograd does is create a graph of all the previous computations for each tensor and traverses this graph
    backward to find the gradient. But I wanted to implement standard backpop even if it takes longer. The variable
    <i>loss</i> is a scalar value.&nbsp;</p>
<p>The straightforward forward pass is given below. There are two passes through the policy network and target network.
    This allows us to calculate the loss in the bellman equation.</p>
<div
    style="background: rgb(240, 240, 240) none repeat scroll 0% 0%; overflow: auto; padding: 0.5em 0.6em; width: auto;">
    <pre style="line-height: 125%; margin: 0px;"># Training policy network
# create design matrix
<span>&nbsp;&nbsp; &nbsp;</span>X = np.hstack((np.ones((n, 1)), X))                       # 256 x 10800+1
# feed forward the batch to get q(s,a)vals
<span>&nbsp;&nbsp; &nbsp;</span>a2 = np.hstack((np.ones((n, 1)), sigmoid(X @ theta1.T)))  # 256 x (24+1)
<span>&nbsp;&nbsp; &nbsp;</span>a3 = np.hstack((np.ones((n, 1)), sigmoid(a2 @ theta2.T))) # 256 x (32+1)
<span>&nbsp;&nbsp; &nbsp;</span>q_current = a3 @ theta3.T                                 # last layer is linear (no activation function as this matches with the target q distribution.
#q_current = h[np.arange(n), actions.astype(int)] # 256 x 2  --&gt; 256 x 1

# calculate loss through another pass
    X_t = np.hstack((np.ones((n, 1)), X_t))                         # 256 x 10800+1
    # take the next state as input for target network
    a2_t = np.hstack((np.ones((n, 1)), sigmoid(X_t @ theta1_t.T)))  # 256 x (24+1) #
    a3_t = np.hstack((np.ones((n, 1)), sigmoid(a2_t @ theta2_t.T)))    # 256 x (32+1)
    h_t = a3_t @ theta3_t.T                                         # 256 x 2</pre>
</div>
<p>The matrix&nbsp;<i>q_current&nbsp;</i>&nbsp;is same as <i>current_q_values.&nbsp;</i>&nbsp;What about next_q_values
    and target_q_values? The target q matrix is obtained from the bellman equation and is used to calculate the total
    network loss and error of the last layer.</p>
<p>If you'll recall, the target values are obtained by adding current rewards to the gamma weighted maximum possible q
    values of the future states. In our case the future q values are given by <i>h_t. </i>Also one important detail is
    that the q values of future are zero if the future state is final (corresponds to a done value). To sum it up, we
    need the maximum value of each row of <i>h_t</i>&nbsp;but also make this value zero if the same row has a 'done
    =True' in the done_all column.</p>
<p>Now that this is done, we still need to calculate the target values and hence the loss. At this point I was confused.
    The loss that PyTorch calculates is a scalar value which is the mean squared error between target q values and
    current q values. We can also do that. But what is the error or 'del' values of the last layer? We need to pass this
    something 'del4' vector to the network while backpropagating.</p>
<p></p>
<ol style="text-align: left;">
    <li>Is it the vector difference between <i>q_target</i>&nbsp;and <i>q_current? (n x 2)</i></li>
    <li>Is it the vector difference between&nbsp;<i>q_target</i>&nbsp;and&nbsp;<i>q_current&nbsp;</i>&nbsp;for each
        corresponding action<i>? (n x 1)</i></li>
    <li>Is it the scalar loss for each element of the layer error?&nbsp;<i>&nbsp;(n x 2)</i></li>
</ol>A similar question was asked <a
    href="https://stackoverflow.com/questions/62418510/backpropagation-for-double-q-learning-dqn">here</a> too. I
debated a lot of options. The one that made sense to me and the internet was 2. but the problem was that the weights of
the last layer (theta3 2x 33) would not match with the dimension (n x 1). I considered backpropagating for only the
relevant actions seemed too complex to&nbsp; be true. Indeed it was. After some more searching I found +realised&nbsp; a
neat concept that could tackle this (probably). We'll use a sort of hybrid of 1. and 2.&nbsp;<div>The idea is to keep
    the dimensions of the last layer error as (n x 2) so that we can backpropagate but make the errors zero at places
    not corresponding to the taken action in the previous step so that they don't have any effect. Also at places of
    taken actions we'll keep the actual <i>q_targets</i> (from bellman).</div>
<div>Let's go over everything step by step to understand it fully.</div>
<div>
    <ol style="text-align: left;">
        <li>Forward pass both policy (<i>q_current</i>) and target (<i>h_t</i>) networks</li>
        <li>Get the <i>q_next </i>vector with zeros at states which have done= True and maximum of each row of h_t where
            done=False</li>
        <li>Create <i>q_target = q_current</i></li>
        <li>For the corresponding actions taken set the value in q_target to (<i>rewards + gamma *q_next</i>)</li>
        <li>calculate cost = mean(( <i>q_target</i>[<i>actions</i>] - <i>q_current</i>[<i>actions</i>])**2)</li>
        <li>Set del4 =&nbsp;<i>q_target - q_current</i></li>
        <li>Use del4 for backprop.</li>
    </ol>Setting the target to current in 3rd step makes the errors 0 at places for actions other that current ones.
    Lets code.
</div>
<div>
    <p></p>
    <div
        style="background: rgb(240, 240, 240) none repeat scroll 0% 0%; overflow: auto; padding: 0.5em 0.6em; width: auto;">
        <pre style="line-height: 125%; margin: 0px;"># initialise temp outside of all loops</pre>
        <pre style="line-height: 125%; margin: 0px;">temp = np.zeros(n)</pre>
        <pre style="line-height: 125%; margin: 0px;"># do the following after the previous snippet
temp[~done_all.astype(bool)] = np.max(h_t[~done_all.astype(bool)], axis=1) # keep maximum of qval for non final states
q_target = np.copy(q_current)
q_target[np.arange(n), actions.astype(int)] = rewards.astype(np.float32) + gamma*temp

# RELU(mean squared error
cost = np.mean((q_target[np.arange(n), actions.astype(int)] - q_current[np.arange(n), actions.astype(int)])**2)# + (lam/2/n)*np.sum(theta1**2 + theta2**2+theta3**2)

# backpropagate loss
del4 = q_current - q_target</pre>
    </div>
    <p><i>There is still an error. Can you spot it?</i></p>
    <p>After this its just standard backprop with basic gradient descent. Or is it?? (Hey vsauce, Mi-)</p>
    <div
        style="background: rgb(240, 240, 240) none repeat scroll 0% 0%; overflow: auto; padding: 0.5em 0.6em; width: auto;">
        <pre style="line-height: 125%; margin: 0px;"><span>&nbsp;&nbsp; &nbsp;</span>del3 = del4 @ theta3*dsigmoid(a3)   <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp; </span># (256x2) * (2x 33) = 256x33 layer3 error
<span>&nbsp;&nbsp; &nbsp;</span>del2 = del3[:, 1:] @ theta2*dsigmiod(a2)         # (256x32) * (32x 25) = 256x25 layer 4 error

<span>&nbsp;&nbsp; &nbsp;</span>delta3 = del4.T @ a3  <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span># (1x256) * (256x33) = 2x33
<span>&nbsp;&nbsp; &nbsp;</span>delta2 = del3[:, 1:].T @ a2  <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;</span># (32x256) * (256x25) = 32x25
<span>&nbsp;&nbsp; &nbsp;</span>delta1 = del2[:, 1:].T @ X  <span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp; &nbsp;</span><span>&nbsp;&nbsp;</span># (24x256) * (256x10801) = 24x10801

<span>&nbsp;&nbsp; &nbsp;</span>#stacked to add regularisation in future
<span>&nbsp;&nbsp; &nbsp;</span>theta1_grad = np.hstack((delta1[:,0].reshape(hidden1_size,1), delta1[:,1:]))/n
<span>&nbsp;&nbsp; &nbsp;</span>theta2_grad = np.hstack((delta2[:,0].reshape(hidden2_size,1), delta2[:,1:]))/n
<span>&nbsp;&nbsp; &nbsp;</span>theta3_grad = np.hstack((delta3[:,0].reshape(outputLayer_size,1), delta3[:,1:]))/n
<br /></pre>
        <pre style="line-height: 125%; margin: 0px;"><span>&nbsp;&nbsp; &nbsp;</span>theta1 = theta1 - alpha*grad
<span>&nbsp;&nbsp; &nbsp;</span>theta2 = theta2 - alpha*grad
<span>&nbsp;&nbsp; &nbsp;</span>theta3 = theta3 - alpha*grad</pre>
        <pre style="line-height: 125%; margin: 0px;">if done:
    episode_durations.append(step)
    break

if int(episode % 10)==0:  # change target thetas every 10 episodes
    theta1_t = theta1
    theta2_t = theta2</pre>
        <pre style="line-height: 125%; margin: 0px;">    theta3_t = theta3</pre>
    </div>
    <p>I have also added the target theta updates in the above code.&nbsp;<i>&nbsp;Spot the error?</i></p>
    <p>Now that gradients were done, it was time for some debugging. Yay?</p>
    <p>I did this in steps:</p>
    <p></p>
    <ol style="text-align: left;">
        <li>Validate that my new state matches with the original deeplizard code.</li>
        <li>Validate the forward pass results (q_current)</li>
        <li>Validate my epsilon decay function</li>
        <li>Validate gradients</li>
        <li>Validate final thetas</li>
    </ol>The first two tests were successful. I got a similar intermediate state as the original code as shown below:<p>
    </p>
    <div class="separator" style="clear: both; text-align: center;"><a
            href="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/pendulum_state_change.png"
            style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="289"
                data-original-width="545" height="236"
                src="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/pendulum_state_change.png"
                width="444" /></a></div><br />
    <p>The third wasn't right. The values for my epsilon were not decaying as fast as they should. Found the solution
        for this relatively quickly. Epsilon needs to be decayed with respect to total cumulative time steps. I was
        decaying it for each episode hence the smaller rate of change.</p>
    <p>Lets move to gradients. Obviously my first run was far from correct. The gradients, q values, activations etc.
        were blowing up very fast. This is a known problem in ML and I knew something about it. But what I did to debug
        this problem was take elements from the original calculated code like the design matrix etc. and ran my backprop
        on it. The first thing I found wrong was my activation. The original code used RELU but I had used sigmoid by
        habit.&nbsp;</p>
    <p>After a lot of stress and hours, I finally got the gradients of the right order but off by a factor of 2. del4!
    </p>
    <p>del4&nbsp; = <b>2*</b>(&nbsp;<i>q_target - q_current)</i></p>
    <p>After running it fully on my code however, it would still not work. But now it was more clear what the problem
        is. The problem of gradients blowing up is caused by not initializing the weights and biases correctly. For a
        network with RELU activations, kaiming initialisation is used. Its easy enough to implement. After doing this,
        the gradients came out perfect!</p>
    <p>Next were the optimized thetas. I knew that the original code uses the Adam optimizer but I didn't bother to
        implement that just to fail. The thetas from standard gradient descent were off from the actual values
        especially for initial episodes. This is because Adam allows a decaying learning rate over the training
        iterations by calculating an exponential moving average. Also, the learning rate is different for each weight
        and bias. So onto implementing Adam. The algo in the <a href="https://arxiv.org/pdf/1412.6980.pdf,">paper</a> is
        pretty self explanatory.</p>
    <div
        style="background: rgb(240, 240, 240) none repeat scroll 0% 0%; overflow: auto; padding: 0.5em 0.6em; width: auto;">
        <pre style="line-height: 125%; margin: 0px;"># place the following two lines out of all loops</pre>
        <pre style="line-height: 125%; margin: 0px;">exp_avg = [np.zeros_like(theta1), np.zeros_like(theta2), np.zeros_like(theta3)]
exp_avg_sq = [np.zeros_like(theta1), np.zeros_like(theta2), np.zeros_like(theta3)]
</pre>
        <div><br /></div>
        <pre style="line-height: 125%; margin: 0px;"><br /></pre>
        <pre style="line-height: 125%; margin: 0px;">below code is to be put in place of gradient descent:</pre>
        <pre style="line-height: 125%; margin: 0px;">opt_step += 1
bias_corr1 = 1 - beta1 ** opt_step
bias_corr2 = 1 - beta2 ** opt_step
for i,grad in enumerate((theta1_grad, theta2_grad, theta3_grad)):
    exp_avg[i] = beta1*exp_avg[i] + (1-beta1)*grad
    exp_avg_sq[i] = beta2*exp_avg_sq[i] + (1-beta2)*(grad**2)
    denom = eps+np.sqrt(exp_avg_sq[i]/bias_corr2)
    step_size[i] = (exp_avg[i]*lr)/bias_corr1/denom

theta1 -= step_size[0]
theta2 -= step_size[1]
theta3 -= step_size[2]</pre>
    </div>
    <p>This definitely made things better. The updated thetas were exact right.</p>
    <p>At this point I ran the learning algorithm. It was admittedly slow but I just wanted to see if it learns anything
        at all. The following graph was plotted on running:</p>
    <div class="separator" style="clear: both; text-align: center;"><a
            href="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/bad_train_graph.png"
            style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="431"
                data-original-width="548" height="327"
                src="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/bad_train_graph.png"
                width="415" /></a></div><br />
    <p>That's just sad.&nbsp;</p>
    <p>For comparison, the original algo's graph looks like:</p>
    <div class="separator" style="clear: both; text-align: center;"><a
            href="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/og_train_graph.png"
            style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="435"
                data-original-width="567" height="307"
                src="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/og_train_graph.png"
                width="400" /></a></div>
    <div><br /></div>What could have gone wrong? A lot as it turns out.<br />
    <p>To find these mistakes I merged my own and the original implementation. Sharing the variable space made it easier
        to debug. And it was definitely better.&nbsp;</p>
    <p>The first and most important mistake was a python mistake. I had updated the target thetas after 10 episodes by
        changing the reference and not the object itself. To solve it I initialized target thetas as a separate
        reference and created copies whenever I wanted to update.</p>
    <p>The second problem was that I wasn't making the new state black when the episode got finished. This meant that
        even if done was true the new state was still a successive difference</p>
    <p>The third error was simple but hard to catch. Remember the temp array we used for storing max q values for future
        states and zeros for final states? I had only initialized this array once and as it got updated, it got filled
        up with maximum q values only and no zeros for final states. All I had to do was initialize this to zeros for
        each time step.</p>
    <p>This is the graph I got after running it now:</p>
    <div class="separator" style="clear: both; text-align: center;"><a
            href="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/train_progress.gif"
            style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="421"
                data-original-width="559" height="324"
                src="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/train_progress.gif"
                width="430" /></a></div><br />
    <p>yay.</p>
    <p>The agent learns at par or better than the original code but its much slower of course. Speed was not a target
        anyways so this will suffice. I call it "dirty_DQN".</p>
    <p>I learnt a lot through this process even though it took some time. The final code is given below. Happy coding.
    </p>
    <div
        style="background: rgb(240, 240, 240) none repeat scroll 0% 0%; overflow: auto; padding: 0.5em 0.6em; width: auto;">
        <pre style="line-height: 125%; margin: 0px;">import numpy as np
import gym
import matplotlib.pyplot as plt
from PIL import Image as PILimg
from itertools import count
import shelve
# import torch
# from IPython import display
# import torchvision.transforms as T
# from collections import namedtuple
# from itertools import count


# constants
plt.figure(2)

lr = 0.001
gamma = 0.999
epsilon = 1
epsilon_max = 1
epsilon_min = 0.01
epsilon_decay = 0.001
beta1 = 0.9
beta2 = 0.999
eps = 1e-8


num_eps = 400
N = 100000 # replay memory size
n = 256 # batch size
replay_mem = []
step_size=[lr]*3

#environment
env = gym.make("CartPole-v0").unwrapped
env.reset()
#env.state=np.array([ 0.00211603,  0.02750717, -0.0075166,   0.04541408])
current_screen = env.render('rgb_array')[160:480, :, :]  # crop
current_screen = np.array(PILimg.fromarray(np.ascontiguousarray(current_screen)).resize((90, 40))).astype(np.float32)/255  # contigous&gt;tensor&gt;resize
current_screen = np.expand_dims(current_screen.transpose((2,0,1)), axis=0) # make CHW&gt;add batch size dimension(helps in concatenation)
# black_screen = np.zeros_like(screen) # initial state is all black

black_screen = np.zeros([1,3,40,90]).astype(np.float32) # initial state is all black

h = black_screen.shape[2]
w = black_screen.shape[3]
ch = black_screen.shape[1] # number of channels

# Neural Network
# create  and initialise the network
inputLayer_size = h*w*ch # for three channels of each image
hidden1_size = 24
hidden2_size = 32
outputLayer_size = env.action_space.n

# kaiming initialisation
theta1 = np.random.uniform(-1, 1, size=(hidden1_size, inputLayer_size+1))    * np.sqrt(1/(inputLayer_size+1))  # 24 x 10801
theta2 = np.random.uniform(-1, 1, size=(hidden2_size, hidden1_size+1))       * np.sqrt(1/(hidden1_size+1))     # 32 x 25
theta3 = np.random.uniform(-1, 1, size=(outputLayer_size, hidden2_size+1))   * np.sqrt(1/(hidden2_size+1))     # |actions| x 33
theta1_t = np.random.uniform(-1, 1, size=(hidden1_size, inputLayer_size+1))  * np.sqrt(1/(inputLayer_size+1))  # 24 x 10801
theta2_t = np.random.uniform(-1, 1, size=(hidden2_size, hidden1_size+1))     * np.sqrt(1/(hidden1_size+1))     # 32 x 25
theta3_t = np.random.uniform(-1, 1, size=(outputLayer_size, hidden2_size+1)) * np.sqrt(1/(hidden2_size+1))     # |actions| x 33
exp_avg = [np.zeros_like(theta1), np.zeros_like(theta2), np.zeros_like(theta3)]
exp_avg_sq = [np.zeros_like(theta1), np.zeros_like(theta2), np.zeros_like(theta3)]

episode_durations = []
cum_step = 0
opt_step = 0
def sigmoid(x):  # i didn't want to do this okay..
    return 1/(1+np.exp(-x))

def RELU(x):
    a = np.copy(x)
    a[a &lt;= 0] = 0
    return a

def dRELU(x):
    a=np.copy(x)
    a[a &lt;= 0] = 0
    a[a &gt; 0] = 1
    return a

# RL
for episode in range(num_eps):
    env.reset()
    #get current screen
    current_screen = env.render('rgb_array')[160:480, :, :]  # crop
    current_screen = np.array(PILimg.fromarray(np.ascontiguousarray(current_screen)).resize((90, 40))).astype(np.float32) / 255  # contigous&gt;tensor&gt;resize
    current_screen = np.expand_dims(current_screen.transpose((2, 0, 1)),axis=0)  # make CHW&gt;add batch size dimension(helps in concatenation)
    # starting state is a black screen
    current_state = black_screen
    cum_cost = 0

    # print(env.state)
    for step in count():
        # initial state forward pass to get q(s) vals
        a1 = np.insert(current_state.reshape(1, inputLayer_size), 0, [1], axis=1) # 1
        a2 = np.insert(RELU(a1 @ theta1.T), 0, [1], axis=1) # 1 x (24+1)
        a3 = np.insert(RELU(a2 @ theta2.T), 0, [1], axis=1) # 1 x (32+1)
        q_current = a3 @ theta3.T # 1 x 2

        epsilon = epsilon_min + (epsilon_max - epsilon_min) * np.exp(-cum_step * epsilon_decay)
        if np.random.rand() &gt; epsilon:
            action = np.argmax(q_current)
        else:
            action = env.action_space.sample()
        #action=0
        _, reward, done, _ = env.step(action)

        # A final state is a black screen
        if done:
            new_state = black_screen
        else:
            new_screen = env.render('rgb_array')[160:480, :, :]  # crop
            new_screen = np.array(PILimg.fromarray(np.ascontiguousarray(new_screen)).resize((90, 40))).astype(np.float32)/255  # contigous&gt;tensor&gt;resize
            new_screen = np.expand_dims(new_screen.transpose((2, 0, 1)), axis=0)  # make CHW&gt;batch size
            new_state = new_screen - current_screen
            current_screen = new_screen

        # push experiences
        if len(replay_mem) &lt; N:
            replay_mem.append([current_state, action, reward, done, new_state])
        else:
            #push newest experience to tail if full
            replay_mem = replay_mem[1:]
            replay_mem.append([current_state, action, reward, done, new_state])

        #iteration update

        current_state = new_state

        # pick random sample from replay memory of input size
        if len(replay_mem) &gt;= n:
            memory_sample = np.array(replay_mem)[np.random.choice(len(replay_mem), n, replace= False), :]
            X = np.concatenate(memory_sample[:, 0]).reshape(n, inputLayer_size)
            actions = memory_sample[:, 1]
            rewards = memory_sample[:, 2]
            done_all = memory_sample[:, 3]
            X_t = np.concatenate(memory_sample[:, 4]).reshape(n, inputLayer_size)
    # Training policy network
        # create design matrix
            X = np.hstack((np.ones((n, 1)), X)) # 256 x 10800+1
        # feed forward the batch to get q(s,a)vals
            a2 = np.hstack((np.ones((n, 1)), RELU(X @ theta1.T))) # 256 x (24+1)
            a3 = np.hstack((np.ones((n, 1)), RELU(a2 @ theta2.T))) # 256 x (32+1)
            q_current = a3 @ theta3.T # last layer is linear (no activation function as this matches with the target q distribution.
            #q_current = h[np.arange(n), actions.astype(int)] # 256 x 2  --&gt; 256 x 1

        # calculate loss through another pass
            X_t = np.hstack((np.ones((n, 1)), X_t))  # 256 x 10800+1
            # take the next state as input for target network
            a2_t = np.hstack((np.ones((n, 1)), RELU(X_t @ theta1_t.T)))  # 256 x (24+1) #
            a3_t = np.hstack((np.ones((n, 1)), RELU(a2_t @ theta2_t.T)))  # 256 x (32+1)
            h_t = a3_t @ theta3_t.T # 256 x 2
            # temp[done_all] = 0  # set all final states to 0 qval
            temp = np.zeros(n)
            temp[~done_all.astype(bool)] = np.max(h_t[~done_all.astype(bool)], axis=1) # keep maximum of qval for non final states
            q_target = np.copy(q_current)
            q_target[np.arange(n), actions.astype(int)] = rewards.astype(np.float32) + gamma*temp

            # RELU(mean squared error
            cost = np.mean((q_target[np.arange(n), actions.astype(int)] - q_current[np.arange(n), actions.astype(int)])**2)# + (lam/2/n)*np.sum(theta1**2 + theta2**2+theta3**2)
            cum_cost +=cost
        # backpropagate loss
            del4 = 2*(q_current - q_target)   # 256x2
            del3 = del4 @ theta3*dRELU(a3)   # # (256x2) * (2x 33) = 256x33 layer3 error
            del2 = del3[:, 1:] @ theta2*dRELU(a2)  # (256x32) * (32x 25) = 256x25 layer 4 error

            delta3 = del4.T @ a3  # (1x256) * (256x33) = 2x33
            delta2 = del3[:, 1:].T @ a2  # (32x256) * (256x25) = 32x25
            delta1 = del2[:, 1:].T @ X  # (24x256) * (256x10801) = 24x10801

            #stacked to add regularisation in future
            theta1_grad = np.hstack((delta1[:,0].reshape(hidden1_size,1), delta1[:,1:]))/n
            theta2_grad = np.hstack((delta2[:,0].reshape(hidden2_size,1), delta2[:,1:]))/n
            theta3_grad = np.hstack((delta3[:,0].reshape(outputLayer_size,1), delta3[:,1:]))/n
            # get new theta vals using GD

            #TODO optimization
            opt_step += 1
            bias_corr1 = 1 - beta1 ** opt_step
            bias_corr2 = 1 - beta2 ** opt_step
            for i,grad in enumerate((theta1_grad, theta2_grad, theta3_grad)):
                exp_avg[i] = beta1*exp_avg[i] + (1-beta1)*grad
                exp_avg_sq[i] = beta2*exp_avg_sq[i] + (1-beta2)*(grad**2)
                denom = eps+np.sqrt(exp_avg_sq[i]/bias_corr2)
                step_size[i] = (exp_avg[i]*lr)/bias_corr1/denom

            theta1 -= step_size[0]
            theta2 -= step_size[1]
            theta3 -= step_size[2]

        cum_step += 1
        if done:
            episode_durations.append(step)
            plt.plot(episode, episode_durations[-1], 'ro')
            #moving avergage
            if episode &gt; 101:
                moving_avg = np.mean(episode_durations[-100:])
            else:
                moving_avg=0
            #plt.pause(0.005)
            plt.plot(episode, moving_avg, "b*")

            plt.pause(0.005)

            print("Training Cost: ", cum_cost/step)
            print("100 Episode average", moving_avg)
            break


    if int(episode % 10)==0:  # change target thetas every 10 episodes
        theta1_t = np.copy(theta1)
        theta2_t = np.copy(theta2)
        theta3_t = np.copy(theta3)


'''
psedocode

-constants

gym.make
reset environment
get current screen
process screen: crop &gt; contigous &gt; resize &gt; add batch dim
get black screen of same size as processed screenshot
create network
initialise random theta weights

for each episode
    reset environment
    current_state = black screen

    for each time step
        forward pass network with current_state(1x10800) to get current q(s) values (1x2)
        if random num &gt; epsilon:
            select action which has larger q value
        else;
            select random action
        reward, done = take above action in environment 
        
        new_screen = take screenshot and process it like above
        new_state = current_state-new_screen
        store (current_state, action, reward, new_state) in replay memory
        
        if replay memory is bigger than the batch size:
            sample batchsize from replay memory
            X = batch sized input for policy
            X_t = batch input for target
            
            q_current = forward pass X into policy net
            q_target = forward pass X_t into target net
            check if any target vals are not finishing states
            q_star = reward + gamma*np.argmax(q_target) # bellman equation
            
            cost = mean squared error of q_star and q_current
            grad = backpropagate the cost over policy net
            check gradient using finite differences
            
            theta1 = theta1 - alpha*grad
            theta2 = theta2 - alpha*grad
            theta3 = theta3 - alpha*grad
        
        current_state = new_state
        if step reaches multiple of 10
            change target network weights to policy weights    
        if done:
            break    
    decay epsilon
    
    
    
# get params
import shelve
s=  shelve.open("shelve") 
theta1=s.get("theta1")
theta2=s.get("theta2")
theta3=s.get("theta3")
theta1_t=s.get("theta1_t")
theta2_t=s.get("theta2_t")
theta3_t=s.get("theta3_t")
actions = s.get("actions")
done_all = s.get("done_all")
rewards = s.get("rewards")
X= s.get("X")
X_t =s.get("X_t")
s.close()

    
'''</pre>
    </div>
</div>