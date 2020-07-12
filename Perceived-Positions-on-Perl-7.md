This document attempts to summarize where everyone is at in their preferred plan for Perl 7. If you are mentioned here and do not agree, please reach out to us or edit the document. Parsing everyone's changing opinions over time on multiple mediums is very difficult to do accurately.


No information:  Ilmari, Steve Hay, Slaven Rezic, Craig Berry, Father Christiomos, Zefram
- We don't know where these people stand because they have not responded in any known forum.

Tony Cook, Yves Orton, Aaron Crane, Stevan Little, John Lightsey, Chris Williams (bingos), Nick Clark, David Golden, Hugo
- Whatever Sawyer decides.

[Nicolas R.](https://github.com/atoomic), Jim, Sawyer, [Todd](https://github.com/toddr), Tux, cPanel, Ovid
- perl 7 defaults to v7 behaviors
- use v7 is implied.
- use compat::p5 at the top of files runs all old perl 5 code as normal
- distros will likely have /usr/bin/perl5 and /usr/bin/perl7

Karl, Paul
- use v5 is implied for 7.0.0 in ~45 days
- require both v5/v7 for perl 7.2.0 June 2021

Leon, Karen, Toolchain, Dave M
- use v5 is implied potentially for 5-10 years.
- No old code breaks
- /usr/bin/perl can point to perl7
- New users will have no intuitive way to know that you need to 'use v7' to get the 7 you're running on.

PPI folks, Editor Parsers, display parsers (github)
- use v5/v7 required.
- There's never confusion parsing code what protocol you're looking at.