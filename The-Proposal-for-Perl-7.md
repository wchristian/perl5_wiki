This is the proposal with additional details as presented by [xsawyerx](xsawyerx) at [The Conference in the Cloud 2020](https://perlconference.us/tpc-2020-cloud/).

[Sawyer's original announcement in p5p](https://www.nntp.perl.org/group/perl.perl5.porters/2020/06/msg257565.html).

There are additional details in Sawyer's [update about versioning](https://www.nntp.perl.org/group/perl.perl5.porters/2020/07/msg257817.html).

brian's [perl7faq](https://github.com/briandfoy/perl7faq/blob/master/README.md).

# Intro

...

# Perl Versioning

Perl 5 operates mostly on a [semantic versioning system](https://semver.org/) with the exception that development versions will have an odd numbered minor version

1. MAJOR version when you make incompatible API changes,
1. MINOR version when you add functionality in a backwards compatible manner, and
1. PATCH version when you make backwards compatible bug fixes.

# Perl 7.0.0

7.0.0 will be Perl 5.32.0 but enforce a [Perl Protocol Declaration](Perl-Protocol-Declaration) before any code is parsed.

# Dual Life modules

One of the goals of 7.0 will be to eat our own dog food. As much code as possible will be brought up to 7.0 standards to set an example of how code should be used. This also provides a platform to prove that the standards work without issue.

Some dual life modules may present a challenge using 7.0. In some cases, it may be necessary to turn off some features temporarily until a better fix can be found. As an example, prototypes are only compatible with subroutine signatures as of Perl 5.20.0. So Test::Simple would either have to stop using prototypes or have `use if $] < 5.20 no feature 'signatures';` in its code. 

As part of the research for this project we have reached out to some authors and found some of the CPAN maintained dual life modules to be unmaintained. On a case by case basis, it may be wise to move the modules from cpan/ to dist/ in order to provide better maintenance.

# Requirements for shipping Perl 7

The goal for 7.0 is to keep the scope as limited as possible to assure a rapid release of 7 so development of 7.1 can start as soon as possible. The more scope we add to 7.0, the more risk we add that it cannot be delivered in a timely fashion.

1. Agree on the [new defaults for Perl 7](Defaults-for-v7).
1. Update pod/ policy documents to state how development has changed.
1. Agree on how -e and -E will work (-5 by default)
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

# Optional items for Perl 7.

1. Provide tooling to auto-upgrade code to be compatible with Perl 7.
    * Convert prototypes to use `:prototype` so they are compatible with subroutine signatures
    * detect and warn about subroutines named fc, say, state, switch, etc. (depending on what p7 defaults are chosen)
1. Have a plan for documentation changes (7.1?)

# FAQ

---

**Q:** What about github.com/perl/perl5 is it named wrong now?

**A:** Yes it is. Github will let us rename the repo and redirects should work correctly.

---

**Q:** Is the idea that major numbers will be incremented every so often, like say, the linux kernel? or do you think it will be perl7 for a long time? just asking out of curiosity

**A:** Perl 8 will likely come in the next 3-5 years. No specific plans are in place beyond that.

---

**Q:** 

**A:** 

---

