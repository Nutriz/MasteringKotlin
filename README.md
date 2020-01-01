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

> Kotlin in action (p. 190): 7.5.2 Using delegated properties: lazy initialization and “by lazy()”

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

> Kotlin in action (p. 91): 4.3.3 Class delegation: using the `by` keyword

> kotlinlang: https://kotlinlang.org/docs/reference/delegation.html#implementation-by-delegation

> Voir aussi [by lazy()](#by-lazy()), Delegated Properties
