# Intro

All parsed code must begin with use vX; It is a directive to the interpreter that 

# Allowed protocol specification

The following pragmas are required by perl 7 before any code. 

use v5;
use v7;
use current '5'; 

# Examples

```

use v5;

...; # code
```

```
# use VERSION: # 1. Minimum version # 2. Feature bundle # 3. (NEW) Protocol declaration   # use v8 # Previously: 8.0.0 binary and above # Now:        Any version that supports v8 protocol (including 7.x binary)  # use v5.12: # Previously: 5.12.0 binary and above, feature bundle for 5.12.0 ONLY # Now: ??? # options: #   1. Use 5.12.0 binary and above, feature bundle for 5.12.0 AND THAT'S IT #   2. Use 5.12.0 binary and above, feature bundle for 5.12.0 + implicit v5 protocol   use v5;    # protocol use v5.12; # minimum version, enable feature bundle  ---  use v5.12; # same, but must be FIRST LINE OF CODE in a file  ---  use v7; use v7.2;  ---  # THIS IS NOT OKAY package Foo; use v5.12;                  # file scope use List::Util qw< first >; # package-scoped subroutine package Bar; # 5.12 is in effect Foo::first(...); # but the subroutines from v5.12 are not available  ---  # okay use v5; package Foo;  ---  # also okay use v5.18; package Foo;  ---  # also okay use v5; package Foo; use v5.20;  ---  use strict;
```