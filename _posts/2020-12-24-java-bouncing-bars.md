---
layout: post
title: "Bouncing bars with java threading"
date: 2020-11-27 08:00:00 +0200
tags: [Java, algorithms, bouncing bars, threading]
---

As an assignment during a course on "[Operating Systems](http://m3101.parlenet.org/_media/m3101-stud.pdf)" (p. 73) we had to recreate in Java a popular program : the bouncing balls. It's great to teach common concepts in Java like making GUIs with `javax.swing` or `javafx`, using multi-threading with the `Thread` and `Runnable` classes and using other commonly used classes.

<img src="https://koc.csbridge.org/img/projects/multipleBalls/multipleBalls.gif" width=400>





> source: <https://koc.csbridge.org/en/projects/multipleBalls.html>



### Sounds nice, how do I code such a thing ?

"Bouncing balls but instead of balls we use bars... that sounds familiar" was my first thought attacking this project. So I did what all CompSci student do : I went to see if anybody had already coded something similar on github, and of course someone already had. I looked up different implementations of the world's oldest video game, Pong, and hacked different bits of code from various repositories together to get a working prototype. 

> TODO : add gif

### A basic collision system

Neat ! But the bars don't exactly collide, they just fade through each other. My next task was to implement a basic collision system. At first I was heavily inspired by [this awesome blogpost]("https://www3.ntu.edu.sg/home/ehchua/programming/java/J8a_GameIntro-BouncingBalls.html") by Hock-Chuan Chua from the Nanyang Technical University of Singapour.  He goes into great detail on the code, design choices and mathematics used in his Bouncing Balls and the collision system they use. The only problem is, it's waaay too complicated and copy pasting the whole thing *might* arouse suspicion so I'm better off making one myself. My first idea was, when two bars collide, to inverse there speed vector but as you can see it looked slightly off.

> TODO : add gif

My second (and final until I come back to it) version was to "swape" the bars' speed vectors and effectively transfer each bars momentum to each other, creating an [elastic collision](https://en.wikipedia.org/wiki/Elastic_collision) (the first gif is great to visualize). I'll likely come back to this project to implement the "Two-dimensional" elastic collision like shown in the bottom of the Wikipedia page.

And that's about it, the GUI is pretty and the bars collide nicely with each other. I just have to change my code a bit to use multiple threads instead of only one and I'll be done. Surely adding multiple threads won't break anything ! *(epic foreshadowing)*

### Adding threads and breaking everything

> TODO : add shameful gif of poly homo bar action

I'd be lying if I said I didn't expect this to happen. But why does this occur ? The collisions used to be calculated as follows : every frame check if two bars' x and y coordinates overlap each other, if so they're colliding and we need to apply changes to the speed vectors, if not do nothing. Note how these checks are all done in a linear and synchronized way.

```
# TODO : add a proper diagram
+----------------+
|                |
|                v
|        draw all the bars
|                |
|                v
| check if box1 collides with box2
|                |
|                v
| check if box2 collides with box3
|                |
|                v
|               ...
|                |
|                v
| check if boxn-1 collides with boxn
|                |
|                |
+----------------+  
```

Threads by nature aren't synchronized with each other and since we can't control them after they're started they have to do all the collision checking themselves. This can lead a [race conditions](https://stackoverflow.com/questions/34510/what-is-a-race-condition) where one bar (thread) checking to see if it is colliding with any other bars won't be able to move out of the way of an incoming bar. The two bars will overlap and continuously see themselves as colliding, exchanging their speed vectors every game tick, effectively freezing them in place.

> A race condition occurs when two or more threads can access shared data and they try to change it at the same time. Because the thread scheduling algorithm can swap between threads at any time, you don't know the order in which the threads will attempt to access the shared data. Therefore, the result of the change in data is dependent on the thread scheduling algorithm, i.e. both threads are "racing" to access/change the data.
>
> Posted by Lehane, edited by Amit Joki - Stackoverflow

This can be counteracted by defining the `Bar.move` method as `synchronized` using the appropriate keyword. 

```java
public synchronized void move(Barre barre) { ... }
```

In this method, we use the `wait()` method to [freeze the thread](https://stackoverflow.com/questions/16758346/how-pause-and-then-resume-a-thread) until the `notify()` or `notifyAll()` method is called. These two blogposts [(1)](https://ducmanhphan.github.io/2020-03-26-Understanding-basic-concepts-in-Java-multithreading/) [(2)](https://ducmanhphan.github.io/2019-12-07-Using-wait-notify-in-synchronized-method-block-of-Multithreading-Java/) by github user DucManhPhan explain the basics of threading in java and the use of `wait`/`notify` in synchronized methods. This way of doing might not seem as the most optimal one but it's what we've been asked to do so I'm sticking to that.

### So, does it work now ?

Yes ! It works even better than I expected. All source code and a `.jar` file with the code ready to use can be found on this [github repository](). If you've noticed any inaccuracies or mistakes in this blog or the code please feel free to let me know and for all those of you who've read this far, enjoy the end result. 

> TODO : add victory.gif

### notes & references

https://koc.csbridge.org/en/projects/multipleBalls.html

https://www3.ntu.edu.sg/home/ehchua/programming/java/J8a_GameIntro-BouncingBalls.html

https://en.wikipedia.org/wiki/Elastic_collision

https://ducmanhphan.github.io/2020-03-26-Understanding-basic-concepts-in-Java-multithreading/

https://ducmanhphan.github.io/2019-12-07-Using-wait-notify-in-synchronized-method-block-of-Multithreading-Java/

