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
        'feature_switch'      => 1,
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
1. Walk a directory tree and find any perl modules or perl scripts. As the first line (after the BOM and `#!`), `use compat::perl5` will be inserted.
2. Find any C or XS code and correct `#if` logic which assumes the major version is Perl 5.

Once this code is run, we believe that all CPAN modules can install on Perl 7 without issue. This is not necessarily how the migration from 7 to 8 will go. But it provides a quick win and overcomes the major blocker to making it to Perl 7.

