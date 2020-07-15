# Intro

This document attempts to summarize where everyone is at in their preferred plan for Perl 7. If you are mentioned here and do not agree, please reach out to us or edit the document. Parsing everyone's changing opinions over time on multiple mediums is very difficult to do accurately.

At this time we have no informationon where these people stand because they have not responded in any known forum.

Ilmari, Steve Hay, Slaven Rezic, Craig Berry, Father Chrysostomos, Zefram

## Yves Orton, Aaron Crane, Stevan Little, John Lightsey, Chris Williams (bingos), Nick Clark, David Golden, Hugo
- Whatever Sawyer decides.

## In Perl 7 the defaults for `$^H`, `%{^H}, and `${^WARNINGS},` change

[Nicolas R.](https://github.com/atoomic), Jim, Sawyer, [Todd](https://github.com/toddr), Tux, cPanel, Ovid, Karl

- perl 7 defaults to [v7 behaviors](Defaults-for-v7) when code is first parsed.
- Perl 7 does not break well behaved code.
- use compat::p5 switches the parser to parse code exactly like it would have in perl 5.32.0
- Perl 5 goes into maintenance mode. There is no fork.
- distros May have `/usr/bin/perl5` and `/usr/bin/perl7`
    - This is ok. They're used to that.

## Tony
- use v5 is implied for 7.0.0 in ~45 days
- require both v5/v7 for perl 7.2.0 June 2021

## Paul
- Add `module` (and eventually `class`) keywords; similar to `package` but implies current major version inside
- Code still using the older `package` to be considered legacy v5 and will one day be removed

## Leon, Karen, Dave M
- No widespread breakage of the perl ecosystem
- **No fork between perl5 and perl7 community and ecosystem**
- Only one /usr/bin/perl
- perl7 will use perl5 defaults, explicit use v7 for greater messaging and usage of new features

## PPI folks, Editor Parsers, display parsers (github)
- use v5/v7 required.
- There's never confusion parsing code what protocol you're looking at.