---
layout: post
title: "Track High Score with Phaser"
date: 2014-02-27 17:20:50 -0600
comments: true
categories: [tags]
tags: [html5, javascript, phaser]
---
The Phaser framework has many resources behind it, such as the examples on the Phaser site, blogs with detailed howto guides and many open source games. I found it trivial to make a quick proof of concept for a simple idea, with a minimal knowledge of the framework before I started. However, as I continued to develop my first game, adding certain elements became more difficult and less documented.

Once I figured out how to add different states, my code was instantly cleaner. At first, I had 3 states:

* Menu - Initial game instructions
* Play - Game is being played and bird is alive
* Game Over - Game is over, bird is dead

Pesky state variables and flags disappeared. The only issue I ran into after doing this was - how do I store a high score when my score variable is in a different state? As far as I knew, everything was destroyed between states. What is a game over screen without a high score (or even the score that caused the game to end)?

I could not find any examples or tutorials on how to do this, nor did I have any experience with local HTML5 browser storage. After searching through the Phaser docs, I stumbled upon this [function](http://docs.phaser.io/Phaser.Device.html#toc26) that returns whether or not there is local storage on the device playing the current game. If I could use local storage, I could store the current score and the high score there!

As it turns out, accessing local browser storage is incredibly simple. Through Javascript, you can access local storage (if the browser supports it) through the window.localStorage object. You can store whatever you want here and access it from anywhere. From my code:

{% highlight groovy %}
if (this.game.device.localStorage) {
    localStorage.score = this.score;
    if (localStorage.highScore) {
        if (localStorage.score > localStorage.highScore) {
            localStorage.highScore = localStorage.score;
        }
    }
    else {
        localStorage.highScore = localStorage.score;
    }
}
{% endhighlight %}
This is the logic to store the current score and update the high score. Incredibly simple. The high score is even stored between browser sessions! You can see this in action by opening up the Developer Tools (if you use Chrome) and going to the Resources Tab.

![Phaser Bird Dev Tools](/assets/phaser-bird-dev-tools.png)

Links:

[HTML5 storage][HTML5 storage]

[Phaser official examples][Phaser official examples]

[Howto with a zombie shooting game][Howto with a zombie shooting game]

[Howto with another Flappy Bird clone][Howto with another Flappy Bird clone]

[HTML5 storage]: http://diveintohtml5.info/storage.html
[Phaser official examples]: http://gametest.mobi/phaser/examples/
[Howto with a zombie shooting game]: http://jessefreeman.com/game-dev/building-a-html5-game-with-phaser/
[Howto with another Flappy Bird clone]: http://blog.lessmilk.com/how-to-make-flappy-bird-in-html5-1/

