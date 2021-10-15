```xml

<androidx.compose.ui.platform.ComposeView 
    android:id="@+id/composeView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    />
  
```

```kotlin

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    setContent {
        // composables can be used here
        MessageWidget()
    }
}
  
```