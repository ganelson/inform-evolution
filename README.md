# inform-evolution

A repository for proposals for changes to the Inform programming language
(see [ganelson/inform](https://github.com/ganelson/inform)).

## Policy

Inform is a mature and widely used programming language, and changes to its
design which are visible to users (i.e., causing a minor or major version
number bump), or implementation changes on any large scale, should always
now begin with a proposal document on this repository.

- Proposals are numbered IE-0001, IE-0002, ... in order of creation. They
are unlikely to be implemented in numerical order.
- The baseline design is Inform version 10.1.0.
- Any change to the Inform design visible to users (and causing a minor or
major version number bump) should be related to a proposal here.
- For now, proposals are made by the Inform team.
- "Accepted" proposals are those accepted _in principle_, but details may
change during implementation or as a result of comments from users, and some
may ultimately be withdrawn in favour of better solutions.

For now, we ask users not to open their own pull requests against this repository,
but users are very welcome to contribute comments in the PR associated with each proposal
for exactly that purpose. For example, [PR#1](pull/1) exists for comments on IE-0001.
For more free-flowing discussion about future possibilities, [the Inform area of the intfiction.org forum](https://intfiction.org/c/authoring/inform-7/)
would be the place.

This repository was made public on 30 July 2022, and was described at
[the Inform talk at the Narrascope III conference](https://ganelson.github.io/inform-website/talks/2022/07/31/narrascope-iii.html).

## List of proposals

Proposal                                                                                                 | Began        | Status   | Comments 
-------------------------------------------------------------------------------------------------------- | ------------ | -------- | -------
[(IE-0001) Directory format for extensions with resources](proposals/0001-extensions-with-resources.md)  | 1 May 2022   | Accepted | [PR#1](pull/1)
[(IE-0002) Automatic gitignores for Inform projects](proposals/0002-inform-project-gitignores.md)        | 1 May 2022   | Accepted | [PR#2](pull/2)
[(IE-0003) Dividing source text into multiple files](proposals/0003-multiple-source-files.md)            | 1 May 2022   | Partly implemented | [PR#3](pull/3)
[(IE-0004) Access to data files embedded in blorbs](proposals/0004-using-data-files-in-blorbs.md)        | 5 May 2022   | Accepted | [PR#4](pull/4)
[(IE-0005) Removing translates into Unicode](proposals/0005-removing-translates-into-unicode.md)         | 5 June 2022  | Accepted | [PR#5](pull/5)
[(IE-0006) New annotations for I6 syntax](proposals/0006-i6-syntax-annotations.md)                       | 16 June 2022 | Accepted | [PR#6](pull/6)
[(IE-0007) Double-precision real numbers](proposals/0007-double-precision-reals.md)                      | 3 July 2022  | Accepted | [PR#7](pull/7)
[(IE-0008) Faster memory allocation](proposals/0008-faster-memory-allocation.md)                         | 13 July 2022 | Accepted | [PR#8](pull/8)
[(IE-0009) Dialogue sections](proposals/0009-dialogue-sections.md)                                       | 21 July 2022 | Accepted | [PR#9](pull/9)
[(IE-0010) Concepts](proposals/0010-concepts.md)                                                         | 21 July 2022 | Accepted | [PR#10](pull/10)
