---
date: 2020-05-15 02:00:00
layout: post
title: "How to Hack HackTheBox?"
subtitle:
description: Want to improve your Pentetration Testing skills? Join Hack The Box Today! 
image: /assets/img/uploads/joinhackthebox/htb.jpg
optimized_image: /assets/img/uploads/joinhackthebox/htb-optimized.jpg
category: walkthrough
tags: hackthebox tutorial
author: madhavmehndiratta
paginate: false
keywords: hackthebox, hackthebox invite code, how to hack hackthebox, hackthebox signup, how to login in hackthebox, hackthebox walkthrough, htb invite code, infosec articles
---

<p align="justify">
For those who do not know about <a href="https://www.hackthebox.eu/">Hack The Box,</a> it is an online platform that allows you to test and improve your skills in Penetration Testing. The platform contains assorted challenges that are updated continuously. Some of the challenges simulate real world scenarios, while others are more like CTFs. Needless to say, Hack the box is beyond resourceful if you want to level up your pentesting skills; especially as a beginner. </p>

<center><br>
<img src="/assets/img/uploads/joinhackthebox/hackthebox.png">
</center>

<p align="justify">
But the fun part is that you cannot just go to the site and start using it. You have to take the Invite Challenge first and hack your way in to get the invitation code and create an account on the website. This sounds intimidating right ? Don’t worry, this article will help you to achieve this goal. However, I highly recommend that you first try yourself to hack in(on your own), and only use this article as a guide in case you need help. </p>

First, visit the official <a href="https://www.hackthebox.eu/">Hack the Box</a> website. As you scroll down to read more information, you will see a join button. 

This will take you to the <a href="https://www.hackthebox.eu/invite"> invite challenge.</a>

<center><br>
<img src="/assets/img/uploads/joinhackthebox/invitepage.png">
</center>

Let’s begin by analyzing the source code. Just right click on the page and go to ‘Inspect Element’ (or simply press Ctrl+Shift+I)

<center><br>
<img src="/assets/img/uploads/joinhackthebox/inspect.png">
</center>

We can see some javascript files in the end. Let's have look at <i>inviteapi.min.js</i>

<center><br>
<img src="/assets/img/uploads/joinhackthebox/inviteapi.png">
</center>

It seems like there are some runnable functions such as <i>makeInviteCode</i> and <i>generate</i> etc.

```js
//This javascript code looks strange...is it obfuscated???
eval(function (p, a, c, k, e, r) {
  e = function (c) {
    return c.toString(a)
  };
  if (!''.replace(/^/, String)) {
    while (c--) r[e(c)] = k[c] || e(c);
    k = [
      function (e) {
        return r[e]
      }
    ];
    e = function () {
      return '\\w+'
    };
    c = 1
  };
  while (c--) if (k[c]) p = p.replace(new RegExp('\\b' + e(c) + '\\b', 'g'), k[c]);
  return p
}('0 3(){$.4({5:"6",7:"8",9:\'/b/c/d/e/f\',g:0(a){1.2(a)},h:0(a){1.2(a)}})}', 18, 18, 'function|console|log|makeInviteCode|ajax|type|POST|dataType|json|url||api|invite|how|to|generate|success|error'.split('|'), 0, {
}))
```
Let’s first try running the <i>makeInviteCode</i> function. We can execute any function in the Console tab. Input the name of the function that you want to execute and press Enter key.

<center><br>
<img src="/assets/img/uploads/joinhackthebox/console.png">
</center>

What the hell is that Skull image?

Anyway if we see the response of the function that we executed, we get a hash which is base64 encoded.

<center><br>
<img src="/assets/img/uploads/joinhackthebox/response.png">
</center>

There is also a message which says that we need to decrypt the base64 hash. We can easily do this in our terminal.

```r
m1m3@ubuntu:~$ echo SW4gb3JkZXIgdG8gZ2VuZXJhdGUgdGhlIGludml0ZSBjb2RlLCBtYWtlIGEgUE9TVCByZXF1ZXN0IHRvIC9hcGkvaW52aXRlL2dlbmVyYXRl | base64 -d

In order to generate the invite code, make a POST request to /api/invite/generate
```

The hash says that we need to create a post request to <i>/api/invite/generate</i> to generate our invite code. Again, this can be done in our terminal:

```r
m1m3@ubuntu:~$ curl -X POST https://www.hackthebox.eu/api/invite/generate

{"success":1,"data":{"code":"RUJTTVktUlVCTUktSERCSk4tSUNSRUMtWVJVTUk=","format":"encoded"},"0":200}
```

Again we got another encoded result. This time also it is base64 encoded. (I guessed that from the "=" in the end.)

We can decrypt it in the same way as we did before:

```r
m1m3@ubuntu:~$ echo RUJTTVktUlVCTUktSERCSk4tSUNSRUMtWVJVTUk= | base64 -d

EBSMY-RUBMI-HDBJN-ICREC-YRUMI
```

Now we have our invitation code. You can simply go to the <a href="https://www.hackthebox.eu/invite">invite page,</a> and submit the invitation code you got.

<center><br>
<img src="/assets/img/uploads/joinhackthebox/congratulations.png">
</center>

Now you can scroll down and register on the website. Also, remember that simply copying the invite code from this article won't work because the code expires after use. So you need to create your own invite code :D

That’s it! Thanks for reading. Stay tuned for similar tutorials and much more coming up in the near future!
If you have any queries, you can contact me <a href="/contact">here.</a>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/fengquan">Fengquan Li.</a>
