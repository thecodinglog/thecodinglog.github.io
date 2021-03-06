---
layout: post
title: Tomcat HTTP Connector 역할과 속성
description: Tomcat HTTP Connector 역할과 속성
date:   2020-08-07 09:00:00 +0900
author: Jeongjin Kim
categories: tomcat
tags:	tomcat servlet container
---

커넥터는 클라이언트와의 통신을 처리합니다. Tomcat에서 사용할 수있는 여러 커넥터가 있습니다. 여기에는 특히 Tomcat을 독립형 서버로 실행할 때 대부분의 HTTP 트래픽에 사용되는 HTTP 커넥터와 Tomcat을 Apache HTTPD 서버와 같은 웹 서버에 연결할 때 사용되는 AJP 프로토콜을 구현하는 AJP 커넥터가 있습니다.

HTTP Connector는 HTTP/1.1 프로토콜을 지원하는 커넥터입니다. Connector는 서버의 특정 TCP 포트 번호에서 연결을 수신하고 요청 처리와 응답을 생성하기 위해 연관된 엔진으로 전달합니다. 

각 요청에는 해당 요청 기간 동안 스레드가 필요합니다. 현재 사용 가능한 요청 처리 스레드에서 처리할 수 있는 것보다 더 많은 동시 요청이 수신되면 구성된 최대 값 (maxThreads 속성 값)까지 추가 스레드가 생성됩니다. 더 많은 동시 요청이 수신되면 커넥터가 만든 서버 소켓 내부에 구성된 최대 값 (acceptCount 속성 값)까지 쌓입니다. 추가 동시 요청은 처리에 리소스를 사용할 수있을 때까지 _connection refused_ 오류를 응답합니다.

# 주요 Attributes

### port

커넥터가 서버 소켓을 만들고 들어오는 연결을 기다리는 TCP 포트 번호입니다. 운영 체제는 하나의 서버 응용 프로그램 만 특정 IP 주소에서 특정 포트 번호를 수신하도록 허용합니다. 특수 값 0 (영)이 사용되면 Tomcat은이 커넥터에 사용할 사용 가능한 포트를 임의로 선택합니다. 이는 일반적으로 임베디드 및 테스트 응용 프로그램에서만 유용합니다.

### acceptCount

사용 가능한 모든 요청 처리 스레드가 사용 중일 때 들어오는 연결 요청의 최대 큐 길이입니다. 대기열이 꽉 찼을 때 수신 된 모든 요청은 거부됩니다. 기본값은 100입니다.

### acceptorThreadCount

연결을 수락하는 데 사용할 스레드 수입니다. 다중 CPU 시스템에서이 값을 늘리는 것이 좋습니다. 그러나 실제로 2개 이상은 필요하지 않을 것입니다. 또한 많은 _non keep alive_ 연결이있는 경우이 값도 늘릴 수 있습니다. 기본값은 1입니다.

### connectionTimeout

커넥터가 연결을 수락 한 후 요청 URI 행이 표시 될 때까지 기다리는 시간 (밀리 초)입니다. 시간 제한 없음 (즉, 무한)는 -1로 설정합니다. 기본값은 60000 (60초)이지만 Tomcat과 함께 제공되는 표준 server.xml은 20000 (20초)으로 설정합니다. _disableUploadTimeout_이 false로 설정되어 있지 않으면 이 제한 시간은 요청 본문 (있는 경우)을 읽을 때도 사용됩니다.

### maxConnections

주어진 시간에 서버가 수락하고 처리 할 최대 연결 수입니다. 이 숫자에 도달하면 서버는 추가 연결을 허용하지만 처리하지는 않습니다. 이 추가 연결은 처리중인 연결 수가 _maxConnections_ 아래로 떨어질 때까지 차단되며, 이때 서버가 새 연결을 다시 수락하고 처리하기 시작합니다. 제한에 도달하면 운영 체제가 _acceptCount_ 설정에 따라 연결을 계속 수락 할 수 있습니다. 기본값은 커넥터 유형에 따라 다릅니다. NIO 및 NIO2의 경우 기본값은 10000입니다. APR / native의 경우 기본값은 8192입니다.

NIO / NIO2의 경우에만 값을 -1로 설정하면 _maxConnections_ 기능이 비활성화되고 연결이 계산되지 않습니다.

### maxThreads

커넥터에서 만드는 최대 요청 처리 스레드 수입니다. 즉 요청을 동시에 처리 할 수 있는 최대 수를 결정합니다. 지정되지 않은 경우 200으로 설정됩니다. Connector에 executor 를 연계한 경우 이 속성은 무시되고 executor를 사용합니다. executor가 구성되면 속성은 설정한 값으로 저장되지만 사용되지 않음을 명확히하기 위해 (JMX를 통해) -1로 보고됩니다.

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- 컨텐츠내 -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-3234744071843247"
     data-ad-slot="1671969273"
     data-ad-format="auto"
     data-full-width-responsive="true"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>