.. _rrd:

장기통계
******************

5분 이전의 통계는 "평균 Gauge" 로 RRD와 Graph API를 통해 제공된다.

===================== ================ ================== =============== ========== =========  ========= ===========
Graph 이름             시간단위           보관날짜(일)          # of lines      Interval   RRA-                                         
===================== ================ ================== =============== ========== =========  ========= ===========
*_day.png             5 min            90                 25920           0          LAST
*_week.png            30 min           120                5760            6          AVERAGE    MINIMUM   MAXIMUM
*_month.png           1 hour           180                4320            12         AVERAGE    MINIMUM   MAXIMUM
*_year.png            6 hour           365                1460            72         AVERAGE    MINIMUM   MAXIMUM
*_year2.png           1 day            730                730             288        AVERAGE    MINIMUM   MAXIMUM
===================== ================ ================== =============== ========== =========  ========= ===========


.. toctree::
   :maxdepth: 2


RRD
====================================

가상호스트/채널마다 RRD를 생성하고 가상호스트가 삭제되면 파기한다.

.. note:: 

   개발단계에서 RRD 파일 크기가 너무 커지는 경우를 감안하여 최대 라인수는 협의가 필요함

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




Graph
====================================

STON Edge 서버와 다른 점에 대해서만 명시한다.

-  dash 그래프 제거
-  *_year2.png 그래프 추가
-  채널 그래프 제공
-  조회 기간 추가



채널 그래프
---------------------

채널은 가상호스트가 제공하는 API를 대부분 동일하게 제공한다. 

.. note::

   제공하지 않는 API 목록 ::

      /graph/vhost/filecount_*.png
      /graph/vhost/mem_*.png
      /graph/vhost/wf2d_*.png



예를 들어 가상호스트 ``www.example.com/bar`` 에 대한 (5분 단위) 클라이언트 트래픽 조회 API 아래와 같다. ::

    /graph/vhost/client_traffic_day.png?vhost=www.example.com/bar


해당 가상호스트에 ``/myLiveStream`` 라는 채널이 있다면 다음과 같이 호출한다. ::

    /graph/vhost/client_traffic_day.png?vhost=www.example.com/bar/myLiveStream

채널이 존재하지 않는 경우 404 Not Found로 응답한다.




조회 범위
---------------------

절대시간을 파라미터로 입력하여 그래프를 생성한다. ::

   /graph/vhost/client_traffic_day.png?vhost={가상호스트명}&start={yyyyMMddHHmm}&end={yyyyMMddHHmm}

분(mm) 조건은 5분 단위로 내림된다. 
예를 들어 HHmm 조건을 1722 라고 입력했다면 이 수치는 5분 단위를 맞추기 위해 11720 으로 내림된다.

``start`` 파라미터가 생략되면 5분 전 통계를 기준으로 아래와 같이 그래프를 제공한다.

================= =================
Graph 이름         기본 기간
================= =================
*_day.png         2일 (48시간)
*_week.png        2주 (14일)
*_month.png       7주
*_year.png        9개월
*_year2.png       18개월
================= =================

``end`` 파라미터가 생략되면 start를 기준으로 위 표에 명시된 기간만큼 그래프가 그려진다. 

예를 들어 2017년 7월 1일의 가상호스트 ``www.example.com/bar`` 의 채널 ``/myLiveStream`` 의 주간 평균 클라이언트 트래픽을 보고 싶다면 아래와 같이 입력한다. ::

  /graph/vhost/client_traffic_week.png?vhost=www.example.com/bar&start=201707010000


