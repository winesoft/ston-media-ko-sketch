.. _rrd:

RRD + Graph
******************

5분 이전의 통계는 "평균 Gauge" 로 RRD에 기록되며, Graph API를 통해 제공된다. 

================ ================== =================== ============== =============  ========= ===========
시간단위           보관날짜(일)          # of lines          Interval       RRA
================ ================== =================== ============== =============  ========= ===========
5 min            90                 25920               0              LAST
30 min           120                5760                6              AVERAGE        MINIMUM   MAXIMUM
1 hour           180                4320                12             AVERAGE        MINIMUM   MAXIMUM
6 hour           365                1460                72             AVERAGE        MINIMUM   MAXIMUM
1 day            730                730                 288            AVERAGE        MINIMUM   MAXIMUM
================ ================== =================== ============== =============  ========= ===========


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

API를 통해 저장된 RRD를 Graph로 추출한다. 
한 그래프에는 최소 1개 이상의 선이 그려지는데 주로 Main 라인은 녹색, Sub 라인은 파란색으로 그려진다. ::

   /graph_global?target=...&step=...&start=...&end=...
   /graph_vhost? target=...&step=...&start=...&end=...&vhost=...&protocol=...

공통적으로 지원되는 Key/Value는 아래와 같다.

- **target** - `전역자원`_ 과 `가상호스트`_ 에서 상세기술
- **step** - 시간단위로 ``5m`` , ``30m`` , ``1h`` , ``6h`` , ``1d`` 중 선택
- **start** - 시작시간
- **end** - 끝시간

시간표현은 `rrdfetch <https://oss.oetiker.ch/rrdtool/doc/rrdfetch.en.html>`_ 에서 제공하는 표현을 그대로 사용한다. 
이 중 시간(start, end)에서 사용할 수 있는 표현은 아래와 같다.

======================== ================================================================================
표현                       설명
======================== ================================================================================
Oct 12                   October 12 this year
-1month or -1m           current time of day, only a month before (may yield surprises, see NOTE3 above).
noon yesterday -3hours   yesterday morning; can also be specified as 9am-1day.
23:59 31.12.1999         1 minute to the year 2000.
12/31/99 11:59pm         1 minute to the year 2000 for imperialists.
12am 01/01/01            start of the new millennium
end-3weeks or e-3w       3 weeks before end time (may be used as start time specification).
start+6hours or s+6h     6 hours after start time (may be used as end time specification).
931225537                18:45 July 5th, 1999 (yes, seconds since 1970 are valid as well).
19970703 12:45           12:45 July 3th, 1997 (my favorite, and its even got an ISO number (8601)).
======================== ================================================================================

다양한 표현을 통해 서비스 모니터링에 최적화된 Graph를 지원한다.


.. note::

   Graph 포맷을 PNG 와 GIF 중 선택할 수 있으나, PNG 가 GIF 보다 20~30% 더 빠르고 40% 더 적은 크기를 가지므로 PNG 만을 지원한다.


전역자원
---------------------

전역자원 Graph는 시스템 상태 또는 STON 미디어 서버가 사용하는 시스템 자원들의 상태를 제공한다. 
API 호출규격은 다음와 같다. ::

   /graph_global?target=...&step=...&start=...&end=...

지원되는 ``target`` 은 다음과 같다.

========================= ================================================= ============================================== =========================== 
Target                    설명                                               Main line                                      Sub line                 
========================= ================================================= ============================================== =========================== 
cpu                       CPU 사용량                                          Kernel + User                                  Kernel                   
ston_media_server_cpu     STON 미디어 서버 CPU 사용량                            Kernel + User                                  Kernel                   
memory                    메모리 사용량                                         전체 사용량                                       STON 미디어 서버 사용량            
iowait                    IO Wait                                            IO Wait               
loadavg                   Load Average                                       Load Average          
ssockevent                서버소켓 이벤트 (클라이언트 -> STON)                     Accepted                                        Closed                   
ssockusage                서버소켓 사용량 (클라이언트 -> STON)                     전체                                              Established               
csockevent                클라이언트소켓 이벤트 (STON -> 원본서버)                   Connected                                       Closed                   
csockusage                클라이언트소켓 사용량 (STON -> 원본서버)                   전체                                            Established               
acldenied                 차단된 IP접근                                         차단된 클라이언트            
eq                        이벤트 큐                                            이벤트 큐                 
wf2w                      쓰기 대기 중인 파일개수                                 쓰기 대기중              
tcpsocket                 TCP 소켓 상태                                       .. figure:: img/graph_tcpsocket_detail.png 
========================= ================================================= ============================================== =========================== 



가상호스트
---------------------

가상호스트 Graph는 서비스 전체 또는 개별 가상호스트의 서비스 상태를 제공한다. 
API 호출규격은 다음와 같다. ::

   /graph_vhost?target=...&step=...&start=...&end=...&vhost=...&protocol=...

- ``vhost`` - 특정 가상호스트를 지정할 수 있으며, 생략된 경우 전체 가상호스트를 합친 결과를 제공한다.
- ``protocol`` - 프로토콜별 그래프를 제공한다. ``all (기본)`` , ``rtmp`` , ``http`` , ``hls`` , ``mpegdash`` 중 선택한다.

