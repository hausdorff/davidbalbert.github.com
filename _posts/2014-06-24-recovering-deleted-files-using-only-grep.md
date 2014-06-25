---
layout: post
title: "Recovering deleted files using only grep"
permalink: recovering-deleted-files-using-only-grep.html
comments: true
technique: ■
---


In my college systems class we were required to implement `malloc`.

I spent a week or so on it. No version control &mdash; I was both youthful and arrogant.

After ironing out all the little systems bugs, I began cleaning up the directory to package up and send off for grading. I went to remove something in the same directory that also started with the letter m, and when I hit tab, zsh helpfully completed this to `malloc.c`.

By the time I had noticed, it was too late.

I had just run `rm malloc.c`.


## `grep` to the rescue

I knew that `grep` works on a lot of the virtual filesystems available in *nix, like `/proc`. I thought, "why not `/dev/*` too?"

I figured I could use a command like this to grep over raw disk data &mdash; not over files or anything like that, just on the raw data on disk &mdash; and if successful, I'd have a resonable shot at recovering my homework.

After RTFM'ing for awhile, eventually I ran a command that looked vaguely like this:

```
$ grep --binary-files=text --context=x 'stringfromyourfile' \
    /dev/whateverPartition > someFile.txt
```

The gist of what we're doing here is running grep over the partition `/dev/whateverPartition`, finding the string `'stringfromyourfile'`, and grabbing the `x` lines bookending that string. If you pick `x` to be big enough, you should get the entire file, plus a bit of junk around the edges. Though, of course, the string has to be unique on your disk, or this will fail.

The key to this is actually the flag `--binary-files=text`; that tells grep to run on the binary disk data even though it doesn't really make any sense.

In the end, this proved good enough and I got my homework back. It's a nifty hack that I don't expect to need to use again, though it's worth knowing that in principle it can be done, especially if you need to look for something else in a binary file.