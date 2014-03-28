===========================================================================
 A proposal for improving the current implementation of datetime64 in NumPy
===========================================================================

:Author: Sankarshan Mudkavi
:Contact: smudkavi@uwaterloo.ca
:Date: 2014-03-18


Executive summary
=================

The current datetime64 module in NumPy handles timezone related operations that cause problems for a number of NumPy use cases. This proposal attempts to rectify this problem by introducing a few simple methods of handling such operations, which can later be enhanced further to include full time zone support if needed. 


Current Implementation
======================

The current implementation of datetime64 handles timezones as follows [1]_:

It assumes datetime64 objects are in UTC [2]_ internally, and timezone translation is done on I/O. On creating a datetime64 object as an ISO string, if timezone info is supplied, that information is respected. If no timezone info is supplied it is assumed to be in locale (system) time. On outputting, the loacale time zone is used if no time zone is specified.

However if a datetime64 is created using a `datetime.datetime` object without any time zone info, it is assumed to be UTC.

Motivation for changing the current implementation
==================================================
As may be immediately obvious from the description above, there is an inherent inconsistency in the behaviour of two different forms of valid input and output using datetime64's, that is I/O with ISO strings and with ```datetime.datetime``` objects. 

Furthermore, many applications in NumPy that deal with date and time data are likely completely separate from the time of system that the code is running on, and  therefore the long-standing tradition of using the locale time in the C standard library time functions.

This problem has also been unadressed for sometime and even a minor improvement enough to make NumPy's treatment of dates and times consistent would be better than the current implementation. This proposal aims to rectify this problem as well as providing a more robust datetime64 object as detailed below.

Possible Solutions
==================
After much discussion within the numpy community, it looks like there are four possible solutions to deal with this issue:

1) Naive datetime64: A naive datetime64 would be the simplest possible solution, where there is no timezone handling at all. Everything is assumed to be in the UTC internally, or alternatively all data is in the same time zone. I/O will have no offset and produce an object without tzinfo. If a datetime64 object is created with a timezone offset, we throw an error that states that numpy cannot currently handle timezone offsets.
 
Example behaviour::

	>>> np.datetime64('2005-02-25T03:00')
	np.datetime64('2005-02-25T03:00')

	>>> np.datetime64('2005-02-25T03:00Z')
	np.datetime64('2005-02-25T03:00')

	>>> np.datetime64('2005-02-25T03:00+0500')
	TZException: numpy does not currently handle tzinfo in datetime64 objects

2) UTC datetime64: This would be very similar to the above, expect everything would be in UTC internally and it would assume UTC if no tzinfo is provided. If it is provided then, the tzinfo is respected but stored as UTC internally. On output UTC is used, no offset computed.

Example behaviour::

	>>> np.datetime64('2005-02-25T03:00')
	np.datetime64('2005-02-25T03:00Z')

	>>> np.datetime64('2005-02-25T03:00-0500')
	np.datetime64('2005-02-25T08:00Z')

3) Optional timezone support: This approach would be completely analogous to the approach taken by the stdlib datetime. It would provide a hook for a tzinfo object and handle it in the same way the standard library does. I have not included examples here since further discussion would be required on how exactly this would operate.

4) Full timezone support: Every array would be required to carry timezone info. This would require a dedicated tzinfo package to be included and maintained in numpy, something which doesn't seem to be warranted right now.

Suggested improvement
=====================
The current suggested improvement would be the simplest possible fix, which would be the naive datetime solution (solution 1). In the discussion in the threads about this problem, this seems to be the least objectionable approach, as well as possibly the most implementable. Since we need to handle the I/O problems currently faced, this would be the best way to ensure those problems are fixed and improvements on this can be made later. This also seems to be compatible with pandas and hence would be the most acceptable to the numeric community at large.



.. [1] http://article.gmane.org/gmane.comp.python.numeric.general/53805
.. [2] http://en.wikipedia.org/wiki/Unix_time


.. Local Variables:
.. mode: rst
.. coding: utf-8
.. fill-column: 72
.. End:
