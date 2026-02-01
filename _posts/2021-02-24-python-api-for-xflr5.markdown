---
layout: post
title: "Making a python API for XFLR5: part 1"
date: 2021-02-24
categories: xflr5 python pythonqt rpc
---

<p>&nbsp;I recently treaded into the dangerous (but much awaited) territory of learning to program in C++. It has been
    held off for a long while but no longer!<br />Anyone who knows how to program knows that doing a project by yourself
    is THE single best way. Yes. THE. Nothing else. No.</p>
<p>Okay that was weird. and not true. But it's the best way for me.&nbsp;</p>
<p>So I started. For a long time I had been wanting to develop a python wrapper/API for this great program novice
    aircraft enthusiasts like me use to design their airplanes. It's called XFLR5 and it is written in C++. That's all
    that's relevant for this blog anyway. The API will be helpful for:</p>
<ul style="text-align: left;">
    <li>Writing custom scripts and optimization loops.</li>
    <li>Develop custom functions aerospace engineers use IF and ONLY IF the API is exposed correctly and efficiently.
    </li>
</ul>
<p>**Let's put a cute disclaimer here </p>
<p>I really don't know if anything I said above is possible. My anxiety tells me that it shouldn't be as it doesn't
    exist as of now. So everything from this point on is an educated guess and experimental at best. TeeHee.</p>
<p>** <br /></p>
<p>There is already a program called OpenVSP which does something very similar with a cool utility called SWIG. This is
    basically a wrapper generator which takes some index/configuration files and makes it easy to call C++ functions in
    python.&nbsp;</p>
<p>But I'm looking for something more dynamic. I need to be able to control the XFLR5 GUI from within python and
    manipulate it's state etc. This might mean that there is a lot of C++ code that will be re-written and well-That was
    the goal anyway.</p>
<p>I started by exploring the viability of this upto a certain degree. These were the tools I found which might be of
    good use:</p>
<ul style="text-align: left;">
    <li>SWIG</li>
    <li>Pybind11</li>
    <li><a href="https://doc.qt.io/archives/qq/qq23-pythonqt.html" target="_blank">PythonQt</a></li>
    <li>PyQt/Pyside? <br /></li>
    <li>Writing the entire wrapper on your own.</li>
</ul>
<p>&nbsp;This project won't be a priority of course. I have a lot on my mind with regard to the ongoing fellowship.
    &nbsp;&nbsp;</p>
<p>Let's see how it pans out.</p>
<p><br /></p>
<div></div>