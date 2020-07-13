# Initial proposal for defaults (**FOR DISCUSSION**)

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
1. no feature indirect

## Not to be included

NOTE that anything not included in 7.0, should probably lead to a discussion in 7.1 about if the feature could be improved in some way to make it more generally available down the road.

1. feature_switch
1. use utf8 (`$^H |= 0x00800000;`)
    * This setting is currently very disruptive to several major code bases.
1. unicode_strings / bitwise / unicode_eval
    * I would have loved to add these at 7.x but I don’t know how feasible they may be. Python has tried doing it for a good few years now with continuous difficulties. They’ve resolved to keep releasing versions of 2.x since people kept refusing to upgrade to Python 3.0. Python 2.7.18 is supposedly the final Python 2 release. In short: Subtly changing behavior of functions (which the above do) is not something to be taken lightly. We should discuss these.
