---
title: Instance Pudding
author: Dan Buch
twitter: meatballhat
layout: post
created_at: Mon 22 Dec 2014 18:00:00 UTC
permalink: 2014-12-22-instance-pudding
---

<figure class="small right">
  <a href="https://github.com/travis-ci/pudding"><img alt="Tanita Pudding!" src="/images/2014-12-22-pudding.jpg"></a>
</figure>

This is a very exciting time for infrastructure and automation at Travis CI. As you may have noticed in a recent [blog
post](2014-12-17-faster-builds-with-container-based-infrastructure), we've begun moving some of our Linux build capacity
to EC2.  With this change come new challenges and opportunities, and we've been busy building up tooling around how we
interact with the infrastructure.

## the importance of chat

Before delving too much into what we've built, it's worth mentioning that we like to talk to each other.  A lot.  I'm
personally very thankful for this as a remote employee, since it means I'm able to lurk in our various Slack channels or
read through the backlog and have a decent idea of what's going on.  In addition to conversations my (amazing!)
teammates are having, I get to see activities from GitHub, Twitter, HelpScout, PagerDuty, Librato, and Papertrail, among
others.  This is all part of our belief in the power of [chatops](https://speakerdeck.com/jnewland/chatops-at-github).

Those familiar with chatops will know that it's not all about having alerts and such coming into the stream of
conversation.  Executing tasks from chat instead of context switching to a console and reporting that you've run some
commands is important both for visibility and knowledge sharing, not to mention being incredibly handy when all you've
got is your phone.

In preparation for moving Linux builds onto the EC2 infrastructure, we decided that the existing tools for managing EC2
instances simply weren't going to cut it.  The command line tools had to be cloned down and configured, and the docs and
help system were next to nonexistent.  Rather than making the command line tooling setup and usage more user-friendly,
we decided to instead put effort into rebuilding the functionality in the form of an API.

## the start of pudding

The first draft of what is now [pudding](https://github.com/travis-ci/pudding) started out as a port of the command line
scripts we'd been using to manage our EC2 instances.  The basic steps were to create a security group, create an
instance with the security group, wait until the instance was running, then use an SSH connection to upload some
last-mile configuration.  In the process of porting to pudding, we ended up switching from waiting for SSH to instead
using [cloud init](https://help.ubuntu.com/community/CloudInit) to perform such configuration.

As we built the pudding API, we were also building a hubot script which has now been extracted into the
[hubot-pudding](https://github.com/travis-ci/hubot-pudding) repo.  Being able to get feedback on our EC2 estate in chat
became addictive, and we started adding Slack integrations to pudding.  In its current incarnation, starting an
instance via pudding results in notifications from the initial creation request, instance start, and at the completion
of cloud init running the user data script.

![hubot pudding usage](/images/2014-12-22-hubot-pudding-usage.png)

## what's next

It's been very handy to be able to manage our EC2 capacity via chat, but we know that we can do much better.  For
starters, the process of starting and terminating individual instances has not scaled well.  We also have the problem of
operating well under capacity for easily 75% of a given weekday, and all weekend long.  The seemingly obvious solution
to this is to use [EC2 autoscaling groups](http://aws.amazon.com/autoscaling/). Unfortunately, the nature of our
workload (running everyone's tests!) is not a great match for the default autoscaling mode of operation in which
instances are terminated immediately when scaling in.

What we're working to add next is a concept of instance pools in pudding, borrowing concepts from autoscaling groups
when it makes sense.  By rolling our own solution (which we don't do lightly), we'll have the level of control we need
to ensure no build jobs are killed mid-run when we're scaling in.  We're also planning to add more commands to the hubot
script for providing things like a summary of all EC2 resources and re-provisioning all instances in a pool with a
newly-baked AMI.

We're also planning to make the pudding codebase even less Travis-specific, as there are currently several bits in there
that only exist because of how [travis-worker](https://github.com/travis-ci/travis-worker) is deployed and configured.
This concern isn't only about shipping an open source project that may be useful to others; it's about taking care to
prevent concept leakage, which so often results in unexpected behavior and maintenance issues (citation needed).

<figure class="small left">
  <img alt="pudding heroku button" src="/images/2014-12-22-pudding-heroku-button.png">
</figure>

In the interest of being able to try out pudding for yourself, and so that we can easily re-provision it if needed,
we've even added a [heroku button](https://blog.heroku.com/archives/2014/8/7/heroku-button)!

Give pudding a shot, or just look at the [source code](https://github.com/travis-ci/pudding), and please provide
feedback.  Happy provisioning!

![pudding terminate instance](/images/2014-12-22-pudding-terminate-instance.png)
