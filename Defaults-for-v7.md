# Intro

The following defaults have been proposed for Perl 7. They were chosen because the belief is that they would break very little well behaved code. 

One or more audits of these defaults against CPAN should be attempted in order to understand how much of it would break if we chose these defaults.

# On by default

1. `use strict;`
1. `use warnings;`
1. `feature unicode_strings`
1. `feature refaliasing`
```perl
my \$x = \$y;
my \%hash = $hashref;  $hash{key} eq $hashref->{key}
foreach my \%inner (@list_of_hashrefs) {
  print "$inner{thing}"
}
```

# New keywords on by default.

1. `feature state`
1. `feature evalbytes`
1. `feature say`
    - Yes there is evidence it's going to break a little bit on CPAN.
    - This is a highly desired feature so the value outweighs the breaks.
3. `feature fc`
    - it's a short keyword and might conflict with existing subroutines.
4. `feature current_sub`
   - Technically this is a keyword `__SUB__()` -- Sawyer
    
# Features not experimental anymore
1. `current_sub`
1. `postderef`
1. `postderef_qq`
1. `signatures`
1. `lexical_sub`

# Not to be included

1. `use re 'strict'`
    - Could break a big chunk of CPAN.
    - Compile time errors?
    - Low confidence we would have a way to test if the code would break.
1. `feature postderef_qq`
    - This might break existing stuff without warning. How could we determine the scope of risk?
3. `feature declared_refs`
    - Of the people on the call, there was confusion on this item and refaliasing.
    - The docs seemed to be limited. We think they should be improved.
5. `feature bitwise`
6. `feature isa`
    - Too new
7. `no indirect`
    - Too new
    - Initial testing shows a high amount of breakage on CPAN.
    - Should this be a warning first?
8. `feature signatures`
    - Poor compatibiltiy with old prototypes.
10. `use utf8` (`$^H |= 0x00800000;`)
    - This setting is currently well known to be very disruptive to several major code bases.
11. `feature switch`
    - In its current state we are very unhappy with it.
12. `unicode_eval`