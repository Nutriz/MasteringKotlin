# Ma maîtrise de Kotlin

Ce document a pour but de regrouper des explications et resources nécessaires concernant la maîtrise du langage Kotlin. Il peut s'apparenter à un **cheat sheet**, mais au delà de la syntax, le but est d'expliquer les concepts et mécanique sous le capot. Cela permettra d'utiliser chaque spécificité du langage en maîtrise total afin d'éviter les mauvaises surprises et effet de bord.

Ce document ne s'attarde pas sur les bases du langage mais va plutôt creuser dans les concepts avancées et est plutôt à destination des développeurs Java.

ps: :gb: An english version will be born at a certain point.

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