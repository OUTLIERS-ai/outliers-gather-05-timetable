# Outliers Gather — Layer 5 — The Timetable

Runs any command you like, on a schedule, while you are doing something else.

Every other layer in this set is about one website. This one is about none of them, which is why it
is the piece you will still be using long after the rest. It starts commands. What those commands
do is entirely their own affair.

---

## Before you start

| | What | How to check |
|---|---|---|
| 1 | **Python 3.8 or newer** | `python --version` |
| 2 | **Your CRM** | the folder has a `_layers` folder inside it |

Nothing here costs money, and nothing here needs anything installed beyond Python itself.

---

## Install

```
git clone https://github.com/OUTLIERS-ai/outliers-gather-05-timetable
cd outliers-gather-05-timetable
python install.py
```

It finds your CRM, asks a few questions, and copies the layer into `_engine` inside it.

**Nothing reaches the outside world during installation.** It writes files, asks the questions, and
stops. It starts no command, and it will not put a window on your screen, now or ever.

### The questions

| Question | What it changes |
|---|---|
| What is the first command you want it to run? | One entry on the timetable. Leave it empty for none. |
| At what time, and on which days? | When that entry fires. |
| Should it start every time you log in? | A logon task on Windows, a LaunchAgent on a Mac. |

---

## Use it

The tools live in `_engine` inside your CRM, alongside the ones your earlier layers installed. Open
a terminal **there**.

```
python timetable.py list
python timetable.py add "<command>" --at 09:30 --days mon,tue,wed,thu,fri [--label "..."]
python timetable.py remove <id>
python timetable.py enable <id>
python timetable.py disable <id>
python timetable.py run
python timetable.py start
python timetable.py stop
python timetable.py status
python timetable.py install-startup
```

`add` also takes two optional settings:

| | What it does |
|---|---|
| `--minutes N` | How long this entry may take before it is ended. Default 45. |
| `--shift N` | How far its minute may move, either way. Default 4. |

### Look before you leap

```
python timetable.py run --dry-run
```

Shows what today holds and when, and starts nothing at all. It reads the timetable and works out
times. There is no route from it to starting a command, which is what makes it honest rather than a
promise, and the tests fail if that ever stops being true.

`run` on its own is the loop itself, in the terminal you are looking at, with everything it does
printed in front of you. `start` is the same loop in the background, with no window. Run it in the
terminal a few times before you set it loose.

---

## One entry. Once a day. Watched.

That is the first step, and it is worth taking it slowly.

A timetable of six entries built in one sitting is six ways to be wrong at once, and no way to tell
which one went wrong. Add one entry, let it run for a week, and read the log:

```
_state/timetable/timetable.log
```

Every wake-up decision and every line the command printed goes in there, stamped with the moment it
happened rather than the moment the run ended.

---

## Nothing appears on your screen

This is the detail the whole layer is built around, and the reason people keep a timetable rather
than switching it off.

**On Windows**, starting a program from a program makes the operating system give the new program
its own console window — a black box that appears over whatever you were doing, every single time
the entry fires. Around forty scheduled jobs on one machine behaved that way in July 2026, and the
work they did was correct. The windows were very nearly the reason the whole set was abandoned.

Two corrections, and this layer applies both, one over the other:

- The background program runs as `pythonw`, the copy of Python that has no console at all.
- Every command it starts carries a flag that stops Windows creating a console for it. This is
  separate and both are needed: a program with no console of its own that starts another program
  makes Windows give **that** one a fresh window, and capturing the output does not prevent it —
  capturing changes where the output goes, it does not stop the window being created.

**On a Mac**, none of this exists. There is no console window for a background program to be given,
so there is nothing to suppress and nothing to configure. A Mac member is not missing a feature
here; the problem does not arise on that machine.

---

## What the timetable actually does

### The firing minute moves, and it is worked out rather than rolled

An entry set to 09:30 does not fire at 09:30. Its minute moves by up to a few minutes either way,
so it lands at 09:27 one day and 09:33 the next. A job that fires at exactly the same minute every
day is a pattern no person produces, and staying inside every limit does not help you if the rhythm
gives you away.

The move is **not** a random number. It is worked out from the entry's name and the date. Ask the
same question twice and you get the same answer.

That property is the whole point. The background program asks "when does this fire today?" on every
wake-up, dozens of times an hour. With a rolled number the answer would be different every time it
was asked: the entry would drift forwards and backwards through the morning, fire early, and fire
again after a restart because the new answer no longer matched the old one. Worked out from the
name and the date, a restart at noon computes the identical minute it computed at dawn.

