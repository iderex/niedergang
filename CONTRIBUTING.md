# Contributing

This repository builds an open tool for reentry breakup and the ground risk that
follows from it. The output is a number somebody may act on, so the standard for
a change here is not that it works but that a reader can check it.

## Start with an issue

Planning happens on the tracker before the code that depends on it exists. An
issue says what is wrong, what the evidence is, and what done means for it, and
it names the files it expects to touch on a line beginning `Scope:`. If the
evidence is a number, the issue carries the command that produced it.

A change that shapes the architecture is written down as a decision record under
`docs/decisions/` before the code that assumes it. The shape of a record is
fixed by `docs/decisions/0001-decision-records.md` and the field list is the
template next to it.

## Sign your work

Every commit carries a `Signed-off-by` trailer whose name and address match the
commit author, and the gate on a pull request refuses one that does not. What
you are certifying is the Developer Certificate of Origin 1.1, in `./DCO`, which
is the published text unchanged. Read it once; it is short.

    git commit -s

If you already committed without it:

    git rebase --signoff <base>

The trailer has to match the author exactly, so a commit authored under one
address and signed off under another is refused. Commits authored by GitHub's
own automation are exempt, because they cannot sign for themselves.

There is one thing wrong with this arrangement today and it is written here
rather than left for a contributor to find. The certificate says you have the
right to submit the contribution under the open source license indicated in the
file. This repository has no license file, so there is no such license, and a
sign-off currently certifies against terms that do not exist. Choosing the
license is the first entry of issue #1, and until it is answered a contribution
from outside cannot be accepted whatever its quality.

## Building and testing

There is nothing to build yet. The tracked tree today is documents and workflow
files:

    git ls-files

The build gate and the test gate are issues #27 and #29, and the commands a
contributor runs before pushing arrive with them, in this document, once they
exist. Anything stated here before that would be a command nobody can run.

## What refuses a change

The pull request page shows which checks ran and what each one said, and that is
the authority. This document does not carry a list of them, because a list here
goes stale against the workflow files that decide it, and a contributor who
trusts the stale list is worse off than one who reads the page.

The one thing you can get wrong before pushing and fix in a second is the
sign-off above. The rest is read on the pull request.

The ruleset on the default branch refuses a direct push and requires a pull
request. It requires no status check to pass, so a red check does not by itself
block a merge:

    gh api repos/iderex/niedergang/rulesets/20530587 --jq '[.rules[].type]'
    ["deletion","non_fast_forward","pull_request"]

Which checks become required is entry 3 of issue #1. Until it is answered, a red
check is stopped by a person reading it.

## Commits and pull requests

One topic per commit and one topic per pull request. A commit carrying two
unrelated changes gets a message describing one of them.

A commit message says what changed and what failure it prevents. Where it
corrects something, it says what was wrong and how it was found.

A pull request names the issue it closes. Any number in its body carries the
command that produced it, run at the commit being reviewed rather than in a
working tree. A claim that was not checked is written as a claim, and a sentence
saying something was not done stays in the body in those words.

The pull request template asks for these directly, and the hygiene gate that
would refuse a body missing them is issue #34. It is not in the tree yet, so
today they are read by a person.

## Contributions from outside

Whether outside contributions are accepted, and on what terms, is entry 2 of
issue #1 and is not yet decided. It cannot be decided before the license.

Until it is, the useful thing an outside contributor can do is open an issue.
Finding a wrong number, a bad citation or a model that does not do what the
documentation says is worth more to this project than code it currently has no
terms to accept.

## Reporting something you should not report in public

`SECURITY.md` has the private route and says which kinds of defect belong in it.
A defect that makes the tool return a confident wrong risk number is one of
them.
