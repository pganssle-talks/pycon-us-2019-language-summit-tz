# IANA Time Zones
- Provides historical time zone information
- Standard open source (public domain) source for time zone information
- Shipped with many operating systems
- Source for `dateutil` and `pytz`'s data.
- 2-21 releases per year (average 9)
<br/>
<img src="images/all_zones.png" alt="Map of IANA time zones"/>

Notes:

So now that we have solved the issue of ambiguous datetimes, it is now possible to make a *correct* implementation of concrete time zones, and the most obvious candidate here is to add support for the zoneinfo files provided by the IANA time zone database, also called the Olson database or tzdb.

This is an open source project that maintains historical time zone information. Transition information is compiled to a binary `tzfile`, and this is shipped with most operating systems.

--

# Data sources

## System

**Pros:**

- Use normal operating system update mechanisms
- Update cadence is independent of CPython's updates
- Uses the same data as other applications

**Cons:**

- Not guaranteed to exist (not officially supported on Windows *yet*)
- Less control for programmers

<br/>
<br/>

<div class="fragment">

## PyPI package

**Pros:**

- Guaranteed on all platforms
- Update cadence is mostly independent of CPython

**Cons:**

- Requires either a dependency or a mechanism for regular updates
- Python-specific time zone data not managed by the system

</div>

--

# `tzfile(5)` and DST offset

**From `man tzfile(5)`**:

> * tzh_typecnt ttinfo entries, each defined as follows:
>   ```
>   struct ttinfo {
>        int32_t        tt_gmtoff;
>        unsigned char  tt_isdst;
>        unsigned char  tt_abbrind;
>   };
>   ```

<br/>

... but `tzinfo.dst` returns a `datetime.timedelta` with the *amount* of daylight saving time offset, not a boolean.

<br/>
<div class="fragment">

## Possible solutions

<table>
<tr>
<td>

- **Heuristics**
    - Works with all current time zones
    - `pytz` and `dateutil` use this strategy
    - Not guaranteed to work for future zones

</td>
<td>

- **Custom compiler for tzdata**
    - Guaranteed to fix this problem
    - May be more memory-efficient
    - Rules out using system time zone database
    - Requires ongoing maintenance

</td>
</tr>
</table>
</div>

Notes:

Unfortunately, one problem with using the compiled binary outputs provided by the system is that for each transition, they only contain the UTC offset, the abbreviation and *whether or not an offset is a DST offset*, but Python is expecting `tzinfo` to know the *amount* of DST offset. This information is in the tz database source files, but is lost during compilation.

`pytz` and `dateutil` use heuristics to detect this successfully in all current cases, but it's possible that those heuristics will fail with future time zone rule changes.

One option that can guarantee that the problem is fixed is to write a custom compiler to translate the tzdata into a format that preserves this information. This would unfortunately mean that we would be unable to use the system time zone information.

Another problem with the custom compiler is that this would require ongoing maintenance to transpile everything into our own time zone database. Java does this, and they frequently have problems where something breaks their compiler, and users on old versions need to use custom backwards-compatibility `tzdata` tarballs even after the compiler fix comes out.

--

# Proposal

<br/>

1. **Support `tzfile`**: On balance, heuristics should be "good enough"
2. **Use system data with fallback to PyPI package**
3. **Enable configuration**
    - `PYTHONTZPATH` analogous to `PYTHONPATH`
    - `datetime.tzpath` analogous to `sys.tzpath`
    - Option to explicitly specify a data source

<br/>
<br/>

```python
class zoneinfo:
    def __init__(self, tzkey, *, data_source=None):
        ...

    @classmethod
    def fromfile(self, tzfile, tzkey):
        ...
```

<br/>
<br/>

**Should use a PEP**: Many design decisions that need to be documented.

