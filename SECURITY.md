# Security

## What to report privately

Two kinds of defect belong in the private route rather than the open tracker.

The usual kind: anything that lets code run, data leak or a build be tampered
with. That includes the workflow files, which are the only executable content
this repository has today.

The kind that is specific to this project: a defect that makes the tool return a
plausible risk number that is wrong, without saying so. A casualty risk number
is read by people deciding whether a reentry is acceptable, and a wrong number
that looks right is the failure this project exists to avoid. If you find one,
treat it as a vulnerability until it is fixed, because an open issue describing
how to make the tool understate a risk is a working recipe for anyone who wants
that answer.

A wrong number that the tool is honest about is an ordinary bug and belongs in
the open tracker.

## How to report

Use GitHub's private vulnerability reporting on this repository, through the
Security tab, Report a vulnerability. It is enabled:

    gh api repos/iderex/niedergang/private-vulnerability-reporting
    {"enabled":true}

Please include what you did, what happened, what you expected, and the commit
you saw it at. For a numerical defect, the scenario file and the output are
worth more than a description of them, and if the difference is a number, the
command that produced it makes the report checkable.

## What to expect

This repository is maintained by one person, so there is no response time
commitment and inventing one here would be a promise nobody can keep. Reports
are read and answered.

There is no bounty.

## Versions

There is no release yet, so there is nothing to patch and no supported version
list to keep current.

    gh release list
    (no output)

Until a release exists, the only thing anybody can be running is a checkout of
the default branch, and the fix for anything found is a commit on it.

## Disclosure

Coordinated. A report is fixed first and described afterwards, and the person
who found it is credited unless they would rather not be.