``target`` 마다 ``protocol`` 지원이 다르다.

============================ =================================================== ================================================================ ========================================= ============================
Target                       설명                                                 Protocol                                                         Main line                                 Sub line                  
============================ =================================================== ================================================================ ========================================= ============================
hitratio    
============================ =================================================== ================================================================ ========================================= =======================


client_res_hit               클라이언트 캐쉬 히트                                     ``all`` , ``rtmp`` , ``http`` , ``hls`` , ``mpegdash``           .. figure:: img/graph_filehit.png                         
client_session               클라이언트 세션                                           ``all`` , ``rtmp`` , ``http`` , ``hls`` , ``mpegdash``           전체 세션                              전송 중 세션                     
client_traffic               클라이언트 트래픽                                          ``all`` , ``rtmp`` , ``http`` , ``hls`` , ``mpegdash``           Inbound                            Outbound                  
client_res                   클라이언트 응답                                           ``all`` , ``rtmp`` , ``http`` , ``hls`` , ``mpegdash``           응답횟수                               요청횟수                        
client_res_complete          클라이언트 트랜잭션                                         ``http`` , ``hls`` , ``mpegdash``                                완료된 응답횟수                          요청횟수                         
client_res_time              클라이언트 응답시간                                         ``all`` , ``rtmp`` , ``http`` , ``hls`` , ``mpegdash``           응답시간                      
client_res_complete_time     클라이언트 완료시간                                         ``http`` , ``hls`` , ``mpegdash``                                트랜잭션 완료시간             
client_rtmp_res_detail       .. figure:: img/sms_rtmp_graph_detail.png                                                                                                                                 
                             RTMP 클라이언트 상세응답                                                                                                                                                           
client_http_res_detail       .. figure:: img/graph_rescode_detail.png                                                                                                                                  
                             HTTP 클라이언트 상세응답                                                                                                                                                           
client_hls_res_detail        .. figure:: img/graph_rescode_detail.png                                                                                                                                  
                             HLS 클라이언트 상세응답                                                                                                                                                            
client_mpegdash_res_detail   .. figure:: img/graph_rescode_detail.png                                                                                                                                  
                             MPEG-DASH 클라이언트 상세응답                                                                                                                                                      
origin_session               원본서버 세션                                            ``all`` , ``rtmp`` , ``http`` , ``hls`` , ``mpegdash``           전체 세션                              전송 중 세션                     
origin_traffic               원본서버 트래픽                                           ``all`` , ``rtmp`` , ``http`` , ``hls`` , ``mpegdash``           Inbound                            Outbound                  
origin_res                   원본서버 응답                                            ``all`` , ``rtmp`` , ``http`` , ``hls`` , ``mpegdash``           응답횟수                               요청횟수                        
origin_res_complete          원본서버 트랜잭션                                          ``http`` , ``hls`` , ``mpegdash``                                완료된 응답횟수                          요청횟수                         
origin_res_time              원본서버 응답시간                                          ``all`` , ``rtmp`` , ``http`` , ``hls`` , ``mpegdash``           응답시간                     
origin_res_complete_time     원본서버 완료시간                                          ``http`` , ``hls`` , ``mpegdash``                                트랜잭션 완료시간       
origin_rtmp_res_detail       .. figure:: img/sms_rtmp_graph_detail.png                                                                                                                                 
                             RTMP 원본서버 상세응답                                                                                                                                                            
origin_http_res_detail       .. figure:: img/graph_rescode_detail.png                                                                                                                                  
                             HTTP 원본서버 상세응답                                                                                                                                                            
origin_hls_res_detail        .. figure:: img/graph_rescode_detail.png                                                                                                                                  
                             HLS 원본서버 상세응답                                                                                                                                                             
origin_mpegdash_res_detail   .. figure:: img/graph_rescode_detail.png                                                                                                                                  
                             MPEG-DASH 원본서버 상세응답                                                                                                                                                       
filecount                    .. figure:: img/graph_filecount_detail.png                                                                                                                                
                             캐싱 콘텐츠 분포                                                                                                                                                                 
mem                          메모리에 적재된 콘텐츠 데이터량                                                                                                                                                         


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

해당 가상호스트에 속한 ``/myLiveStream`` 라는 채널이 있다면 vhost에 추가적으로 채널명을 붙인다. ::

    /graph/vhost/client_traffic_day.png?vhost=www.example.com/bar/myLiveStream

가상호스트는 존재하나 해당 채널이 없다면 404 Not Found로 응답한다.




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

예를 들어 2017년 7월 1일부터 가상호스트 ``www.example.com/bar`` 의 채널 ``/myLiveStream`` 의 주간 평균 클라이언트 트래픽을 보고 싶다면 아래와 같이 입력한다. ::

  /graph/vhost/client_traffic_week.png?vhost=www.example.com/bar&start=201707010000


/graph ? 
          target=
          vhost=
          foramt=png
          start= 
          end=