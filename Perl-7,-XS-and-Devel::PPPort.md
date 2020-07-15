# Perl 7, XS and Devel::PPPort

☛ feel free to edit, comment, react on this documment

## Introduction

As [proposed here](The-Proposal-for-Perl-7), The Perl major version will soon be bumped from 5 to 7, opening the road to 8, 9, ... in the near future. 

## The Problems

When bumping the Perl Major version to 7, this is going to raise two different but related issues with XS code.

1. Incorrect and common usage of `PERL_VERSION` checks in xs code
1. An existing protection in `ppport.h` provided by [Devel::PPPort](https://metacpan.org/pod/Devel::PPPort) to die if the major version is not 5

# Incorrect usage of `PERL_VERSION`

## Description of the problem

It is a common pattern(even if incorrect) to check only `PERL_VERSION` in xs code:

```c
#if PERL_VERSION < 13 || (PERL_VERSION == 13 && PERL_SUBVERSION < 9)
#  define PERL_HAS_BAD_MULTICALL_REFCOUNT
#endif
```
[read more from cpan/Scalar-List-Utils/ListUtil.xs](https://github.com/Perl/perl5/blob/e4543a2055aeeb5ef42eb7a84fae20b71643c972/cpan/Scalar-List-Utils/ListUtil.xs#L137)

The check is incomplete. It is missing a check on `PERL_REVISION`. This code assumes Perl will never exceed version 5.

## Exploring solutions

### Using `PERL_REVISION`

An improved usage of `PERL_VERSION` would always check the `PERL_REVISION`

```c
#if PERL_REVISION == 5 && ( PERL_VERSION < 13 || (PERL_VERSION == 13 && PERL_SUBVERSION < 9) )
#  define PERL_HAS_BAD_MULTICALL_REFCOUNT
#endif
```

### Using a macro to perform the comparison

A much better solution would be to use some alternate macro designed for the job.
If you look at existing XS, these macros can bee seen copied in multiple XS files.

Here is a version of them using multiple flavors depending what check you need
```c
#ifndef PERL_VERSION_DECIMAL
#  define PERL_VERSION_DECIMAL(r,v,s) (r*1000000 + v*1000 + s)
#endif
#ifndef PERL_DECIMAL_VERSION
#  define PERL_DECIMAL_VERSION \
        PERL_VERSION_DECIMAL(PERL_REVISION,PERL_VERSION,PERL_SUBVERSION)
#endif
#ifndef PERL_VERSION_GE
#  define PERL_VERSION_GE(r,v,s) \
        (PERL_DECIMAL_VERSION >= PERL_VERSION_DECIMAL(r,v,s))
#endif
#ifndef PERL_VERSION_LE
#  define PERL_VERSION_LE(r,v,s) \
        (PERL_DECIMAL_VERSION <= PERL_VERSION_DECIMAL(r,v,s))
#endif
#ifndef PERL_VERSION_GE
#  define PERL_VERSION_GT(r,v,s) \
        (PERL_DECIMAL_VERSION > PERL_VERSION_DECIMAL(r,v,s))
#endif
#ifndef PERL_VERSION_LE
#  define PERL_VERSION_LT(r,v,s) \
        (PERL_DECIMAL_VERSION < PERL_VERSION_DECIMAL(r,v,s))
#endif
#  define PERL_VERSION_EQ(r,v,s) \
        (PERL_DECIMAL_VERSION == PERL_VERSION_DECIMAL(r,v,s))
#endif
...
```

The check above could then become
```c
#if PERL_VERSION_LT(5,13,9)
#  define PERL_HAS_BAD_MULTICALL_REFCOUNT
#endif
```

Note that IMO, this is cleaner, easier to read and probably reduces mistakes...

## Impact on CPAN

Many XS modules (152 on CPAN) are using `#if PERL_VERSION` Most do not check for `PERL_REVISION` correctly.
We can explore multiple solutions to fix them when bumping Perl to 7.

1. provide the compare macros via Devel::PPPort - (view work from Karl)[https://github.com/Dual-Life/Devel-PPPort/commit/edccecc51ea9235e17d0a4a8e8a18e48caa3a3f1]
1. patch Perl core itself rather than patching all modules

### Add the compare macros to Devel::PPPort 

I think adding the compare macros to Devel::PPPort is a good think and should be done and released.
Karl already worked on providing such macros:
- [Karl's work](https://github.com/Dual-Life/Devel-PPPort/commit/edccecc51ea9235e17d0a4a8e8a18e48caa3a3f1)
- [doc](https://github.com/Dual-Life/Devel-PPPort/commit/a15c0190a80c319517b97564e436922e664eb3c1)

My understanding is these macros are going to be added to CORE in the next development cycle.

I think we should consider patching all, if not most, of popular modules (high in the CPAN river or dual life) to enforce the usage of these new macros. We should also consider add a documentation section for XS to recommend using these macros.

One minor issue to consider: conflicting redefined macros if they are not protected with `#ifundef`.

Are we considering adding and using these macros to be the only solution?
If so any xs modules on CPAN (or outside) using `PERL_VERSION` without `PERL_REVISION` is going to be broken.

### Can we patch Perl itself?

This is exploring the idea that it would be much harder to patch the whole CPAN (and world) rather than fixing Perl internals during the next development cycle.

Here are multiple suggestions, in no specific order:

1. Freeze the PERL_VERSION, PERL_REVISION, ...
2. Lie to the world... 

#### Freeze the PERL_VERSION, PERL_REVISION, ... variables

These variables could always stay at `5.34.0` for example.
Potentially this could raise a warning when using any of them.

But internally we would stop using them in favor of alternate macros (not sure what would be a better name) which would be used widely in core.

Note: I could see doing it in the reversed way.

i.e.: defining a single variable like `5000320001` or maybe `5.32.1` then having C macro built around them to extract major, minor, build id... 

#### Lie to the world

Bumping `PERL_REVISION` from 5 to 7 is probably fine by itself and I expect the number of XS modules being broken by that simple bump being very limited. (we could probably gather some metrics)

What's problematic is the bump of `PERL_VERSION` from 32 to 0 for example. (decreasing the number)
Suggestion make the `PERL_VERSION` bump from 32 to 7000 (for example), and adjust internaly where it makes sense to remove 7000 for `~cosmetic` usages. Example with `perl -V`.

With such a solution, I expect that it would not be mandatory (even if recommeneded) to patch all the XS distribution out in the world.

### Exploring other ideas?

.... 

# Protections in `ppport.h`

The second issue we are facing comes from an internal protection in `ppport.h` itself which breaks if `PERL_REVISION` is not 5. Over 1017 XS modules on CPAN include a `ppport.h` that needs to be updated and a re-released to CPAN.

For example `Class-XSAccessor-1.19` comes with [ppport.h](https://metacpan.org/source/SMUELLER/Class-XSAccessor-1.19/ppport.h) 
This has been fixed by Karl via [this commit](https://github.com/Dual-Life/Devel-PPPort/commit/293861b9234e45fb1a3018959caaab2086bc6fbd).

The main issue was coming from the check in [parts/inc/version](https://github.com/Dual-Life/Devel-PPPort/blob/v3.58/parts/inc/version#L49)
```c
/* It is very unlikely that anyone will try to use this with Perl 6
   (or greater), but who knows.
 */
#if PERL_REVISION != 5
#  error ppport.h only works with Perl version 5
#endif /* PERL_REVISION != 5 */
```

## Questions / Exploring Solutions

1. Why `ppport.h` need to be shipped with a distro?
2. Could Devel-PPPort start shipping `ppport.h` in a shared_lib directory which could then be consume by other modules?
3. Could we envision a toolchain patch to automatically update `ppport.h` when detecting a version?

Is patching the whole CPAN the only way?
What other solutions can we consider to try to solve that problem?

