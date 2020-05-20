---
layout: my-post
title: Git directory configuration (conditional includes)
date: 2020-05-20 21:37
---

On my laptop I have two types of Git repositories.  Some of them are open source projects, to which I may occasionally (try to) contribute.  There are also my daily work repositories which belong to a company I work for.  If I happen to make some changes and mean to publish them, my name along with my email address appear in commits.  For open source projects, preferably I use my private email, while for the company projects it's a requisite company's email address.

Currently, in my global Git configuration the default email address is my private one.  But now for each company's repository I had to configure the company's email address.  This was annoying and quite often I forgot about it and even submitted my changes for a review with the wrong email address.  Also in the past, when my company email address was the default one, I had the same situation with open source projects.

I thought there must be a way to get this right.  I already had repositories organised with all the open source ones stored under `~/sources/oss`, while the company ones under `~/sources/company`.  So I wanted to tell Git that anything under `~/sources/company` should use my company email address.

Some time ago I saw on Mastodon someone suggesting a convenient alias to improve Git skills by checking some more esoteric, but quite often extremely useful features, on a random basis:

{% highlight terminal %}
alias git-rtfm='man $(find /usr/share/man -name "git-*" | sed "s/.*\///;s/\..*//" | shuf -n1)'
{% endhighlight %}

Honestly, I'm not there yet to read a single Git man page a day, but at least it's something always at the back of my mind:  if I know what I need, I'll likely find it.  So this time, `git-config`'s [conditional includes](https://git-scm.com/docs/git-config#_conditional_includes) and [gitdir](https://git-scm.com/docs/git-config#Documentation/git-config.txt-codegitdircode) to the rescue.  I ended up with this in my `~/.config/git/config`:

{% highlight terminal %}
[user]
	name = Krzysztof Konopko
	email = krzysztof.konopko@amk.zone

[includeIf "gitdir:~/sources/company/"]
	path = ~/sources/company/company-git.inc
{% endhighlight %}

and `~/sources/company/company-git.inc`:

{% highlight terminal %}
[user]
	email = kris@company.tld
{% endhighlight %}

Simples!