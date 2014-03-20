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
As may be immediately obvious from the description above, there is an inherent inconsistency in the behaviour of two different forms of valid input and output using datetime64's, that is I/O with ISO strings and with `datetime.datetime` objects. 

Furthermore, many applications in NumPy that deal with date and time data are likely completely separate from the time of system that the code is running on, and  therefore the long-standing tradition of using the locale time in the C standard library time functions.

This problem has also been unadressed for sometime and even a minor improvement enough to make NumPy's treatment of dates and times consistent would be better than the current implementation. This proposal aims to rectify this problem as well as providing a more robust datetime64 object as detailed below.

Suggested improvement
=====================
The suggested improvement involves implementing a function to accept an iso string or a `datetime.datetime` object as UTC with an optional flag to provide tzinfo, so as not to confuse applications that expect a naive datetime without any time zone info. UTC is used for output, with no offset. The internal storage would be in UTC.

This fix is simplistic and avoids having to include a comprehensive tzinfo package which would be required if complete timezone support were to be implemented. Furthermore this would likely involve a dip in perfomance since it would require heavy interaction with the python layer for each operation. This could be incorporated in the future, since the proposed fix could easily be extended without jeopardizing the option of providing complete support at a later time.


.. [1] http://article.gmane.org/gmane.comp.python.numeric.general/53805
.. [2] http://en.wikipedia.org/wiki/Unix_time


.. Local Variables:
.. mode: rst
.. coding: utf-8
.. fill-column: 72
.. End:
