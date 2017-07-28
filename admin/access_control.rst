.. _access-control:

접근제어
******************

이 장에서는 원치않는 VOD/LIVE 접근을 차단하는 방법에 대해 설명한다.
접근차단은 보통 ACL(Access Control List)에 차단목록(Black-list)을 작성하지만 설정편의상 허용목록(White-list)을 작성하기도 한다.

접근제어는 접속단계에서 수행하는 서버 접근제어와 가상호스트마다 설정하는 가상호스트 접근제어로 나뉜다.
수준별로 시점과 판단기준이 다르므로 효과적인 차단시점을 결정해야 한다.
접근제어의 동작은 모두 로그에 기록된다.

.. note::

   통상적으로 클라이언트는 VOD나 LIVE를 시청하는 사용자를 의미하지만, 이 장에서는 STON 미디어 서버에 접근하는 모든 IP연결(클라이언트 또는 인코더 등)을 의미한다.

.. toctree::
   :maxdepth: 2


.. _access-control-serviceaccess:

서버 접근제어
====================================

클라이언트가 서버에 접속하는 순간 IP정보를 통해 차단여부를 결정한다.
접속단계에서 처리되기 때문에 가장 확실하며 빠르다.
전역설정(server.xml)에 설정하며 가장 높은 우선순위를 가진다. ::

   # server.xml - <Server><Host>

   <ServiceAccess Default="Allow">
      <Deny>192.168.7.9-255</Deny>
      <Deny>192.168.8.10/255.255.255.0</Deny>
   </ServiceAccess>

-  ``<ServiceAccess>``
   IP기반의 ACL을 설정한다.
   IP, IP Range, Bitmask, Subnet 이상 네 가지 형식을 지원한다.
   순서를 인식하며 상위에 설정된 표현이 우선한다.
   ``Default (기본: Allow)`` 속성은 일치하는 조건이 없을 때 처리방법이다.
   이 속성을 ``Deny`` 로 설정하면 하위에 ``<Allow>`` 로 허가할 조건들을 명시해주어야 한다.

차단된 IP는 :ref:`admin-log-deny` 에 기록된다.


.. _access-control-geoip:

GeoIP
====================================

GeoIP를 사용하여 국가별로 접근을 차단할 수 있다.
`GeoIP Databases <http://dev.maxmind.com/geoip/legacy/downloadable/>`_ 중
Binary Databases를 `GEOIP_MEMORY_CACHE and GEOIP_CHECK_CACHE <http://dev.maxmind.com/geoip/legacy/benchmarks/>`_ 로
링크하여 실시간으로 변경내용을 반영한다. ::

   # server.xml - <Server><Host>

   <ServiceAccess GeoIP="/var/ston/geoip/">
      <Deny>AP</Deny>
      <Deny>GIN</Deny>
   </ServiceAccess>

``<ServiceAccess>`` 의 ``GeoIP`` 속성에 GeoIP Databases 경로를 설정한다.
국가코드는 `ISO 3166-1 alpha-2 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2>`_ 와
`ISO 3166-1 alpha-3 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3>`_ 를 지원한다.

.. note::

   GeoIP는 파일명이 예약되어 있으므로 반드시 저장된 로컬경로를 입력하도록 설정한다.
   또한 자동으로 변경이 반영되기 때문에 별도로 설정을 Reload하지 않아도 된다.


GeoIP가 설정되어 있다면 해당 디렉토리에 저장된 파일목록을 조회한다.
설정되어 있지 않다면 404 NOT FOUND로 응답한다. ::

   http://127.0.0.1:20040/monitoring/geoiplist

결과는 JSON형식으로 제공된다. ::

   {
       "version": "1.2.0",
       "method": "geoiplist",
       "status": "OK",
       "result":
       {
           "path" : "/usr/ston/geoip/",
           "files" :
           [
               {
                   "file" : "GeoIP.dat",
                   "size" : 766255
               },
               {
                   "file" : "GeoLiteCity.dat",
                   "size" : 12826936
               }
           ]
       }
   }


.. _access-control-vhost:

가상호스트 접근제어
====================================

