---
layout: post
title:  "homebrew tap for concourse and Pivotal cli's"
date:   2019-06-07 08:52:00 -0600
categories: homebrew tap
---

Downloading and placing the CLI's in the right location on Mac's is always a pain. I was looking to automate the upgrade of CLI's on my mac, and while scripting, I thought, it would be useful, if I created a brew tap, that could install the required cli's for me, and also help others around the globe.

So here is how you can go about installing the `fly`, `concourse`, `om` and `cred-alert` cli's on your mac, using `homebrew`

```
brew tap rahulkj/tap

brew install om fly concourse cred-alert
```

To automate this repo, so it's pinned to the latest released versions from Pivotal and concourse, I wrote a pipeline, that executes every day, and if there is a new version of those cli's available, then it would update the repo.

Hope this helps you all. Happy brewing!
