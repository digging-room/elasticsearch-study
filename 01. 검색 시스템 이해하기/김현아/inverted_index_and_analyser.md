#데이터 색인과 텍스트 분석
엘라스틱 공식 가이드북와 여러 자료를 바탕으로 용어 및 개념을 정리함

> Elastic 공식 가이드북 : <https://esbook.kimjmin.net/06-text-analysis>

##1. 역 인덱스 (Inverted Index)
- 역 인덱스 구조 : 
 ```
테이블구조 : [{'id':1, 'text':'the fox'}, {'id':2, 'text':''fox'}] 
역인덱스   : [{'term':'fox', 'id':1, 2}, {'term':'the', 'id':1}] 
 ```
- term : 추출된 각 키워드
- 색인 : 데이터를 저장하는 과정에서 역 인덱스 구조로 변환하기 때문에 elasticsearch에서는 저장이 아닌 색인을 한다고 표현함


##2. 텍스트 분석 (Text Analysis)
  - ES에서는 저장되는 모든 doc의 텍스트 필드 별로 역 인덱스를 생성함
  - 문자열 필드는 Text Analysis 과정을 거쳐 검색어 토큰으로 저장됨
  - Text Analysis 과정을 처리하는 기능을 애널라이저(Analyser)라고 함
  - ES의 애널라이저의 구성
  <img src="https://gblobscdn.gitbook.com/assets%2F-Ln04DaYZaDjdiR_ZsKo%2F-LntYrdKmTe441TqYAJl%2F-LntZ63SAIfHu6Q_OgzJ%2F6.2-02.png?alt=media&token=52213afe-e6ab-4bc2-b9e0-20027542a79e"></img>
    + 0~3개의 캐릭터필터(Character Filter)
      + 전체 문장에서 특정 문자 대치 및 제거
    + 1개의 토크나이저(Tokenizer)
      + 문장에 속한 단어들을 텀(term) 단위로 분리하여 처리하는 기능
      + 1개만 적용이 가능함
    + 0~N개의 토큰필터(Token Filter)
      + 분리된 term들을 하나씩 가공하는 기능
      + 대소문자처리 : lowercase 토큰 필터 (보통 디폴트로 쓰임)
      + 불용어(stopword) 처리 : the, a, am과 같은 검색어로 가치가 없는 단어 제거
      + 형태소 분석기 (언어마다 다름) : 영어는 snowbell 토큰 필터를 통해 jumping과 jumps를 "jump"라는 하나의 term으로 병합
      + 동의어 처리 (synonym 토큰필터) : quick 텀에 동의어로 fast를 지정하면, fast로 검색 시 quick 텀을 포함하는 문서도 검색되도록 할 수 있음


##3. 애널라이저 (Analyser)
  1. analyze API : ES에서 텍스트에 tokenizer와 N개의 filter를 적용했을 때 어떤 term이 추출되는 지 확인 가능
      > 공식문서 : https://esbook.kimjmin.net/06-text-analysis/6.3-analyzer-1/6.3-analyzer
      - 토크나이저는 tokenizer 하나 입력
      - 토큰 필터는 filter 항목의 값으로 [ ] 안에 배열 형식으로 입력 
      - 애널라이저는 캐릭터 필터, 토크나이저 그리고 토큰 필터들을 조합해서 사용자 정의 애널라이저, 혹은 엘라스틱에서 제공되는 snowball과 같이 바로 사용 가능한 것이 있다.
      - 엘라스틱서치 빌트인 애널라이저 : <https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html>
  2. term 쿼리
    - match 쿼리와 유사
    - 애널라이저를 적용하지 않고 검색어 그대로 일치하는 term 검색
    - jumps, jumping, jump 중 정확히 "jumps" 라고 입력해야 하나 나옴
  3. 사용자 정의 애널라이저 및 인덱스에 적용
    - analyze API로 애널라이저, 토크나이저, 토큰필터들의 테스트가 가능함
    - 하지만 실제로 인덱스에 저장되는 데이터의 처리에 대한 설정은 애널라이저만 적용 가능
  4. 텀 벡터 termvectors API
    - 색인된 도큐먼트의 역 인덱스의 내용을 확인할 때는 도큐먼트 별로 _termvectors API를이용해서 확인이 가능   
     ```
     GET <인덱스>/_termvectors/<도큐먼트id>?fields=<필드명>
     ```
  * 참고 : 텍스트 분석(Analysis) 과정은 검색에 사용되는 역 인덱스에만 관여합니다. 원본 데이터는 변하지 않으므로 쿼리 결과의 _source 항목에는 항상 원본 데이터가 나옵니다.


##4. 캐릭터 필터 (Character Filter)
  - 텍스트 분석 중 가장 먼저 처리되는 과정
  - 색인된 텍스트가 토크나이저에 의해 텀(term)으로 분리되기 전에 사용되는 전처리 도구
  - 7.0버전 기준으로 3개의 필터 존재
  - 하나만 적용하거나, 모두 적용할 수 있음
  1. HTML Strip
    - HTML 태그 및 &nbsp;와 같은 특수문자 제거하여 텍스트로 변환
  2. Mapping
    - 지정한 단어를 다른 단어로 치환
    - 다수의 애널라이저들이 특수문자를 불용어로 처리하기 때문에 "C", "C++"의 검색결과가 동일한 결과를 나타냄
    - C++ 텀을 cpp 처럼 치환할 경우 검색될 가능성이 있는 모든 특수문자를 치환해야하기 때문에 번거로움
    - 특수문자 "+" 를 "_plus"로 치환하여 색인하면 C++ 검색 시, C++에 해당하는 문서가 검색됨
  3. Pattern Replace
    - 정규식을 이용한 복잡한 패턴의 치환 처리 가능
    
##5. 토크나이저 (Tokenizer)

##6. 토큰 필터 (Token Filter)

##7. 형태소 분석 (Stemming)