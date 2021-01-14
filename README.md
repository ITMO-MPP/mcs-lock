# MCS Lock

Оказывается, что замечательное свойтсво алгоритма MCS блокировки, которое заключется в том, что он ждет на своем узле, 
можно использовать для реализации честной (First Come, First Served) блокировки, которая усыпляет
ждущий поток через `park` и пробуждает его через `unpark`. Вместо того, чтобы крутиться в бесконечном цикле, 
ожидая пока взявший блокировку поток её освободит, можно крутиться в цикле вида:

```kotlin 
while (node.locked) park()
```                

А при освобождении блокировки можно будить другой поток, если в его узле предварительно запомнить ссылку на 
создавший его поток для последующего вызова `unpark`:

```kotlin 
node.locked = false
unpark(node.thread)                                                                                           
```

Рассуждение выше касается только ожидание блокировки, которая в данный момент занята другим потоком. 
Вспомогательное ожидание в коде `unlock` алгоритма MCS блокировки должно выполняться простым циклом.  

Вам предостоит написать и отладить этот код.

## Задание

В файле [`src/Lock.kt`](src/Lock.kt) 
находится описание интерфейса блокировки, который вам предстоит реализовать. Так как алгоритму MCS необходимо
передать указатель на свой узел между методами `lock` и `unlock`, то для вашего удобства соотвествующие 
методы определены так, что `lock` возвращает указатель на узел, который потом будет передан в `unlock`
тестирующим кодом:

```kotlin
fun lock(): N
fun unlock(node: N)
```                            

То есть, тестирующий код всегда входит в критическую секцию используюя вот такую функцию `withLock`:

```kotlin
inline fun <N> Lock<N>.withLock(block: () -> Unit) {
    val node = lock()
    block()
    unlock(node)
}
```

Ваше решение должно быть в файле `src/Solution.(kt|java)`. Основной класс должен называться `Solution`.
Шаблоны решений вы найдете здесь: 
* На языке Kotlin в файле [`src/SolutionTemplateKt.kt`](src/SolutionTemplateKt.kt) 
* На языке Java в файле [`src/SolutionTemplateJava.java`](src/SolutionTemplateJava.java)

Класс решения должен принимать в конструкторе параметр типа [`Environment`](src/Environment.kt) и должен взывать 
методы `park` и `unpark` на экземпляре этого класса, чтобы тесты могли передать вашего алгоритму
тестовую реализацию этих методов для проверки корректности вашего кода.  

В классе решения вы должны использовать объекты типа `AtomicRefernce` для общих пременных между потоками.
Для хранения флажка `locked` используйте `AtomicReference<Boolean>`. В языке Kotlin вы можете менять значение 
атомарных переменных через свойство `value`, например:

```kotlin 
val locked = AtomicReference<Boolean>(false) // объявление переменной в классе
locked.value = true // выставить значение в коде 
```

Все поля классов должны быть неизменяемые (`val` в Kotlin, `final` в Java). Массивы использовать нельзя. 
Также вам потребуется вспомогательный класс для узла, на который вы так же можете ссылаться и на который 
накладываются такие же ограничения. В шаблоне уже есть пример такого класса. Он также запоминает указатель на поток.   
Вспомогательные классы должны быть внутри вашего класса решения. 

Для проверки вашего решения запустите из корня репозитория:
* `./gradlew build` на Linux или MacOS
* `gradlew build` на Windows
 
Тесты:
* [`СodeTest`](test/CodeTest.kt) проверяет выполнение ограничений описанных в условии. 
* [`CorrectnessTest`](test/CorrectnessTest.kt) проверяет правильность работы блокировки в соответсвии с FCFS и
  корректность использования методов part/unpark.
* [`StressTest`](test/StressTest.kt) проверяет взаимную блокировку и прогресс под нагрузкой.
