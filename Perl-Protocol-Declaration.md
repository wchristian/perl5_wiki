# Intro

All parsed code must begin with use vX; It is a directive to the interpreter which tells it how to parse the following code.

# Allowed ways to specify the protocol

The following pragmas are required by perl 7 before any code is allowed to be parsed. Comments and whitespace are permissible. -x applies.

```perl
use v5;
use v7;

# This is a placeholder. Open for bikeshed.
use current; # Tell the parser to use the major of the current binary. 
```

# Mixed protocols

The following is allowed but won't technically work on perl 5:

```perl
use v7;
sub foo ($bar) {
    ...;
}

use v5;

package foo;
sub bar ($@) {
    ...;
}

1;
```

# How it works

At this time (5 and 7) it really only affects `$^H`, `%^H`, and `$^WARNINGS` but this may change at a future date.

The behavior can be simulated with this code on 5.30.0. It would be different per major version of perl 5.

```perl
package p7;

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

Note that the only changes are the manipulation of `${^WARNING_BITS}`, `$^H`, `%^H`. As a result backwards compatibility for older perl code can be achieved easily by injecting use compat::perl5 which effectively reverses the new perl 7 defaults for the entire file.

```perl
package p5;

# use p5 should be the first thing in your code. Especially before use strict, warnings, v5.XXX, or feature.

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