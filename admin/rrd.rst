.. _rrd:

RRD
******************


.. toctree::
   :maxdepth: 2


.. note::

   아래 기준으로 생성하고 RRD 파일 크기가 너무 클 경우 라인 수를 적절히 줄일 필요가 있음

========= ================ ================== =============== ========== =========  =========
시간단위    보관날짜(일)        # of lines         Interval        RRA-                                         
========= ================ ================== =============== ========== =========  =========
5 min     90               25920              0               LAST
30 min    120              5760               6               AVERAGE    MINIMUM    MAXIMUM
1 hour    180              4320               12              AVERAGE    MINIMUM    MAXIMUM
6 hour    365              1460               72              AVERAGE    MINIMUM    MAXIMUM
1 day     730              730                288             AVERAGE    MINIMUM    MAXIMUM
========= ================ ================== =============== ========== =========  =========


::

   rrdtool create x.rrd
   — start 현재 시간
   — step 300
   DS:variable_name:DST:heartbeat:min:max
   RRA:CF:xff:step:rows


- DS 파트 - 데이터 소스를 지정하는 부분
- DST = GAUGE // 현재 STON이 모든 수치를 GAUGE 형태로 RRD Update
- heartbeat = 600 // 현재 5분 (300초) 단위로 업데이트가 이루어지는데, 업데이트가 불가한 경우 (수치를 못 가져왔다는 등의 Wait)에 얼마나 기다릴지를 나타냄.
- min, max = 데이터 업데이트 시의 최소 최대 수치 -> 지정해주는 것이 좋겠음. U는 Unknown. CPU를 예로 든다면, 최소는 0%, 최대는 100% (또는 CPU 갯수에 따라 나눠서 나오는 수치는 100%*CPU 이지만, 적당히 큰수 6400% 등으로 크게 잡아두는 것도 괜찮음)



RRA 파트 데이터의 구조 지정하는 부분

- CF = LAST(마지막 값, 가공없음, 5분 최초 단위는 LAST를 보통 사용함), AVERAGE, MIN, MAX 등
- step = 데이터가 업데이트되는 Inverval - 여기서는 300 (5분)
- rows = 데이터 베이스의 저장 길이 (몇 라인까지 저장할 것인가)

예제 ::

   rrdtool create x.rrd
   — start 현재시간
   — step 300
   DS:var1:GAUGE:600:0:6400
   DS:var2:GAUGE:600:0:100
   DS:var3:GAUGE:600:0:10000000
   DS:var4:GAUGE:600:0:U
   RRA:LAST:0.5:1:25920
   RRA:AVERAGE:0.5:6:5760
   RRA:AVERAGE:0.5:12:4320
   RRA:AVERAGE:0.5:72:1460
   RRA:AVERAGE:0.5:288:730
   RRA:MIN:0.5:6:5760
   RRA:MIN:0.5:12:4320
   RRA:MIN:0.5:72:1460
   RRA:MIN:0.5:288:730
   RRA:MAX:0.5:6:5760
   RRA:MAX:0.5:12:4320
   RRA:MAX:0.5:72:1460
   RRA:MAX:0.5:288:730


::

   rrdtool update x.rrd {int(현재시간/300)*300}:데이터값 {int(현재시간/300)*300}:데이터값 {int(현재시간/300)*300}:데이터값 



- ``{int(현재시간/300)*300}`` 를 통해서 5분 단위로 업데이터 단위 시간이 딱 잘라지도록 함. (현재시간 -> epoch time)

- N으로 시간값이 들어가게 되면, 업데이트 시마다 미묘하게 몇초 간격으로 차이가 날 수 있어서 그래프나 데이터 fetch 시에 좋지 않음.

- 데이터 확인 방법은 rrdtool fetch x.rrd AVERAGE —start 시간 —end 시간

- RRD가 시간이 딱 끊어져서 들어가는 제약이 있는 database이므로 시작 시간과 끝 시간도 300 단위에 맞게 끊어주는것이 좋음.

