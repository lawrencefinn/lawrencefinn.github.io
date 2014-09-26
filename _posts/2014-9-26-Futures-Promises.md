---
layout: post
title: Futures, Promises, and Async Programming on the JVM
---

Futures are becoming a very hot topic in the JVM world.  I have asked many friends and collegues if they know what a future is and how to use it.  
The answers tend to be "sort of" with a few people using them but not knowing how or why, which of course leads to improper use.  I was not very
pleased with the current state of literature to help engineers understand futures better so I figured I would give it a shot.

#Why Futures?#
If you know about asynchronous programming, feel free to skip this section.
In typical programming, all the code you write gets executed serially and in one thread / execution context.  Each command is only executed AFTER the
prior command finishes executing.  
