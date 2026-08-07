## Closes

The issue this change closes, as `Closes #N`. If it does not close one, say
which issue it belongs to and why it does not finish it.

## What changed and what failure it prevents

What the change does, and what goes wrong without it. Where this corrects
something, say what was wrong and how it was found.

## Numbers and the commands behind them

Every number in this body carries the command that produced it, run at the
commit being reviewed rather than in a working tree. If there are no numbers,
write that there are none.

## Does this change an output an operator would compare against a previous run

Yes or no, and if yes, what moves and by how much. A change to a coefficient, a
default, a tolerance, a random stream or an output format all count, and so does
a change that makes a run refuse where it used to answer. Somebody re-running an
old scenario after this lands has to be able to tell an intended change from a
regression.

## What was not checked

Anything you could not verify, could not run, or left out of scope. This section
stays negative if it was negative; it is not a place to reassure.
