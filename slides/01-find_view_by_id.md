```kotlin

// ViewModel.kt
findViewById<TextView>(R.id.textView)?.run {
    text = "Lorem ipsum"
}
  
```

```xml

<!-- layout.xml -->
<TextView 
    android:id="@+id/textView"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{vm.sampleText}"
    />
  
```
