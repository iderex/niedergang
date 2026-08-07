# 0001. Decision records

## Date

2026-08-08

## Status

accepted

## Question

Where does a decision that shapes this tool get written down, what does it have
to contain to be worth reading a year later, and what happens to it when it
turns out to be wrong.

The question is asked before any of the physics decisions because none of them
can be written until there is a shape to write them in.

## Options considered

Nothing written down, with the reasoning left in commit messages and in the
heads of whoever was there. Cheapest, and it is the state this repository would
drift into by default. It costs the ability to revisit: a reader who disagrees
with a model choice cannot tell whether the alternative was weighed and rejected
or never considered, so the argument is had again from the start.

The issue tracker as the record. It is where the reasoning already happens and
it needs no new convention. It costs two things. An issue is a conversation with
a timeline, and a decision is a statement, so a reader has to reconstruct the
outcome from the thread and can reconstruct it wrongly. And the tracker is a
service outside the artefact: a clone of this repository, which is what somebody
auditing a risk number will have, does not carry it.

A wiki. Easy to write and easy to reorganise. It has no pull request, so a
decision can change without review, and no history a reader can bisect against
the code, so there is no way to ask what the rule was on the day a result was
produced.

Records in the repository, in version control, next to the code they govern.
Reviewed on the way in like any other change, versioned with the thing they
describe, present in every clone, and greppable. It costs a convention that has
to be followed, and it puts a decision behind a pull request, which is slower
than writing a comment.

## Decision

Decision records live in this repository under `docs/decisions/`, one file per
decision, in Markdown, reviewed and merged like any other change.

The required fields are the second level headings of
`docs/decisions/0000-template.md`. That file is the authority for the list. This
record carries the same headings, because it is a record and the rule applies to
it, and a reader who wants to know whether the two have drifted apart runs:

    diff <(grep '^## ' docs/decisions/0000-template.md) \
         <(grep '^## ' docs/decisions/0001-decision-records.md)

Empty output is agreement. The same command with any other record in the second
position says whether that record carries every required field. There is one
list of fields in this project and it is the template's headings; this paragraph
deliberately does not repeat them, because a second copy is a copy that can go
stale while still reading as authoritative.

Two of the fields are the reason this format was not taken off a shelf
unchanged. Most decision record formats stop at the reasons for the option
taken. `What would change this` asks for the measurement or the constraint that
would make the record wrong, written while the decision is fresh, so that
revisiting it later is a matter of checking a stated condition rather than
re-running the whole argument. `What depends on this` is what makes a
supersession tractable, because it says which other records and which parts of
the tree have to be looked at again.

`Reasons against` is required and may not be empty. A record whose author could
find no argument against the option taken has not looked, and that record is the
one that will be quoted years later by somebody who cannot tell it was never
tested.

The number is fixed by the issue that plans the record, on its `Scope:` line,
and that line is the authority for which number belongs to which decision. A
number is never reused, including where a record is withdrawn before it is
accepted. Four digits, zero padded, so the directory sorts.

A record is superseded, never edited into a different decision. The old file
stays where it is, its status becomes `superseded by NNNN`, and the new record
says what it replaced and why. Fixing a typo, a broken link or a wrong file path
in a landed record is a correction and not a supersession; changing what was
decided, or the reasons it was decided, is a supersession however small the edit
looks.

A record is not a place to argue. The argument happens in the issue and in the
pull request that lands the record, and the record states what came out of it.

## Reasons

The tool this project is building produces a number that somebody will act on,
and the number is only as trustworthy as the reader's ability to see what was
assumed. A model choice, a frame convention or a threshold is not visible in the
output, so the place it is visible has to travel with the code.

Version control gives the property the other options cannot. A reader can check
out the commit a result was produced at and read the decisions as they stood
that day, which is the same property the run manifest is for on the data side.

Review is the second reason. A decision that lands as a pull request has been
read by somebody other than its author before it binds anything, or, where no
second reader exists, the absence is on the record. A wiki edit and a tracker
comment have neither property.

## Reasons against

This is more ceremony than a project with one committer needs, and ceremony that
is not maintained is worse than none, because a directory of stale records reads
as current. The failure mode is real and it is not hypothetical in this format:
`What depends on this` is exactly the field most likely to be right on the day
it is written and wrong six months later.

Putting decisions behind a pull request also slows down the case where the
decision is obvious and the record is a formality. Some of the records this
milestone plans will be short, and the shape will look heavier than what it
carries.

Against the specific format: nine required fields is more than the widely used
short forms carry, and every field that is required is a field somebody will
fill with a sentence that says nothing rather than leave visibly empty. The
answer to that is review, not a shorter list, and review is a person rather than
a check.

## What would change this

A check that reads a record and refuses one that is missing a field would move
the shape rule from prose into enforcement. Today no check does. The command
above is the whole mechanism, it is run by a person or not at all, and nothing
in this repository refuses a record that skips a heading or a template that
grows one this record does not. The gate that would carry such a rule is
issue #35, and until it does, this section is the disclosure rather than a plan.

If the numbering scheme collides in practice, because two branches plan two
records with the same number, the authority moves from the issue's `Scope:` line
to something that cannot collide. It has not happened yet and the scheme is not
changed in anticipation.

If a decision turns out to need a machine readable form, because something in
the tool has to read it rather than a person, Markdown with headings is the
wrong container and this record is superseded rather than stretched.

## What depends on this

Every decision record planned in milestone 02 and every one written after it.
The records already planned name their files on their issues' `Scope:` lines, so
this record fixes their shape and not their content.

The contributing document, which tells somebody where a decision goes.

Anything that later reads a record mechanically, which nothing does today.
