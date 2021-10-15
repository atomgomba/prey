```kotlin

@Composable 
fun MessageList(messages: List<@Composable () -> Unit>) {
    Column {
        for (msg in messages) {
            Row { msg() }
        }
    }
}
  
```