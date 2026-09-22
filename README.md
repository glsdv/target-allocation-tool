# Personalised Target Allocation Tool (Excel)

Splits a store's daily sales target fairly across staff, based on when each
person actually works — how many hours, at what time of day, and in what
role — rather than splitting it evenly or by hours alone.

![Main sheet](screenshots/main-sheet.png)

## The problem

A flat or hours-only target ignores that an hour on a Saturday afternoon
isn't worth the same as a quiet Tuesday morning. Splitting targets that way
made them feel arbitrary and hard to compare between staff, especially
across a mix of full shifts, short shifts, and different roles.

## What it does

- Takes each person's rota — their shift on each day of the week — as input
- Counts only the part of each shift inside the store's trading hours
- Deducts a lunch break, scaled proportionally to shift length
- Weights the remaining time by an hour-by-hour trading pattern, so busier
  hours count for more
- Adjusts for role, since a more senior person's target is set relative to
  the team's, not equal to it
- Splits each day's target across everyone working that day, in proportion
  to their resulting "weight" — so the targets always add up exactly to the
  day's total

## How it's built

The workbook has three sheets:

- **Employee_Shifts** — the sheet you actually use. Enter each day's target,
  and each person's role and shift from a dropdown. Targets calculate
  automatically.
- **Settings** — every input that drives the model: trading hours per day,
  the shift list, role factors, the lunch rule, and an hour-by-hour weight
  table. Yellow cells are the only ones meant to be edited.
- **Shift_Values** — a calculation-only sheet that works out what one shift
  is worth on each day of the week, before Employee_Shifts uses it.

![Settings sheet](screenshots/settings-sheet.png)

The core formula, once a shift's value has been looked up, is short:

```
Person weight = shift value x role factor
Target        = person weight / total weight of everyone working that day
                 x that day's target
```

The hour-by-hour weighting is the more interesting piece. For each hour of
a shift, the sheet counts the fraction of that hour actually worked
(clipped to trading hours) and multiplies it by that hour's weight, then
sums across the shift:

```
=SUMPRODUCT(overlap_per_hour, hourly_weight_row)
```

One Excel quirk worth flagging: `MIN`/`MAX` don't compare a number against
a range element-by-element — they just return the overall smallest or
largest value across everything you give them. Getting a true hour-by-hour
overlap needed the algebraic identities for elementwise min and max
instead (`(a+b−|a−b|)/2` = min, `(a+b+|a−b|)/2` = max), applied across the
whole hour-weight row at once.

## Hourly weights

![Hourly weight table](screenshots/settings-hourly-weights.png)

The current weights are **illustrative**, based on patterns described in
several published retail-footfall reports — a lunchtime peak, an evening
pickup, Friday skewing later in the day, Saturday running flatter across
the whole day, and Sunday (shorter trading hours) front-loaded toward early
afternoon. They are not measured from real transaction data, and the sheet
says so.

The natural next step — not yet built — is to derive real weights from an
actual dataset: group hourly transactions by day of week and hour, and
calculate each hour's share relative to the average. That would replace
the illustrative table with numbers that are actually defensible.

## Data in this workbook

The employees, roles, shifts and daily targets in this copy are randomly
generated for demonstration. No real staff data or actual store targets
are included.

## Built with

Excel formulas and structure developed with AI assistance (Claude), based
on my own weighting logic and ~10 years of retail stock control and
operations experience.

## What I'd improve

- Replace the illustrative hourly weights with ones derived from real
  hourly sales data (see above)
- Add a small validation check that flags if trading hours or shift times
  are entered inconsistently
- Rebuild the calculation in Python/pandas so it can scale across multiple
  stores at once
