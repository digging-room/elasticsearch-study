# Contents

## 1. 클러스터 관점에서 구성요소 살펴보기

  1. 클러스터
  2. 노드
  3. 인덱스
  4. 문서
  5. 샤드
  6. 레플리카
  7. 세그먼트

## 2. 엘라스틱서치 샤드 vs 루씬 인덱스

## 3. 엘라스틱서치가 근실시간 검색을 제공하는 이유

  1. 색인 작업 시 세그먼트의 기본 동작 방식
  2. 세그먼트 불변성
  3. 세그먼트 불변성과 업데이트
  4. 루씬을 위한 Flush, Commit, Merge
  5. 엘라스틱서치를 위한 Refresh, Flush, Optimize API
  6. 엘라스틱서치와 NRT(Near Real-Time)

## 4. 고가용성을 위한 Translog의 비밀

  1. Translog의 동작 순서
  2. Translog가 존재하는 이유

## 5. 엘라스틱서치 샤드 최적화

  1. 운영 중에 샤드의 개수를 수정하지 못하는 이유
  2. 레플리카 샤드의 복제본 수는 얼마가 적당할까?
  3. 클러스터에서 운영 가능한 최대 샤드 수는?
  4. 하나의 인덱스에 생성 사능한 최대 문서 수는?
