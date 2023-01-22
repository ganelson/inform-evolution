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
too for numerous other reporters of related bugs, and to Dannii Willis and
Peter Bates for their comments. Relevant Jira issues include:

* [I7-2220 on the definition of holding](https://inform7.atlassian.net/browse/I7-2220)
* [I7-2219 on directions being held](https://inform7.atlassian.net/browse/I7-2219)

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

## Consistency

Suppose `R` is a relation between two kinds `K` and `L`, and defines the
Inform verb `to arr`. Inform source text can deal with the relation in three ways:

* A sentence can assert that `X arrs Y.` as play begins.
* Phrases like `if` can test whether `X arrs Y` at any point during play.
* The phrase `now` can sometimes force `now X arrs Y`, or `now X does not arr Y`.

We will say that `R` is "consistent" if the following properties hold:

* Writing the assertion sentence `X arrs Y.` has exactly one of these two consequences:
	* A compilation error.
	* The test `if X arrs Y` will be true at the start of play.
* Writing the test `if X arrs Y` must:
	* If `X` is a `K`, and `Y` is an `L`, produce a true/false result without
	issuing a run-time problem.
	* If `X` is not a `K`, or `Y` is not an `L`, either there must be a compilation
	error so that the test is never made, or it must compile but then produce
	the result false, without issuing a run-time problem.
* If the test `if X arrs Y` is made twice in succession, and no state change has
occurred between the two tests, the result must be the same.
* Writing the phrase `now X arrs Y` has exactly one of these three consequences:
	* A compilation error so that this is never executed.
	* A run-time problem message.
	* A change of state so that `if X arrs Y`, if tested immediately afterwards,
	would be true.
* Similarly for `now X does not arr Y`, except that the state is changed to false.

Combining the first two properties above, an assertion sentence `X arrs Y.`
is required to throw a compilation error if `X` is not a `K`, or if `Y` is not
an `L`. For example, if the relation is defined over vehicles and doors, then
the compiler must only allow `X arrs Y.` if `X` is a vehicle and `Y` is a door.

It is *not* a requirement that assertion sentences `X arrs Y.` must be allowed
for any pair of `X` and `Y` in the domains `K` and `L`, and similarly for
`now`. (Indeed, the numerical relation `greater than` over numbers cannot be
asserted or be changed with `now`.)

Consistency seems simple, but it's surprisingly easy to leave edge cases which
violate it. A minimum goal for this proposal is to ensure that all our spatial
relations are consistent.

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
This behaved a little unexpectedly on direction objects, or at least those
currently in play, since it returned the `Compass` pseudo-object (which can
only be referred to from I6 code, and is an awkward leftover from Inform 1).

### Remediation

We have to accept that `hold` means something different in the spatial model
than it might for more abstract collections of objects created by authors.
There is `hold` in the sense of `be immediately subordinate in either the
object tree or component-parts data structure`, and then there is `hold` in
the narrower sense of `be a physical object directly in, say, the possession
of a person`.

Define a "spatial object" as any object which is a `thing`, `region`,
`room` or `direction`. (This of course includes objects of subkinds of those,
such as people or vehicles.) These are the objects in the tree which make up
the spatial model for the world, and often they are the only instances of
`object` in an Inform story. But if the author writes, say, `An idea is a kind
of object`, then any instances of `idea` will not be spatial objects. Such
objects will be called "abstract".

`R_holding` is now defined with terms `object` and `object`, not `person` and
`thing`. It continues to be tested with `HolderOf`, though this function has
been modified slightly so that the `holder of` a direction is `nothing`. As a
result, the `holder of` of a spatial object is now always another spatial object,
or else `nothing`.

Note that the change of domain affects how determiners work with the relation.
For example, suppose there is just one room, `Lab`, and the player is in this
room. Then:

	"whether or not Lab holds the player" = truth state: true
	"whether or not nothing holds the player" = truth state: false
	"whether or not nowhere holds the player" = truth state: false
	"whether or not something holds the player" = truth state: false
	"whether or not somewhere holds the player" = truth state: true
	"whether or not everywhere holds the player" = truth state: true

Note that `nothing` means `no object`, not `no thing`. Since a room, in this
case `Lab` is an object, it is not true that `nothing holds the player`. On
the other hand, `something` means `some object`. So despite appearances, it
is perfectly consistent that `nothing holds the player` and `something holds the player`
are both false.

Asserting that `now X holds Y` imposes a new restriction if either `X` or `Y`
is a spatial object. In these cases, `Y` must be a thing, and `X`
must be either a room, a person, a container or a supporter. Failing that, the
following run-time problem is produced:

	Only things can be made to be held, carried or worn using 'now';
	and in the case of wearing or carrying, the new wearer or carrier
	must be a person, while for holding, the new holder must be a
	room, a container, a supporter, or a person.

The intention here is to prevent `now X holds Y` from breaking the consistency
rules of the world model, while not restricting its use with abstract objects.

The loss of personhood from the defined kind of `Y`, and thingness from `X`,
would affect an assertion sentence like this one:

	Mr Smith holds the silver coin.

since it would create `Mr Smith`, in the absence of any other information, as
a thing and not a person. We don't want that, so the Inform compiler has been
given a special case rule inferring that `Mr Smith` is a person by virtue of
being in the holding relationship, and that the `silver coin` is a thing.

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

* Holding is a consistent relation defined over the kinds `object` and `object`.
* `X holds Y.` can be asserted only in cases when `X` is a person and `Y` is a thing.
* If `if X holds Y` is true then either both `X` and `Y` are spatial objects,
or neither is.
* If both are spatial objects, then `if X holds Y` if one of the following:
	* `Y` is a thing immediately inside a room `X`;
	* `Y` is a thing immediately inside a container `X`;
	* `Y` is a thing supported by a supporter `X`;
	* `Y` is a thing carried by a person `X`;
	* `Y` is a thing which is a component part of a thing `X`;
	* `Y` is a region which is immediately a part of another region `X`.
* If both are spatial objects, then `now X holds Y` is permitted only in the
following cases:
	* `Y` is a thing and `X` is a room;
	* `Y` is a thing and `X` is a container;
	* `Y` is a thing and `X` is a supporter;
	* `Y` is a thing and `X` is a person.
* `now X does not hold Y` is never permitted.
* `if X holds Y` is true if and only if `the holder of Y is X`.
* `if X holds Y` is true, then `if Y holds X` is false. Consequently, `if X holds X`
is always false.

Note that:

* A region can only hold another region: in particular it does not hold rooms.
* Backdrops and two-sided doors are conceptually present in multiple rooms at once, but
this is implemented at runtime by moving them as necessary. Since `holder of B`
for a backdrop `B` can only return the room it's currently in, `R holds B` only
if `R` is that current location. This is not an inconsistency on a careful reading
of the consistency rules above, but it can be a surprise. If the Brass Door is
between the Blue Room and the Green Room, then nevertheless only one of these
tests will be true at any given time:

	if the Blue Room holds the Brass Door, ...
	if the Green Room holds the Brass Door, ...

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

* Carrying is a consistent relation defined over the kinds `person` and `thing`.
* `X carries Y.` can be asserted for any such pair, so long as this does not
contradict other assertions. If `Y` is declared to be a backdrop or a door,
that would be just such a contradiction.
* If `if X carries Y` is true then both `X` and `Y` are spatial objects.
* If `if X carries Y` is true then `Y` does not have the `worn` property.
* `now X carries Y` is permitted in all cases, and causes the `worn` property
to be removed from `Y`.
* If `if X carries Y` is true then `if X holds Y` is true.
* If an Inform phrase `carrier of X` were defined using `CarrierOf`, then
it would be the case that `X carries Y` if and only if `the carrier of Y is X`.

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
* `now X wears Y` either results in a situation where `X wears Y` is true, or
throws a run-time problem message.

There is no Inform phrase `wearer of X` in the Standard Rules, but if we defined:

	To decide which object is the wearer of (something - object):
		(- (WearerOf({something})) -).

then `X wears Y` if and only if `the wearer of Y is X`.

## The regional containment relation `R_regional_containment`

### Infelicities in Inform 10.1

A region regionally contains itself, which is a little odd, and appears to be
an oversight: nothing in the test suite relies on this. Regional containment
is used to give meaning to `if X is in R` where `R` is a region, so the
practical result is we can have `if R is in R` being true. There is a tenuous
argument for this, but it seems more likely to surprise authors than not to.
(Is Europe in Europe? I think I would say not.)

It was impossible to change the region of a room with `now` (though it was
possible to remove regions from other regions, in some cases). Whether this is
infelicitous is a matter of taste.

### Remediation

The function `TestRegionalContainment` now returns `false` if asked to test
whether a region contains itself, and so `if R is in R` is now false.

A new function `SetRegionalContainment` moves either a room or a region to a
region. This enables, for example:

	Nirvana is a region. Shangri-La is a region. Nirvana is in Shangri-La.
	Seventh Heaven is a region. 

	Lebling Monument is a room. Lebling Monument is in Nirvana. A plaque
	is in the Monument.

	Instead of examining the plaque:
		now Nirvana is in Seventh Heaven;
		now Lebling Monument is in Seventh Heaven;
		say "You feel mystic connections being rearranged."

If `now X is in R` is tried in cases where `X` is anything other than a room
or region, or in the case where `X` equals or is contained by `R`, a runtime
problem is thrown:

	If 'R' is the name of a region, then 'now X is in R' is only allowed in
	cases where 'X' is a room or another region. It's otherwise too vague
	where things are to move.

This is technically a change to the containment relation, not to the regional
containment relation, but rooms and regions can be removed from all regions with:

	now Lebling Monument is nowhere;
	now Nirvana is nowhere;

(In Inform 10.1, `now Lebling Monument is nowhere` would throw a run-time problem
if the player was in this room, and otherwise do nothing.)

The `showme` debugging verb now lists the super-region of a region. (Since this
can now change, it seems useful to be able to see what it is: for example
by typing `SHOWME NIRVANA` in the above example.)

### Invariant

If `R` and `S` are regions and `X` is an object, then the following should be now true:

* For all pairs of `X` and `R` where assertion sentences say that `X is in R`
in the initial state, the condition `if X is in Y` is true when play begins.
* At all times `if X is in R` can be true only if `X` is a thing, a room or
a region.
* This relation is transitive, but never reflexive or symmetric. In particular,
`if X is in R` is true, then `if R is in X` is false; and `if R is in R` is false.
* `if S is in R` then `R` must be the `holder of S`, or the `holder of the holder of S`,
or... and so on.
* `now X is in R` either results in a situation where `if X is in R` is true, or
throws a run-time problem message.
