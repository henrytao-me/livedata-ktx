[![GitHub license](https://img.shields.io/badge/license-MIT-lightgrey.svg?maxAge=2592000)](https://raw.githubusercontent.com/shopify/livedata-ktx/master/LICENSE)
[ ![Download](https://api.bintray.com/packages/shopify/shopify-android/livedata-ktx/images/download.svg) ](https://bintray.com/shopify/shopify-android/livedata-ktx/_latestVersion)
[![Build Status](https://travis-ci.org/Shopify/livedata-ktx.svg?branch=master)](https://travis-ci.org/Shopify/livedata-ktx)

# livedata-ktx

Kotlin extension for LiveData, chaining like RxJava.

[README For 1.x](https://github.com/Shopify/livedata-ktx/blob/master/README.1.x.mdx)

# Getting Started

To add LiveData KTX to your project, add the following to your app module's build.gradle:

```groovy
implementation "com.shopify:livedata-ktx:VERSION"
```

#### For `android.arch.lifecycle:livedata:1.x`, please use `com.shopify:livedata-ktx:1.x`

#### For `androidx.lifecycle:livedata:2.x`, please use `com.shopify:livedata-ktx:2.x`

# Usage

### Chaining LiveData

```kotlin
val liveData = MutableLiveData<Boolean>()
liveData
  .distinct()
  .filter { it == false }
  .map { true }
  .nonNull()
  .observe(lifecycleOwner, { result ->
    // result is non-null and always true
  })
```

### Observe with lifecycle

```kotlin
val liveData = MutableLiveData<Boolean>()
liveData
  .distinct()
  .observe(lifecycleOwner, {
    // TODO
  })
```

### Observe forever

```kotlin
val liveData = MutableLiveData<Boolean>()
liveData
  .distinct()
  .observe({
    // TODO
  })
```

### Preserve immutability

```kotlin
val mutableLiveData = MutableLiveData<Boolean>()
val liveData: LiveData<Boolean> = mutableLiveData.distinct()
// liveData.value is invalid

val singleLiveData = SingleLiveData<Boolean>()
val nonNullLiveData: NonNullLiveData<Boolean> = singleLiveData
// nonNullLiveData.value is invalid

val nonNullMutableLiveData = MutableLiveData<Boolean>()
val otherNonNullLiveData: NonNullLiveData<Boolean> = nonNullMutableLiveData.nonNull()
// otherNonNullLiveData is invalid
```

### Remove observer

Because the input observer goes through a wrapper before it observes to source LiveData. So that you can't simply remove it by just calling origin method `liveData.removeObserver`.

The new observe method returns `Removable` interface that allows you to remove observer effectively.

```kotlin
val liveData = MutableLiveData<Boolean>()
val removable = liveData
  .nonNull()
  .observe(lifecycleOwner, {
    // TODO
  })
removable.removeObserver()

val removableFromObserveForever = liveData.observe {
  // TODO
}
removableFromObserveForever.removeObserver()
```

### SingleLiveData

It is a lifecycle-aware observable that sends only new updates after subscription, used for events like navigation and Snackbar messages. `livedata-ktx` has different implementation comparing to SingleLiveEvent from [google samples android-architecture](https://github.com/googlesamples/android-architecture/blob/dev-todo-mvvm-live/todoapp/app/src/main/java/com/example/android/architecture/blueprints/todoapp/SingleLiveEvent.java).

```kotlin
val liveData = SingleLiveData<Int>()
val actuals: MutableList<Int?> = mutableListOf()
val observer: (t: Int) -> Unit = { actuals.add(it) }

liveData.value = 1
liveData.observe(this, observer)
liveData.value = 2
liveData.value = 3

val expecteds = mutableListOf(2, 3)
assertEquals(expecteds, actuals)
```

For more use cases, please see the tests at [LiveDataTest.kt](https://github.com/shopify/livedata-ktx/blob/master/livedata-ktx/src/test/java/com/shopify/livedataktx/LiveDataTest.kt)

# Feel missing methods

It is easy to add your custom extension without requiring to send a PR. For example:

```kotlin
/**
 * filter
 */
internal class FilterExt<T>(private val predicate: (T?) -> Boolean) : Operator<T, T> {

    override fun run(value: T?, source: LiveData<T>, onResult: OnResult<T>) {
        val predicated = predicate(value)
        onResult(Result(value, !predicated))
    }
}

fun <T> LiveData<T>.filter(predicate: (T?) -> Boolean): LiveData<T> = SupportLiveData(this, FilterExt<T>(predicate))
fun <T> NonNullLiveData<T>.filter(predicate: (T) -> Boolean): NonNullLiveData<T> = SupportNonNullLiveData(this, FilterExt {
    predicate(it!!)
})
```

# Contributing

Any contributions are welcome!
Please check the [CONTRIBUTING](CONTRIBUTING.md) guideline before submitting a new issue. Wanna send PR? [Click HERE](https://github.com/shopify/livedata-ktx/pulls)

# Maintainers

- Francisco Cavedon <[@fcavedon](https://github.com/fcavedon)>
- Henry Tao <[@henrytao-me](https://github.com/henrytao-me)>
- Ivan Savytskyi <[@sav007](https://github.com/sav007)>
- Kris Orr <[@krisorr](https://github.com/krisorr)>

# License

    The MIT License (MIT)

    Copyright (c) 2018 Shopify Inc.

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in
    all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
    THE SOFTWARE.
