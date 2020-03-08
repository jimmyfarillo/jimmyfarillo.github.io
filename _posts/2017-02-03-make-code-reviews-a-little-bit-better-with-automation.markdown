---
layout: post
title: "Make Code Reviews a Little Bit Better With Automation"
date: 2017-02-03
---

As a software developer on a team that includes other software developers, code
reviews are an important part of the job. It is probably the one duty of my job
that I can expect to do every single day. Why? What is the purpose of code
reviews? Well, code reviews have several purposes. They:

1. Help ensure that new code is comprised of sound logic.
1. Help ensure that new features follow the specifications.
1. Help ensure that new code conforms to the agreed upon code style (if your
   company/team doesn't have a code style guide, you should consider adopting
   one).
1. Act as announcements of new code and features to the other team members /
   maintainers of the code base.
1. Share knowledge of best practices amongst the team.

When a team of people is working and collaborating together in one code base,
code reviews are a great tool for keeping everyone in sync.

However, it is not uncommon for a developer to dread the task of reviewing a
team member's code. Especially when the pull request is lengthy. I am definitely
guilty of avoiding reviewing a large PR for an entire day, having it constantly
in the back of my head that I'll eventually have to "suffer" through the process
of going through the proposed code line-by-line, looking for errors and
optimizations.

And then at some point last year the title of an episode of the [Ruby
Rogues](https://devchat.tv/ruby-rogues) podcast caught my attention:
["Automating Code Reviews with Mindaugas
Mozūras"](https://devchat.tv/ruby-rogues/251-rr-automating-code-reviews-with-mindaugas-mozuras)
. In the episode, they discuss [pronto](https://github.com/mmozuras/pronto), a
Ruby gem that can be used in conjunction with various runners to automate parts
of your code review process. It even works seamlessly with GitHub and can be
configured to automatically add review comments to new PRs. The best part is
that it only looks at the code diff and only adds review comments on lines of
code that were changed.

Very soon after listening to that episode I proposed adding the gem to the main
application that my team works on.

I decided to only use the rubocop runner, since my goal was to only automate the
"conforms to the agreed upon code style" purpose of code reviews. The setup was
really simple, and I even created a separate account to be the pronto commenter.
It is aptly named **prontosaurus**, thanks to one of my friends at work. The
pronto gem runs during the CI build when a PR is opened or pushed to.

There was some initial tweaking that needed to be done once everything was set
up, such as enabling/disabling certain warnings based on my team's code style
preferences. Even now, prontosaurus will sometimes leave comments that we
outright ignore because we don't agree with the style being enforced… and I
should probably open a PR to make those changes…

But overall I think prontosaurus has been a positive addition to our code review
process. We view his/her comments as suggestions rather than as strict rules
that will prevent us from merging a PR.

### The advantages of using pronto:

- Reviewers can focus on looking at the code at a higher level while leaving
  code style to prontosaurus.
- When a bot tells you about little mistakes you made, you feel less ashamed
  than if it were an actual member of your team.
- Learning new code style best practices. There have been several times when
  prontosaurus has commented on my code with some method or coding style that I
  have not heard of. The more you know!
- A cute little dinosaur makes comments on your PRs!

### The disadvantages of using pronto:

- Will most likely require several rounds of tweaking based on your team's
  preferences.
- Sometimes not suitable for very large code changes. For example, when doing a
  find/replace PR, it can touch many many lines of legacy code. The pronto bot
  may have some not very nice things to say about the code style of that code,
  but it may not be the appropriate time to address those comments.

One extra little config that I added was the ability to turn off prontosaurus's
comments by simply adding a "No Pronto" label to the PR. This alleviates the
issue of prontosaurus going on a rampage for large PRs.

If you are interested in implementing the pronto gem for your application, I
recommend listening to ["Automating Code Reviews with Mindaugas
Mozūras"](https://devchat.tv/ruby-rogues/251-rr-automating-code-reviews-with-mindaugas-mozuras)
and checking out the [pronto GitHub repo](https://github.com/mmozuras/pronto).
