# (IE-0005) Removing translates into Unicode

## Motive

Inform currently contains an obscure language feature enabling Unicode code
points (specified in decimal) to be given names. For example:

	latin small letter s with acute translates into Unicode as 347.
	dentistry symbol light down and horizontal with triangle translates into Unicode as 9156.

In practice this feature is used only (and obsessively) by the extensions
"Unicode Full Character Names" and "Unicode Character Names", built into Inform,
which make a huge number of such translations. Those extensions have to be
explicitly included by the user to have any effect, and they are slow to read
(because huge) and just saturate the compiler's symbols table. We can do better.

## Proposed changes

1. These two extensions will be removed.

2. The Inform installation will contain a stand-alone file of Unicode character
names. This might even be a straight copy of [the Unicode standard name list](https://www.unicode.org/Public/14.0.0/ucd/NamesList.txt),
though it may be sensible to continue to restrict Inform to the Basic Multilingual Plane characters from that.
The file will be read in to the compiler only on demand, that is, when the
compiler finds that it has to make sense of "unicode ...":

	"[unicode Latin capital letter L with stroke]odz Churchyard"

3. A problem message will reject any usage of "translates into Unicode".
