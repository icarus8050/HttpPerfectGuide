# 캐시
캐시를 이용하면 다음과 같은 장점이 있다.
* 불필요한 데이터 전송을 줄여서, 네트워크 비용을 절감해준다.
* 네트워크 병목을 줄여서 대역폭을 늘리지 않고도 페이지를 빨리 불러올 수 있다.
* 원 서버에 대한 요청을 줄일 수 있다.
* 페이지를 먼 곳에서 불러올수록 지연시간이 많이 걸리지만, 캐시는 거리로 인한 지연을 줄여준다.

## Cache hit, Cache miss
* 캐시에 요청이 왔을 때, 해당 요청이 캐싱되어 있다면 바로 응답한다. (Cache hit)
* 캐시에 요청이 왔을 때, 해당 요청이 캐싱되어 있지 않다면 원 서버로 요청을 전달한다. (Cache miss)

## 재검사
* 캐시는 최신 상태인지 점검해야 한다.
* 모든 데이터를 검사하기에는 네트워크 대역폭 문제로 인해 해당 데이터가 검사가 필요할 정도로 오래된 경우에 검사를 진행한다.
* 캐시의 재검사가 필요할 때, 원 서버에 작은 재검사 요청을 보내고, 데이터가 변경되지 않았다면 304 Not Modified 응답을 보낸다.
* 캐시 데이터가 최신화될 필요가 없으면 해당 데이터의 유효시간을 최신화한다.

## 사본을 신선하게 유지하기
* 캐시된 사본은 모두 시간에 따라 변경된다.
* 캐시된 데이터는 서버의 데이터와 일치하도록 관리해야 한다.

### 문서 만료
* HTTP 는 CacheControl 과 Expire 라는 특별한 헤더들을 이용해서 원 서버가 각 문서에 유효기간을 붙일 수 있다.
* 캐시된 문서가 만료되면 캐시는 반드시 서버와 문서에 변경된 것이 있는지 검사해야한다.
* 변경된 것이 있다면 새 유효기간과 함께 새로운 사본을 가져와야 한다.

### 유효기간과 나이
* Cache-Control: max-age
  * max-age 값은 문서의 최대 나이를 정의한다. 최대 나이는 문서가 처음 생성된 이후부터 신선하지 않다고 간주될 때까지 경과한 시간의 최댓값(초 단위)이다.
  * Cache-Control: max-age=484200
* Expires
  * 절대 유효기간을 명시한다.
  * Expires: Fri, 05 Jul 2002, 05:00:00 GMT

### 조건부 메서드와의 재검사
* HTTP 는 캐시가 서버에게 조건무 GET 이라는 요청을 보낼 수 있도록 해준다.
* 서버가 갖고 있는 문서가 캐시가 갖고 있는 것과 다른 경우에만 객체 본문을 보내달라고 하는 것이다.
* HTTP 는 다섯 자기 조건무 요청 헤더를 정의하는데, 그 중 둘은 캐시 재검사를 할 때 가장 유용한 If-Modified-Since 와 If-None-Match 이다.
* 모든 조건부 헤더는 'If-' 접두어로 시작한다.

### If-Modified-Since: <date>
* 문서가 주어진 날짜 이후로 수정되었다면 요청 메서드를 처리한다. 캐시된 버전으로부터 콘텐츠가 변경된 경우에만 가져오기 위해 Last-Modified 서버 응답 헤더와 함께 사용된다.

### If-None-Match: <tags>
* 서버는 문서에 대한 일련번호와 같이 동작하는 특별한 태그(ETag)를 제공할 수 있다. If-None-Match 헤더는 캐시된 태그가 서버에 있는 문서의 태그와 다를 때만 요청을 처리한다.
* 캐시가 객체에 대한 여러 개의 사본을 갖고 있는 경우, 그 사실을 서버에게 알리기 위해 하나의 If-None-Match 헤더에 여러 개의 엔티티 태그를 포함할 수 있다.
* If-None-Match: "v2.6", "v2.7"

