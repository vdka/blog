---
title: System Headers
author: Ethan Jackwitz
date: '2018-03-19'
categories:
tags:
  - macOS
  - Xcode
---

# Do Not Touch System Headers

## But if you do it's ok

Editing system headers can provide the most mind boggling errors imaginable. Especially if what you edit a macro.

I reconfirmed this while playing around with the definition of the `NS_ENUM` macro in `CoreFoundation`. I deleted it, got distracted and left the header file without thinking about it any further. Then suddenly I nothing would build for me.

![Lots of errors](/img/SystemHeadersErrors.png)

It took me probably 15 minutes before remembering I was playing around in system headers. I immediately knew what I had done. Desperate for solutions I looked for options for repairing your system headers, none were found, but I did stumble upon [phracker/MacOSX-SDKs](github.com/phracker/MacOSX-SDKs)

I thought I may be able to replace my SDK with the one located there. Knowing that's a definite bad idea since I have no idea if any alterations have made their way into those SDK's.

Instead I diff'd the two SDK's using [Kaleidoscope](kaleidoscopeapp.com). `diff` would also work

    diff -r Downloads/MacOSX10.13.sdk /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.13.sdk | less

![5 line diff's can be the source of hours of potential pain](/img/SystemHeadersDiff.png)

![Just for the future](/img/SystemHeadersFuture.png)
