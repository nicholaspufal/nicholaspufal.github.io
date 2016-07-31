---
layout: post
title: "Plan to move from Dropbox to Google Drive? Better think twice"
date: 2014-04-19 22:57
comments: true
---

After the huge price drop announced by Google I decided to give it a try. I have been a Dropbox user for a long time, but its prices versus my usage aren't really compatible.

However, we only realize the pros/cons of a service after taking a closer look and comparing it to another one.

In the follow topics, I will cover some experiments/findings that I made.

## Google Drive doesn't support symlinks

At first glance, one thing that really annoyed me is the fact that Google Drive doesn't support symlinks, i.e. it won't upload to the cloud the content that a symbolic link is pointing to.

It's a tiny thing, but incredibly practical: if I want to backup my Documents folder, I simply create a symlink from that folder to my Dropbox folder and that's it. With Google Drive that's not an option, so you need to do the other way around: move everything to Google Drive's folder, and create a symlink from there to the location where the folder should live. Disappointing, to say the least.

I decided to search a bit on this subject and found this article: [Dropbox and symlinks by Paul Ingraham](http://www.paulingraham.com/dropbox-and-symlinks.html)

According to the author, Dropbox's behavior to follow symlinks might be a pain when it's about symlinking internal folders. Based on that, I decided to try it out and to be fair honest I didn't face any of the issues reported in there. I think that's due to the fact that Dropbox is really smart in the way it indexes chunks of data (more on this next).

## A few use cases

Instead of relying on other people's comments about both services, I decided to experiment with it. I performed some basic actions with both, and wrote down the results.

### Uploading new content to the cloud

Both are about the same, with Dropbox slightly better. You can check the results below:

##### Upload Google Drive installer (binary with 26MB) through the desktop app:
* **Dropbox:** 8min 20sec
* **Google Drive:** 8min 37sec

##### Upload MP3 file with 6MB from the web interface:
* **Dropbox:** 1min 2sec
* **Google Drive:** 1min 4sec

##### Kind of complex directories (2.8MB total size with 121 files and 22 sub-folders):

* **Dropbox:** 3min 10sec
* **Google Drive:** 1min 57sec

Yes, the last test had a big difference, due to the way Dropbox works. Depending on how you look at it, it might be something positive though.

Let me explain it a bit better: when you upload a file to Dropbox it will first split that file into several chunks of data which are then indexed. If some of those chunks already exist in the cloud, Dropbox won't re-upload them. On the other side, Google Drive isn't so smart, it simply uploads whatever you put into its folder - there isn't any kind of analysis running upfront.

That's the why sometimes it might take a bit longer for Dropbox to upload your files.

#### Moving content around (content that was already in the cloud)

Due to the exact same reason mentioned in the latest topic, Dropbox is way faster here. If you move/rename content, Dropbox will realize that the content was already present in the cloud and won't re-upload it. It's just a matter of seconds until it indexes the data and verify it against what is already in the cloud.

With Google Drive, you better grab some coffee and relax.

Benchmark is not really relevant here, as the behavior is completely different - Google Drive will re-upload everything.

### Remove content from the cloud

Google Drive wins, as it doesn't have to play with indexes.

##### Remove a folder with 44 files and 12MB total size:

* **Dropbox:** 33sec
* **Google Drive:** 6sec

### Memory/CPU usage during syncing

Both are CPU hungry. When indexing/uploading content my Dropbox was pretty often using over 150% of my CPU. Google Drive was only a few steps behind, with at most 115%.

What got my attention is the fact that Google Drive uses way more memory than Dropbox when syncing. It was using over 1.5GB after 1 hour! Dropbox was never over 200MB during my tests.

(Btw, I'm using a Retina Macbook Pro Early 2013 running OS X 10.9.2)

### Reliability - is all of my data being copied over?

For most of the time both were syncing everything, however I experienced Google Drive not syncing a particular folder from my projects.

I'm a developer, and in a few projects I had my `.bin` folder being completely ignored by Google Drive. Not sure why though.

I had no feedback from Google's app: they were **really** ignored.

## Final thoughts

I want to keep this post short, so I won't dive into privacy/terms of service concerns, but yes, it was another thing that shocked me. Google's terms are way more broad/generic. The information is also spread through several links, and sometimes it's not really clear what is Google Drive specific - which would be important to mention, as more sensitive information tends to be hosted through Google Drive than in other Google's services.

I suggest you to read both and take your own conclusions.

I know a lot of people that just jumped into Google Drive because its appealing prices. My suggestion would be to think twice and evaluate it pretty well before taking an action.
