This is the proposal with additional details as presented by [xsawyerx](xsawyerx) at [The Conference in the Cloud 2020](https://perlconference.us/tpc-2020-cloud/).

[Sawyer's original announcement in p5p](https://www.nntp.perl.org/group/perl.perl5.porters/2020/06/msg257565.html).

There are additional details in Sawyer's [update about versioning](https://www.nntp.perl.org/group/perl.perl5.porters/2020/07/msg257817.html).

# Intro

7.0.0 will be Perl 5.32.0 but [with new defaults](Defaults-for-v7).

# What we need to do to release Perl 7.0.0

The goal for 7.0 is to keep the scope as limited as possible to assure a rapid release of 7 so development of 7.1 can start as soon as possible. The more scope we add to 7.0, the more risk we add that it cannot be delivered in a timely fashion.

1. Discuss
    - Agree on the [new defaults for Perl 7](Defaults-for-v7).
    - Determine how to put perl 7 into perl 5 mode
    - Update [pod / policy documents](Changes-to-policies-in-Perl-7) to state how development has changed.
    - Agree on [how -e and -E will work](How-dash-e-works-in-7)
    - Determine what `use v7` will do.
1. [Fix Perl 7 version checks in XS](https://github.com/Perl/perl5/wiki/Perl-7,-XS-and-Devel::PPPort)
1. Prepare blead to work with the new defaults.
    1. Add `use v5` to all perl code in blead.
    1. Address fresh_perl issues in 7. Ideally by initially injecting use v5 in them.
1. Bump version to 7.0.0 and set new defatuls.
1. Testing
    1. Test Suite passing
    1. Test Perl 7 against some portion of CPAN.
1. RELEASE 7.0.0

# After Perl 7.0.0 is released

## Immediate Changes

1. Merge `v5.32.0..blead7` commits into blead which has had ongoing development during this time.
1. Fix use v5 statements to be in sync with development work that happened in blead for 5.33 cycles.
1. Bump blead to `7.1.0` and release it
1. Bump blead to `7.1.1`

## Next steps in 7.1

1. Review and update code in blead to convert `use v5` code to `use v7`
    1. Review modules in `cpan/` and address orphaned code where necessary.
1. Provide tooling to auto-upgrade code to be compatible with Perl 7.
    * Convert prototypes to use `:prototype` so they are compatible with subroutine signatures
    * detect and warn about subroutines named fc, say, state, switch, etc. (depending on what p7 defaults are chosen)
1. Make and execute a plan for documentation changes (7.1?)
1. Make a plan for Perl "5" things
    * Environment variables **PERL5LIB**
    * Perl 5 Porters
    * Perl 5 debugger.

More questions? [Try the FAQ!](Perl7-FAQ)
