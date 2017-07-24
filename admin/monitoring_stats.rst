.. _monitoring-stats:

모니터링 & 통계
******************


.. toctree::
   :maxdepth: 2



가상호스트 - 종합통계
====================================

가상호스트 종합통계에 Type속성과 RTMP 원본통계가 추가된다. ::

   "VirtualHost":                              <VirtualHost                                 
   [                                               Name="www.example.com"  
     {                                             Uptime="155986"
       "Name":"www.example.com",                   Type="live"
       "Uptime":155996,                            ... (생략) ...
       "Type":"live"                               MpegDashOriginOutbound="0"
       ... (생략) ...                               RtmpOriginSession="0"
       "MpegDashOriginOutbound":2509,              RtmpOriginActiveSession="0"
       "RtmpOriginSession":1,                      RtmpOriginInbound="0"
       "RtmpOriginActiveSession":1,                RtmpOriginOutbound="0"
       "RtmpOriginInbound":2481,                   AllClientSession="0" 
       "RtmpOriginOutbound":8,                     ... (생략) ...
       "AllClientSession":54,
       ... (생략) ...

가상호스트 Type이 ``LIVE`` 인 경우 각 채널별 상세 통계가 Channel로 추가된다.

   "VirtualHost":                              <VirtualHost                                 
   [                                               Name="www.example.com"  
     {                                             Uptime="155986"              
       "Name":"www.example.com",                   AllOriginSession="0"

       ... (생략) ...                               ... (생략) ...

       "RtmpClientOutbound":8,                     ByteHitRatio="0">
       "RequestHitRatio":6387,                   <Memory>784786700</Memory>
       "ByteHitRatio":2926                       <SecuredMemory>0</SecuredMemory>
       "Memory":785740769,                       <Disk> ... </Disk>
       "SecuredMemory":0,                        <CacheFileEvent> ... </CacheFileEvent>
       "Disk": { ... },                          <OriginTraffics> ... </OriginTraffics>
       "CacheFileEvent": { ... },                <ClientTraffic> ... </ClientTraffic>
       "OriginTraffics": { ... },                <Channels> ... </Channels>
       "ClientTraffics": { ... },                <Channels> ... </Channels>
       "Channel": { ... }                        <Channels> ... </Channels>
     },                                        </VirtualHost>
     ...
   ]

-  ``Channel`` LIVE 방송에 대한 상세통계

가상호스트가 LIVE 서비스로 설정되어 있는 경우 가상호스트의 종합통계와 원본/클라이언트 통계는 ``Channels`` 통계를 합한 것과 같다. 



Channel 통계
------------------------------

가상호스트 통계와 마찬가지로 채널 통계가 프로토콜별로 구분되어 제공된다.  ::

   "Channel":                                  <Channel
   [                                               Name="/myLiveStream"  
     {                                             Uptime="155986"              
       "Name":"/myLiveStream",                     AllOriginSession="0"                                   
       "Uptime":155996,                            AllOriginActiveSession="0"   
       "AllOriginSession":33,                      AllOriginInbound="0"                                    
       "AllOriginActiveSession":20,                AllOriginOutbound="0"                                   
       "AllOriginInbound":688177,                  HttpOriginSession="0"                                   
       "AllOriginOutbound":14184,                  HttpOriginActiveSession="0"
       "HttpOriginSession":62,                     HttpOriginInbound="0"                                   
       "HttpOriginActiveSession":62,               HttpOriginOutbound="0"                                  
       "HttpOriginInbound":2375,                   HlsOriginSession="0"
       "HttpOriginOutbound":2509,                  HlsOriginActiveSession="0"
       "HlsOriginSession":62,                      HlsOriginInbound="0"                                     
       "HlsOriginActiveSession":62,                HlsOriginOutbound="0"                                 
       "HlsOriginInbound":2375,                    MpegDashOriginSession="0"                                  
       "HlsOriginOutbound":2509,                   MpegDashOriginActiveSession="0"                            
       "MpegDashOriginSession":62,                 MpegDashOriginInbound="0"                                   
       "MpegDashOriginActiveSession":62,           MpegDashOriginOutbound="0"                                 
       "MpegDashOriginInbound":2375,               AllClientSession="0"                                    
       "MpegDashOriginOutbound":2509,              AllClientActiveSession="0"                              
       "AllClientSession":54,                      AllClientInbound="0"                                    
       "AllClientActiveSession":2327,              AllClientOutbound="0"                                   
       "AllClientInbound":2481,                    HttpClientSession="0"                                   
       "AllClientOutbound":8,                      HttpClientActiveSession="0"                             
       "HttpClientSession":54,                     HttpClientInbound="0"                                   
       "HttpClientActiveSession":2327,             HttpClientOutbound="0"                                  
       "HttpClientInbound":2481,                   HlsClientSession="0"                                    
       "HttpClientOutbound":8,                     HlsClientActiveSession="0"                              
       "HlsClientSession":54,                      HlsClientInbound="0"                                    
       "HlsClientActiveSession":2327,              HlsClientOutbound="0"                                   
       "HlsClientInbound":2481,                    MpegDashClientSession="0"                                   
       "HlsClientOutbound":8,                      MpegDashClientActiveSession="0"                             
       "MpegDashClientSession":54,                 MpegDashClientInbound="0"                                    
       "MpegDashClientActiveSession":2327,         MpegDashClientOutbound="0"                                   
       "MpegDashClientInbound":2481,               RtmpClientSession="0"                                   
       "MpegDashClientOutbound":8,                 RtmpClientActiveSession="0"                             
       "RtmpClientSession":54,                     RtmpClientInbound="0"                                   
       "RtmpClientActiveSession":2327,             RtmpClientOutbound="0"                                  
       "RtmpClientInbound":2481,                   RequestHitRatio="0"
       "RtmpClientOutbound":8,                     ByteHitRatio="0">
       "RequestHitRatio":6387,                   <OriginTraffics> ... </OriginTraffics>
       "ByteHitRatio":2926                       <ClientTraffic> ... </ClientTraffic>
       "OriginTraffics": { ... },              </Channel>
       "ClientTraffics": { ... }
     },
     ...
   ]

원본/클라이언트 상세통계는 가상호스트와 동일하다.

