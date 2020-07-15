# Intro

This document attempts to summarize where everyone is at in their preferred plan for Perl 7. If you are mentioned here and do not agree, please reach out to us or edit the document. Parsing everyone's changing opinions over time on multiple mediums is very difficult to do accurately.

At this time we have no information on where these people stand because they have not responded in any known forum.

Ilmari, Steve Hay, Slaven Rezic, Craig Berry, Father Chrysostomos, Zefram, Aaron Crane

## In Perl 7 the defaults change


Pro: Nicolas R., Jim, Sawyer, Todd, Tux, cPanel, Ovid, Karl

Ok with Sawyer's decision: Yves Orton, Stevan Little, John Lightsey, Chris Williams (bingos), Nick Clark, David Golden, Hugo

- Perl 7 defaults to [Perl 7 behaviors](Defaults-for-v7) when code is first parsed.
- Perl 7 does not break well behaved code.
    - Most code on CPAN will install and function.
    - All critical code on CPAN will  install and function with minor patches that are both compatible with 5 and 7.
    - *No widespread breakage of the perl ecosystem*
    - No fork between perl5 and perl7 community and ecosystem
- A means to get some of the older semantics on demand will exist
     - use compat::p5 switches the parser to parse code exactly like it would have in perl 5.32.0
- Perl 5 goes into maintenance mode. There is no fork.
- distros May have `/usr/bin/perl5` and `/usr/bin/perl7`
    - This is ok. They're used to that when transitioning to a new major version.

## v5 implied a year then v5/v7 required in all code.

Tony, PPI folks, Editor Parsers, display parsers (github)

- use v5 is implied for 7.0.0 in ~45 days
- require both v5/v7 for perl 7.2.0 June 2021
- There's never confusion parsing code what protocol you're looking at.

## module or class

Paul

- Add `module` (and eventually `class`) keywords; similar to `package` but implies current major version inside
- Code still using the older `package` to be considered legacy v5 and will one day be removed

## v5 implied until vX is seen

Leon, Karen, Dave M

- Defaults do not change
- No widespread breakage of the perl ecosystem
- **No fork between perl5 and perl7 community and ecosystem**
- Only one /usr/bin/perl
- perl7 will use perl5 defaults, explicit use v7 for greater messaging and usage of new features
