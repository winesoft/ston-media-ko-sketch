﻿.. _rrd:

RRD
******************


.. toctree::
   :maxdepth: 2


========= ============ ============== =========== ========== =========  =========
시간단위    보관날짜(일)    # of lines     Interval    RRA~
========= ============ ============== =========== ========== =========  =========
 5 min      90         25920          0           LAST
30 min     120         5760           6           AVERAGE    MINIMUM    MAXIMUM
 1 hour    180         4320          12           AVERAGE    MINIMUM    MAXIMUM
 6 hour    365         1460          72           AVERAGE    MINIMUM    MAXIMUM
 1 day     730          730         288           AVERAGE    MINIMUM    MAXIMUM
========= ============ ============== =========== ========== =========  =========