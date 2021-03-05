# Ma maîtrise de Kotlin

Ce document a pour but de regrouper des explications et resources nécessaires concernant la maîtrise du langage Kotlin. Il peut s'apparenter à un **cheat sheet**, mais au delà de la syntax, le but est d'expliquer les concepts et mécanique sous le capot. Cela permettra d'utiliser chaque spécificité du langage en maîtrise total afin d'éviter les mauvaises surprises et effet de bord.

Ce document ne s'attarde pas sur les bases du langage mais va plutôt creuser dans les concepts avancées et est plutôt à destination des développeurs Java.

ps: :gb: An english version will be made at a certain point.

## Concepts avancés

## by lazy()

Sucre syntaxique permettant de faire du [lazy initialization](https://en.wikipedia.org/wiki/Lazy_initialization) à l'aide d'une [delegate property](https://kotlinlang.org/docs/reference/delegated-properties.html#delegated-properties). Concrètement, la valeur de la propriété n'est calculée qu'au premier accès.

```kotlin
val database: AppDatabase by lazy { AppDatabase.getInstance(appInstance) }
```

*La variable `database` et son code d'initialisation `AppDatabase.getInstance(this)` seront executés lors de la première utilisation de `database`.*

Cela peut être utile si la propriété est lourde a initialiser et qu'elle n'est pas utilisée au premier plan (exemple: une classe personne qui stocke des adresses supplémentaires affichées seulement si l'utilisateur demande la vue détaillée de la personne). Également utile pour initialiser une propriété qui a besoin d'un Context ou une Activity, créées que lorsque la classe s'initialise.

> Kotlin in Action (p. 190): 7.5.2 Using delegated properties: lazy initialization and “by lazy()”

> kotlinlang: https://kotlinlang.org/docs/reference/delegated-properties.html#lazy

> api ref: [kotlin-stdlib / kotlin / lazy](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/lazy.html)

## Le mot-clé `by` (class delegation)

Le mot clé `by` permet de déléguer l'implémentation d'une interface à un autre objet de votre classe, ceci afin de faciliter l'utilisation du [decorator pattern](https://fr.wikipedia.org/wiki/D%C3%A9corateur_(patron_de_conception)).

```kotlin
class CountingSet<T>(
    val innerSet: MutableCollection<T> = HashSet<T>()
) : MutableCollection<T> by innerSet {

    var objectsAdded = 0

    override fun add(element: T): Boolean {
        objectsAdded++
    return innerSet.add(element)
    }
}
```

*Dans cet exemple, toutes les méthodes de l'interface `MutableCollection<T>` ont été implémentées par l'objet `innerList` sauf la méthode `add(element: T)` qui a été overridée avec un comportement différent.*

> Kotlin in Action (p. 91): 4.3.3 Class delegation: using the `by` keyword

> kotlinlang: https://kotlinlang.org/docs/reference/delegation.html#implementation-by-delegation

> Voir aussi [by lazy()](#by-lazy()), Delegated Properties

## Annotations

## @JvmOverloads

Permet de générer des surcharge de méthodes Java pour les paramètres optionnels d'une méthode Kotlin. Sans cela, du code Java consommant une classe Kotlin devra à chaque fois utiliser la signature complète.

Exemple simple :
```kotlin
// Kotlin
class ClassWithDefaultArguments @JvmOverloads constructor(
        name: String,
        age: Int = 42,
        isMale: Boolean = true
)
```

```java
// Consomme une classe Kotlin avec des paramètres par défaut en Java  
new ClassWithDefaultArgs("Joe");
new ClassWithDefaultArgs("Joe", 12);
new ClassWithDefaultArgs("Joe", 12, false);// sans @JvmOverloads, seule cette dernière méthode aurai pu être utilisée
```

> Fonctionne également avec les méthodes autre que les constructeur  

Permet également de consommer une classe Java en Kotlin qui attends la surcharge de plusieurs méthodes.

Exemple avec l'héritage de la class `View` :

```kotlin
class MyCustomView @JvmOverloads constructor(
        context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {
}
```

> Kotlin in Action (p. 49): 3.2.2 Default parameter values

> kotlinlang: [Overloads generation](https://kotlinlang.org/docs/reference/java-to-kotlin-interop.html#overloads-generation)

## Coroutines

### Introduction

**kotlinx.coroutines** est une librairie qui permets de gérer l'asynchronisme d'une manière séquentielle. Elle permet de gérer les changements de thread, lancer des tâches en parallèles, rendre [main-safe]() nos fonctions devant être lancées hors main thread et permet de faire de la "concurrence structuré". La librairie permet également de remplacer RxJava et les LiveData grace aux coroutines **Flows**. Tout cela se fait via des coroutines qu'on peut voir comme des threads légés. Toute la librairie est utilisable en code commun kotlin multiplatforme.

Fonctions :

- [`launch`](#launch-{-}) lance une coroutine non-bloquante
- [`witchContext(Dispatchers.IO)`]() lance une coroutine en changeant de dispatcher (thread ou groupe de threads)
- `await` lance une coroutine non-bloquante retournant une promise `Deferred<T>` qui renverra dans le futur un résultat
- `runBlocking` A utilsier pour des tests, permet d'appeler des fonctions suspend

Objets :

- `CoroutineScope` 
- `Job` objet retourné par les fonctions lançant des coroutines. Permet d'attendre ou d'annuler la coroutine et tous ses enfants.
- `Dispatcher` représent un thread/pool de threads dans lequel une coroutine sera exécutée
- ``

Toutes ces méthodes doivent être lancées depuis un `CoroutineScope` leur retour permet d'annuler la coroutines et tous ces enfants

Exemple simple dans Android

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    // launches a new coroutine on the main thread
    lifecycleScope.launch {
        val user = sdk.signIn(username, password) // web api call on background thread
        textview.text = user.name // update views on main thread
        avatar.image = user.image
    }
}
```

Equivalence :

- Dispatcher <-> Pool de threads
- CoroutineContext
### launch { }

Lance une nouvelle coroutine sans bloquer le thread actuel.

La fonction retourne un `job` qui peut être utilisé pour annuler la coroutine.

Sans paramètre, hérite du `scope + dispatcher` parent.

```kotlin
GlobalScope.launch {
    delay(2000L)
    println("This is printed after 2 seconds")
}
println("This is printed before the first println()")
```

### CoroutineScope

Un scope de coroutine représente la "sandbox" dans laquelle la coroutine va être exécutée.

A tout moment un scope peut annuler toutes les coroutines en cours d'execution à l'aide de `scope.cancel()`.

Vous pouvez créer vos propres scopes, en plus d'avoir plusieurs scopes déjà disponible sur Android, notamment `lifecycleScope` et `viewModelScope` ; qui sont des scopes liés au cycle de vie d'une Activity/Fragment/ViewModel et détruits/annulés lorsque 

> [Lifecycle-aware coroutine scopes](https://developer.android.com/topic/libraries/architecture/coroutines#lifecycle-aware)
#### GlobalScope

`GlobalScope` est un scope qui est "vivant" pendant toute la durée de vie de l'application. Il est à utiliser uniquement pour des démos ou tests.
`GlobalScope.launch { }` lancera une nouvelle coroutine avec le `Dispatchers.Default`.

Il est généralement préférable sur Android d'utiliser un scope lié au cycle de vie d'une vue (`Activity` ou `Fragment`), ou d'un `ViewModel`.


> [kotlinx.coroutines launch { }](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html)

### coroutineScope { }

Est une lambda avec `CoroutineScope` comme *receiver*.

`coroutineScope` est conçue pour faire de la décomposition parallèle. Si une coroutine enfant échoue à l'intérieur du scope, le scope échoue et toutes les autres coroutines enfants sont annulées.
La fonction retourne dès que le scope et toutes les coroutines enfants sont complètés

```kotlin
suspend fun concurrentSum(): Int = coroutineScope {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    one.await() + two.await()
}
```

### CoroutineContext

Persiste le contexte de la coroutine.

- CoroutineDispatcher
- CoroutineExceptionHandler (optional)
- CoroutineName (optional)
- Job

Doit être passé en paramètre d'un `CoroutineScope`.

```kotlin
suspend fun concurrentSum(): Int = coroutineScope {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    one.await() + two.await()
}
```


### suspendCoroutine { }

Est utilisé pour suspendre l'exécution de la coroutine.
A l'intérieur du bloc `it = continuation`

### runBlocking { }

Lance une coroutine bloquante sans avoir de scope à spécifier.
Permet d'appeler des `suspend function`. Le thread est bloqué jusqu'à la fin de l'exécution du bloc.

```kotlin
suspend fun demo() {
    println("Dans une suspend fonction")
}

fun main() {
    // ne compile pas
    // suspend function 'demo' should be called only from a coroutine or another suspend function
    demo()
  
    // compile:
    runBlocking {
        demo()
    }
    println("Ceci est affiché seulement après la fin d'exécution du bloc runBlocking")
}
```

> `runBlocking` annule l'avantage non-bloquante des coroutines. D'une manière générale cela doit seulement être utiliser pour faire des tests, jouer avec les coroutines où dans les tests unitaires

### async { }

Lance une coroutine et retourne une valeur sous forme de Deferred.

```kotlin
private suspend fun doNetworkCall(): String {
    delay(2000)
    return "Network Answer"
}

GlobalScope.launch(Dispatchers.IO) {
    Log.d("TAG", "Before async calls")
    val answer = async { doNetworkCall() }
    Log.d("TAG", "After async calls") // this is printed immediately
    Log.d("TAG", "Answer: ${answer.await()}") // this is printed after 2 seconds
}
```

### Android lifecycleScope and viewModelScope

Scopes from lifecycle (of the activity or fragment) and a ViewModel. Launching a new coroutine is done in the main thread.

### Job

Un job est une tâche annulable avec un cycle de vie qui se termine quand il est complété. Généralement retourné par `launch` ou toute autre méthode qui lance une coroutine.

`isActive`, `isCompleted` et `isCancelled` sont des propriétés aidant à vérifier l'état d'un `job`.

`join()` pour attendre la fin d'un job.
`cancel()` pour annuler un job et tous les jobs enfants.

> official doc: [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html)

### SupervisorJob

### Flows

### StateFlow

Les `StateFlow` sont presque comme les `LiveData`.

- Émets la dernière valeur quand un nouveau consommateur écoute.


### Ressources sur Android

https://developer.android.com/kotlin/coroutines/coroutines-best-practices