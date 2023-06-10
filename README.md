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
for exactly that purpose. For example, [PR#1](https://github.com/ganelson/inform-evolution/pull/1) exists for comments on IE-0001.
For more free-flowing discussion about future possibilities, [the Inform area of the intfiction.org forum](https://intfiction.org/c/authoring/inform-7/)
would be the place.

This repository was made public on 30 July 2022, and was described at
[the Inform talk at the Narrascope III conference](https://ganelson.github.io/inform-website/talks/2022/07/31/narrascope-iii.html).

## List of proposals

Proposal                                                                                                 | Began           | Status                     | Comments
-------------------------------------------------------------------------------------------------------- | --------------- | -------------------------- | -------
[(IE-0001) Directory format for extensions with resources](proposals/0001-extensions-with-resources.md)  | 1 May 2022      | In progress                | [PR#1](https://github.com/ganelson/inform-evolution/pull/1)
[(IE-0002) Automatic gitignores for Inform projects](proposals/0002-inform-project-gitignores.md)        | 1 May 2022      | Implemented but unreleased | [PR#2](https://github.com/ganelson/inform-evolution/pull/2)
[(IE-0003) Dividing source text into multiple files](proposals/0003-multiple-source-files.md)            | 1 May 2022      | Implemented but unreleased | [PR#3](https://github.com/ganelson/inform-evolution/pull/3)
[(IE-0004) Access to data files embedded in blorbs](proposals/0004-using-data-files-in-blorbs.md)        | 5 May 2022      | In progress                | [PR#4](https://github.com/ganelson/inform-evolution/pull/4)
[(IE-0005) Removing translates into Unicode](proposals/0005-removing-translates-into-unicode.md)         | 5 June 2022     | Implemented but unreleased | [PR#5](https://github.com/ganelson/inform-evolution/pull/5)
[(IE-0006) New annotations for I6 syntax](proposals/0006-i6-syntax-annotations.md)                       | 16 June 2022    | Implemented but unreleased | [PR#6](https://github.com/ganelson/inform-evolution/pull/6)
[(IE-0007) Double-precision real numbers](proposals/0007-double-precision-reals.md)                      | 3 July 2022     | Accepted                   | [PR#7](https://github.com/ganelson/inform-evolution/pull/7)
[(IE-0008) Faster memory allocation](proposals/0008-faster-memory-allocation.md)                         | 13 July 2022    | Accepted                   | [PR#8](https://github.com/ganelson/inform-evolution/pull/8)
[(IE-0009) Dialogue sections](proposals/0009-dialogue-sections.md)                                       | 21 July 2022    | Implemented but unreleased | [PR#9](https://github.com/ganelson/inform-evolution/pull/9)
[(IE-0010) Concepts](proposals/0010-concepts.md)                                                         | 21 July 2022    | Implemented but unreleased | [PR#10](https://github.com/ganelson/inform-evolution/pull/10)
_(IE-0011) New data structures_ (still drafting: see pull request)                                       | --              | Drafting                   | [PR#11](https://github.com/ganelson/inform-evolution/pull/11)
_(IE-0012) A new GlkKit_ (still drafting: see pull request)                                              | --              | Drafting                   | [PR#12](https://github.com/ganelson/inform-evolution/pull/12)
[(IE-0013) Annotations for kit linking](proposals/0013-annotations-for-kit-linking.md)                   | 10 October 2022 | Implemented but unreleased | [PR#13](https://github.com/ganelson/inform-evolution/pull/13)
[(IE-0014) Inter names for rulebooks](proposals/0014-inter-names-for-rulebooks.md)                       | 23 October 2022 | Implemented but unreleased | [PR#14](https://github.com/ganelson/inform-evolution/pull/14)
[(IE-0015) World model enforcement](proposals/0015-world-model-enforcement.md)                           | 16 January 2023 | In progress                | [PR#15](https://github.com/ganelson/inform-evolution/pull/15)
[(IE-0016) Language extensions reform](proposals/0016-language-extensions-reform.md)                     | 5 February 2023 | Implemented but unreleased | [PR#16](https://github.com/ganelson/inform-evolution/pull/16)
[(IE-0017) Apps and extensions](proposals/0017-apps-and-extensions.md)                                   | 15 April 2023   | In progress                | [PR#17](https://github.com/ganelson/inform-evolution/pull/17)
[(IE-0018) Use options and kit configuration](proposals/0018-use-options-and-kit-configuration.md)       | 2 May 2023      | Implemented but unreleased | [PR#18](https://github.com/ganelson/inform-evolution/pull/18)
[(IE-0019) Unicode command parser](proposals/0019-unicode-command-parser.md)                             | 15 May 2023     | Implemented but unreleased | [PR#19](https://github.com/ganelson/inform-evolution/pull/19)
[(IE-0021) No automatic plural synonyms use option](proposals/0021-no-automatic-plural-synonyms.md)      | 7 June 2023     | Implemented but unreleased | [PR#21](https://github.com/ganelson/inform-evolution/pull/21)
