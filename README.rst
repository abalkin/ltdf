=======
Rationale
=======

In the most world locations there have been and will be times when
local clocks are moved back.  In those times intervals are introduced
in which local clocks show the same time twice in the same day.   In
these situations, the information displayed on a local clock (or
stored in a Python datetime instance) is insufficient to identify a
particular instance in time.   The proposed solution is to add a
boolean flag to the datetime instances that will distinguish between
the two ambiguous times.

=======
Proposal
=======

The "first" flag
------------------

We propose adding a boolean member called ``first`` to the instances of
``datetime.time`` and ``datetime.datetime`` classes.   This member will have the
value True for all instances except those that represent the second
(chronologically) moment in time in an ambiguous case.

Affected APIs
------------------

Attributes
...............

Instances of datetime.time and datetime.datetime will get a new
boolean attribute called "first."

Constructors
....................

The ``__new__`` methods of the datetime.time and datetime.datetime classes
will get a new keyword-only argument ``first=True`` that will control the
value of the "first" attribute in the returned instance.

Methods
.............

The ``replace()`` methods methods of the datetime.time and
``datetime.datetime`` classes will get a new keyword-only argument
first=True that will control the value of the ``first`` attribute in the
returned instance.

Affected behaviors
-------------------------

Implementations of tzinfo
.......................................

Subclasses of ``datetime.tzinfo`` will read the values of ``first`` in
``utcoffset()`` and ``dst()`` methods and set it appropriately in the
instances
returned by the ``fromutc()`` method.  No change to the signatures of
these methods is proposed.

Pickle size
--------------
Pickle sizes for the datetime.datetime and datetime.time objects will
not change.  The ``first`` flag will be encoded in the first bit of the 4th byte of the datetime.datetime
pickle payload or the 1st byte of the datetime.time. In the current
implementation [1] these bytes are used to store hour value (0-23) and
the first bit is always 0.  Note that ``first=True`` will be encoded as 0
in the first bit and ``first=False`` as 1.  (This change only affects
pickle format.  In C implementation, the "first" member will get a
full byte to store the actual boolean value.)

Temporal arithmetics
----------------------------
The value of "first" will be ignored in all operations except
utcoffset() and dst() methods of tzinfo implementations and __eq__ and
``__hash__`` methods of the datetime.datetime and datetime.time  classes.
The only methods that will be able to  produce nonzero values of
"first" are ``__new__`` and replace() methods of the ``datetime.datetime`` and
``datetime.time``  classes and ``fromutc()`` method of some tzinfo
implementations.

Backward and forward compatibility
-----------------------------------------------

This proposal will have no effect on the programs that do not set the
``first`` flag explicitly or use tzinfo implementations that do.
Pickles produced by older programs will remain fully forward
compatible.  Only datetime/time instances with ``first=False`` pickled in
the new versions will become unreadable by the older Python versions.
Pickles of instances with first=False will remain unchanged.


==================
Questions and Answers
==================

1. Why not call the new flag "isdst"?

Alice:  Bob - let's have a stargazing party at 01:30 AM tomorrow!
Bob:  Should I presume initially that summer time (for example,
Daylight Saving Time) is or is not (respectively) in effect for the
specified time?
Allice: Huh?

Bob: Alice - let's have a stargazing party at 01:30 AM tomorrow!
Alice: You know, Bob, 01:30 AM will happen twice tomorrow. Which one
do you have in mind?
Bob:  I did not think about it, but let's pick the first.


[1]: https://hg.python.org/cpython/file/d3b20bff9c5d/Include/datetime.h#l17
