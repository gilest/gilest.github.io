---
layout: post
title: Run ruby services on OS X with chruby & launchd
---

I've been running some ruby services on OS X machines lately and ran into a few roadblocks. Hopefully this will help someone!

# Preparing the server

Getting a fresh ruby installed with the excellent [chruby](https://github.com/postmodern/chruby) and [ruby-install](https://github.com/postmodern/ruby-install) is no problem.

You'll need to have enabled chruby [system-wide](https://github.com/postmodern/chruby#system-wide) in order for it to be loaded during deployment.

Otherwise deployment is fairly straightforward. I got along just fine with the sensible defaults provided by [capistrano](https://github.com/capistrano/capistrano) and [capistrano-chruby](https://github.com/capistrano/chruby).

Now your fresh code and dependencies are all ready to go. This is where it gets interesting.

## Intro to launchd

On a Linux OS you have a number of options: runit, upstart, systemd etc. But to manage services on OS X you really want to be using launchd.

At first it seem a bit counterintuitive, but there are some tools and abstractions that make it a little easier to manage.

* [LaunchControl](http://www.soma-zone.com/LaunchControl/) provides a GUI for managing and creating launchd services. It's especially useful when creating a new service from scratch.
* [lunchy](https://github.com/eddiezane/lunchy) is a wrapper for the not-intuitive launchctl CLI provided by OS X.

A lot of launchd itself is outside of the scope of this post. For a good reference check out the [launchd Tutorial](http://www.launchd.info/).


## Running your ruby code

The key to running a chruby-installed ruby in a launchd job definition is the `chruby-exec` command.

For example, on my system I have `ruby-2.4.0` installed and I want to run a rake task called `roll_call`. I could run the following command.

`/usr/local/bin/chruby-exec ruby-2.4.0 -- bundle exec rake roll_call`

Which translates to the following in a job definition:

```
<key>ProgramArguments</key>
<array>
  <string>/usr/local/bin/chruby-exec</string>
  <string>ruby-2.4.0</string>
  <string>--</string>
  <string>bundle</string>
  <string>exec</string>
  <string>rake</string>
  <string>roll_call</string>
</array>
```

However, running `bundle exec` won't work from outside of a ruby project directory. So set the working directory to the currently deployed project.

```
<key>WorkingDirectory</key>
<string>/Users/giles/roll-call/current</string>
```

This job will now successfully run but in the development environment. Easily fixed by setting an environment variable.

```
<key>EnvironmentVariables</key>
<dict>
  <key>RACK_ENV</key>
  <string>production</string>
</dict>
```