가상호스트별로 접근을 제어한다.
클라이언트가 요청(HTTP 또는 RTMP)을 보낼 때 차단여부를 결정한다.
왜냐하면 연결만하고 구체적인 요청을 보내지 않는다면 가상호스트를 찾을 수 없기 때문이다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <AccessControl Default="Allow">OFF</AccessControl>

-  ``<AccessControl>``

   - ``OFF (기본)`` ACL이 활성화되지 않는다. 모든 클라이언트 요청을 허가한다.

   - ``ON`` ACL이 활성화된다.
     차단된 요청에 대해서는 프로토콜에 따른 적절한 응답코드를 제공한다.

     ``Default (기본: Allow)`` 속성이 ``Allow`` 라면 ACL은 거부목록이 된다.
     반대로 ``Deny`` 라면 ACL은 허가목록이 된다.

Deny된 요청은 :ref:`admin-log-access` 에 TCP_DENY로 기록된다.



.. _access-control-vhost-acl-denialcode:

Deny 응답
---------------------

각 프로토콜 별로 요청이 차단될 때 보낼 응답을 결정한다. ::

   # server.xml - <Server><VHostDefault><Options><Http>
   # vhosts.xml - <Vhosts><Vhost><Options><Http>

   <DenialCode>401</DenialCode>

-  ``<DenialCode> (기본: 401 Unauthorized)`` HTTP 요청이 차단될 때 보낼 응답코드를 설정한다.
   HTTP 응답코드는 `RFC2616 <https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html>`_ 을 참고한다. 


.. note::

   HTTP 기반 프로토콜(HLS, MPEG-DASH)은 모두 ``<HTTP><DenialCode>`` 를 사용한다.

RTMP 프로토콜에서는 NetStream을 통해 재생하는 단계가 Connect, Play로 나뉘어 있어 각 단계마다 보낼 수 있는 메시지가 다르다. ::

   # server.xml - <Server><VHostDefault><Options><Rtmp>
   # vhosts.xml - <Vhosts><Vhost><Options><Rtmp>

   <DenialCodeConnect>Rejected</DenialCodeConnect>
   <DenialCodePlay>Failed</DenialCodePlay>

-  ``<DenialCodeConnect> (기본: Rejected)`` NetStream.Connect 요청이 차단될 때 보낼 응답코드를 설정한다.
-  ``<DenialCodePlay> (기본: Failed)`` NetStream.Play 요청이 차단될 때 보낼 응답코드를 설정한다.

ActinScript 3.0 - `NetStatusEvent <http://help.adobe.com/ko_KR/FlashPlatform/reference/actionscript/3/flash/events/NetStatusEvent.html>`_ 에서 공식적으로 언급하는 응답 메시지는 다음과 같다.

=========================== ========= ============================================
NetStream 코드               Level     의미
=========================== ========= ============================================
Connect.Closed 	            status	  P2P 연결이 성공적으로 종료되었습니다.
Connect.Failed  	        error	  P2P 연결 시도에 실패했습니다.
Connect.Rejected  	        error     P2P 연결 시도에 다른 피어에 대한 액세스 권한이 없습니다.
Connect.Success             status    P2P 연결 시도에 성공했습니다.
Play.Failed 	            error     이 테이블에 나열되지 않은 원인(예: 구독자에게 읽기 액세스 권한이 없음)으로 인해 재생에 오류가 발생했습니다.
Play.FileStructureInvalid   error     응용 프로그램이 잘못된 파일 구조를 감지했습니다.
Play.InsufficientBW 	    warning   클라이언트가 정상적인 속도로 데이터를 재생하기에 충분한 대역폭을 가지고 있지 않습니다.
Play.NoSupportedTrackFound  status    응용 프로그램이 지원되는 추적(비디오, 오디오 또는 데이터)을 감지하지 못했습니다.
Play.PublishNotify          status    스트림의 첫 배급이 모든 구독자에게 보내집니다.
Play.Reset                  status    재생 목록 재설정이 원인입니다. 참고: AIR 3.0 for iOS에서는 지원되지 않습니다.
Play.Start                  status    재생이 시작되었습니다.
Play.Stop                   status    재생이 중지되었습니다.
Play.StreamNotFound         error     NetStream.play() 메서드에 전달된 파일을 찾을 수 없습니다.
Play.Transition             status    서버가 비트율 스트림의 전환 결과로 다른 스트림으로 전환하는 명령을 받았습니다.
Play.UnpublishNotify        status    스트림의 배급 정지가 모든 구독자에게 보내집니다.
=========================== ========= ============================================



