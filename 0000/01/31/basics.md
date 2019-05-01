# Time zones are Rules

- `UTC+1` is an offset
- `Europe/London` is a time zone

<br/>
<br/>
<br/>

# Time zones in Python
`tzinfo` objects represent a mapping between naive times and offset information:


```python
>>> from datetime import datetime
>>> from dateutil import tz
>>> EASTERN = tz.gettz("America/New_York")
>>> print(datetime(2019, 3, 1, 12, tzinfo=EASTERN))
2019-03-01 12:00:00-05:00

>>> print(datetime(2019, 4, 1, 12, tzinfo=EASTERN))
2019-04-01 12:00:00-04:00
```

Notes:

A time zone is a mapping between the time on the wall and UTC.

This means that the offset is not sufficient.

In my opinion the datetime module gets this right, because time zones are
represented by `tzinfo` objects, which provide an interface mapping local
times to the offset information that applies at the time.

