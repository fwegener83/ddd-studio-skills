# Checks for the example data of a flow

Run them in this order. Each check says what to compare and what counts
as a finding. A flow is one storyline in time (no branches), so every
slice may rely on everything to its left.

## 1. Identity: the same thing keeps its name

- Every entity the story follows (an aggregate instance by its
  `identifiedBy` field, or a value that repeats across slices: an order
  number, a vehicle number, a customer id) has the same value wherever
  it is meant to be the same thing.
- One id always comes with the same attributes (the same order belongs
  to the same customer in every slice).
- Finding: an id or number that changes between slices of the same story
  without a slice that explains it.

## 2. Command and event agree

- Within one command slice, fields the command and its events share by
  name carry the same values, unless the description says the aggregate
  changes them.
- Finding: the event says something other than the command asked for,
  without a reason in the model.

## 3. Time runs forward

- Timestamps grow along the flow: a slice to the right does not happen
  before a slice to the left.
- Within one entity's life, the order of its own times is right (begun
  before ended, ordered before delivered, reserved before expired).
- Planned times stay the same across the story unless a slice changes
  them (a planned date in one slice and a different one in another for
  the same thing is a contradiction).
- Times in the slice description ("at 5:15") match the data.

## 4. Invariants and constraints hold

- For every command slice that ends in events: the example data and the
  state the story has built so far satisfy the aggregate's invariants
  and the command's constraints. If they do not, the slice should be a
  rejection, or the data is wrong.
- For a slice that ends in a rejection: the data really breaks the
  invariant it points to. A rejection the data does not justify is a
  finding too.

## 5. Triggers and policies

- A slice triggered by a policy, process manager or event handler: the
  triggering event (the occurrence the slice points to, or the nearest
  one to its left) exists, and its data satisfies the policy's condition
  (its constraints and rule text).
- Compute the condition with the story's values (times, amounts,
  states). For a time window, "now" is the time of the triggering event
  unless the model says otherwise. A policy that fires although its
  condition is not met, or a story where the condition is met and the
  reaction is missing, is a finding.

## 6. Read models show what happened

- For every read-model slice, replay the events of all earlier slices
  that the read model projects from, using its projection rules, and
  compare the result with the example rows.
- Finding: a row that should have changed but did not (a state left
  behind), a row that should exist but is missing, a row with no event
  behind it, a value that differs from the event it came from.
- Also check that some projection rule creates the row at all. Rules that
  only update an existing row ("set the status of the row") need an
  earlier event that adds it; if no slice shows that event, the row has
  no origin in the story (a gap).
- The query's answer (where set) is consistent with the rows and the
  query's parameters.
- A projection rule that seems to contradict the read model's name or the
  slice description (a view of "open" items that keeps closed ones) is a
  question for the modeller, not a finding about the data. Offer the
  change of the rule as a second proposal at most.

## 7. Quantities add up

- Amounts, counts and prices that relate to each other are consistent:
  a shortage is not larger than what was asked for, parts add up to a
  total, a balance follows from its bookings. These are common sense,
  not model rules: report a mismatch as a contradiction only when the
  model or a description ties the two values together, otherwise as
  suspicious.
- Units and currencies stay the same for the same value.

## 8. References point backwards

- A value that refers to something (an id of another entity) refers to
  something introduced earlier in the story, or the slice description
  says where it comes from.
- Finding: a reference to something the story never introduced.

## 9. Description and data tell the same story

- The slice description and the flow description match the data
  ("two parts" and `quantity: 2`; "the courier brings the parts" and a
  later slice that uses them).
- A step the description claims but no slice shows is a **gap**, not a
  contradiction. Report it as such.
