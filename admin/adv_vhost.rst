﻿.. _adv-vhost:

14장. 가상호스트 고급기법
******************

이 장에서는 가상호스트를 활용하여 서비스를 유연하게 구성하는 여러 기법에 대해 설명한다.

가상호스트는 보통 원본(Domain 또는 IP목록)과 1:1로 구성되는 것이 기본이다.
하지만 상황에 따라 대표 가상호스트를 여러 하위 가상호스트로 분기하거나,
반대로 독립적인 여러 가상호스트를 하나의 서비스로 포장해야 하는 경우도 발생한다.
각 기능에 따라 :ref:`monitoring_stats_vhost_client` / :ref:`admin-log-access` 의 정책이 다를 수 있음을 유의해야 한다.


.. toctree::
   :maxdepth: 2


.. _adv-vhost-facadevhost:

Facade 가상호스트
====================================

``<Alias>`` 는 가상호스트의 별명만을 추가하는 것이므로 통계와 로그가 분리되지 않는다.
가상호스트는 공유하지만 도메인에 따라 :ref:`monitoring_stats_vhost_client` 와 :ref:`admin-log-access` 를 분리하고 싶은 경우 Facade가상호스트를 설정한다.

.. figure:: img/sms_adv_vhost_facade1.png
   :align: center

   Facade는 통계와 로그만 수집한다.

::

    # vhosts.xml - <Vhosts>

    <Vhost Name="example.com">
       ...
    </Vhost>

    <Vhost Name="another.com" Status="facade:example.com">
       ...
    </Vhost>

``Status`` 속성의 값을 ``facade:`` + ``가상호스트`` 로 설정한다.
예제의 경우 :ref:`monitoring_stats_vhost_client` 와 :ref:`admin-log-access` 는 example.com이 아닌 클라이언트가 요청한 도메인인 another.com으로 수집된다.



.. _adv-vhost-sub-path:

Sub-Path 지정
====================================

한 가상호스트에서 경로에 따라 다른 가상호스트가 처리하도록 설정할 수 있다.

.. figure:: img/sms_adv_vhost_subpath1.png
   :align: center

   통계/로그는 요청을 최종처리한 각각의 가상호스트에 기록된다.


::

   # vhosts.xml - <Vhosts>

   <Vhost Name="sports.com">
     <Sub Status="Active">
       <Path Vhost="baseball.com">/baseball/<Path>
       <Path Vhost="football.com">/football/<Path>
     </Sub>
   </Vhost>

   <Vhost Name="baseball.com" />
   <Vhost Name="football.com" />

-  ``<Sub>`` 경로나 패턴이 일치하면 해당 요청을 다른 가상호스트로 보낸다.
   일치하지 않는 경우만 현재 가상호스트가 처리한다.

   - ``Status (기본: Active)`` Inactive인 경우 무시한다.

   -  ``<Path>`` 클라이언트가 요청한 URI와 경로가 일치하면 ``Vhost`` 로 해당 요청을 보낸다.
      값은 경로 또는 패턴만 가능하다. ::

예를 들어 클라이언트가 다음과 같이 요청했다면 해당 요청은 가상호스트 football.com이 처리한다. ::

   GET /football/top10.mp4 HTTP/1.1
   Host: sports.com


.. _adv-vhost-redirection-trace:

HTTP Redirect 추적
====================================

원본서버에서 Redirect계열(301, 302, 303, 307)로 응답하는 경우 Location헤더를 추적하여 콘텐츠를 요청한다.

   .. figure:: img/sms_adv_vhost_redirectiontrace1.png
      :align: center

      클라이언트는 Redirect여부를 모른다.

::

   # server.xml - <Server><VHostDefault><OriginOptions><Http>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions><Http>

   <RedirectionTrace>OFF</RedirectionTrace>

-  ``<RedirectionTrace>``

   - ``OFF (기본)`` 3xx 응답으로 저장된다.

   - ``ON`` Location헤더에 명시된 주소에서 콘텐츠를 다운로드 한다.
     형식에 맞지 않거나 Location헤더가 없는 경우에는 동작하지 않는다.
     무한히 Redirect되는 경우를 방지하기 위하여 1회만 추적한다.



.. _adv-vhost-trafficcap:

대역폭 제한
====================================

가상호스트의 최대 Outbound 대역폭을 제한한다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <TrafficCap>0</TrafficCap>

-  ``<TrafficCap> (기본: 0)``
   가상호스트의 최대 Bandwidth를 Mbps단위로 설정한다.
   0으로 설정하면 Bandwidth을 제한하지 않는다.

STON 미디어 서버와 모든 클라이언트 사이에 발생하는 대역폭의 총합은 ``<TrafficCap>`` 을 넘을 수 없다.
``<TrafficCap>`` 을 50 (Mbps)로 설정했다면 50Mbps NIC를 설치한 것과 같은 효과를 낸다.
