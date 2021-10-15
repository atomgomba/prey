```kotlin

@Composable fun MessageWidget(message: State<String>) {
    Text(message.value)
}

@Composable fun MessageWidget() {
    // do a heavy operation once and keep 
    // the result during recompositions
    val msg by remember { fetchChuckNorrisJoke() }
    Text(msg)
}
  
```
