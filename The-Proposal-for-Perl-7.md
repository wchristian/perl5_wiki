This is the proposal with additional details as presented by [xsawyerx](xsawyerx) at [The Conference in the Cloud 2020](https://perlconference.us/tpc-2020-cloud/).

[Sawyer's original announcement in p5p](https://www.nntp.perl.org/group/perl.perl5.porters/2020/06/msg257565.html).

There are additional details in Sawyer's [update about versioning](https://www.nntp.perl.org/group/perl.perl5.porters/2020/07/msg257817.html).

brian's [perl7faq](https://github.com/briandfoy/perl7faq/blob/master/README.md).

# Intro

7.0.0 will be Perl 5.32.0 but enforce a [Perl Protocol Declaration](Perl-Protocol-Declaration) before any code is parsed. Versioning will proceed according to the [existing versioning plan](Perl-7-Versioning).

# What we need to do before Perl 7

The goal for 7.0 is to keep the scope as limited as possible to assure a rapid release of 7 so development of 7.1 can start as soon as possible. The more scope we add to 7.0, the more risk we add that it cannot be delivered in a timely fashion.

1. Agree on the [new defaults for Perl 7](Defaults-for-v7).
1. Update [pod / policy documents](Changes-to-policies-in-Perl-7) to state how development has changed.
1. Agree on [how -e and -E will work](How--e-works-in-7) (-5 by default)
1. Add use v5 to all perl code
1. Change perl to [require v5/v7 protocol specification](Perl-Protocol-Declaration)
1. Fix fresh_perl issues.
1. Fix all blead XS to support `PERL_MAJOR=7`
1. [Module::Compatibility](Making-CPAN-work-on-Perl-7)
1. Update core perl code to use v7 where no changes are required.
1. Update cpan clients and EUMM/M::B to work with Perl 7
1. Testing
    1. Test Suite passing
    1. Test Perl 7 against CPAN via Module::Compatibility
1. RELEASE 7.0.0

# After Perl 7.0.0 is released

## Immediate Changes

1. Merge `v5.32.0..blead7` commits into blead which has had ongoing development during this time.
1. Bump blead to `7.1.0` and release it
1. Bump blead to `7.1.1`

## Next steps in 7.1

1. Review and update code in blead to convert `use v5` code to `use v7`
    1. Review modules in `cpan/` and address orphaned code where necessary.
1. Provide tooling to auto-upgrade code to be compatible with Perl 7.
    * Convert prototypes to use `:prototype` so they are compatible with subroutine signatures
    * detect and warn about subroutines named fc, say, state, switch, etc. (depending on what p7 defaults are chosen)
1. Make and execute a plan for documentation changes (7.1?)

# FAQ

---

**Q:** What about github.com/perl/perl5 is it named wrong now?

**A:** Yes it is. Github will let us rename the repo and redirects should work correctly.

---

**Q:** Is the idea that major numbers will be incremented every so often, like say, the linux kernel? or do you think it will be perl7 for a long time? just asking out of curiosity

**A:** Perl 8 will likely come in the next 3-5 years. We hope to have [Cor](https://github.com/Ovid/Cor/wiki) in blead by then. No specific plans are in place beyond that.

---

**Q:** 

**A:** 

---
