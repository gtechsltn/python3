# python3

Python 3.11.5 | packaged by Anaconda, Inc. | (main, Sep 11 2023, 13:26:23) [MSC v.1916 64 bit (AMD64)] on win32

Type "help", "copyright", "credits" or "license" for more information.

## Python version
```
C:\Users\ADMIN> python --version
Python 3.11.5
```

## Python work with DateTime
* **Weekdays**: "next Monday", "last Friday"
* **Holidays**: "last Christmas"
* **Relative days**: "3 days ago", "in 2 weeks", "yesterday", "tomorrow"

```
from datetime import datetime, timedelta
import dateparser

def next_weekday(base_date, weekday):
    """Return the next weekday (0=Monday, 6=Sunday) after base_date."""
    days_ahead = (weekday - base_date.weekday() + 7) % 7
    if days_ahead == 0:
        days_ahead = 7
    return base_date + timedelta(days=days_ahead)

def last_weekday(base_date, weekday):
    """Return the last weekday (0=Monday, 6=Sunday) before base_date."""
    days_behind = (base_date.weekday() - weekday + 7) % 7
    if days_behind == 0:
        days_behind = 7
    return base_date - timedelta(days=days_behind)

def parse_relative_dates_general(relative_dates, anchor_date=None):
    """
    Parse relative dates from strings including weekdays, holidays, relative days.
    """
    if anchor_date is None:
        anchor_date = datetime.now()
    results = {}

    weekday_map = {
        "monday": 0,
        "tuesday": 1,
        "wednesday": 2,
        "thursday": 3,
        "friday": 4,
        "saturday": 5,
        "sunday": 6
    }

    for rel in relative_dates:
        abs_date = None
        rel_lower = rel.lower()

        # --- Handle specific weekdays ---
        for wd_name, wd_index in weekday_map.items():
            if f"next {wd_name}" in rel_lower:
                abs_date = next_weekday(anchor_date, wd_index)
                break
            elif f"last {wd_name}" in rel_lower:
                abs_date = last_weekday(anchor_date, wd_index)
                break

        # --- Handle holidays manually ---
        if abs_date is None and "christmas" in rel_lower:
            if "last" in rel_lower:
                abs_date = datetime(anchor_date.year - 1, 12, 25,
                                    anchor_date.hour, anchor_date.minute)
            elif "next" in rel_lower:
                abs_date = datetime(anchor_date.year, 12, 25,
                                    anchor_date.hour, anchor_date.minute)
                if abs_date <= anchor_date:
                    abs_date = datetime(anchor_date.year + 1, 12, 25,
                                        anchor_date.hour, anchor_date.minute)

        # --- Try dateparser fallback for relative days like "3 days ago" ---
        if abs_date is None:
            abs_date = dateparser.parse(
                rel,
                settings={
                    "RELATIVE_BASE": anchor_date,
                    "STRICT_PARSING": True
                }
            )

        results[rel] = abs_date

    return results

# --- Example usage ---
if __name__ == "__main__":
    anchor = datetime(2026, 1, 21, 12, 0, 0)
    relative_dates = [
        "3 days ago",
        "next Monday",
        "in 2 weeks",
        "yesterday",
        "tomorrow",
        "last Christmas",
        "next Christmas",
        "last Friday"
    ]

    results = parse_relative_dates_general(relative_dates, anchor)
    for rel, abs_date in results.items():
        print(f"{rel:20} -> {abs_date}")
```

## Results

```
C:\Users\ADMIN> python main.py
3 days ago           -> 2026-01-18 12:00:00
next Monday          -> 2026-01-26 12:00:00
in 2 weeks           -> 2026-02-04 12:00:00
yesterday            -> 2026-01-20 12:00:00
tomorrow             -> 2026-01-22 12:00:00
last Christmas       -> 2025-12-25 12:00:00
next Christmas       -> 2026-12-25 12:00:00
last Friday          -> 2026-01-16 12:00:00
```
