### 요약
* use 를 사용하면 Closeable/AutoCloseable 을 구현한 객체를 쉽게 안전하게 처리할 수 있다.
* 파일을 처리할 때는 한 줄씩 읽어 들이는 useLines 를 사용하는 것이 좋다.

### use 를 사용하여 리소스를 닫아라
* 더 이상 필요하지 않을 때, close 메서드를 사용해서 명시적으로 닫아야 하는 리소스가 있다.
* 예시
  * InputStream, OutputStream
  * java.sql.Connection
  * java.io.Reader(FileReader, BufferedReader, CSSParser)
  * java.new.Socket, java.util.Scanner
  * etc...
* 이러한 리소드들은 AutoCloseable 을 상속받는 Closeable 인터페이스를 구현하고 있고 최종적으로 리소스에 대한 참조가 없어질 때, GC 가 처리한다.
* 하지만 이는 굉장히 느리기 때문에 명시적으로 close 를 호출해주는 것이 좋다.
* 자바에서는 이럴 때 try-with-resources 구문을 이용하여 처리하는게 좋다고 가이드하고있다.
* 코틀린에서는 use 키워드를 사용하여 구현할 수 있다.
* use 함수는 모든 Closeable 객체에 사용할 수 있다.

* 파일 업로드 코드 예시
```kotlin
fun upload(
    inputStream: InputStream,
    size: Long,
    originalFilename: String,
) { 
    val checkedInputStream = CheckedInputStream(inputStream, CRC32())
    checkedInputStream.use {
        uploadService.upload(
            it,
            size,
            originalFilename,
        )
    }
}
```

* 파일을 한 줄씩 처리할 때 활용할 수 있는 useLines 함수도 제공한다.