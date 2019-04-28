# Concrete Time Zones

- UTC / Fixed Offsets <span class="fragment" data-fragment-index="1">✔ Added in 3.2</span>
- Local time <span class="fragment" data-fragment-index="2">✘</span>
- IANA Time Zones <span class="fragment" data-fragment-index="2">✘</span>


Notes:

The large majority of people only want to use three types of time zones. They want a UTC type, and possibly some fixed offset from UTC. They want a representation of the system local time, and they want a representation of the IANA or Olson time zones.

The UTC case in particular is both very commonly useful and easy to implement, and was finally added in Python 3.2, but the other two options have presented more significant challenges.

--

# Historical Problem: Ambiguous times

```python
>>> from datetime import datetime, timedelta
>>> from datetime import tz
>>> NYC = tz.gettz("America/New_York")

>>> dt0 = datetime(2004, 10, 31, 5, 30, tzinfo=tz.UTC)
>>> print(dt0.astimezone(NYC))
2004-10-31 01:30:00-04:00

>>> print((dt0 + timedelta(hours=1)).astimezone(NYC))
2004-10-31 01:30:00-05:00
```

<br/>
<br/>

<ul>
    <li> <b>PEP 431</b>: "Time zone support improvements" – ✘ Withdrawn for lack of DST support</li>
<!--    <li class="fragment"><b>PEP 500</b>: "A protocol for delegating datetime methods to their tzinfo implementations" – ✘ Rejected</li> -->
    <li class="fragment" data-fragment-index="1"><b>PEP 495</b>: "Local time disambiguation" – ✔ Accepted for Python 3.6</li>
</ul>

<div class="fragment" data-fragment-index="1">
<br/>

Introduces the `fold` attribute:

```python
>>> print(datetime(2004, 10, 31, 1, 30, tzinfo=NYC))
2004-10-31 01:30:00-04:00

>>> print(datetime(2004, 10, 31, 1, 30, fold=1, tzinfo=NYC))
2004-10-31 01:30:00-05:00
```

</div>


Notes:

When people started to look into the possibility of implementing other concrete time zone types, there was one major hitch, which is that Python had no support for handling ambiguous datetimes. Ambiguous datetimes occur when a time zone's offset shifts backwards, for example during a transition from daylight saving time to standard time. In the example here, the local time 1:30 occurs twice in the Eastern time zone.

This is a problem, because if you'll recall, Python's time zone model is that `tzinfo` is a mapping between the naive portion of the datetime and its offset information, but in this case there are *two* wall times distinguished *only* by their offset information!

Having no solution to this led to the withdrawal of PEP 431. This problem was, however, fixed in Python 3.6 with the acceptance of PEP 495, which adds a `fold` attribute to `datetime` to distinguish the cases. This is why in this example, `dateutil` is able to correctly distinguish between the DST and STD times.

