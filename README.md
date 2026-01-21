# Python vs C# vs VB.NET

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

## Results (Python)

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

# Version CSharp (C#)
```
using System;
using System.Collections.Generic;
using System.Globalization;

class RelativeDateParser
{
    // --- Helper: next weekday ---
    public static DateTime NextWeekday(DateTime baseDate, DayOfWeek day)
    {
        int daysAhead = ((int)day - (int)baseDate.DayOfWeek + 7) % 7;
        if (daysAhead == 0) daysAhead = 7;
        return baseDate.AddDays(daysAhead);
    }

    // --- Helper: last weekday ---
    public static DateTime LastWeekday(DateTime baseDate, DayOfWeek day)
    {
        int daysBehind = ((int)baseDate.DayOfWeek - (int)day + 7) % 7;
        if (daysBehind == 0) daysBehind = 7;
        return baseDate.AddDays(-daysBehind);
    }

    // --- Main parser ---
    public static Dictionary<string, DateTime?> ParseRelativeDates(List<string> relativeDates, DateTime? anchor = null)
    {
        DateTime baseDate = anchor ?? DateTime.Now;
        var results = new Dictionary<string, DateTime?>();

        foreach (var rel in relativeDates)
        {
            DateTime? absDate = null;
            string lower = rel.ToLowerInvariant();

            // --- Weekdays ---
            foreach (DayOfWeek wd in Enum.GetValues(typeof(DayOfWeek)))
            {
                string wdName = wd.ToString().ToLower();
                if (lower.Contains("next " + wdName))
                {
                    absDate = NextWeekday(baseDate, wd);
                    break;
                }
                else if (lower.Contains("last " + wdName))
                {
                    absDate = LastWeekday(baseDate, wd);
                    break;
                }
            }

            // --- Holidays: Christmas ---
            if (absDate == null && lower.Contains("christmas"))
            {
                if (lower.Contains("last"))
                {
                    absDate = new DateTime(baseDate.Year - 1, 12, 25, baseDate.Hour, baseDate.Minute, baseDate.Second);
                }
                else if (lower.Contains("next"))
                {
                    absDate = new DateTime(baseDate.Year, 12, 25, baseDate.Hour, baseDate.Minute, baseDate.Second);
                    if (absDate <= baseDate)
                        absDate = new DateTime(baseDate.Year + 1, 12, 25, baseDate.Hour, baseDate.Minute, baseDate.Second);
                }
                else
                {
                    absDate = new DateTime(baseDate.Year, 12, 25, baseDate.Hour, baseDate.Minute, baseDate.Second);
                }
            }

            // --- Relative days: "3 days ago", "in 2 weeks", "yesterday", "tomorrow" ---
            if (absDate == null)
            {
                absDate = ParseSimpleRelativeDays(lower, baseDate);
            }

            results[rel] = absDate;
        }

        return results;
    }

    // --- Simple relative day parser ---
    private static DateTime? ParseSimpleRelativeDays(string input, DateTime baseDate)
    {
        try
        {
            if (input.Contains("yesterday"))
                return baseDate.AddDays(-1);
            if (input.Contains("tomorrow"))
                return baseDate.AddDays(1);

            // "X days ago"
            if (input.Contains("days ago"))
            {
                string[] parts = input.Split(' ');
                if (int.TryParse(parts[0], out int n))
                    return baseDate.AddDays(-n);
            }

            // "in X days"
            if (input.StartsWith("in ") && input.Contains("days"))
            {
                string[] parts = input.Split(' ');
                if (int.TryParse(parts[1], out int n))
                    return baseDate.AddDays(n);
            }

            // "in X weeks"
            if (input.StartsWith("in ") && input.Contains("weeks"))
            {
                string[] parts = input.Split(' ');
                if (int.TryParse(parts[1], out int n))
                    return baseDate.AddDays(7 * n);
            }

            return null;
        }
        catch
        {
            return null;
        }
    }

    // --- Example usage ---
    static void Main()
    {
        DateTime anchor = new DateTime(2026, 1, 21, 12, 0, 0);

        var relativeDates = new List<string>
        {
            "3 days ago",
            "next Monday",
            "in 2 weeks",
            "yesterday",
            "tomorrow",
            "last Christmas",
            "next Christmas",
            "last Friday"
        };

        var results = ParseRelativeDates(relativeDates, anchor);

        foreach (var kv in results)
        {
            Console.WriteLine($"{kv.Key,-20} -> {kv.Value}");
        }
    }
}
```

## Results (C#)
```
3 days ago           -> 1/18/2026 12:00:00 PM
next Monday          -> 1/26/2026 12:00:00 PM
in 2 weeks           -> 2/4/2026 12:00:00 PM
yesterday            -> 1/20/2026 12:00:00 PM
tomorrow             -> 1/22/2026 12:00:00 PM
last Christmas       -> 12/25/2025 12:00:00 PM
next Christmas       -> 12/25/2026 12:00:00 PM
last Friday          -> 1/16/2026 12:00:00 PM
```

