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
    - tokenizer는 사용 불가하며, 일부의 캐릭터필터와 토큰필터를 적용할 수 있다
    - 글자당 적용하는 filter는 가능한데, 전체단어를 봐야하는 filter는 적용 불가
    - ex) lowercasing filter 적용 가능하지만, stemming filter는 불가함
    - 사용 가능한 filters
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
    - 아직 잘 이해가 안감
4. index와 enabled의 차이점
    - https://stackoverflow.com/questions/50836504/elasticsearch-mapping-parameters-index-vs-enabled
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