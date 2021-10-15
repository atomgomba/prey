```kotlin

@Composable fun MessageList(messages: List<String>) {
    Column {
        for (msg in messages) {
            Text(
                msg, 
                modifier = Modifier.padding(8.dp).
                textAlign = TextAlign.Center,
            )
        }
    }
}
  
```