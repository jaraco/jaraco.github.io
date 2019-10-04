---
layout: post
title: Tiny Hack—Extracting Small File from a Large Zip File in the Cloud
tags: english, grammar
---

Today I had a problem. I have a large (2GB) file in the cloud (Dropbox) which I suspect has a small file that I need from it. I don't want to download the whole 2GB file to my local machine and I definitely don't want to extract the whole file to my local hard disk just to see if the file is there.

I discovered that Dropbox actually has a nice feature where it will open largish zip files in preview to extract files, but it's limited to .5GB, so not quite large enough for my needs.

I found with [Expandrive](https://www.expandrive.com/) (highly-recommended), I can mount my Dropbox folder without downloading any content, so at least that exposes the target zip file to my local workstation without downloading it.

Now I just need an application to open the zip file locally and explore/extract its contents. On Windows, I would have just opened the file in Explorer, but since I'm on macOS, I've got to track down an alternative.

Since I'm on macOS, I tried [Fuse](https://osxfuse.github.io/). I installed Fuse and through Homebrew installed [fuse-zip](https://bitbucket.org/agalanin/fuse-zip/wiki/Home). From there, I was able to attempt to mount the file, but when I ran `fuse-zip file.zip /Volumes/file`, it blocked for a very long time, as if it had to read the whole file just to set up the mount :(.

I realized I could download some third-party Zip file explorer, but searching online and in the mac App Store, I found no obvious reputable solution. Then I remembered that Python 3.8 has a new routine to make it easier to traverse zip files like pathlib objects, and since [my shell](https://xon.sh/) is already running under Python 3.8, I just started inspecting the file:

```
$ import zipfile
$ root = zipfile.Path('/Volumes/Dropbox/path/to/myfile.zip')
$ list(root.iterdir())
[Path('/Volumes/Dropbox/path/to/myfile.zip', 'Takeout/')]
$ [x.name for x in (root / 'Takeout').iterdir()]
['archive_browser.html',
 'Contacts',
 'Calendar',
 'Tasks',
 'Hangouts',
 'Drive',
 'Mail']
```

Once I found the file I needed, I wanted to save it, so I wrote this quick function:

```
$ def save_text(path):
.     with open(path.name, 'wb') as strm:
.         strm.write(path.read_bytes())
.
```

Then ran it:

```
$ save_text(root / 'Takeout/Drive/some doc.docx')
```

And then I had a copy of the file I needed, transmitting only the bytes of the file and some overhead metadata. Hooray!
