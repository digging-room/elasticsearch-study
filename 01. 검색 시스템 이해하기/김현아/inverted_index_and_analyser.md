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
    - match 쿼리 : 애널라이저를 적용하여 검색
    - term 쿼리 : term 쿼리는 검색어에 애널라이저를 적용하지 않고 검색어 그대로 일치하는 term 검색
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
  - 데이터 색인 과정에서 검색 기능에 가장 큰 영향을 미치는 단계
  - 한 개만 적용 가능
  1. Standard, Letter, Whitespace
     - Standard 토크나이저
        - 공백으로 텀을 구분하면서 "@"과 같은 일부 특수문자를 제거
        - 단어 끝의 특수문자는 제거하지만 중간에 있는 특수문자는 제거되거나 분리되지 않음
     - Letter 토크나이저
        - 알파벳을 제외한 모든 공백, 숫자, 기호들을 기준으로 텀을 분리
     - Whitespace 토크나이저
        - 스페이스, 탭, 그리고 줄바꿈 같은 공백만을 기준으로 텀을 분리
     - 3개의 토크나이저 중에 Letter 토크나이저의 경우 검색 범위가 넓어져서 원하지 않는 결과가 많이 나올 수 있음
     - 반대로 Whitespace의 경우 특수문자를 거르지 않기 때문에 정확하게 검색을 하지 않으면 검색 결과가 나오지 않을 수 있음
     - 보통은 Standard 토크나이저를 많이 사용함
  2. UAX URL Email
    - 이메일 주소 또는 웹 URL 경로용 토크나이저
  3. Pattern
    - 공백 기준이 아닌 다른 문자 (",", ":") 등으로 구분자를 사용하고 싶을 때 이용
    - 분리할 패턴을 기호 또는 Java 정규식 형태로 지정 가능
  4. Path Hierarchy
    - 디렉토리나 파일 경로용 토크나이저
    - 경로 데이터를 계층별로 저장해서 하위 디렉토리에 속한 도큐먼트들을 수준별로 검색하거나 집계 가능
     
##6. 토큰 필터 (Token Filter)
  - 토크나이저를 이용한 텀 분리 과정 이후에는 분리된 각각의 텀 들을 지정한 규칙에 따라 처리하는 과정을 담당
  - 나열된 순서대로 처리되기 때문에 순서를 잘 고려해야 함
  - 토큰 필터 역시 종류가 상당히 많고 계속 업데이트 되기 때문에 공식문서 참고
  - 주로 사용되는 토큰 필터는 아래와 같음
  1. Lowercase, Uppercase : 대소문자 구분없이 검색이 가능하도록 처리
  2. Stop : 불용어(stopword) 처리
  3. Synonym : 동의어 처리
     - ex) amazon을 aws로 동의어 처리
        - docs : ["Amazon Web Service", "AWS"]
        - 첫번째 문서의 텀벡터(_termvectors) : aws, web, service
        - 두번째 문서의 텀벡터(_termvectors) : aws
     - 동의어 처리 옵션
        - expand (디폴트는 true) : 설정에 토큰들을 모두 저장하지 않고 맨 처음에 명시된 토큰 하나만 저장. "synonyms": "aws, amazon => aws" 로 설정한 것과 동일하게 동작
        - lenient (디폴트는 false) : synonym 설정에 오류가 있는 경우 오류가 있는 부분을 무시하고 실행
  4. NGram, Edge Ngram, Shingle : 자동 완성 기능 구현 시 사용(일반적인 텍스트분석에는 적합하지 않음)
     - NGram
        - 텀이 아닌 단어의 일부만 가지고도 검색해야 하는 기능이 필요한 경우 사용
        - 단어의 일부를 나눈 부위를 NGram 이라고 함
        - 옵션 : min_gram (디폴트 1), max_gram (디폴트 2)
     - Edge Ngram 
        - 검색을 위해 NGram을 저장하더라도 보통은 단어의 맨 앞에서부터 검색하는 경우가 많음
        - 텀 앞쪽의 ngram 만 저장하기 위해서는 Edge NGram 토큰필터를 이용
        - ex) edgeNGram의 옵션을 "min_gram": 1, "max_gram": 4 으로 설정
        - "house" 를 분석하면 다음과 같이 "h", "ho", "hou", "hous" 4개의 토큰이 생성됨
     - Shingle
        - NGram과 Edge NGram은 모두 하나의 단어로부터 토큰을 확장하는 토큰 필터
        - 옵션은 공식문서 참고
  5. Unique
    - 하나의 doc에 중복되는 중복되는 텀 들은 하나만 저장
    
