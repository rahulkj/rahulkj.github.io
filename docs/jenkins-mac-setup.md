Install Jenkins setup on Mac
---

Had troubles with setting the remote access and changing the port for Jenkins. It was pretty easy to change the default configurations. Here are the steps:

- `brew install jenkins`
- `nano /url/local/opt/jenkins/homebrew.mxcl.jenkins.plist`
- change the `--httpPort=8080`to anything you like, in my case `--httpPort=8090`
- change the `--httpListenAddress` from `127.0.0.1` to `0.0.0.0` to allow remote access
- `ln -sfv /url/local/opt/jenkins/*.plist ~/Library/LaunchAgents`
- `launchctl load ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist`

Now you can access the jenkins instance on port `8090` and you can also access this remotely.

That was quick and easy. Hope this helps you!!
