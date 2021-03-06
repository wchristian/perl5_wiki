### What's Perl 7?

Perl 7 was proposed to be Perl v5.n (n = current version at that time) with saner, more modern defaults. A transitional major version before we get to language-breaking changes. We thought that would be right after v5.32, but some time has passed, and p5p announced that they'd release v5.34 in May 2021. As well, significant resistance to the proposal was mounted requiring more discussion. At this point nobody knows what changes Perl 7 will actually contain.

There's a [Perl 7 proposal](https://github.com/Perl/perl5/wiki/The-Proposal-for-Perl-7) in the wiki for the [perl5 repo](https://github.com/Perl/perl5/).

### What's the state right now?

Perl has made the move to having a three-person Steering Committee instead of the single "pumpking". The Perl Steering Committee controls the future evolution of the language. As of January 2021, they are just getting started and have not made a statement on Perl 7 yet.

### Why not Perl 32 (or 34)?

There was some thought to dropping the 5 from the version and using what we already think of as the major version. However, that would reinforce the idea that it's just a continuation of the Perl 5 line. And, 32 is a big number. 

We could have done many things, but we had to choose one of them. Perl 7 is what we chose.

### What are the new defaults?

We're still talking about that in [Defaults for v7](Defaults-for-v7). We don't know for sure because we don't have Perl 7 yet. These are some of the things already on the list, although there's no guarantee on anything yet:

* enable strict by default
* enable warnings by default
* enable some keywords

### When will this happen?

The Perl Steering Committee has not released any news on Perl 7.

### How can I help?

Right now we're early in planning. The details as we understand them are in [The Proposal for Perl 7](The-Proposal-for-Perl-7). What we know right now is that well behaved CPAN code should just work. 

If you are an author, take a look at your own code. You can see a report on your modules by updating this URL with your own PAUSE ID: https://cpants.cpanauthors.org/author/XSAWYERX

You could also look at this list and provide fixes to the authors who are not yet using strict/warnings in all locations: 
 https://cpants.cpanauthors.org/kwalitee/use_strict

### Will Perl 7 run my Perl 5 code?

If your code can run under Perl 5.n (n = current), you should be able to run it under Perl 7, although you might have to unset some defaults). Perl 5 has removed features that were deprecated even when v5.0 was released in 1994. If you are using those, you may need to fix those too. Trying your code under v5.n will find many of those problems for you.

There may be a compatibility mode to put code sections back into Perl 5 mode. it will possibly be something simple like `use v5` or `use compat::perl5`

But, no one is taking Perl 5 away from you. 

### What happens to Perl 5?

Perl 5 goes into long term maintenance. That can be as long as a decade or more. You'll still get important security and bug fixes, just for a little bit longer than usual.

The current [Perl support policy](https://perldoc.perl.org/perlpolicy.html) guarantees support for the previous two maintenance releases. We intend to extend that for the ultimate Perl 5 release.

### What will `/usr/bin/perl` be?

Distros control what `/usr/bin/perl` is, not Perl development. We can provide a suggestion or guidance, especially in [Configure](https://github.com/Perl/perl5/blob/blead/Configure). Even so, we feel this should be a late decision in the process and advice from distros will be sought before we make any final decisions.

As the user, your decision is likely made for you. If you aren't going to install your own `perl` for your application, you're stuck on whatever the system provides. If you do install your own `perl`, you will get to decide how that happens.

### What happens if Perl 7 doesn't happen?

We are endeavoring to make [the scope of the bump to 7.0.0](The-Proposal-for-Perl-7#what-we-need-to-do-before-perl-7) as small as possible to make sure it does happen. 

If Perl 7 doesn't happen, we get Perl v5.(n+2) and keep doing that.

We already have all of the code, there are simply some details to work out. This is more paper shuffling and administrative work than anything else. The Perl community already has the tools in place to test all of CPAN against any Perl release. Perl 7 is more about process adjustment than new code. Hopefully this will also shake out any issues so make future major version bumps easier to deal with.

We have been planning this for a year. We've been carefully considering how big a bite we can take. It is our goal to have as modest plan as possible in order to ensure success.

### Will feature X be included?

Perl 7 is v5.n with different defaults. You won't see new features and you won't have any taken away from you if you're running on v5.32. If it's already in Perl, you can have it. If it isn't, probably not.

There are many things that people want, a significant amount of excitement is already happening around features we want to see in 7.2. Once that is out, you will already be able to use it by declaring `use v8` in your code.

### Will Perl 7 have a new object system?

Many people are interested in Ovid's [Cor proposal](https://github.com/Ovid/Cor). It will go into Perl when and if it is ready. Until it is merged to blead, no promises will be made. It is important we not cause the progress of the Perl language to stall waiting on any one thing that may never come.

### Will UTF-8 features be on by default?

We don't think so. People from several major companies have already reported in that utf8 feature defaults are the most likely thing to break their code bases. We'd all like to have this, but there's death and destruction in the wake of a big switch like that. Perl has all the features it needs to handle that, but we don't know how much of the community is ready for that.

But, this is not simply enabling some features. Changing the default behavior of encodings is likely to break many programs because the program itself needs to be ready for Unicode (there's a primer in an appendix to the latest [Learning Perl](https://www.learningperl.com). Tom Christiansen answered [Why does modern Perl avoid UTF-8 by default?](https://stackoverflow.com/a/6163129/2766176).

We do think it is time to discuss this problem once 7.0.0 is out. Maybe there's a 3rd choice.

### What about Perl 6?

Perl 6 started as an ambitious effort to completely rewrite Perl. For various reasons, that didn't replace Perl and went off to live a separate life as the language now known as [Raku](https://www.raku.org). The number 7 was next in line.

### So when's Perl 8 coming out?

While we are excited about Perl 7, we're most excited about Perl 8 and beyond. We would like Perl 8 to happen in less than 5 years. It would have several new features enabled by default.

### What about github.com/perl/perl5? Is it named wrong now?

With Perl 7, a repo named perl5 would be weird. Github will let us rename the repo and redirects should work correctly.

### What about the perl 5 porters mailing list.

It will soon be anachronistic to call it this. We don't have an answer for this yet.

### Is the idea that major numbers will be incremented every so often, like say, the linux kernel? or do you think it will be perl7 for a long time?

Perl 8 will likely come 3-5 years after Perl 7. That's a rough idea that's saying "not a year but not a decade". We don't even have Perl 7 yet, so we have no idea what Perl 8 could be.

### How can I find out more about Perl 7?

* [Getting Better, not Getting By](https://www.youtube.com/watch?v=6wPMh-3qYJM), Sawyer's keynote at The Perl Conference in the Cloud
* [The Proposal for Perl 7](https://github.com/Perl/perl5/wiki/The-Proposal-for-Perl-7)
* [Sawyer's announcement on p5p](https://www.nntp.perl.org/group/perl.perl5.porters/2020/06/msg257565.html)

#### External resources
* [Announcing Perl 7](https://www.perl.com/article/announcing-perl-7/) on Perl.com
* [Perl 7 tl;dr](http://blogs.perl.org/users/brian_d_foy/2020/06/the-perl-7-tldr.html)
* [Preparing for Perl 7](https://leanpub.com/preparing_for_perl7), from [Perl School](https://perlschool.com)
