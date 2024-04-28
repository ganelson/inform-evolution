# (IE-0038) Division of time

* Proposal: [IE-0038](0038-division-of-time.md)
* Discussion PR link: [#38](https://github.com/ganelson/inform-evolution/pull/38)
* Authors: Graham Nelson
* Status: Accepted
* Related proposals: [IE-0037](0037-relative-kinds.md)
* Implementation: Implemented but unreleased

## Summary

Inform's existing kind `time` is split into two different concepts. `time`
continues to be used for values like `12:23 pm`, which are possible times
of day, whereas `time period` is introduced as a new built-in kind to
hold `12 minutes`, `minus 34 hours 10 minutes` and so on.

## Motivation

At present Inform has the awkward convention that `time` can hold _both_
times of day, and also time periods. The Inform kind system more or less
forced this convention on us, but see [IE-0037](0037-relative-kinds.md),
which provides a way out.

The existing convention is confusing for authors. It is downright strange
that `showme 10 minutes plus 5 minutes` produces `time: 12:15 am`, for
example.

Almost all declarations of properties or phrases which involve `time`
understand this as _either_ a time of day _or_ a time period, and so the
fact that existing Inform allows these to be confused just leads to
potentially strange situations. For example,

	To decide which time is the time since (sc - scene) began: ...

clearly ought to return a time period, whereas

	To decide which time is the time when (sc - scene) began: ...

clearly ought to be a time of day. There's no good sense in which either
of these phrases is simultaneously meaningful in both ways. Or consider:

	To decide which time is (t - time) before (t2 - time): ...

This allows `3:04 pm before 25 minutes` as a valid phrase, for example.
Clearly `t` is intended to be a time of day and `t2` a time period, and
the result should be a time of day.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Minor changes to runtime kits.
- [x] Small but consequential changes to the Standard Rules, but not Basic Inform.
- [x] Minor changes to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

This is a breaking change, and a handful of the Inform examples needed to be
corrected to allow for it. On the other hand, the corrections were all minor
and fairly obvious. For example, the example "MRE" featured food items with
a property for how long they keep you fed, in minutes: that had the kind
`time`, and the example immediately worked again once this was changed
to `time period`.

The issue is mitigated somewhat with an explanatory problem message for
kind mismatches as between `time` and `time period`.

## Details

As noted above, a new built-in kind `time period` is introduced. This has
dimension, whereas `time` becomes dimensionless, and the Standard Rules now
include:

	A time minus a time specifies a time period.

Inform previously accepted constant literals such as `12 minutes` and `4:12 pm`
as `time` values, and now accepts the first sort as `time period` values
instead.

The following changed phrase definitions are then made:

	To decide which time is (t - time period) before (t2 - time)

	To decide which time is (t - time period) after (t2 - time)

	To decide which time period is (n - number) hours

	To decide which time period is (n - number) hours (m - number) minutes

	To (R - rule) in (t - time period) from now

	To decide which time period is the time since (sc - scene) began

	To decide which time period is the time since (sc - scene) ended

In each case the only change is that a usage of `time` has become `time period`.

In addition, the Understand token `"[a time period]"`, which already existed
and matched fragments of commands like `2 MINUTES`, now produces a `time period`
instead of a `time`. This means that stories using this token now need to use
the result as `time period understood`, not `time understood`.

Because `"[a time period]"` is no longer a special case, this line has been
removed from the Standard Rules:

	The understand token a time period is defined by Inter as "RELATIVE_TIME_TOKEN".

## Alternatives considered

It wasn't entirely clear how to name the two kinds. There would be a case
for removing `time` and having two entirely new kind names, perhaps `time of day`
and `time period`. But we already have a variable called `time of day`.
Even if we called it something like `clock time`, the two would be mixed up.
It seemed best to keep `time`, especially as in many uses in existing code
the kind continues to work as it should.

For the new kind, `interval` or `duration` were both possibilities, but
these are relatively common words to be redefining, and they both have other
interpretations (such as the interval at the theatre). `diurnality` was less
likely to cause name-clashes, but also less familiar to authors born after
1450.

A similar concept had already been called `elapsed time` in the Metric Units
extension, so to avoid nameclashes that wasn't chosen. `time difference` is
suggestive of time zones, so that seemed awkward.

The tie-breaker in favour of `time period` was simply that Inform had the
`Understand` grammar token, `"[a time period]"`, already as a special case.
