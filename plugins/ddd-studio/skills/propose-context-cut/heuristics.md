# Heuristics for finding context boundaries on a Big Picture

A Big Picture Event Storming board (Alberto Brandolini, *Introducing
EventStorming*) shows a process as events on a timeline, with the people,
systems, rules and views around them. Boundaries are not drawn on it;
they show up as places where several signals change at once. Each
section is one signal: what to look for, and what counts as evidence.

None of these is a rule on its own. A boundary is worth proposing where
two or more agree.

## 1. Pivotal events

Some events end one phase and start another: after them, different
people care, different questions are asked, and the earlier details
stop mattering (Brandolini calls them pivotal events).

- Look for: an event after which the actors, the read models or the
  vocabulary on the following cards change; phase tags that change;
  events that hand something over ("released", "handed over", "ready").
- Evidence: the pivotal event and the cards just before and after it.

## 2. Change of actor (who is in charge)

A context usually has its own deciders. Where the people who act on the
cards change, a boundary is likely.

- Look for: consecutive events whose actors differ; a role that appears
  only in one stretch of the timeline; a handover between roles.
- Careful: one department is not automatically one context, and one
  role can work in two contexts. The actor is evidence, not proof.

## 3. Change of language

A bounded context is the boundary within which a model and its words
have one meaning (Eric Evans, *Domain-Driven Design*, 2003, ch. 14).

- Look for: the same word with different meanings on different cards or
  in notes ("available", "train", "order"); different words for the
  same thing; a word that is defined by words from another stretch of
  the board.
- A word with two meanings is a strong boundary signal: the two meanings
  belong in two contexts, and the boundary runs between them.

## 4. Who decides a rule

Policies and rules on the board carry the business decisions. A context
should own the rules it applies (Vlad Khononov, *Balancing Coupling in
Software Design*, 2024: functional coupling across a boundary makes every
change touch both sides).

- Look for: policy cards and rule texts; whose threshold or condition it
  is; whose events trigger it and whose command it causes.
- A rule whose condition needs knowledge from one stretch and whose
  threshold is set by people from another stretch is a **split rule**.
  Do not draw a boundary through it without saying who owns it; usually
  it is an open question for the business.

## 5. Pace and time horizon

Things that change for different reasons or at a different rhythm
belong apart: planning days ahead against acting tonight, a catalogue
that changes monthly against orders that change every minute.

- Look for: phase tags or notes about time ("the day before", "tonight",
  "weekly"); plans that are made in one stretch and overridden in
  another.
- A plan made by one group and changed by another at a different pace is
  a boundary signal, and the question "who may change the plan" belongs
  to the humans.

## 6. External systems

An external system is a boundary by definition. The question is which
context talks to it and translates its language.

- Look for: external-system cards and the events they hang on; system
  names inside event texts.
- Usually one context owns the translation (an anti-corruption layer).
  Two contexts talking to the same system in different words is a
  signal for a question, not a reason for a context of its own.

## 7. Open questions and conflicts

Hotspots and notes cluster where people disagree. Disagreement often
means two models meet there.

- Look for: notes that point at cards from two stretches of the
  timeline; notes about words; notes that ask "who decides".
- Do not resolve them. Put them at the boundary they suggest and list
  them as questions.

## 8. Counter-heuristics: cuts to avoid

Test every candidate cut against these (DDD Crew, *Bounded Context
Canvas*; Nick Tune; the anti-patterns most reviews find):

- **Cut by noun** ("everything about the train"): behaviour ends up
  spread over the callers. Cut by decisions and language instead.
- **Cut by department only**: a department can span two models, and two
  departments can share one.
- **Cut by technology or legacy** ("notifications", "the old system")
  with no business decision of its own.
- **Pass-through context**: receives and forwards, decides nothing.
- **Split rule**: the condition in one context, the threshold or the
  decision in another (§4).
- **Chatty boundary**: one business step crosses the same boundary back
  and forth. Play a case along the timeline and count.
- **God context**: most events, most rules and most words in one place,
  the rest as satellites. Split by §1 to §5 or say why not.