##7. 형태소 분석 (Stemming)
  - Stemming = 어간 추출 또는 형태소 분석
  1. Snowbell : Elasticsearch 내장되어 제공함
  2. 노리(Nori) 한글 형태소 분석기 (추가 공부 필요)
     - 아리랑 (arirang) : <https://github.com/HowookJeong/elasticsearch-analysis-arirang>
     - 은전한닢 (seunjeon) : mecab-ko-dic 기반, 시스템 사전에 등록되어 있는 단어에 한하여 복합명사 분해와 활용어 원형 찾기가 가능, <https://bitbucket.org/eunjeon/seunjeon>
     - Open Korean Text : 텍스트 정규화와 형태소 분석, 스테밍을 지원 <https://github.com/open-korean-text/open-korean-text>
     - 노리(Nori)
        - Elasticsearch 6.6 버전 부터 공식적으로 Nori(노리) 한글 형태소 분석기 지원
        - 공식문서 : <https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-nori.html>
  

## Tips
* 인덱스 새로고침 (기존의 사전 파일의 내용이 변경 시 새로고침을 통해 토큰필터 새로 적용)
```
기존의 사전 파일의 내용이 변경 된 경우 인덱스를 새로 고침을 해 주어야 토큰 필터가 새로 적용됩니다.
이것은 stop 외에도 뒤에 설명할 synonym 이나 nori 한글 형태소 분석기 사전에도 동일하게 적용됩니다.
새로 고침을 하는 방법은
POST <인덱스명>/_close
POST <인덱스명>/_open
을 차례대로 실행 해 주면 됩니다. 
인덱스가 close 된 중에는 색인이나 검색이 불가능 하게 되니 주의해야 합니다.
또한 애널라이저의 사전만 갱신되는 것이기 때문에 이미 색인된 도큐먼트들의 역 색인 내용은 변경되지 않습니다. 
인덱스 새로 고침 이후에 색인되는 데이터들과 match 쿼리의 검색 등에만 적용이 됩니다. 
기존 도큐먼트의 역 색인을 변경하려면 데이터를 모두 다시 재색인을 해야 합니다.
<https://esbook.kimjmin.net/06-text-analysis/6.6-token-filter/6.6.2-stop>
```
* 동의어 처리 시 사전 관리 방법 추천
```
동의어 여러 개를 입력 할 때는 "synonyms": [ ... ] 항목 안에 배열로 넣어도 되지만, 그 보다는 파일을 따로 만들어 관리하는 것이 편합니다. 
stop 토큰 필터와 마찬가지로 synonyms_path 항목에 config 디렉토리 기준의 상대 경로에 파일을 저장하고 경로명을 입력하면 됩니다. 
동의어는 하나의 규칙당 한 줄씩 입력해야 하며 파일은 UTF-8로 인코딩 되어야 합니다.
```
* Ngram 토큰필터와 자동완성
```
ngram 토큰필터를 사용하면 저장되는 텀의 갯수도 기하급수적으로 늘어난다.
검색어를 "ho"로 검색 했을 때 house, shoes 처럼 검색 결과를 예상하기 어렵기 때문에 일반적인 텍스트 검색에는 사용하지 않는 것이 좋습니다. 
ngram을 사용하기 적합한 사례는 카테고리 목록이나 태그 목록과 같이 전체 개수가 많지 않은 데이터 집단에 자동완성 같은 기능을 구현하는 데에 적합합니다.
```

* Unique 토큰필터와 TF(Term Frequency)
```
match 쿼리를 사용해서 검색하는 경우 unique 토큰 필터를 적용한 필드는 텀의 개수가 1개로 되기 때문에 TF(Term Frequency) 값이 줄어들어 스코어 점수가 달라질 수 있습니다. 
match 쿼리를 이용해 정확도(relevancy) 를 따져야 하는 검색의 경우에는 unique 토큰 필터는 사용하지 않는 것이 바람직합니다.
```