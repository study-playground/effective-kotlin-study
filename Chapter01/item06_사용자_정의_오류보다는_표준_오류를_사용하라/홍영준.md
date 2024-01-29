### 사용자 정의 오류보다는 표준 오류를 사용하라
* 가능하다면, 직접 오류를 정의하는 것보다는 최대한 표준 라이브러리의 오류를 사용하는 것이 좋다.
* 표준 라이브러리의 오류는 많은 개발자들이 알고 있으므로, 이를 재사용하는 것이 좋다.
* 잘 만들어진 규약을 가진 널리 알려진 요소를 재사용하면, 다른 사람들이 API 를 더 쉽게 배우고 이해할 수 있다.

### 대표 예시
* IllegalArgumentException
* IllegalStateException
* IndexOutOfBoundsException : 인덱스 파라미터의 값이 범위를 벗어났을 때 
* ConcurrentModificationException : 동시 수정을 금지했는데, 발생했을 때
* UnsupportedOperationException : 사용자가 사용하려고 했던 메서드가ㅏ 현재 객체에서는 사용할 수 없다는 것을 나타냄
* NoSuchElementException : 사용자가 사용하려고 했던 요소가 존재하지 않음을 나타냄
* ArithmeticException : 산술 연산 중 오류가 발생할 때 
* ClassCastException : 객체의 형변환이 불가능할 때 
                                                                                      