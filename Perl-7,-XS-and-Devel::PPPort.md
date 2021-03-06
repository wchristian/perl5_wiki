# Perl 7, XS and Devel::PPPort

☛ feel free to edit, comment, react on this documment

## Introduction

As [proposed here](The-Proposal-for-Perl-7), The Perl major version will soon be bumped from 5 to 7, opening the road to 8, 9, ... later on. 

## The Problems

When bumping the Perl Major version to 7, this is going to raise two different but related issues with XS code.

1. Incomplete and common usage of `PERL_VERSION` checks in xs code (without checking `PERL_REVISION`)
1. An existing protection in `ppport.h` provided by [Devel::PPPort](https://metacpan.org/pod/Devel::PPPort) to die if the major version is not 5

# Incomplete usage of PERL_VERSION

## Description of the problem

It is a common pattern(even if incomplete) to check only `PERL_VERSION` in xs code:

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
If you look at existing XS, these macros can be seen copied in multiple XS files.

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

# Impact on CPAN

## Protections in `ppport.h`

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

### Questions / Exploring Solutions

1. Why does `ppport.h` need to be shipped with each CPAN distro?
    - This is similar mistake we made with [Module::Install](https://rt.cpan.org/Public/Bug/Display.html?id=118958)
2. Devel-PPPort could start shipping `ppport.h` in a shared_lib directory which could then be consumed by other modules
3. Should toolchain patch CPAN installs automatically update `ppport.h` when detecting a dated copy of the header file?
    - Is patching all of CPAN the only way?
1. What other solutions can we consider to try to solve that problem?

## Incomplete PERL_REVISION checks

Many XS modules (152 on CPAN) are using `#if PERL_VERSION` Most do not check for `PERL_REVISION` correctly.

There are several ways we have thought of already to fix this problem. There may be more:

1. Provide the compare macros (`PERL_VERSION_GT(5,21,2`) via Devel::PPPort - [view work from Karl](https://github.com/Dual-Life/Devel-PPPort/commit/edccecc51ea9235e17d0a4a8e8a18e48caa3a3f1)
1. Patch Perl core itself rather than patching all modules

### Compare macros

Karl has already worked on providing these macros:
- [Karl's work](https://github.com/Dual-Life/Devel-PPPort/commit/edccecc51ea9235e17d0a4a8e8a18e48caa3a3f1)
- [doc](https://github.com/Dual-Life/Devel-PPPort/commit/a15c0190a80c319517b97564e436922e664eb3c1)

These macros will also be added to CORE in the next development cycle.

I think we should consider patching all, if not most, of popular modules (high in the CPAN river or dual life) to enforce the usage of these new macros.

We should also [add a documentation section for XS](https://github.com/Perl/perl5/tree/blead/pod) to recommend using these macros.

Conflicting redefined macros may be a problem. Example: [version.pm defines PERL_VERSION_LT](https://metacpan.org/source/JPEACOCK/version-0.9924/vutil/vutil.h#L81). At first glance, it looks like this will only result in a compiler warning which might actually be a good thing!

This solution only fixes the issue if authors update the `ppport.h` they ship with their module.

I think adding the compare macros to Devel::PPPort is a good thing and should be done and released.

### Patch Perl 7

Patching CPAN is hard. We are still working on fixing modules on CPAN which assume `.` is in `@INC`. 3 years later, 5000 modules on CPAN make incorrect use of Module::Install and are broken for the basic use case of `perl Makefile.PL; make install`

The following approach would allow any existing XS code to warn but continue to work as it did previously in Perl 7 and beyond.

1. Make these constants forever stuck at:
   - `PERL_REVISION  = 5`
   - `PERL_VERSION   = 255` # Something high.
   - `PERL_SUBVERSION= 255`.
1. Replace these constants in core with: `PERL_VERSION_MAJOR`, `PERL_VERSION_MINOR`, `PERL_VERSION_PATCH`
    - Discourage their use outside of core (update pod somewhere)
1. Encourage the usage of `PERL_VERSION_GT` macros instead.
1. Update `Devel::PPPort` 
    - Provide version comparison (`PERL_VERSION_LE`, etc.) macros for versions < 7.
    - Provide a warning on legacy use of `PERL_REVISION`, `SUBVERSION`, `REVISION`, etc.

ppport.h would do something like this:
```c
#ifndef PERL_VERSION_MAJOR <--- we're on Perl 5
# define GT macros.
#else
... Do nothing
#endif
```

### Versioning schemes from other langs

- Python uses [MAJOR.MINOR.MICRO.RELEASE.SERIAL](https://docs.python.org/3/c-api/apiabiversion.html)
    - Using each version as a byte and using hex to show the version is certainly interesting and makes comparisons easier.
- Ruby does [MAJOR.MINOR.TEENY.PATCH](https://www.ruby-lang.org/en/news/2013/12/21/ruby-version-policy-changes-with-2-1-0/)
- Semver.org recommends [MAJOR.MINOR.PATCH](https://semver.org/)

# Comments

## From tonyc:

As to the version macros I'm not entirely comfortable with making the old PERL_REVISION macros lie, but I don't strongly object to it if it's going to keep backward compatibility that well.  Will `$]` also return 5.999999?

> answer from Nico
> `$]` is going to stay accurate, we are only changing the macro to avoid XS breakage
> > ./perl -Ilib -E 'say $]'
> > 7.001000

## From ether:

- remove PERL_VERSION and PERL_REVISION macros in p7 core.
- add PERL_VERSION and PERL_REVISION macros in new Devel::PPPort (shipped with v7 core (and also made available to v5 distributions) - under v7, PERL_VERSION will be fixed to 5 and PERL_REVISION to some high value (as suggested above). under v5 they retain their real values. But add deprecation warnings to them to push authors to switch to the new macros where major and minor versions must be tested together.

> answer from Nico
>> This is a great idea & sounds something we could do in blead once bumped to 7.1.0
>> we should deprecate PERL_VERSION & co including PATCHLEVEL and SUBVERSION
>> they only belong to D-PPP as deprecated

I would also add a check to all ppport.h versions starting today, advising that the author upgrade if the file is over N days old (with N to be bikeshed but a value of something like 60 to 90 days might be reasonable, as Devel::PPPort sees frequent releases). This requires embedding the release timestamp into the file, so it won't work for any existing ppport.h files in the wild of course.  Or, alternatively, the age of the file could be inferred from the latest blead version that the file supports. but since changes need to be made anyway to add these checks, we might as well be direct and use a timestamp.

(Note: a common trick used in toolchain to do author-only warnings, so users installing from cpan do not see the same warnings, is `-d '.git' or !-f 'META.json'`.)

## From LeoNerd:

I'd strongly suggest keeping the variable names on a theme of `MAJOR`.`MINOR`.`PATCH`. As mentioned in https://github.com/Perl/perl5/wiki/Perl-7-Versioning it follows the semver standard.

In addition, it has nice properties that the three components are the same length, and they sort correctly:

```
$ perl -MConfig -E 'm/^PERL_VERSION_/ and print "$_ = $Config{$_}\n" for sort keys %Config'
PERL_VERSION_MAJOR = 7
PERL_VERSION_MINOR = 1
PERL_VERSION_PATCH = 3
```

The proposed other name of `MAJOR`.`MINOR`.`MICRO` does not match semver, and does not sort nicely in this manner.

> answer from Nico
>> This sounds great, would have to synchronize with Karl on this one for D-PPP, but the sooner the better
>> as we are going to rename all the three variables
>> This is a great idea

## From Karl

The values like PERL_VERSION, PERL_MAJOR_VERSION are #define constants, and not variables.  They must be so, as they control what gets compiled and what gets skipped.  It is much harder to deprecate on the use of a preprocessor token, relying on C99 features that are not all that well defined.  xenu has concluded that it is impossible in gcc, but works in clang.  I spent way too long trying to get something to work in gcc, but haven't tried clang.  Is clang in widespread enough use to justify deprecating for just compiles under it?

The idea of putting a timer in ppport.h scares me. D:P wasn't updated for years, and I think that most that can be expected is 4 times a year.  There is also the potential for breakage from new versions, as alh has pointed out.

> answer from Nico
>> The deprecation warning is something nice to have. If we cannot implement it, I do not think it's a big deal
>> I agree with the timer idea, it's going to be either wrong or right... 
>> if network is available we could simply check the last published version on CPAN while using it?