### 약한 검사기와 강한 검사기
* 모든 캐시된 사본을 무효화시키지 않고 문서를 살짝 고치더라도 같은 문서라고 허용하고 싶은 경우에 '약한 검사기(weak validator)' 를 지원한다.
* 강한 검사기(strong validator)는 콘텐츠가 바뀔 때마다 바뀐다.
* 서버는 'W/' 접두사로 약한 검사기를 구분한다.
* ETag: W/"v2.6"
* If-None-Match: W/"v2.6"

### 캐시 제어
* 서버가 캐시를 제어하는 설정 방법에 대해서 우선순위대로 나열해보면 아래와 같다.
* Cache-Control: no-store
* Cache-Control: no-cache
* Cache-Control: must-revalidate
* CAche-Control: max-age
* Expires 날짜 헤더
* 아무 만료 정보를 주지 않고, 캐시가 휴리스틱한 방법으로 결정

### no-cache 와 no-store 응답 헤더
* 'no-store' 가 표시된 응답은 캐시가 그 응답의 사본을 만드는 것을 금지한다.
* 'no-cache' 가 표시된 응답은 로컬 캐시 저장소에 저장될 수 있지만, 먼저 서버와 재검사 하지 않고서는 캐시에서 클라이언트로 제공될 수 없다.

### max-age 응답 헤더
* 신선하다고 간주되었던 문서가 서버로부터 온 이후로 흐른 시간이고, 초로 나타낸다.
* s-maxage 헤더는 max-age 처럼 행동하지만 공용 캐시에만 적용된다.
* 서버는 maximum aging 을 0으로 설정함으로써, 캐시가 매 접근마다 문서를 캐시하거나 리프레시하지 않도록 요청할 수 있다.

### Expires 응답 헤더
* 더 이상 사용하지 않기를 권하는 Expires 헤더는 초 단위의 시간 대신 실제 만료 날짜를 명시한다.
* Expires: Fri, 05 Jul 2002, 05:00:00 GMT

### Must-Revalidate 응답 헤더
* 캐시는 성능을 개선하기 위해 신선하지 않은 객체를 제공하도록 설정될 수 있다.
* 만약 캐시가 만료 정보를 엄격하게 따르길 원하는 경우에 사용한다.
* 캐시가 이 객체의 신선하지 않은 사본을 원 서버와의 최초의 재검사 없이는 제공해서는 안 됨을 의미한다.

### 클라이언트의 신선도 제약
* 클라이언트는 Cache-Control 요청 헤더를 사용하여 만료 제약을 엄격하게 하거나 느슨하게 할 수 있다.
* Cache-Control: max-stale
  * 캐시는 신선하지 않은 문서라도 자유롭게 제공될 수 있다.
* Cache-Control: max-stale = <s>
  * 클라이언트는 만료시간이 S의 값만큼 지난 문서도 받아들인다.
* Cache-Control: min-fresh = <s>
  * 클라이언트는 지금으로부터 적어도 <s>초 후까지 신선한 문서만을 받아들인다.
* Cache-Control: max-age = <s>
  * 캐시는 s초보다 오랫동안 캐시된 문서를 반환할 수 없다. age 가 유효기간을 넘어서게 되는 max-stale 지시어가 함께 설정되지 않는 이상, 이 지시어는 캐싱 규칙을 더 엄격하게 만든다.
* Cache-Control: no-cache
  * 이 클라이언트는 캐시된 리소스는 재검사하기 전에는 받아들이지 않는다.
* Cache-Control: no-store
  * 이 캐시는 저장소에서 문서의 흔적을 최대한 빨리 삭제해야 한다.
* Cache-Control: only-if-cached
  * 클라이언트는 캐시에 들어있는 사본만을 원한다.

---

## 참고자료

[HTTP 완벽 가이드](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966261208&orderClick=LEA&Kc=)