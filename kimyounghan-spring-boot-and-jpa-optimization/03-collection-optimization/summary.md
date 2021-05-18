# 정리

## 엔티티 조회

### V1

- 엔티티를 조회해서 직접 반환한다.

### V2

- 엔티티 조회 후 DTO로 변환한다.
- 여러 테이블을 join 할 때 성능이 나오지 않는다.

### V3

- fetch join으로 쿼리 수를 최적화한다.

### V3.1

- 컬렉션 페이징의 한계를 돌파한다.
- 컬렉션은 fetch join 시 페이징이 불가능하다.
- 실무에서는 페이징을 할 일이 많으므로 다음과 같이 구현한다.
    - ToOne 관계는 fetch join으로 쿼리 수를 최적화한다.
    - 컬렉션은 fetch join 대신 지연 로딩을 유지한다.
        - fetch join을 하면 페이징이 안되기 때문이다.
    - `hibernate.default_batch_fetch_size`, `@BatchSize`로 최적화한다.
        - 지연 로딩을 하면 N+1 문제가 나타나므로 이 옵션으로 해결한다.

## DTO 직접 조회

V4, 5, 6에서 단순하게 쿼리가 1번 실행된다고 V6이 항상 좋은 방법인 것은 아니다.

### V4

- JPA에서 DTO를 직접 조회한다.
- 코드가 단순하다.
    - 그냥 loop만 돌면 된다.
- 특정 주문 한 건만 조회하면 이 방식으로도 성능이 잘 나온다.
    - Order 조회 결과가 1건이면 OrderItem 조회 쿼리도 1번만 실행하면 된다.

### V5

- 컬렉션 조회를 최적화한다.
    - 일다대 관계인 컬렉션은 in 절을 활용해 모두 불러온다음 map으로 만들어 메모리에서 미리 조회한다.
- 코드가 복잡하다.
- 여러 주문을 한 번에 조회하는 경우에는 이 방식을 사용한다.
    - Order 데이터가 1000건일 때 V4는 쿼리가 1 + 1000번 실행된다.
        - 1은 Order를 조회한 쿼리 수, 1000은 조회된 Order의 row 수다.
    - V5를 쓰면 쿼리가 총 1 + 1만 실행된다.
- 쿼리가 2번 나가지만(네트워크를 2번 타지만) 딱 정규화된 데이터를 보낸다.
- DTO를 직접 조회해야할 때는 거의 V5 방식을 많이 사용한다.

### V6

- 플랫 데이터로 최적화한다.
    - join 결과를 그대로 조회한 후 애플리케이션에서 원하는 모약으로 직접 변환한다.
- 쿼리 한 번으로 조회하므로 좋아보이지만 Order 기준으로 페이징할 수 없다.
- 실무에서는 이 정도로 데이터가 많으면 페이징이 꼭 필요하므로 선택하기 어렵다.
- 데이터가 많으면 중복 전송이 증가해서 V5와의 성능 차이도 미비하다.
    - 데이터가 뻥튀기 되어서 네트워크 전송량이 많아지기 때문에 그걸 따지면 V5가 더 좋을 수도 있다.

## 권장 순서

1. 엔티티 조회 방식으로 우선 접근한다.
    - fetch join으로 쿼리 수를 최적화한다.
    - 컬렉션을 최적화한다.
        - 페이징이 필요하다면 `hibernate.default_batch_fetch_size`, `@BatchSize`로 최적화한다.
        - 페이징이 필요없다면 fetch join을 사용한다.
2. 엔티티 조회 방식으로 해결이 안되면 DTO 조회 방식을 사용한다.
3. DTO 조회 방식으로 해결이 안되면 NativeSQL이나 스프링 JdbcTemplate을 사용한다.

## 참고

- 엔티티 조회 방식은 fetch join이나 `hibernate.default_batch_fetch_size`, `@BatchSize`처럼 코드를 거의 수정하지 않는다.
    - 옵션만 약간 변경해서 다양한 성능 최적화를 시도할 수 있다.
    - 반면 DTO를 직접 조회하는 방식은 성능을 최적화하거나 성능 최적화 방식을 변경할 떄 많은 코드를 변경해야 한다.
    - 엔티티 조회 방식으로 대부분의 애플리케이션이 해결된다.
        - 해결이 안되는 문제는 정말 트래픽이 많은 것이므로 캐시 등 다른 방법을 생각해봐야 한다.
        - 참고로 엔티티는 영속성 컨텍스트에서 관리되는 상태가 있기 때문에 캐싱하면 안된다.
        - DTO로 변환해서 캐시해야 한다.
- 엔티티 조회 방식은 JPA가 많은 부분을 최적화해주기 때문에 단순한 코드를 유지하면서 성능을 최적화할 수 있다.
    - DTO 조회 방식은 SQL을 직접 다루는 것과 유사하기 때문에 성능 최적화와 코드 복잡도 사이에서 줄타기를 해야한다.