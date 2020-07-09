# Intro

All parsed code must begin with use vX; It is a directive to the interpreter which tells it how to parse the following code.

# Allowed ways to specify the protocol

The following pragmas are required by perl 7 before any code is allowed to be parsed. Comments and whitespace are permissible. -x applies.

```
use v5;
use v7;
use current; # Tell the parser to use the major of the current binary.
```

# Mixed protocols

The following is allowed but won't technically work on perl 5:

```
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