A Compose UI toolkit több szempontból is elmozdulást jelent a korábbi View-ra épülő rendszertől miközben kompatibilis (vagy interoperábilis) marad vele:

* nem része a platformnak
    * az OS-től függetlenül updatelhető, gyorsabban jönnek a bugfixek és open-source
* teljesen Kotlinban van írva
    * kihasználja a nyelv remek szintaktikai adottságait, pl. lambdák, keyword argumentek, top-level functions
* sokkal egyszerűbb a Composable widgetek életciklusa
    * megjelenik
    * kirajzolódik vagy újrarajzolódik
    * eltűnik
* deklaratív

A klasszikus rendszerben imperatív módon írtuk le a UI-t, azaz megmondtuk a rendszernek, h mikor mit változtasson a felületen. Az alábbi példák mindenkinek ismerősek:

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

A fenti példában a szöveget view binding segítségével állítjuk be, ami látszólag deklaratív eljárás, de a motorháztető alatt a Gradle plugin legenerálja
nagyjából ugyanazt a boilerplate kódot, amit amúgyis használnánk és az első példában látszik. 

A deklaratív paradigmában ezzel ellentétben leírjuk, azaz deklaráljuk, hogy a UI miként nézzen ki és a rendszer az adatok függvényében magától tudja 
eldönteni, h mikor mit változtasson meg. Ez a paradigma a webfejlesztésben a jQuery korszakra hasonlít, amikor kiválasztottunk a DOM-ból elemeket és 
azokat mutáltuk manuálisan. A Compose ezt haladja meg a DSL használatával. Ebben a paradigmában View osztályok helyett egyszerű függvényeket tudunk 
használni a UI definiálására és nem kell többet XML-ben mókolni, hiszen a UI kódból fog összeállni.

```kotlin
// Composable function names are UpperCamelCase
@Composable fun MessageWidget() {
    Text("Lorem ipsum", color = Color.RED)
}
```

Itt a `@Composable` annotáció jelzi a compilernek, h ez egy composable függvény, aminek a törzse a UI leírását tartalmazza. A példában megjelenítünk egy
egyszerű hard kódolt szöveget anélkül, h külön létre kellene hozni ehhez egy XML fájlt, egyszerűen kódból.

Ez természetesen nagy könnyebség nekünk, fejlesztőknek, mert az XML fájlok szerkesztgetése körülményes, de plusz hozadéka az is, h így jobban érvényesíthető
az enkapszuláció elve. A kinézet és a viselkedés egy helyen definiálható és nem kell az adatok kezelését (ami Kotlinban van) külön kezelni a kinézettől, 
azaz pl. nem fogunk tudni olyan hibát elkövetni, h egy időközben az XML fájlból kitörölt widgetettel akarunk valamit csinálni vagy valamilyen működés
attól függ, h egy widgetnek mi a parentje, hol szerepel a hierarchiában, stb. Másképpen mondva csökkenthető a UI logika külső függése.
Itt gondoljon mindenki valami régi jó kis jQuery-alapú web appra, ahol külön figyelmet kellett szentelni annak, h a HTML szinkronban legyen a JS kóddal 
és a DOM strukturája valóban megfeleljen annak, amire az üzleti logika számít.

A rekompozíció (újrarajzolás) reaktívan történik meg, tehát minden alkalommal, amikor a mögöttes adat változik. Ezeket az adatokat `State` objektumokban
lehet tárolni, amire a kapcsolódó Composable fel tud iratkozni és változás esetén automatikusan újrarajzolódni.

```kotlin
@Composable fun MessageWidget(message: State<String>) {
    Text(message.value)
}
```

Ez a nomenklatúra ismerős lehet a `LiveData` miatt és valóban, egy `LiveData` objektumból egy extension functionnel (`LiveData<T>.observeAsState()`)
létrehozható `State` objektum, illetve használható ugyanígy Coroutine Flow vagy RxJava2 is. Tehát a leggyakrabban használt reaktív megoldásokkal
múködik együtt, illetve hát az összessel.
A fenti példában az állapot élettartama hosszabb, mint magának a composablenek az élettartama, mert kívülről kapja meg, de lehetne az állapot lokális is, 
ha a függvényen belül jön létre. Összetettebb widgetek esetén érdemes is egy magasabb szintú konténer widgetből lepasszolni a gyermekeknek, h ne
sok kis elaprózodott állapottal kelljen dolgozni.

```kotlin
@Composable fun MessageWidget() {
    // do a heavy operation once and keep the result during recompositions
    val msg by remember { fetchChuckNorrisJoke() }
    Text(msg)
}
```

Mivel itt a widgetek csak függvények, ezért argumentumként bármilyen értéket át lehet nekik adni. A beépített komponensek például mind tartalmaznak
egy `modifier` argumentumot, amivel a widget layout tulajdonságait lehet beállítani és még egyéb specifikus argumentumokat. Ezzel lehet a widgeteket
úgy módosítani, ahogy XML-ben az XML attribútumokat használtuk.

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

A lambdaszintaxisnak köszönhetően könnyedén át lehet adni más composable függvényeket is.

```kotlin
@Composable fun MessageList(messages: List<@Composable () -> Unit>) {
    Column {
        for (msg in messages) {
            Row {
                msg()
            }
        }
    }
}
```

A Compose interoperábilis a View-s szisztémával, ezért meglévő projektbe be lehet vezetni lépésenként, pl. egy új képernyőnél elkezdeni használni.
Ehhez nyújtja a library a `ComposableView` osztályt, ami beilleszthető bármilyen View hierarchiába és azon belül elérhető a teljes Compose toolkit.


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

@Preview