The step from one day to the next is chosen so that it can never land back where it was yesterday.
The honest cost of that guarantee is that one entry's minutes walk the range in a repeating order
rather than scattering freely. The alternative — a plain hash of the name and the date — would hand
you the same minute two days running roughly one day in nine, and "occasionally identical" is
exactly the pattern being avoided.

### A missed entry still runs, up to two hours late

A laptop asleep at nine still runs the nine o'clock entry when it wakes at ten. Without this,
closing the lid means the day's work never happens and nothing says why. Two hours is long enough
to cover a lie-in, a meeting or a restart, and short enough that a job set for the morning never
lands in the evening.

### Nothing fires twice

The record of what has already run is written the moment **before** an entry is started, never
after it finishes. A machine switched off mid-run restarts into "that one has had its turn" rather
than starting it again.

### Each entry says how long it may take

Not one limit over everything. A job that walks a long list is paced by its own allowance, not by
the clock, and forty items at three minutes each is two hours of correct, authorised work. A single
45-minute limit sitting over that ends the job in the middle of work it was allowed to do — it cost
26 actions on one occasion and ended a run at 15 of 40 on another. So the limit belongs to the
entry: `--minutes 180` for the long walk, `--minutes 2` for copying a file.

A longer allowance can never raise how much work happens. Whatever limits the command carries still
bind, because this starts it exactly as you would from a terminal.

### Being alive is judged on progress

"It has been running for an hour" says nothing about whether it is working. So output is read as it
is produced, and every line counts as progress. A command that keeps talking keeps the timetable
looking healthy however long it takes. A command that goes silent for ten minutes stops refreshing
the sign of life, so a genuine wedge is still caught.

### It knows which running program is its own

Every running program has a number. The operating system reuses those numbers, so a number on its
own is a label rather than a name. On one occasion a dead job's number was handed to an unrelated
laptop utility, after which the checker reported the dead job as running **and** refused to start
the real one — two days of nothing, reported as healthy. The moment the program started is recorded
alongside its number, and the pair is what identifies it.

---

## What is in here

| File | What it is |
|---|---|
| `engine/timetable.py` | The command you type |
| `engine/timetable_store.py` | The timetable file, the record of what has run, the log |
| `engine/timetable_clock.py` | When each entry fires, and whether it still may |
| `engine/timetable_runner.py` | Starting one command, watching it, knowing when it has wedged |
| `engine/timetable_daemon.py` | The loop, the sign of life, which program is ours |
| `engine/timetable_platform.py` | The three places Windows and a Mac genuinely differ |
| `engine/crm_paths.py`, `engine/safe_write.py` | Shared with your CRM. Written only if missing, never overwritten |
| `tests/test_timetable.py` | The proof |

Three files are written into your CRM:

| Path | What it holds |
|---|---|
| `_layers/timetable.json` | The timetable. Yours. Edit it by hand if you prefer. |
| `_state/timetable/state.json` | What has already run, and on which date. |
| `_state/timetable/timetable.log` | What happened, one line at a time. |

---

## The tests are the proof

```
python tests/test_timetable.py
```

Every check exists because of something that would otherwise be believed rather than known. Four of
them pin a dated failure directly: the sign of life written too late, the reused process number,
the single time limit stretched over work paced by its own allowance, and the console windows.

| Exit code | What it means |
|---|---|
| **0** | Everything passed. |
| **1** | Something failed. The line that failed says what. |
| **2** | Your CRM was not found, so the tests stopped rather than run a shorter version of themselves. Not a fault in this layer — install your CRM's first layer and run it again. |

That third case matters. This layer writes your timetable through two files that belong to your CRM
and are shared by every layer, and the promise that a write can never damage the file already there
lives in **your** copy of them. Without it there is nothing to test against, and a check that did
not run is not a check that passed. If your CRM is somewhere unusual, point at it:

```
OUTLIERS_CRM=/path/to/your/CRM python tests/test_timetable.py
```

The tests schedule nothing, start no background program, run no real command, and write nowhere
near your real CRM — every test points at a throwaway folder first.

---

## What this layer leaves unsolved

The timetable will run a command at the wrong moment as faithfully as the right one, and it has no
opinion about whether the command should have run at all. It answers "when", and nothing else. The
question of whether today is a day to be doing this — and how much of it — belongs to the layer
underneath, and stays there.
