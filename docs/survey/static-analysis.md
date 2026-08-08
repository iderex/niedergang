# Static analysis for this codebase, and what each candidate would actually find

Issue #83 asks what stands in for the two first-party static analysis contexts
on the target gate, on the assumption that they analyse a language this board
does not use. That assumption is the first thing this document has to correct,
and correcting it changes the shape of the question rather than the answer.

Nothing here was run. There is no source in this tree to run it on:

    git ls-tree -r --name-only 37e72533f6821e0f082911245e5fc0dc05fc4485 | wc -l
    32
    git ls-tree -r --name-only 37e72533f6821e0f082911245e5fc0dc05fc4485 | grep -Ec '\.rs$|Cargo\.toml|rust-toolchain'
    0

Thirty-two tracked paths at that commit, none of them source and none a package
manifest. The commands name the commit rather than reading a working tree, so a
reader gets the same answer.

Every row below is therefore a claim about a tool, taken from that tool's own
documentation, and not a result measured on this codebase. Where the
documentation does not answer a question, the row says so rather than filling it
in.

## The premise has moved

CodeQL supports Rust. The changelog entry of 14 October 2025 states it plainly:

> Rust joins the list of generally available languages (C/C++, Java/Kotlin,
> JS/TS, Python, Ruby, C#, Go, GitHub Actions, and Swift) for CodeQL. [1]

The supported-languages reference carries the row and one footnote that matters
to this tree:

> Rust | Rust editions 2021 and 2024 | Rust compiler | `.rs`, `Cargo.toml` [2]

> Requires `rustup` and `cargo` to be installed. Features from nightly
> toolchains are not supported. [2]

So the question #83 was written against, which analyser replaces one that cannot
read this language, has no subject. The analyser can read it. What is left is a
narrower and more useful question: whether what it looks for is what this
codebase gets wrong.

## The yardstick

#83 names four defect classes for this codebase specifically: arithmetic
overflow and precision loss, indexing and slicing that can panic on malformed
input, unchecked conversion at the data boundary, and anything unsafe. The first
three are numerical-tool defects rather than security defects, and a scanner
built to find injection and credential handling will not look for them. That is
the yardstick every row below is measured against, and it is the reason
availability and coverage have to be judged separately.

## The candidates

### CodeQL

Reads Rust, per the two quotations above. Its findings reach the code scanning
dashboard, and that route is already proven in this tree rather than assumed:
two workflows upload SARIF through the same action today.

    git grep -l 'codeql-action/upload-sarif' 37e72533f6821e0f082911245e5fc0dc05fc4485 -- .github/workflows
    37e72533f6821e0f082911245e5fc0dc05fc4485:.github/workflows/scorecard.yml
    37e72533f6821e0f082911245e5fc0dc05fc4485:.github/workflows/zizmor.yml

It can gate a pull request, since a code scanning job is a check like any other
and this tree already fails a build on findings in `zizmor.yml`.

On licence, the CodeQL CLI terms permit exactly the case this repository is in
and exclude the other one. The permitted use is to "Perform analysis on the Open
Source Codebase", with database generation allowed "If the Open Source Codebase
is hosted and maintained on GitHub.com", and the terms bar use "in connection
with any codebase that is not an Open Source Codebase". [3]

What is not established is coverage against the yardstick. The two changes to
Rust queries visible in the 2026 changelog entries read here are both to one
query about hard-coded cryptographic values [4][5]. That is a claim about where
the Rust pack's attention has recently been and it is not a count of the pack or
a statement of what the pack contains. Reading the query pack itself is what
would settle whether CodeQL finds precision loss, a panicking index or an
unchecked conversion, and that was not done here.

### The analyser the toolchain ships

Clippy comes with the toolchain record 0002 pins, so it costs no new dependency
and no new runtime. It is not this issue's to adopt: the format and lint gates
are #28, and clippy is that issue's subject.

What belongs here is one observation about the yardstick. Lints matching two of
the four classes exist and are off by default. `arithmetic_side_effects` sits in
the `restriction` group at level allow, and `cast_possible_truncation` and
`cast_precision_loss` sit in `pedantic` at level allow [6]. So the arithmetic and
conversion classes are a configuration decision rather than a missing capability,
and the decision belongs to #28. The full mapping from the four classes to a
lint set was not enumerated here and should not be enumerated in two places.

### Semgrep

Rust is listed as generally available for Semgrep Code, with cross-function
dataflow analysis, and without the cross-file dataflow that the larger languages
get [7]. The engine repository is under LGPL-2.1 [8], which is a licence this
project can consume as a tool regardless of what it later chooses for itself,
since running an analyser over a codebase is not distribution of that analyser.

Two things are not established. The row that gives Rust its maturity level also
attributes cross-function analysis and a Pro rule count to it, and which of those
belong to the freely available engine rather than to the paid platform was not
determined here. Whether Semgrep's Rust rules look for any of the four classes
was also not determined.

Semgrep can emit SARIF, so the dashboard route is available on the same
mechanism the two workflows above already use. That is an inference from the
format rather than something run here.

### Miri

Miri detects undefined behaviour, which is the fourth class on the yardstick and
the one no other candidate here addresses directly. It is installed on Rust
nightly [9].

That is a real cost against record 0002 rather than a detail. The record pins the
toolchain to an exact version rather than a channel, so that a fresh clone
selects that version and no other. Adding Miri adds a second toolchain to the
tree, and no record in the tree named a nightly toolchain before this document
did:

    git grep -n -i 'nightly' 37e72533f6821e0f082911245e5fc0dc05fc4485 -- docs/

No output, at that commit. The command names the commit deliberately: run against
a later tree it finds this document, which says nothing about the records.

The CodeQL footnote quoted above points the same way: features from nightly
toolchains are not supported there either. A codebase that stays on stable keeps
both of those simple, and a codebase that reaches for Miri should do it knowing
it is buying a second toolchain.

Miri also runs the test suite under an interpreter rather than reading the
source, so what it covers is what the suite executes. That makes it a suite
property rather than a scanner, and it would answer to #29 and #30 rather than to
a static analysis workflow.

### Advisory and licence tooling

`cargo-audit` and `cargo-deny` read the dependency set rather than the source, so
they find none of the four classes. They are #32 and they are named here only so
that a reader comparing this document against the gate does not count them twice.

### Adopting nothing

#83 asks for this option to be recorded rather than left as the absence of a
decision, and it is a real option. A scanner adopted so that a table has a row
in it makes the table read as covered, which is worse than a stated gap, because
the next reader stops looking. What makes this option defensible here is that the
first three classes on the yardstick are reachable by a lint configuration in #28
and by the greppable invariants of #35, both of which are this board's own work
rather than a purchased capability.

What makes it weaker is the fourth class. Nothing in #28 or #35 as currently
written detects undefined behaviour, and `unsafe` is the one class where an
outside tool is doing something this board cannot do by reading its own tree.

## What this document does not answer

Whether any candidate finds the four classes. Every row above says what its tool
reads and what it can gate; none of them says what it would have found in this
codebase, because there is no codebase. That question becomes answerable as soon
as a workspace exists, and the honest sequence is to run the candidates on real
source before choosing between them rather than to choose from documentation.

Nothing here was read off the target gate's own configuration. The comparison
against it is the parity table of #82, and this document supplies a row for that
table rather than writing one into it.

## References

[1] GitHub Changelog, "CodeQL scanning Rust and C/C++ without builds is now
generally available", 14 October 2025,
https://github.blog/changelog/2025-10-14-codeql-scanning-rust-and-c-c-without-builds-is-now-generally-available/

[2] CodeQL documentation, "Supported languages and frameworks",
https://codeql.github.com/docs/codeql-overview/supported-languages-and-frameworks/

[3] GitHub CodeQL CLI licence,
https://github.com/github/codeql-cli-binaries/blob/main/LICENSE.md

[4] GitHub Changelog, "CodeQL 2.24.0 adds Swift 6.2 and .NET 10 support",
29 January 2026,
https://github.blog/changelog/2026-01-29-codeql-2-24-0-adds-swift-6-2-support-net-10-compatibility-and-file-handling-for-minified-javascript/

[5] GitHub Changelog, "CodeQL 2.26.1 improves analysis accuracy and framework
coverage", 29 July 2026,
https://github.blog/changelog/2026-07-29-codeql-2-26-1-improves-analysis-accuracy-and-framework-coverage/

[6] Clippy lint index, https://rust-lang.github.io/rust-clippy/master/index.html

[7] Semgrep documentation, "Supported languages",
https://docs.semgrep.dev/supported-languages

[8] Semgrep repository, https://github.com/semgrep/semgrep

[9] Miri, https://github.com/rust-lang/miri
