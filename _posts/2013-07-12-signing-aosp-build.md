---
author: michael
title: Signing Your Custom AOSP Build With Your Release Keys
layout: blog
number: 4
tags:
- android
- aosp
- build
- sign
- release
- key
keywords:
- android
- aosp
- build
- sign
- release
- key
---
<p> The procedure described <a href="http://www.kandroid.org/online-pdk/guide/release_keys.html">here</a> actually works. I only had to edit <code>sign_target_files_apks</code> to change the display property (that was added in 4.2 or 4.1) so the version is correctly identified as a release version. </p>

<p> I am writing this because I searched the net for a solution - even after I found the web page above, as I didn't trust it - and didn't find sufficient proof to support the procedure. </p>

