A module called `Module::Compatibility` is being worked on right now. It will be invoked by CPAN clients but potentially other code. It will do the following like the following, although the specifics may change:

1. Walk a directory tree and find any perl modules or perl scripts. As the first line (after the BOM and `#!`), `use v5` will be inserted.
    * Some string in the perl code or possibly `META.yml` can be used to prevent compat::perl5 from being inserted into it.
2. Find any C or XS code and correct `#if` logic which assumes the major version is Perl 5.

Once this code is run, we believe that all CPAN modules can install on Perl 7 without an issue. This is not necessarily how the migration from 7 to 8 will go. But it provides a quick win and overcomes the major blocker to making it to Perl 7.

# XS Changes needed

There are roughly 150 distros on CPAN that [mention PERL_VERSION](https://grep.metacpan.org/search?size=20&q=%23if+PERL_VERSION&qd=&qft=*.xs). We believe a few regexes can be applied to `.x`, `.c`, and `.h` code by M:C to fix the majority of modules to make them work.

**TODO:** This isn't the only regex. More research needs to be documented here.

There are few enough that it should be easy enough just to test them.