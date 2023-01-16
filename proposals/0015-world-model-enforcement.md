# (IE-0015) World model enforcement

* Proposal: [IE-0015](0015-world-model-enforcement.md)
* Discussion PR link: [#15](https://github.com/ganelson/inform-evolution/pull/15)  
* Authors: Graham Nelson
* Language feature name: --
* Status: Draft
* Related proposals: --
* Implementation: Experiments in progress

## Summary

`WorldModelKit` and the Standard Rules, together with support hard-wired into
the `if` module of the Inform compiler, collectively provide a "world model".
At present it is not as consistent or as well-enforced as we would like, and
this leads to unexpected and unhelpful behaviour. IE-0015 gathers up a
set of changes to improve (though not fully fix) matters in one place, rather
than leaving these in scattered responses to bug reports.

## Motivation

Under the Inform world model, a tree of objects together with their properties
are used to represent spatial layout and containment for a fictional world.

This is not a free-for-all. For example, in the object tree an object representing
a room cannot be the parent of another room, and there are many other such
restrictions. So authors cannot freely rearrange this object tree at run-time.

While these restrictions are usually common sense, and the Inform compiler is
good at making sure the initial state of the world model (as set up by assertion
sentences like `Peter is in the Dining Room`) follows all of the rules, the
Inform documentation is not always explicit, and the run-time code in
`WorldModelKit` does not always ensure that the situation remains compliant.
In some cases, a lack of clarity about what it means to "hold" or "carry" has
also led to linguistic inconsistencies with these verbs.

Zed Lopez has been instrumental in finding some of these issues, and I'm grateful
too for numerous other reporters of related bugs. For example, see Jira issue
I7-2220.

This is an incomplete draft, but what is described below is implemented on
the `master` branch of the repository. These features are not in any release
branch as yet.

## Components affected

- [ ] Minor change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Minor changes to runtime kits.
- [x] Minor changes to the Standard Rules and Basic Inform.
- [x] Minor change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

All of these changes inevitably affect existing source text, and usually they
invalidate something which was a bad idea. For example:

	now the Tapestry Room is wearing the leather jacket;

was never going to end well, but did not previously throw a run-time problem.
(Compile-time checking is in general too weak to throw compiler errors here:
subkinds of `object` are allowed considerable latitude.)

It would be tempting to take a zero-tolerance approach to inconsistencies,
but we have to recognise a trade-off. Authors do sometimes use the ability of
objects to contain each other (outside of the spatial model) for abstract
purposes we don't want to prohibit. So changes must be made cautiously.

At present no change in this proposal affects any test case in the Inform
test suite other than for indexing. (The test cases `Index-Tokens` and
`Index-Relations` are affected because they describe the kinds of the terms
in the spatial relationships.)

## The holding relation

### Infelicities in Inform 10.1

In Inform 10.1, `R_holding` - the internal representation of the holding
relation, which defines the verb `X holds Y` - was set up inconsistently.
`X` had the kind `person` and `Y` the kind `thing`, but at run-time there
was no enforcement of this, and so for example:

	now the shelf holds the trophy;
	if the shelf holds the trophy, ...;

were both allowed, even with `shelf` being a supporter, not a person. Moreover,
since holding was tested with the `WorldModelKit` function `HolderOf`, which
allows for rooms as well as things, the following could simultaneously be true:

	if the Tapestry Room holds the player, ...;
	if nothing holds the player, ...;

Note that the function `HolderOf` was also used to power the phrase `holder of Y`.
In particular, `holder of the player` could well evaluate to `Tapestry Room`.

### Remediation

`R_holding` is now defined with terms `thing` and `thing`, not `person` and
`thing`. At run-time this is tested with a new function `RelationalHolderOf`, thus:

	[ RelationalHolderOf o;
		o = HolderOf(o);
		if (o ofclass K2_thing) return o;
		return nothing;
	];

Moreover, asserting `now X holds Y` throws a new run-time problem unless both
`X` is a thing and `Y` is a person, a container or a supporter:

	Only objects which are things can take part in holding or carrying.
	For example, 'now the Viper Pit Room holds the snake' is incorrect
	because the Viper Pit Room, being a room, is not a thing. The eventual
	holder or wearer has to be not only a thing, but a special kind of
	thing: either a person, or a supporter, or a container.

The loss of personhood from the defined kind of `Y` would affect an assertion
sentence like this one:

	Mr Smith holds the silver coin.

since it would create `Mr Smith`, in the absence of any other information, as
a thing and not a person. We don't want that, so the Inform compiler has been
given a special case rule inferring that `Mr Smith` is a person by virtue of
being in the holding relationship.

Now, this does mean that:

	The oak shelf holds the Toby jug. The oak shelf is a supporter.

now throws a problem message, since it gives contradictory information about
the kind of the shelf. That is the case even though this:

	The Toby jug is on the oak shelf.

(i.e., which creates the shelf as a supporter) would allow the following to work:

	When play begins:
		if the oak shelf holds the Toby jug, say "Jug ahoy!"

But this is not an inconsistency because there are several ways in which `X` can
be said to hold `Y`, and the assertion `X holds Y` is simply choosing one of those
ways, the one in which `X` is a person carrying something. In the same way,
`now X holds Y` cannot make `Y` a component part of `X`, only a something carried,
contained or supported by `X`.

### Invariant

The following should be now true:

* For all pairs of `X` and `Y` where assertion sentences say that `X holds Y`
in the initial state, the condition `if X holds Y` is true when play begins.
* At all times `if X holds Y` can be true only if `X` and `Y` are both things.
Moreover either `Y` is a component part of `X`, or `X` has one of the
kinds person, container or supporter.
* `now X holds Y` either results in a situation where `X holds Y` is true, or
throws a run-time problem message.

It is also true that:

* If `X holds Y`, then `the holder of Y is X`.

However, the converse is not true. In particular, if `X` is a person in a room
`R`, then `the holder of X holds X` is false, because `the holder of X` is `R`
and `R`, being a room and not a thing, cannot `hold X`. Also, though this is
a less significant issue, `the holder of X` can sometimes be `nothing`.

We have to accept that `holder of X` has a different and wider meaning than
the relation `holding`, allowing it to be used in more abstract settings.
The meaning of `holder of X` is unchanged from Inform 10.1; to change it would
break the Inform test suite, including some examples from the existing
documentation. But the situation is at least improved over 10.1: in 10.1,
neither of `X holds Y` and `the holder of Y is X` implied the other.

## The carrying relation

### Infelicities in Inform 10.1

This is simpler because `R_carrying` was always defined so that `X carries Y`
required `X` to be a person, `Y` to be a thing, and `Y` not to have the
either/or property `worn`. The relation was tested with a function `CarrierOf`
which, unlike `HolderOf`, enforced these restrictions.

However, `now X carries Y` was implemented with the `MoveObject` function,
which did not. So it was possible for this to happen:

	now the blue box carries the glass marble;
	if the blue box carries the glass marble, ...; [False]
	if the glass marble is in the blue box, ...; [True]

### Remediation

`now X carries Y` is now implemented with a new `MakeCarrierOf` function, which
throws a run-time problem message unless `X` is a person and `Y` is a thing,
and which removes the `worn` property from `Y`.

### Invariant

The following should be now true:

* For all pairs of `X` and `Y` where assertion sentences say that `X carries Y`
in the initial state, the condition `if X carries Y` is true when play begins.
* At all times `if X carries Y` can be true only if `X` is a person and `Y` is
a thing, and `Y` does not have the `worn` property.
* `now X carries Y` either results in a situation where `X carries Y` is true, or
throws a run-time problem message.

There is no Inform phrase `carrier of X` in the Standard Rules, and probably
nothing to be gained by adding one. But if we defined:

	To decide which object is the carrier of (something - object):
		(- (CarrierOf({something})) -).

then it would always be true that `X carries Y` if and only if `the carrier of Y is X`.

## The wearing relation

### Infelicities in Inform 10.1

Again a simple case, because `R_wearing` was always defined so that `X wears Y`
required `X` to be a person, `Y` to be a thing, and `Y` to have the
either/or property `worn`. The relation was tested with a function `WearerOf`
which, unlike `HolderOf`, enforced these restrictions.

However, `now X wears Y` was implemented with the `WearObject` function,
which did not. The result was that `now the shelf wears the framed certificate`
would move the certificate to the shelf and give the certificate the `worn`
property, which it certainly shouldn't have. Also, `if X wears Y` did not
check that `Y` was a thing (though it is hard to imagine any other class of
object at runtime having the `worn` attribute).

### Remediation

`WearObject` now throws a run-time problem message unless `X` is a person and
`Y` is a thing. `WearerOf(X)` now returns `nothing` if `X` is not a thing.

### Invariant

The following should be now true:

* For all pairs of `X` and `Y` where assertion sentences say that `X wears Y`
in the initial state, the condition `if X wears Y` is true when play begins.
* At all times `if X wears Y` can be true only if `X` is a person and `Y` is
a thing, and `Y` has the `worn` property.
* `now X wears Y` either results in a situation where `X carries Y` is true, or
throws a run-time problem message.

There is no Inform phrase `wearer of X` in the Standard Rules, but if we defined:

	To decide which object is the wearer of (something - object):
		(- (WearerOf({something})) -).

then it would always be true that `X wears Y` if and only if `the wearer of Y is X`.
