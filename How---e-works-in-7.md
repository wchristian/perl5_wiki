 In Perl 5, -e/-E work as follows:

perl -e behaves like an empty perl 5 program.

perl -E behaves the same as perl -e 'use v5.$MAJOR.0'

In Perl 7, ideally perl -e would behave like `perl -Mv7` and perl -E would behave like `perl -Mv8`. The problem is that this would break most of the existing one liners. 