.. _access-control-vhost_acl:

가상호스트 ACL
---------------------

모든 클라이언트 요청에 대하여 허용/거부/Redirect 여부를 판단한다.
ACL은 /svc/{가상호스트 이름}/acl.txt에 설정한다. ::

   # /svc/www.example.com/acl.txt
   # 구분자는 콤마(,)이며 {조건},{키워드 = allow | deny | redirect} 순서로 표기한다.
   # n 개의 조건을 결합(AND)하기 위해서는 &를 사용한다.

   $IP[192.168.1.1], allow
   $IP[192.168.2.1-255]
   $IP[192.168.3.0/24], deny
   $IP[192.168.4.0/255.255.255.0]
   $IP[AP] & !HEADER[referer], allow
   $HEADER[cookie: *ILLEGAL*], deny
   $HEADER[via: Apache]
   $HEADER[x-custom-header]
   !HEADER[referer] & !HEADER[user-agent] & !HEADER[host], deny
   $URL[/bar/myLiveStream], allow
   $URL[/private/*], deny
   /broadcast/*adult*, deny
   /secure/*.dat


   # Redirect는 HTTP 에서만 동작한다.
   # Redirect일 경우 키워드 뒤에 이동시킬 URL을 명시한다. (HTTP 응답의 Location헤더의 값으로 명시)

   $IP[GIN], redirect, /page/illegal_access.html
   $HEADER[referer:], redirect, http://another-site.com


   # referer헤더가 존재하지 않는다면 example.com에 요청 URI를 붙여서 Redirect한다.
   # 클라이언트 요청은 /로 시작하기 때문에 #URI 앞에 /를 붙이지 않도록 주의한다.

   !HEADER[referer], redirect, http://example.com#URI



설정은 우선순위를 가지며 조건은 IP, GeoIP, Header, URL 4가지로 설정이 가능하다.

-  ``IP`` 
   $IP[...]로 표기하며 IP, IP Range, Bitmask, Subnet 네 가지 형식을 지원한다.

-  ``GeoIP`` 
   $IP[...]로 표기하며 반드시 GeoIP설정이 되어 있어야 동작한다.

-  ``Header``
   $HEADER[Key : Value]로 표기한다.
   Value는 명확한 표현과 패턴을 인식한다.
   $HEADER[Key:]처럼 구분자는 있지만 Value가 빈 문자열이라면 요청 헤더의 값이 비어 있는 경우를 의미한다.
   $HEADER[Key]처럼 구분자 없이 Key만 명시되어 있다면 Key에 해당하는 헤더의 존재유무를 조건으로 판단한다.

-  ``URL``
   $URL[...]로 표기하며 생략이 가능하다. 명확한 표현과 패턴을 인식한다.

$는 "조건에 맞다면 ~ 한다"를 의미하지만 !는 "조건에 맞지 않는다면 ~ 한다"를 의미한다.
다음과 같이 부정조건으로 지원한다. ::

   # 국가가 KOR이 아니라면 deny한다.
   !IP[KOR], deny

   # referer헤더가 존재하지 않는다면 deny한다.
   !HEADER[referer], deny

   # /secure/ 경로 하위가 아니라면 allow한다.
   !URL[/secure/*], allow

.. note::

   HTTP와 RTMP는 형식과 의미가 다르지만 URL이외의 정보를 Key-Value구조로 다룬다는 점에서는 동일하다. 
   따라서 $HEADER 표현은 RTMP에서 Object의 Property를 검사하는 것으로 사용된다.

   .. figure:: img/sms_acl_vhost_rtmp_property.png
      :align: center

      RTMP의 Connect 요청 예제
   
   위 요청은 아래 조건으로 차단시킬 수 있다. ::

      $HEADER[flashVer: LNX 9,0,124,2], deny
      $HEADER[fpad: false], deny
      $HEADER[videoCodecs: 4071], deny

