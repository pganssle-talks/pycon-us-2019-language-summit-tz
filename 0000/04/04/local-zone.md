# Local zone

Some operations treat naive timezones as local times:

```python
>>> from datetime import datetime, timezone
>>> dt = datetime(2020, 1, 1, 12)
>>> dt.astimezone(timezone.utc)
2020-01-01 17:00:00+00:00
```

<br/>

But others do not:

```python
>>> datetime(2020, 1, 1, 12) - datetime(2020, 1, 1, tzinfo=timezone.utc)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can't subtract offset-naive and offset-aware datetimes
```

<br/>

And the offset and `tzname` information are not accessible:

```python
>>> print(datetime(2020, 1, 1).utcoffset())
None
>>> print(datetime(2020, 1, 1).tzname())
None
>>> datetime(2020, 1, 1).strftime("%Z%z")
''
```

<br/>

--

# Local zone from `time`

`dateutil` provides `tzlocal` that is a wrapper around the `time` module:

```python >>> from dateutil import tz
>>> dt = datetime(2020, 1, 1, tzinfo=tz.tzlocal())
>>> dt.strftime("%Z%z")
'EST-0500'
```

<br/>
<br/>

Uses:

- `time.tzname` for abbreviations
- `time.timezone` and `time.altzone` for offsets
- `time.localtime` for the mapping

--

## Changing system time

1. Requires `tzset` to be called:

```python
>>> import time, os
>>> os.environ["TZ"] = "America/Los_Angeles"
>>> datetime(2020, 1, 1, tzinfo=tz.tzlocal()).strftime("%Z%z")
'EST-0500'
>>> time.tzset()
'PST-0800'
```

<br/>

2. Doesn't work at *all* on Windows (no `tzset`)

<br/>
<br/>

3. Changing system time will change offsets for *existing* times:

```python
>>> from datetime import timezone
>>> dt = datetime(2019, 3, 15, tzinfo=timezone.local)
>>> dt.strftime("%Z%z")
'EDT-0400'
>>> os.environ["TZ"] = "UTC"
>>> time.tzset()
>>> dt.strftime("%Z%z")
'UTC+0000'
```

--

# Possible Options

1. No support for changing the system time zone during a Python run

2. New implementation not using system calls

3. Allow datetime offsets to change

**Suggestion**: Needs one or more PEPs