# Version VB.NET
```
Imports System
Imports System.Collections.Generic
Imports System.Globalization

Module RelativeDateParser

    ' --- Helper: next weekday ---
    Function NextWeekday(baseDate As DateTime, day As DayOfWeek) As DateTime
        Dim daysAhead As Integer = ((CInt(day) - CInt(baseDate.DayOfWeek) + 7) Mod 7)
        If daysAhead = 0 Then daysAhead = 7
        Return baseDate.AddDays(daysAhead)
    End Function

    ' --- Helper: last weekday ---
    Function LastWeekday(baseDate As DateTime, day As DayOfWeek) As DateTime
        Dim daysBehind As Integer = ((CInt(baseDate.DayOfWeek) - CInt(day) + 7) Mod 7)
        If daysBehind = 0 Then daysBehind = 7
        Return baseDate.AddDays(-daysBehind)
    End Function

    ' --- Simple relative days parser ---
    Function ParseSimpleRelativeDays(input As String, baseDate As DateTime) As DateTime?
        Try
            Dim lower = input.ToLowerInvariant()
            If lower.Contains("yesterday") Then Return baseDate.AddDays(-1)
            If lower.Contains("tomorrow") Then Return baseDate.AddDays(1)

            If lower.Contains("days ago") Then
                Dim parts = lower.Split(" "c)
                Dim n As Integer
                If Integer.TryParse(parts(0), n) Then Return baseDate.AddDays(-n)
            End If

            If lower.StartsWith("in ") AndAlso lower.Contains("days") Then
                Dim parts = lower.Split(" "c)
                Dim n As Integer
                If Integer.TryParse(parts(1), n) Then Return baseDate.AddDays(n)
            End If

            If lower.StartsWith("in ") AndAlso lower.Contains("weeks") Then
                Dim parts = lower.Split(" "c)
                Dim n As Integer
                If Integer.TryParse(parts(1), n) Then Return baseDate.AddDays(7 * n)
            End If

            Return Nothing
        Catch
            Return Nothing
        End Try
    End Function

    ' --- Main parser ---
    Function ParseRelativeDatesGeneral(relativeDates As List(Of String), Optional anchorDate As DateTime? = Nothing) As Dictionary(Of String, DateTime?)
        Dim baseDate As DateTime = If(anchorDate, DateTime.Now)
        Dim results As New Dictionary(Of String, DateTime?)()

        Dim weekdayMap As New Dictionary(Of String, DayOfWeek) From {
            {"monday", DayOfWeek.Monday},
            {"tuesday", DayOfWeek.Tuesday},
            {"wednesday", DayOfWeek.Wednesday},
            {"thursday", DayOfWeek.Thursday},
            {"friday", DayOfWeek.Friday},
            {"saturday", DayOfWeek.Saturday},
            {"sunday", DayOfWeek.Sunday}
        }

        For Each rel In relativeDates
            Dim absDate As DateTime? = Nothing
            Dim lower = rel.ToLowerInvariant()

            ' --- Weekdays ---
            For Each kvp In weekdayMap
                If lower.Contains("next " & kvp.Key) Then
                    absDate = NextWeekday(baseDate, kvp.Value)
                    Exit For
                ElseIf lower.Contains("last " & kvp.Key) Then
                    absDate = LastWeekday(baseDate, kvp.Value)
                    Exit For
                End If
            Next

            ' --- Holidays: Christmas ---
            If absDate Is Nothing AndAlso lower.Contains("christmas") Then
                If lower.Contains("last") Then
                    absDate = New DateTime(baseDate.Year - 1, 12, 25, baseDate.Hour, baseDate.Minute, baseDate.Second)
                ElseIf lower.Contains("next") Then
                    absDate = New DateTime(baseDate.Year, 12, 25, baseDate.Hour, baseDate.Minute, baseDate.Second)
                    If absDate <= baseDate Then
                        absDate = New DateTime(baseDate.Year + 1, 12, 25, baseDate.Hour, baseDate.Minute, baseDate.Second)
                    End If
                Else
                    absDate = New DateTime(baseDate.Year, 12, 25, baseDate.Hour, baseDate.Minute, baseDate.Second)
                End If
            End If

            ' --- Relative days ---
            If absDate Is Nothing Then
                absDate = ParseSimpleRelativeDays(lower, baseDate)
            End If

            results(rel) = absDate
        Next

        Return results
    End Function

    ' --- Example usage ---
    Sub Main()
        Dim anchor As New DateTime(2026, 1, 21, 12, 0, 0)
        Dim relativeDates As New List(Of String) From {
            "3 days ago",
            "next Monday",
            "in 2 weeks",
            "yesterday",
            "tomorrow",
            "last Christmas",
            "next Christmas",
            "last Friday"
        }

        Dim results = ParseRelativeDatesGeneral(relativeDates, anchor)

        For Each kvp In results
            Console.WriteLine($"{kvp.Key,-20} -> {kvp.Value}")
        Next
    End Sub

End Module
```

## Results (VB.NET)
```
3 days ago           -> 1/18/2026 12:00:00 PM
next Monday          -> 1/26/2026 12:00:00 PM
in 2 weeks           -> 2/4/2026 12:00:00 PM
yesterday            -> 1/20/2026 12:00:00 PM
tomorrow             -> 1/22/2026 12:00:00 PM
last Christmas       -> 12/25/2025 12:00:00 PM
next Christmas       -> 12/25/2026 12:00:00 PM
last Friday          -> 1/16/2026 12:00:00 PM
```
