# Three way to format datetime object in Python

1. Dates have a default representation, but you may want to print them in a specific format. In that case, you can get a custom string representation using the `strftime()` method.

```py
print today.strftime('We are the %d, %b %Y')
>>> 'We are the 22, Nov 2008'
```

All the letter after a "%" represent a format for something :

```py
%d is the day number
%m is the month number
%b is the month abbreviation
%y is the year last two digits
%Y is the all year
```

> Have a look at the [official documentation](http://docs.python.org/2/library/datetime.html#strftime-and-strptime-behavior), or [McCutchen's quick reference](http://strftime.org/) you can't know them all.

1. Since PEP3101, every object can have its own format used automatically by the method format of any string. In the case of the datetime, the format is the same used in strftime. So you can do the same as above like this:

```py
print "We are the {:%d, %b %Y}".format(today)
>>> 'We are the 22, Nov 2008'
```

The advantage of this form is that you can also convert other objects at the same time.

3. With the introduction of Formatted string literals (since Python 3.6, 2016-12-23) this can be written as

```py
import datetime
f"{datetime.datetime.now():%Y-%m-%d}"
>>> '2017-06-15'
```