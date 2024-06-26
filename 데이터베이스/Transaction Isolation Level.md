- Read Uncommitted
  - 가장 낮은 수준의 격리 수준
  - 한 트랜잭션의 변경사항이 커밋되지 않아도 다른 트랜잭션에서 조회가 가능합니다.
  - 이로 인해 데이터가 보였다 안 보였다 하는 Dirty Read 현상이 발생
- Read Committed
  - 한 트랜잭션의 변경사항이 커밋되어야 해당 변경사항을 다른 트랜잭션에서 조회 가능한 격리 수준
  - 다른 트랜잭션은 undo 영역을 확인하여 데이터를 참조
  - 한 트랜잭션 내의 동일한 조회 쿼리의 결과가 다르게 나타날 수 있는 Non-repeatable Read 현상이 발생
- Repeatable Read
  - 한 트랜잭션 내의 동일한 조회 쿼리의 결과가 동일한 격리 수준
  - 각 트랜잭션 별로 고유한 Id값을 가지어서 undo영역에서 데이터를 가져올 때 자신의 Id보다 낮은 id의 트랜잭션의 변경으로 발생한 값들만 조회
- Serializable
  - 모든 트랜잭션을 순차적으로 실행하는 격리 수준
  - 실시간 서비스에 사용 시 동시성이 매우 떨어져 성능이 제대로 나오지 않을 수 있음
