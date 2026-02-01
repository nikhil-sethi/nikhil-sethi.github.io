---
layout: post
title: "Deep Reinforcement Learning (Part 1)"
date: 2020-09-17
categories: reinforcement-learning deep-learning
---

<p>Welcome to part 1 of this series of blog posts that I'll be doing on reinforcement learning. I don't know how
    religious I'll be in updating it, but I'll start anyhow.</p>
<p></p>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container"
    style="margin-left: auto; margin-right: auto;">
    <tbody>
        <tr>
            <td style="text-align: center;"><a
                    href="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/bird_flocking.png"
                    style="margin-left: auto; margin-right: auto;"><img alt="Birds flocking in a V-formation"
                        data-original-height="565" data-original-width="849" height="448"
                        src="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/bird_flocking.png"
                        title="Birds flocking in a V-formation" width="674" /></a></td>
        </tr>
        <tr>
            <td class="tr-caption" style="text-align: center;"><i><span style="font-size: x-small;">Birds flocking in a
                        V-formation</span></i><br /></td>
        </tr>
    </tbody>
</table>
<p></p>
<p>I started reinforcement learning with an objective to develop algorithms for multi-agent control of unmanned aerial
    vehicles. While I have a lot of interest in natural and emergent optimization methods, learning it also supplements
    my job as a research associate under a professor of swarm intelligence.</p>
<p>Even though RL is relatively new, I won't dwell on what it is. There are plenty of resources for that already (a
    simple google search will suffice). I'll get straight to what I am up to.</p>
<p>If you're a complete newbie like me, I would suggest gaining some knowledge of the following. If you are curious
    enough, however, you might gain some of it along the way too:</p>
<p></p>
<ol style="text-align: left;">
    <li>Basic Machine/deep learning strategies</li>
    <li>Beginner-intermediate Python</li>
    <li>Some(not a lot) experience with any ML library (I was familiar with PyTorch)</li>
    <li>Linear algebra, Calculus, and basic statistics (while not completely necessary, it is recommended if you wanna
        understand stuff thoroughly.</li>
</ol>The list is not exhaustive. You might need some coffee and the will to rigorously debug mistakes as well :)&nbsp;
<p></p>
<p>Like almost everyone, I started on youtube and google with <a
        href="https://www.youtube.com/watch?v=2pWv7GOvuf0&amp;list=PLqYmG7hTraZDM-OYHWgPebj2MfCFzFObQ"
        target="_blank">these videos</a> by David Silver. They're a bit math-intensive and detailed but I liked that. It
    helped a lot in understanding standard conventions and mostly in understanding research papers which is the primary
    source of knowledge in a developing field.&nbsp;</p>
<p>Speaking of conventions, I also read up on the famous <a
        href="https://web.stanford.edu/class/psych209/Readings/SuttonBartoIPRLBook2ndEd.pdf" target="_blank">book by
        Sutton and Barto</a>. I still haven't read it comprehensively like one should but I started some of it anyway.
    It's a good book.&nbsp;</p>
<p>When I was somewhat acquainted with the terminology and basic concepts, I couldn't wait to implement a project and
    get on programming what I had learned. A great help at this point was <a
        href="https://www.youtube.com/watch?v=nyjbcRQ-uQ8&amp;list=PLZbbT5o_s2xoWNVdDudn51XM8lOuZ_Njv"
        target="_blank">this course</a> by the channel deeplizard on youtube. They build up from the basics to develop
    an RL project using gym (a library to manage RL environments) and PyTorch.&nbsp;</p>
<p>It is my habit to reinvent the wheel whenever I am learning something new- It is a great way to understand
    fundamentals. So, instead of following the approach in the videos, my objective was to develop the entire algorithm
    in pure Python (well, almost ;) go numpy!).&nbsp;<br /></p>
<p>The code in the series is good and fast but is highly abstracted due to PyTorch and its vast number of operations on
    Tensors (not to mention the dynamic computation graphs). So, another thing that I always do on my first
    implementations is to make the entire code into one script and with the least amount of functions possible. This
    makes me understand the flow of control better and makes it far easier to debug.</p>
<p>And thus I began. I'll be continuing in the next post. Thank you for reading :)</p>