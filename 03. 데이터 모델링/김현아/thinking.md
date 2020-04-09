#생각해보기
##3.1 매핑 파라미터
1. analyzer 파라미터를 사용할 수 있는 데이터 타입은 데이터 타입 text 말고 어떤 게 있을까?
    - text 타입만 가능함
    - 공식문서 내용 중 "Only text fields support the analyzer mapping parameter."
    - 7.6 버전 문서 : https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html
2. normalizer는 불용어 처리나 다른 토큰필터도 적용이 가능한가? 중첩도 가능한가?
    - 불용어 처리와 같은 전체단어를 봐야하는 필터는 적용이 안된다
    - 다른 토큰필터도 적용 가능하다
    - 여러 개의 필터를 중첩하여 사용 가능하다
    - analyzer과 유사하나 다른점은 tokenizer 적용 여부이다
    - tokenizer는 사용 불가하며, 캐릭터필터와 일부 토큰필터를 적용할 수 있다
    - 글자당 적용하는 filter는 가능한데, 전체단어를 봐야하는 filter는 적용 불가
    - ex) lowercasing filter 적용 가능하지만, stemming filter는 불가함
    - 사용 가능한 token filters
        - ```
          버전 7.6
          he current list of filters that can be used in a normalizer is following: 
          arabic_normalization, 
          asciifolding, 
          bengali_normalization, 
          cjk_width, 
          decimal_digit, 
          elision, 
          german_normalization, 
          hindi_normalization, 
          indic_normalization, 
          lowercase, 
          persian_normalization, 
          scandinavian_folding, 
          serbian_normalization, 
          sorani_normalization, 
          uppercase. 
          ```
3. 색인하지 않고 검색하는 것의 의미는?
    - 아래 두 파라미터를 보고 색인을 하지 않고 검색하거나, 검색은 안되지만 _source에 표시된다는 것이 어떤 의미인지 궁금해짐
        - dynamic 옵션 중 false : 색인되지 않아 검색은 안되지만 _source에 표시됨
        - enabled : 검색은 하고 싶지만 색인은 하고 싶지 않는 경우 사용
    - 아직 잘 이해가 안감 => 온라인 스터디를 통해 이해함 (회의록_2020-04-04)
    - 검색 조건으로는 제외하지만, 검색 결과로 노출하고 싶다는 의미
4. index와 enabled의 차이점
    - https://stackoverflow.com/questions/50836504/elasticsearch-mapping-parameters-index-vs-enabled
    - enabled : 
    - index : 
5. similarity 유사도 알고리즘 BM25 VS TF/IDF
    - 단어의 빈도수
        - TF/IDF : common words can still influence the score
        - BM25 : limits influence of term frequency
    - 길이가 짧은 문장의 적중도 (긴 문장보다 당연히 높게 나올테니까)
        - TF/IDF : short fields(title, ...) are automatically scored higher
        - BM25 : scales field length with average

##3.2 메타 필드
1. 특정 문서를 하나의 샤드로 몰아넣고 싶을 때 _routing 필드를 쓴다. 그런데 특정 샤드로 몰아 넣는 것은 어떻게 하는 것일까?
   ```
   # 별도의 설정 없이 문서를 색인하면 샤드에 골고루 분산되어 저장된다
   Hash (document_id) % num_of_shards
   
   # _routing 메타 필드를 사용할 경우 색인할 때 해당 문서들은 동일한 라우팅ID를 지정한다. 문서 ID를 사용하는 대신 파라미터로 입력한 _routing 값이 샤드를 결정하는데 사용된다.
   Hash (_routing) % num_of_shards
   ```
##3.3 필드 데이터 타입
1. 매핑 파라미터 'norms'과 Text 데이터타입의 'norms' 파라미터는 같은가?
    - 이 둘은 다른 것이다.
    - 매핑 파라미터 norms
        - p73 설명 : 문서의 _score 값 계싼에 필요한 정규화 인수를 사용할지 여부를 설정함. 기본값은 true. _score 계산이 필요없는 경우 비활성화해서 디스크 공간 절약 가능
        - 공식문서 : https://www.elastic.co/guide/en/elasticsearch/reference/current/norms.html
    - Text 데이터 타입의 norms 파라미터
        - p88 설명 : 유사도 점수를 산정할 때 필드 길이를 고려할지를 결정한다. 기본값은 true
        - 공식문서 : https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html

2. 문자열의 종류
    - 공식문서 : https://esbook.kimjmin.net/07-settings-and-mappings/7.2-mappings/7.2.1
    - 선언 가능한 문자열 타입은 text, keyword (2.x 버전 이전에는 string 하나의 타입만 존재)

3. Array 타입의 문자열의 타입은 무엇일까?
    - 내가 지정한 걸로 되겠지

##3.4 엘라스틱서치 분석기
1. 동의어 사전 적용 시점에 내용 중 이해가 가지 않아 여러 번 읽은 문단 (p128)
    > 검색 시점에는 사전의 내용이 변경되더라도 해당 내용이 반영된다. 하지만 색인 시점에 동의어 사전이 사용됐다면 사전의 내용이 변경되더라도 색인이 변경되지 않는다. 이 경우에는 기존 색인을 모두 삭제하고 색인을 다시 생성해야만 변경된 사전 내용이 적용된다. 이러한 문제점 때문에 동의어 사전이 빈번하게 수정되는 인덱스의 경우 색인 시점에는 적용하지 않고 검색 시점에만 저굥하는 방식으로 이러한 문제점을 해결하기도 한다.
    - 동의어 사전의 내용 변경되었을 때, 검색 시점에는 이를 반영하는데 색인 시점에는 반영하지 않는다는 뜻이겠지?
###3.5 Document API 이해하기
1. 예전 버전의 문서는 어떻게 확인할까?
    - 안된대. 버전만 관리하고 이전 _source는 갖고 있지 않나봐.
    - Is it possible to get older versions of any document : <https://discuss.elastic.co/t/is-it-possible-to-get-older-versions-of-any-document/7019>
2. 예전 버전의 문서 삭제
    - segments가 인덱스를 병합하기 때문에 언젠가는 예전 버전의 문서는 자동적으로 삭제됨
    - Deleting lod versions <https://discuss.elastic.co/t/deleting-old-versions/4689/2>
3. _source_exclude로 제외한 필드는 검색 결과에서 제외된다. 그렇다면 검색 대상에서도 제외될까?
    - 웅먄ㅇ먕먕
4. _source가 비활성화 혹은 일부 필드가 검색이 안될 경우, Update API 가능한가?
    - _source 는 원천데이터이고, Update API는 _source의 값을 가지고 재색인을 함
    - 그렇다면 Update API가 _source에 항상 접근할 수 있나? 흐음. 수정되면 안되는 필드는 어떻게 보호할까?
