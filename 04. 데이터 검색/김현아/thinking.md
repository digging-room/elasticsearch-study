#생각해보기
##4.2.3 Query DSL의 주요 파라미터
1. 기본적으로 Multi Index 및 Multi 타입 검색으로 다수의 인덱스에 대해 한 번의 요청으로 결과를 얻을 수 있다.
  - 서버에 N개의 인덱스로 구성하는 경우도 있을 수 있다.
  - 만약 서로 다른 성격의 인덱스라면, 불필요하게 검색하는 것은 비효율적이다.
  - 인덱스명을 지정하여 조회하는 습관을 들이는 것이 좋을 것 같다.
