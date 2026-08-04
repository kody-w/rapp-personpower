# RAPP Personpower (pp)

**A universal unit for rating automation, the way horsepower rates
engines.**

In 1783 James Watt needed to sell steam engines to people who owned
horses, so he measured what a horse could sustain and priced his machines
in the buyer's own units. The unit outlived the horse. This repo does the
same for automation: it defines **personpower** so that any automated
process — a testing harness, a deployment pipeline, an agent doing a
person's clicking — can carry one honest, comparable number.

> **1 personpower (1 pp) = one attentive, competent power user performing
> the task hands-on at the interface, without dawdling and without
> assistance.**

An automation's rating is how many of those people it replaces on a given
workload, measured on the stopwatch:

```
P (pp) = T_person / T_engine
```

- **T_person** — wall-clock time for one attentive power user to execute
  the task's checklist by hand (measured directly, or estimated from the
  standard rate table in `spec/rates.json`).
- **T_engine** — wall-clock time for the automation to execute the SAME
  checklist, end to end, unattended.

**Worked example** (the measurement that named the unit): a UI test pass
of 33 checks across two surfaces — clicks, dialog expectations, layout
measurements, URL-parameter verification, a file download inspected.
Hand-executed by a power user: ~20 minutes. The automated harness,
measured with `/usr/bin/time`: **19.1 seconds**. Rating: **~60 pp**.

## The attention corollary

The stopwatch understates. The person's twenty minutes were twenty
minutes of full human attention; the engine's nineteen seconds cost none.
Report attention alongside power when it matters:

```
A (attention ratio) = attention_person / attention_engine
```

where `attention_engine` is the human-minutes consumed WHILE the
automation runs (usually ≈ 0, so A is effectively unbounded — say
"unattended" rather than dividing by zero).

## Measurement rules (what makes a rating honest)

1. **Same checklist both sides.** The engine must perform the checks a
   person would perform, through the same front door — real clicks, real
   dialogs, real downloads. An API call impersonating a click rates a
   different task.
2. **T_person is a power user, not a novice.** The unit is deliberately
   conservative: rate against someone who knows the tool and the
   checklist. Use `spec/rates.json` when you estimate instead of measure.
3. **Count the annoying things.** Unexpected dialogs, layout measurement,
   absence checks ("this must NOT be visible") belong in the checklist on
   both sides.
4. **No fake pulls.** If the engine skipped a check, it doesn't count.
   A run that didn't execute rates 0 pp, loudly.
5. **State the workload.** A pp rating is per-workload, like horsepower
   is per-engine: "60 pp on a 33-check UI regression pass", not "60 pp"
   in the void.

## Estimating T_person: the rate table

`spec/rates.json` carries standard per-check hand-execution rates
(seconds) for common interactive verifications — click-and-observe,
URL/parameter inspection, devtools layout measurement, download-and-open,
form fill, console review. Sum the checklist against the table when a
live human measurement is impractical. The table is versioned; cite the
version with your rating (e.g. `61 pp (rates v1)`).

## Calculator

```
python3 personpower.py --checks checks.json --engine-seconds 19.1
# -> {"T_person_s": 1155, "T_engine_s": 19.1, "personpower": 60.5, "rates": "v1"}
```

`checks.json` is a list of `{"type": "<rate-table key>", "count": N}`
entries — see `spec/rates.json` for the keys.

## Lineage

Coined in the RAPP ecosystem's *Personless Harness* work — the pattern of
an engine (a Brainstem) pulling the same carriage (Copilot, a browser, an
app) a person used to pull. Essay: <https://kodyw.com/the-personless-harness/>.
Related: the [RAPP Agent Registry](https://github.com/kody-w/RAR).

## License

MIT. Use the unit, cite the repo.
