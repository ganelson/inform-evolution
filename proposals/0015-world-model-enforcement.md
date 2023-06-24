# (IE-0015) World model enforcement

* Proposal: [IE-0015](0015-world-model-enforcement.md)
* Discussion PR link: [#15](https://github.com/ganelson/inform-evolution/pull/15)
* Authors: Graham Nelson
* Language feature name: --
* Status: Draft
* Related proposals: --
* Implementation: In progress

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
* [I7-2211 on world model violations](https://inform7.atlassian.net/browse/I7-2211)
* [I7-2046 on output for containers with concealed contents](https://inform7.atlassian.net/browse/I7-2046)
* [I7-2036 on examining things with concealed contents](https://inform7.atlassian.net/browse/I7-2036)

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

The changes described below regarding output for supporters or containers all of
whose contents are concealed altered the output for the "Undescribed" test case.
(Additionally, the test cases `Index-Tokens` and `Index-Relations` are affected
because they describe the kinds of the terms in the spatial relationships.)

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

`HolderOf` contained an awkward special case to deal with the situation before
the "position player in the world model" rule has run; it then returned the
pseudo-object `thedark` in a desperate attempt to avoid returning `nothing`,
which caused certain past-tense conditions tested later in play to go wrong.

### Improvements

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
the other hand, `something` means `some thing`. So despite appearances, it
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

Before the "position player in the world model" rule has run, `HolderOf`
the player returns the position which the player will once it does; so it now
never returns `thedark`, which is not a spatial object.

### New specification

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
	* `Y` is a thing and `X` is a person;
	* `Y` is a region and `X` is a region.
* If both are abstract objects, then `now X holds Y` is permitted in all cases.
* `now X does not hold Y` is never permitted.
* `if X holds Y` is true if and only if `the holder of Y is X`. In particular,
for any given `Y`, there is at most one `X` such that `X holds Y`.
* `if X holds Y` is true, then `if Y holds X` is false. Consequently, `if X holds X`
is always false.

Note that:

* Using `now` to change holding can result in a situation where an object
indirectly or directly holds itself. This is not allowed, either for spatial
or abstract objects, but technically results in a "programming error" message
rather than a run-time problem message.
* A region can only hold another region: in particular it does not hold rooms.
* Backdrops and two-sided doors are conceptually present in multiple rooms at once, but
this is implemented at runtime by moving them as necessary. Since `holder of B`
for a backdrop `B` can only return the room it's currently in, `R holds B` only
if `R` is that current location. This is not an inconsistency on a careful reading
of the consistency rules above, but it can be a surprise. If the Brass Door is
between the Blue Room and the Green Room, then only one of `the Blue Room holds the Brass Door`
and `the Green Room holds the Brass Door` will be true, and it will not always
be easy to be sure which.

## The enclosure relation

### Infelicities in Inform 10.1

Because the enclosure relation is (almost) the transitive closure of holding,
all the defects of holding in 10.1 could in principle harm enclosure as well,
but in practice enclosure was mostly unaffected by those defects.

In 10.1, both adjoining rooms enclosed the two sides of a door, but none of
the containing rooms enclosed a backdrop (even if it was, in fact, found in
only that room). This treated doors and backdrops differently.

### Improvements

The change that makes the holder of a direction `nothing` affects enclosure,
since it means the `Compass` pseudo-object no longer encloses any directions.

A room now encloses a backdrop if it's any of the rooms in which the backdrop
is found (unless the backdrop is `absent`).

### New specification

The following should be now true:

* Enclosure is a consistent relation defined over the kinds `object` and `object`.
* `X encloses Y.` cannot be asserted: it is too vague.
* If `if X encloses Y` is true then either both `X` and `Y` are spatial objects,
or neither is.
* Enclosure is the transitive closure of a relation we'll call "almost-holding".
That is, `X encloses Y` if there is a sequence `H1`, `H2`, ..., `Hn` such that
`X almost-holds H1`, `H1 almost-holds H2`, ..., `Hn almost-holds Y`.
* For abstract objects, almost-holding is the same thing as holding.
* For spatial objects, almost-holding is the same as holding except that:
	* Both sides of a two-sided door almost-hold the door, whereas only the
	side most recently seen by the player actually holds it;
	* Any of the rooms in which a backdrop is found almost-hold it (unless it is `absent`),
	whereas only the one most recently seen by the player actually holds it.
* Since holding implies almost-holding, holding also implies enclosure. Thus
`X holds Y` always implies that `X encloses Y`. 
* `now X encloses Y` is never permitted.
* `now X does not enclose Y` is never permitted.
* Enclosure is transitive (unsurprisingly, since it's a transitive closure):
that is, if `X encloses Y` and `Y encloses Z` then `X encloses Z`.
* `if X encloses Y` is true, then `if Y encloses X` is false.
Consequently, `if X encloses X` is always false.

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

### Improvements

`now X carries Y` is now implemented with a new `MakeCarrierOf` function, which
throws a run-time problem message unless `X` is a person and `Y` is a thing,
and which removes the `worn` property from `Y`.

`now X does not carry Y` has been enabled for the first time. It's analogous
to somebody putting an object down on whatever the floor is for them.

### New specification

The following should be now true:

* Carrying is a consistent relation defined over the kinds `person` and `thing`.
* `X carries Y.` can be asserted for any such pair, so long as this does not
contradict other assertions. If `Y` is declared to be a backdrop or a door,
that would be just such a contradiction.
* If `if X carries Y` is true then both `X` and `Y` are spatial objects.
* If `if X carries Y` is true then `Y` does not have the `worn` property.
* `now X carries Y` is permitted in all cases, and causes the `worn` property
to be removed from `Y`.
* `now X does not carry Y` is permitted in all cases. If `X carries Y` then
`Y` is moved to the `holder of X`; if not, nothing happens. Note that if `X`
is currently out of play (so that `holder of X` is `nothing`) then `Y` is
made separately out of play.
* If `if X carries Y` is true then `if X holds Y` is true.
* If an Inform phrase `carrier of X` were defined using `CarrierOf`, then
it would be the case that `X carries Y` if and only if `the carrier of Y is X`.
 In particular, for any given `Y`, there is at most one `X` such that `X carries Y`.

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

### Improvements

`WearObject` now throws a run-time problem message unless `X` is a person and
`Y` is a thing. `WearerOf(X)` now returns `nothing` if `X` is not a thing.

`now X does not wear Y` has been enabled for the first time. It's analogous
to somebody taking a hat off but still carrying it.

### New specification

The following should be now true:

* Wearing is a consistent relation defined over the kinds `person` and `thing`.
* `X wears Y.` can be asserted for any such pair, so long as this does not
contradict other assertions. If `Y` is declared to be a backdrop or a door,
that would be just such a contradiction.
* If `if X wears Y` is true then both `X` and `Y` are spatial objects.
* If `if X wears Y` is true then `Y` has the `worn` property.
* `now X wears Y` is permitted in all cases, and causes the `worn` property
to be given to `Y`.
* `now X does not wear Y` is permitted in all cases. If `X wears Y` then
`Y` has the `worn` property removed, so that now `X carries Y` instead; if not,
nothing happens.
* If `if X wears Y` is true then `if X holds Y` is true.
* If an Inform phrase `wearer of X` were defined using `WearerOf`, then
it would be the case that `X wears Y` if and only if `the wearer of Y is X`.
In particular, for any given `Y`, there is at most one `X` such that `X wears Y`.

## The support relation

### Infelicities in Inform 10.1

This was technically inconsistent, because `now X supports Y` was allowed even
in cases where `X` was not a supporter.

### Improvements

A run-time problem is now thrown by `now X supports Y` unless `X` is a supporter
and `Y` is a thing.

### New specification

The following should be now true:

* Support is a consistent relation defined over the kinds `supporter` and `thing`.
* `X supports Y.` or `Y is on X.` can be asserted for any such pair, so long as
this does not contradict other assertions.
* If `if X supports Y` is true then both `X` and `Y` are spatial objects.
* `now X supports Y` is permitted in all cases.
* `now X does not support Y` is not permitted in any case.
* If `if X supports Y` is true then `if X holds Y` is true.
* If an Inform phrase `supporter of X` were defined using `SupporterOf`, then
it would be the case that `X supports Y` if and only if `the supporter of Y is X`.
In particular, for any given `Y`, there is at most one `X` such that `X supports Y`.

## The incorporation relation

### Infelicities in Inform 10.1

Mostly fine, except that it produced programming errors rather than runtime problems
if attempts were made to `now` this on objects other than things.

### Improvements

The `MakePart` function now throws a runtime problem if applied to non-things.

### New specification

* Incorporation is a consistent relation defined over the kinds `thing` and `thing`.
* `X is part of Y.` can be asserted for any such pair, so long as this does not
contradict other assertions.
* If `if Y is part of X` is true then both `X` and `Y` are spatial objects.
* If `if Y is part of X` is true then `if X holds Y` is true.
* `now Y is part of X` is permitted in all cases.
* `now Y is not part of X` is not permitted in any case.
* If an Inform phrase `incorporator of X` were defined using `PartOf`, then
it would be the case that `Y is part of X` if and only if `the incorporator of Y is X`.
In particular, for any given `Y`, there is at most one `X` such that `Y is part of X`.

## The containment relation

### Infelicities in Inform 10.1

Containment is the most problematic spatial relation in 10.1. There were three
main problems:

(1) First, containment could be tested and sometimes
also created with `now` for cases which ought to be impossible. `now the coin is
in the granite slab` would work even if the `granite slab` isn't a container,
and there were numerous more subtle edge cases like that. For example, the
|Compass| pseudo-object contained some (but not necessarily all) directions;
and in general abstract objects could be tested for containment, though not
always moved with `now` or looped correctly over. If containment is supposed
to mean something like its natural-language meaning, this is all wrong.

(2) The second problem is a subtle inconsistency to do with backdrops and doors.
`now B is in R` is allowed in the case where `B` is a backdrop and `R` is a
region; but `if B is in R` (if this is given the containment meaning) is then
often false because the backdrop has been removed from the object tree, unless
`R` is the player's current room. (If this is given the regional-containment
meaning, it is true unless the region contains no rooms.)

The same issue affects two-sided doors. `now D is in R` can move a two-sided door,
and that potentially leads to inconsistency. In fact, though, this has pretty
disastrous consequences anyway, because doors should not be mobile. (It is already
the case that doors cannot be removed from play, and that map connections to doors
cannot be altered. It seems to be just an oversight that doors could be moved
with `now` in 10.1.)

(3) The third problem arises from the way that `X is in Y` is interpreted as meaning
containment most of the time, but as regional containment when `Y` is the literal
name of a region. So consider this:

	repeat with Y running through objects:
		if the player is in Y:
			say "You look around [Y].";
	if the player is in the Vast Wasteland:
		say "You look around the dreary landscape beyond."

Here `Y` is not necessarily a region, so the compiler uses regular containment
to decide whether `player is in Y`, and that applies on every iteration of the
loop at runtime, including when `Y` equals `Vast Wasteland`; but it uses regional
containment to decide whether `player is in the Vast Wasteland`. We might like
the outcome to be the same in both cases. But it is not. In 10.1, regions do
not contain anything except possibly other regions. But they regionally contain
not only regions (including those only indirectly children in the object tree) but
also all rooms inside them (or their indirect child regions) and also all things
in any of those rooms. It's unclear what if anything we should do about this.

### Improvements

(1) Containment is now defended so that `X is in Y` can be true only for spatial objects.

An attempt to have a non-container-or-room contain something now produces a
runtime problem message. This is quite a consequential change: some may feel it
breaks existing code, others that it identifies genuine bugs. As evidence of
that, it turned up two bugs in the test suite and one in the Standard Rules:

(a) The documentation example `Some Assembly Required`, which involves cutting
up shirts, includes the line:

	now every thing which is part of the noun is in the holder of the noun.

This should have been (and now is) `is held by the holder of the noun`: otherwise
it goes wrong if the player, say, is holding the thing being cut up into pieces.

(b) The example `Transmutations` goes out of its way to court trouble by allowing
the "inserting" action to bypass the requirement on being a container:

	The can't insert into what's not a container rule does nothing when inserting something into the machine.

A bug (and it is genuinely that) in the example went on to handle this case like so:

	Carry out inserting something into the machine: 
		now the noun is nowhere; 
		now the player carries the new form of the noun.

I'm pretty certain that the writer did not intend the `carry out` rulebook to
continue executing from there, so that the regular `carry out inserting` would
execute:

	Carry out an actor inserting something into (this is the standard inserting rule):
		now the noun is in the second noun.

But in fact it did, and this is where the `now` throws a runtime problem message.
The example should have written (and now does write) this:

	Carry out inserting something into the machine: 
		now the noun is nowhere; 
		now the player carries the new form of the noun;
		rule succeeds.

(c) The Standard Rules included:

	Carry out an actor dropping (this is the standard dropping rule):
		now the noun is in the holder of the actor.

This is now incorrect in the case when the actor is standing on an enterable
supporter, because the `noun` cannot be `in` a supporter. (For example, this
happens in the example `Priority Lab`.) The rule should have read, and now does read,

	Carry out an actor dropping (this is the standard dropping rule):
		now the noun is held by the holder of the actor.

If a bad containment of this kind is made with `now`, the following new runtime
problem message is produced:

	When "now X is in Y" is used, "X" has to be a thing, and "Y" has to be a
	container or a room. Sometimes this problem is triggered when an overly
	broad phrase has been used such as "now X is in the holder of Y", which
	would be fine if "Y' is in a room or container, but not if it's a supporter
	or a person. (Things can be "on" a supporter, but not "in" it. Equally, it
	can't be a person: people can "carry" or "wear" things, but not contain
	them.) The safe way to avoid that is to write "now X is held by the holder
	of Y". Whatever holds "Y", if it can hold "Y" then it can hold "X".

(2) Now for the inconsistencies on moving doors or backdrops.

Doors are easy enough to deal with. An attempt to move a door with `now D is in R`
now produces a runtime problem message. (And similarly for `now P is wearing D`
or `now P is carrying D` or `now S supports D`.)

But backdrops remain problematic. For the moment the inconsistency remains.

(3) Nothing has been done about this.

### New specification

* Containment is a currently inconsistent relation defined over the kinds `object` and `object`.
It is consistent except for backdrops, where asserting `B is in R.` does not
guarantee that `B is in R` will be true at start of play, and where `now B is in R`
does not guarantee that `B is in R` will be true immediately afterwards (unless
`R` is the literal name of a region, in which case it will work because regional
containment not regular containment will be used).
* `X is in Y.` can be asserted only where `X` is a thing and `Y` is either a room
or a container. (Important to remember here that `X is in Y` is taken as meaning
regional containment, not containment, in the case where `Y` is a region. Here
we're describing the relation `containment`, not the meaning of `to be in`.)
* If `if X is in Y` is true then both `X` and `Y` are spatial objects.
* `X is in Y` is true if one of:
	* `X` is a room and `Y` is its region;
	* `X` is a thing and `Y` is a container or room holding `X`;
	* `X` is a region and `Y` is a region holding `X`.
* `now X is in Y` is permitted only where:
	* `X` is a thing and `Y` is a container or room, or
	* `X` is a room and `Y` is a region, or
	* Both `X` and `Y` are regions.
* `now X is not in Y` is not permitted in any case.
* If `if X is in Y` is true then either `if X holds Y` is true or `X` is a room
and `Y` is its region.
* If an Inform phrase `container of X` were defined using `ContainerOf`, then
it would be the case that `X is in Y` if and only if `the container of X is Y`.
In particular, for any given `Y`, there is at most one `X` such that `X is in Y`.

## The regional-containment relation

This provides a meaning for `if X is regionally in Y`, but this is not a verb
used very much. The relation more often arises because the predicate calculus
engine in Inform silently reads `if X is in R` as `if X is regionally in R`,
whenever `R` is a value whose kind is `region`. (In particular, this will
certainly happen if `R` is the explicit name of a region, which is the most
common case.)

Regional containment cannot be eliminated from Inform because to do so would
require code about ordinary containment to be compiled much less efficiently.

### Infelicities in Inform 10.1

A region regionally contains itself, which is a little odd, and appears to be
an oversight: nothing in the test suite relies on this. Regional containment
is used to give meaning to `if X is in R` where `R` is a region, so the
practical result is we can have `if R is in R` being true. There is a tenuous
argument for this, but it seems more likely to surprise authors than not to.
(Is Europe in Europe? I think I would say not.)

If a region contains a door or backdrop, it sometimes does not contain component
parts of that door or backdrop, or their further contents.

It was impossible to change the region of a room with `now` (though it was
possible to remove regions from other regions, in some cases). Whether this is
infelicitous is a matter of taste.

### Improvements

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

`TestRegionalContainment` has been rewritten to make clearer its
connection to holding, and to ensure that component parts of backdrops
and doors work correctly.

The `showme` debugging verb now lists the super-region of a region. (Since this
can now change, it seems useful to be able to see what it is: for example
by typing `SHOWME NIRVANA` in the above example.)

### New specification

* Regional containment is a consistent relation defined over the kinds `region`
and `object`.
* If `if Y is regionally in R` is true then both `R` and `Y` are spatial objects.
* `if Y is regionally in R` if one of the following:
	* `Y` is a region directly or indirectly in `R`.
	* `Y` is a room whose region is either `R` or is another region directly or indirectly in `R`.
	* `Y` is a backdrop which is not `absent` and any one of whose locations is such a room.
	* `Y` is a two-sided door which is not `absent`, either of whose locations is such a room.
	* `Y` is any other thing, whose location is such a room.
* Regional containment nearly, but does not quite, imply the transitive closure of holding.
(Regions can only hold other regions, and the rules for backdrops and doors are different.)
What can be said is that if `Y is regionally in R` and `Y holds Z` then `Z is regionally in R`.
In particular this applies to component parts, even of backdrops and doors.
* There can be multiple regions `R1`, `R2`, ..., such that `Y is regionally in R1`
and so on, for the same thing `Y`. This can be true even if neither `R1` holds `R2`
nor vice versa. There is therefore no consistent way to define the `region of`
a thing.
* `now Y is regionally in R` is permitted only when `Y` is also a region.

## The room-containment relation

This is used less even than regional containment. By definition, X is room-contained
by Y if Y is equal to `the location of X`, a phrase which is an Inform 7 wrapper
for the `WorldModelKit` function `LocationOf`.

Authors could in theory choose to create a verb with this as meaning:

	The verb to be roomwise in means the room-containment relation.

Otherwise, though, room containment is used only internally to provide a meaning
for -where words in sentences like `if X is somewhere`, or `if X is everywhere`.
For example, `if X is somewhere` is read as `if the location of X is a room`,
or equivalently `if the location of X is not nothing`.

(So note that room containment is _not_ the meaning of `if X is in R` in cases
where `R` can be proved at compile-time to have the kind room: in that sense it's
not a parallel to regional containment.)

### Infelicities in Inform 10.1

This is technically inconsistent. It is defined over kinds `room` and `thing`,
but in 10.1, a room room-contains itself. So for example if `Chapter House`
is a room, then `Chapter House is somewhere` is true, and `Chapter House is nowhere`
is false; `Chapter House is everywhere` is true if and only if this is the
only room. Also, it was in theory possible to `now` this relationship between
abstract objects.

### Improvements

Redefining it over `room` and `object` makes it consistent.

A new function `MakeRoomContainerOf` makes suitable checks.

### New specification

For convenience, we'll pretend that the verb `to be roomwise in` has been created.

* Room containment is a consistent relation defined over the kinds `room` and `object`.
* It cannot be asserted: `X is roomwise in Y.` throws a problem message.
* If `if X is roomwise in Y` is true then both `X` and `Y` are spatial objects.
* `if X is roomwise in Y` is true if and only if `the location of X is Y`.
* This happens if `Y` is a room, and one of:
	* `X` is a two-sided door, and `Y` is either the front side of the door, or
	else the back side if the player is currently in that room.
	* `X` is a backdrop, and `Y` is one of the rooms `X` is in, and either the
	player is currently in `Y` or else `Y` is the first room `X` is in.
	* `X` is a thing directly or indirectly held by `Y`.
	* `X` is `Y` itself, and therefore a room.
* `now X is roomwise in Y` is permitted only if `X` is a thing and `Y` is a
room, and is then equivalent to `now X is in Y`.
* `now X is not roomwise in Y` is not permitted in any case.

The reason we want a room to room-contain itself is so that wordings such as
`if R is somewhere dark` will work. This is read as testing if `R` is room-contained
by a dark room: which, if `R` is itself a room, is equivalent to testing whether
`R` is dark, and that's clearly what the writer of these words intended to find out.
If we didn't allow `R` to room-contain itself, `if R is somewhere dark` would
always fail when `R` is a room.

This is once again awkward on backdrops and doors. That awkwardness means that,
for example, `if the white mist is everywhere` will be false even if the mist
is a backdrop which is indeed found in every room: because when the test is
performed, only one of those rooms will be the `location of the white mist`.

It is tempting to get rid of that issue by defining the relation differently,
so that `white mist` is roomwise in every room it is found inside. But that
would make tests such as `if X is somewhere` very inefficient, because they
would then have to involve a loop through all rooms in the world. At present,
`if X is somewhere` executes quickly because it only needs to find the location
of `X` and consider that. Speed seems more important here: nobody really needs
to test whether a backdrop is currently found in every location.

## The visibility relation

This is a relatively simple relation, since it can be tested but not asserted.

### Infelicities in Inform 10.1

Though defined over things, it can be tested for any objects, with potentially
unpredictable outcomes: so this may be inconsistent.

`X can see B`, for a backdrop or two-sided door `B`, may not work as expected,
since the test is performed on the basis of the current holder of `B` -- a
single location, where `B` most recently was.

### Improvements

`TestVisibility(A, B)` now returns `false` in all cases where either `A` or `B`
is not a thing. 

### New specification

The following should be now true:

* Visibility is a consistent relation defined over the kinds `thing` and `thing`.
* `X can see Y.` cannot be asserted.
* If `if X can see Y` is true then both `X` and `Y` are spatial objects.
* `X can see Y` provided that both:
	* The holder of the core of `X` offers light. (The core of a thing is the
	result of repeatedly taking whatever thing it is a part of, until it is not
	a part any more. So if a terminal is part of a battery which is part of a
	camera which is carried by a photographer, then the core of the terminal
	is the camera, and the holder of the core is the photographer.)
	* `Y` is in scope from the point of view of `X`.
* `now X can see Y` is not permitted.
* `now X cannot see Y` is not permitted.

## The audibility relation

This is a relatively simple relation, since it can be tested but not asserted.

### Infelicities in Inform 10.1

Inform 10.1 does not have audibility, which was introduced as part of the new
dialogue system.

`X can hear B`, for a backdrop or two-sided door `B`, may not work as expected,
since the test is performed on the basis of the current holder of `B` -- a
single location, where `B` most recently was.

### Improvements

`TestAudibility(A, B)` now returns `false` in all cases where either `A` or `B`
is not a thing.

### New specification

The following should be now true:

* Audibility is a consistent relation defined over the kinds `thing` and `thing`.
* `X can hear Y.` cannot be asserted.
* If `if X can hear Y` is true then both `X` and `Y` are spatial objects.
* `X can hear Y` provided that `Y` is in scope from the point of view of `X`.
* `now X can hear Y` is not permitted.
* `now X cannot hear Y` is not permitted.

Note that this is visibility minus the requirement for light.

## The touchability relation

This is a relatively simple relation, since it can be tested but not asserted.

### Infelicities in Inform 10.1

Though defined over things, it can be tested for any objects, with potentially
unpredictable outcomes: so this may be inconsistent.

Unlike the visibility and audibility relations, touchability makes a real effort
to get backdrops and two-sided doors right. It gets just one case wrong, and then
only sometimes: if both `A` and `B` are backdrops (or doors) where both are
in principle present, but they are currently in two different rooms in the object
tree and in both cases these are rooms where the other is not in principle present,
there is a false negative. But I think it is always correct in cases where at most
one of `A` and `B` is a backdrop/door.

### Improvements

`TestTouchability(A, B)` now returns `false` in all cases where either `A` or `B`
is not a thing. 

### New specification

The following should be now true:

* Touchability is a consistent relation defined over the kinds `thing` and `thing`.
* `X can touch Y.` cannot be asserted.
* If `if X can touch Y` is true then both `X` and `Y` are spatial objects.
* `X can touch Y` provided that both:
	* `Y` is in scope from the point of view of `X` (but see note about backdrops/doors above).
	* The accessibility rulebook allows `Y` to be touched by `X` (see function `ObjectIsUntouchable`).
* `now X can touch Y` is not permitted.
* `now X cannot touch Y` is not permitted.

In very early IF, direction objects sometimes doubled as objects representing walls:
thus "south" could be typed to refer either to the direction or to the south wall
of (doubtless) a cave. Inform no longer takes that view, and you can't see or
touch a direction.

## The concealment relation

This is a relatively simple relation, since it can be tested but not asserted.
Concealment has no backdrop or door issues since neither can be concealed, nor can
conceal.

### Infelicities in Inform 10.1

When listing inventory, if a player had a container or supporter with concealed
contents, those contents were listed but were *not* in scope.

Output between truly empty containers or supporters and those which held one or
more concealed items differed in several places, constituting "tells" of the
existence of the concealed item:

* a falsely empty scenery supporter would get a locale paragraph to say "On
(the supporter) is nothing"; a truly empty one wouldn't get a paragraph
* in room descriptions, a truly empty container would be tagged "(empty)"; a
falsely empty one wouldn't
* opening a truly empty container yielded "You open (the container)"; opening
a falsely empty one would add ", revealing nothing"
* the default message on examining a truly empty container was "(The container)
is empty" or, for a truly empty supporter, "You see nothing special about (the
supporter)"; for a falsely empty receptacle, it would be "In/on (the receptacle)
is nothing"
* when entering a falsely empty supporter or container, subsequent to the "You
get into/onto..." message, you would be told "In/on (the receptacle) you can
see nothing"
* upon searching a truly empty container, the output was "(The container) is
empty" or, for a truly empty supporter, "There is nothing on (the supporter)";
for a falsely empty receptacle, the output was "In/on (the receptacle) is
nothing"

### Improvements

The concealed contents of things the player has are omitted from inventory.

The output for falsely empty things matches the output for truly empty ones.

### New specification

The following should be now true:

* While it continues to be the case that a player is aware of all the things they
directly carry or wear, it is no longer guaranteed that the player is aware of all
the things they *enclose*.
* The default output for actions on falsely empty receptacles is consistent with
that for actions on truly empty ones.
* Concealment is a consistent relation defined over the kinds `thing` and `thing`.
* `X conceals Y.` cannot be asserted.
* If `if X conceals Y` is true then both `X` and `Y` are spatial objects.
* `X conceals Y` provided that `X` encloses `Y`, and the deciding the concealed
possessions activity says that it's concealed.
* `now X conceals Y` is not permitted.
* `now X does not conceal Y` is not permitted.

Note that although this is almost always used with `X` being a person, there's
no requirement for that: containers and supporters can also conceal things.
(Rooms however cannot, since rooms are not things.)

There continues to be a potential tell of concealed items in that they continue to
count against carrying capacity. This is deemed to be the appropriate default
behaviour; how or whether to deal with that situation is left to the individual
author.
