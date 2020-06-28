This is the proposal with additional details as presented by [xsawyerx](xsawyerx) at [The Conference in the Cloud 2020](https://perlconference.us/tpc-2020-cloud/)

# Intro

...

# Perl Versioning

Perl operate mostly on a [semantic versioning system](https://semver.org/) with the exception that development versions will have an odd numbered minor version

1. MAJOR version when you make incompatible API changes,
1. MINOR version when you add functionality in a backwards compatible manner, and
1. PATCH version when you make backwards compatible bug fixes.

# Perl 7.0.0

7.0.0 will be Perl 5.32.0 but with different defaults.

The defaults can be simulated with this code on 5.30.0. It would be different per major version of perl 5.

```perl
package compat::perl7;

BEGIN {
    # This code is compiled for 5.030 to give you an idea of how you code will work on Perl 7.
    $] >= 5.030 && $] < 5.031 or die("Perl 5.030 is required to use this module.");
}

sub import {

    # use warnings; no warnings qw/experimental/;
    ${^WARNING_BITS} = pack( "H*", "55555555555555555555555515000440050454" );

    # use strict;
    $^H = 0x1C0207E2;

    %^H = (
        'feature_bitwise'      => 1,
        'feature_current_sub'      => 1,
        'feature_declared_refs'      => 1,
        'feature_evalbytes'      => 1,
        'feature_fc'      => 1,
        'feature_postderef_qq'      => 1,
        'feature_refaliasing'      => 1,
        'feature_say'      => 1,
        'feature_signatures'      => 1,
        'feature_state'      => 1,
        'feature_unicode_eval'      => 1,
    );
}

1;
```

Note that the only changes are the manipulation of `${^WARNING_BITS}`, `$^H`, `%^H`. As a result backwards compatibility for older perl code can be achieved easily by injecting `use compat::perl5` which effectively reverses the new perl 7 defaults for the entire file.

```perl
package compat::perl5;

# use compat::perl5 should be the first thing in your code. Especially before use strict, warnings, v5.XXX, or feature.

BEGIN {
    $] <= 8 or die("This code is incompatible with Perl 8. Please see XXX for more information.");
    if ( $] > 6 ) {
        warn_quietly_once("This code is being run using Perl $]. It should be updated or may break in Perl 8. See YYY for more information.");
    }
}

sub import {

    # no warnings;
    ${^WARNING_BITS} = undef;

    # perl  -e'my $h; BEGIN {  $h = $^H } printf("\$^H = 0x%08X\n", $h); '
    $^H = 0x0;

    %^H = ();
}

1;
```

# CPAN compatibility with Perl 7

A module called `Module::Compatibility` is being worked on right now. It will be invoked by CPAN clients but potentially other code. It will to the following:
1. Walk a directory tree and find any perl modules or perl scripts. As the first line (after the BOM and `#!`), ```use compat::perl5``` will be inserted.
    * Some string in the perl code or possibly `META.yml` can be used to prevent compat::perl5 from being inserted into it.
2. Find any C or XS code and correct `#if` logic which assumes the major version is Perl 5.

Once this code is run, we believe that all CPAN modules can install on Perl 7 without issue. This is not necessarily how the migration from 7 to 8 will go. But it provides a quick win and overcomes the major blocker to making it to Perl 7.

# Dual Life modules

One of the goals of 7.0 will be to eat our own dog food. As much code as possible will be brought up to 7.0 standards to set an example of how code should be used. This also provides a platform to prove that the standards work without issue.

Some dual life modules may present a challenge using 7.0. In some cases, it may be necessary to turn off some features temporarily until a better fix can be found. As an example, prototypes are only compatible with subroutine signatures as of Perl 5.20.0 So Test::Simple would either have to stop using prototypes or have `use if $] < 5.20 no feature 'signatures';` in its code. 

As part of the research for this project we have reached out to some authors and found some of the CPAN maintained dual life modules to be unmaintained. On a case by case basis, it may be wise to move the modules from cpan/ to dist/ in order to provide better maintenance.

# Requirements for shipping Perl 7

The goal for 7.0 is to keep the scope as limited as possible to assure a rapid release of 7 so development of 7.1 can start as soon as possible. The more scope we add to 7.0, the more risk we add that it cannot be delivered in a timely fashion.

1. Agree on what defaults will change for Perl 7.
1. Agree on how -e and -E will work.
1. Update blead perl code so that all files loaded by Perl 7 work the new defaults.
1. Update XS to be compatible with perl 7
1. Test dist/ modules and determine if they need to be modified in order to live on CPAN
1. Get as many blead cpan/ authors to update their code to be compatible with Perl 7.
1. Test Perl 7 against CPAN via Module::Compatibility
1. Update cpan clients to work with Perl 7
1. Update pod/ policy documents to state how development has changed.

# Optional items for Perl 7.

1. Provide tooling to auto-upgrade code to be compatible with Perl 7.
    * Convert prototypes to use `:prototype` so they are compatible with subroutine signatures
    * detect and warn about subroutines named fc, say, state, switch, etc. (depending on what p7 defaults are chosen)
1. Have a plan for documentation changes (7.1?)

# Initial proposal for defalts (**FOR DISCUSSION**)

## Features not experimental anymore
1. current_sub
1. postderef
1. postderef_qq
1. signatures
1. lexical_sub

## To be included
1. use strict;
1. use warnings;
    * Are there warnings we should turn off by default?
1. feature bitwise
1. feature current_sub
1. feature declared_refs
1. feature evalbytes
1. feature fc
1. feature postderef_qq
1. feature refaliasing
1. feature say
1. feature signatures
1. feature state
1. feature unicode_eval
1. no feature indirect

## Not to be included

NOTE that anything not included in 7.0, should probably lead to a discussion in 7.1 about if the feature could be improved in some way to make it more generally available down the road.

1. feature_switch
1. use utf8 (`$^H |= 0x00800000;`)
    * This setting is currently very disruptive to several major code bases.
1. unicode_strings / bitwise / unicode_eval
    * I would have loved to add these at 7.x but I don’t know how feasible they may be. Python has tried doing it for a good few years now with continuous difficulties. They’ve resolved to keep releasing versions of 2.x since people kept refusing to upgrade to Python 3.0. Python 2.7.18 is supposedly the final Python 2 release. In short: Subtly changing behavior of functions (which the above do) is not something to be taken lightly. We should discuss these.




# FAQ

**Q:** Why not `use v7` and then default all other code to Perl 7.

**A:** This continues to expect the people least in the know to put "tribal knowledge" in their code it make it run in a more modern way.

---

**Q:** What about github.com/perl/perl5 is it named wrong now?

**A:** Yes it is. Github will let us rename the repo and redirects should work correctly.

---

**Q:** 

**A:** 

---
