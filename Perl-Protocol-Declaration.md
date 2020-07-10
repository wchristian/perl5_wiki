# Intro

All parsed code must begin with use vX; It is a directive to the interpreter which tells it how to parse the following code.

# Allowed ways to specify the protocol

To specify the Perl 7 protocol, you specify the `v7` protocol:

```perl
use v7;
```

To fall back to Perl 5.8, specify `v5`:

```perl
use v5;
```

You can get the preview version of the next major version

```perl
use v8;
```

We are still discussing the possibility of providing a pragma `current` which will set the defaults to the major version of the installed Perl. Both the name and the behavior are continuing to be discussed.

```perl
use current; # Tell the parser to use the major of the current binary. 
```

# Manipulating the defaults once they are set

Perl 7 will honor `use v5.32` under `v5` code not not in 7 or later.

You can still turn off defaults in all protocols by doing:

```perl
no strict;
no warnings;
...
```
    
Don't rely on these as a crutch though. If you want to stick to Perl 5 behavior, you can still use Perl 5.

# Backward compatibility with Perl 5

Perl 5 already understands `use v5;`. If Perl 5 sees `use v7`, it will stop with:

```
Perl v7.0.0 required--this is only v5.30.0, stopped.
BEGIN failed--compilation aborted.
```

Which is exactly what should happen with code written for the v7 protocol but run from a perl5 binary.

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




