---
title: "Your Ide Is Not an Excuse for Unreadable Code"
date: 2018-03-16T21:25:26Z
draft: true
tags: ["technical", "clean code", "php"]
---

## Your IDE Is Not an Excuse for Unreadable Code

I was reviewing some pull requests recently and I got into a  little debate. Just before I go into what that was exactly let me prefix this with: as a developer I am a very strong believer in writing clean, readable code. From top to bottom it should read like a story and if I handed it to someone who had never coded in their life, I want them to be able to understand roughly what the code is doing.

So the debate started with the following comment;

_“My IDE tells me what the that parameter is meant to do when I hover over it”_

Some of you may agree with this point and some of you, like myself, will disagree with it. To me just because your IDE provides a utility does not mean you should rely on it. It should be there to aid your development, but it should not be a crutch that your code relies on to be readable.

Lets dig into it with a quick example involving boolean flags, something which can often refer to code smell on their own (and for good reason!).


```php
$postgresql->truncateTable('user', true);
```


Could you tell purely by scanning over the above line what the flag was doing? I could hover over and check which inevitably I would be forced to do in this situation but that’s just extra cognitive load. Load which could easily be avoided with a few simple techniques. My personal take is to avoid boolean flags wherever possible, but if I really need to utilise them I’ll often create well named constants to replace passing through `true/false` - for example:

```php
$postgresql->truncateTable('user', PostgreSQL::RESTART_SEQUENCE)
```


A parameter like the above allows me to scan through the code and understand at a glance what each part is doing without me having to hover over every other parameter or worse jump into a never-ending rabbit hole of methods. Alternatively (and preferably in my opinion) you could create separate methods to remove the boolean flags.

```php
$postgresql->truncateTable('user');
$postgresql->restartSequence('user');
```


Overall we have added an extra line but that trade off gives us more readable code that does not rely on an IDE to understand at first glance. Even better we don’t have to dodge, duck, dip, dive and dodge into every other method to check its parameters, or what the flags mean if our IDE does not support hovering hints. We simply read well named methods and carry on our merry way.

Oh and lets not forget there are Developer’s who choose not to use an IDE, or the times that you are in an environment where you do not have access to your favourite IDE. Maybe that urgent fix for that server that’s on fire at at 4:00am in the morning that you just had to SSH into.

These days we have so many tools at our disposal that they can make us lazy and reliant. As Developers we should always strive to write the best code we possibly can, and avoid the shortcuts that can lead to technical debt.

To finish on I believe it was John Woods who said _“Always code as if the guy who ends up maintaining your code will be a violent psychopath who knows where you live.”_<sup>[1](#quote-source)</sup> Words a Developer should live by, especially in an age where we post our location on social media at every turn.

<a name="quote-source">1</a>: https://groups.google.com/forum/#!msg/comp.lang.c++/rYCO5yn4lXw/oITtSkZOtoUJ
