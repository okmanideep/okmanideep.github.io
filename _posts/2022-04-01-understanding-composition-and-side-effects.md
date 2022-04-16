---
layout: post
cover: false
title: '[Draft] Understanding Composition and Side Effects'
description: 'Important aspect to any framework that relies on functions is to understand when they are invoked. Read along to get a crystal clear understanding'
date:   2022-04-01 18:00:00
tags: jetpack compose android
class: 'post-template'
subclass: 'post tag-android'
categories: 'okmanideep'
---

<style>
.insight {
    background: #3B4354;
    border-radius: 3px;
    padding: 12px;
}
.insight > :first-child {
    margin-top: 0;
}
.insight > :last-child {
    margin-bottom: 0;
}
</style>

With compose, we write a lot of functions and lambdas in our code. It is essential to know the order of execution of these functions, if and when are they executed.

Let's dive in. 

We are going to use the following `Composable` to find out the order of execution and understand in what order our code executes during runtime.

## 🛠 Setup
```kotlin
@Composable
private fun TrafficLight(
    lightEmoji: String,
    modifier: Modifier = Modifier
) {
    log("$lightEmoji Composition 1")
    LaunchedEffect(lightEmoji) {
        log("$lightEmoji LaunchedEffect 1")
    }
    SideEffect {
        log("$lightEmoji SideEffect 1")
    }
    DisposableEffect(lightEmoji) {
        log("$lightEmoji DisposableEffect 1")

        onDispose {
            log("$lightEmoji onDispose 1")
        }
    }

    Text(lightEmoji, fontSize = 120.sp, modifier = modifier)

    log("$lightEmoji Composition 2")
    LaunchedEffect(lightEmoji) {
        log("$lightEmoji LaunchedEffect 2")
    }
    SideEffect {
        log("$lightEmoji SideEffect 2")
    }
    DisposableEffect(lightEmoji) {
        log("$lightEmoji DisposableEffect 2")

        onDispose {
            log("$lightEmoji onDispose 2")
        }
    }
}
```
Just a bunch of logs around a `Text`

## Touch And Go
We are going to start by adding and removing this `TrafficLight` on touch as shown below

```kotlin
@Composable
fun TouchAndGo() {
    var isVisible by remember { mutableStateOf(false) }
    Box(
        contentAlignment = Alignment.Center,
        modifier = Modifier
            .fillMaxSize()
            .clickable {
                log("---Click---------------------")
                isVisible = !isVisible
            },
    ) {
        if (isVisible) {
            TrafficLight(lightEmoji = "🟢")
        }
    }
}
```
Initially we show nothing. On click, we show the green light 🟢.

<video width="320" height="320" autoplay muted loop>
  <source src="https://i.imgur.com/TCy72F9.mp4" type="video/mp4">
</video>

So these are the logs

```
---Click---------------------
🟢 Composition - 1
🟢 Composition - 2
🟢 DisposableEffect - 1
🟢 DisposableEffect - 2
🟢 SideEffect - 1
🟢 SideEffect - 2
🟢 LaunchedEffect - 1
🟢 LaunchedEffect - 2
---Click---------------------
🟢 onDispose - 2
🟢 onDispose - 1
```

<div class="insight">
<h3>✨ Insights</h3>

<p>When a <code>Composable</code> is added to the UI:</p>

<ul>
<li>Composition (our <code>@Composable</code> function) runs first</li>
<li>Then all the <code>DisposableEffect</code>s in the order present in the code</li>
<li>Then all the <code>SideEffect</code>s in the order present in the code</li>
<li>and then all the <code>LaunchedEffect</code>s in the order present in the code</li>
</ul>

<p>When a <code>Composable</code> is removed from the UI:</p>

<ul>
<li>All the <code>onDispose</code>s run in <strong>reverse order</strong> of their corresponding <code>DisposableEffect</code>s</li>
</ul>
</div>

---

## Stop And Go
Toggle between 🟢 and 🔴 on click

```kotlin
@Composable
fun StopAndGo() {
    var go by remember { mutableStateOf(true) }

    Box(
        contentAlignment = Alignment.Center,
        modifier = Modifier
            .fillMaxSize()
            .clickable {
                log("---Click---------------------")
                go = !go
            },
    ) {
        val light = if (go) "🟢" else "🔴"
        TrafficLight(lightEmoji = light)
    }
}
```


<video width="320" height="320" autoplay muted loop>
  <source src="https://i.imgur.com/pJWzmgh.mp4" type="video/mp4">
</video>

Let's look at the logs. For brevity, we removed multiple logs for Composition and Effects

```
🟢 Composition
🟢 DisposableEffect
🟢 SideEffect
🟢 LaunchedEffect
---Click---------------------
🔴 Composition
🟢 onDispose
🔴 DisposableEffect
🔴 SideEffect
🔴 LaunchedEffect
```
One might have expected this, but it is an important aspect to keep in mind.

<div class="insight">
<h3>✨ Insights</h3>
<ul>
<li>Composition of the <i>incoming composable</i>* runs before the <i>outgoing composable</i>* disposes (<code>onDispose</code>)</li>
<li><code>DisposableEffect</code> of the <i>incoming composable</i>* runs after the <i>outgoing composable</i>* disposes (<code>onDispose</code>)</li>
</ul>

</div>


> *There is no notion of incoming and outgoing composable _objects_ at runtime in the above example. I have chosen to word it that way since we are accustomed to object oriented thinking. Essentially at runtime, compose notices a `State` change, re-runs the corresponding code that is accessing the state. The re-run is what informs the runtime of which `DisposableEffect`s to dispose. We will come back to this notion of objects at runtime later

## Stop Fade Go
More often than not, we animate our changes. Let's look at the order of execution when we add animation to the above example

```kotlin{15-17}
@Composable
fun StopFadeGo() {
    var go by remember { mutableStateOf(true) }

    Box(
        contentAlignment = Alignment.Center,
        modifier = Modifier
            .fillMaxSize()
            .clickable {
                log("---Click---------------------")
                go = !go
            },
    ) {
        val light = if (go) "🟢" else "🔴"
        Crossfade(targetState = light) {
            TrafficLight(lightEmoji = it)
        }
    }
}
```

```
🟢 Composition
🟢 DisposableEffect
🟢 SideEffect
🟢 LaunchedEffect
---Click---------------------
🔴 Composition
🔴 DisposableEffect
🔴 SideEffect
🔴 LaunchedEffect
🟢 onDispose
```

<div class="insight">
<h3>✨ Insights</h3>

<p>When animated, the outgoing composable is disposed only after the animation is complete</p>
</div>

Might feel obvious in hindsight. But it is important to keep in mind that since the `DisposableEffect` of the incoming composable runs before the `onDispose` of the outgoing composable, if they are trying to attach to the same instance or acquire the same resource, it might create issues.

---
## Ready Set Go
🔴 Ready -> 🔴 Set -> 🟢 Go on click

```kotlin
@Composable
fun ReadySetGo() {
    var count by remember { mutableStateOf(1) }

    Box(
        contentAlignment = Alignment.Center,
        modifier = Modifier
            .fillMaxSize()
            .clickable {
                log("---Click---------------------")
                count++
            },
    ) {
        val step = count % 3
        val light = if (step == 0) "🟢" else "🔴"
        val message = when (step) {
            1 -> "Ready"
            2 -> "Set"
            0 -> "GO!"
            else -> "Uh Oh!"
        }
        Column(horizontalAlignment = Alignment.CenterHorizontally) {
            TrafficLight(lightEmoji = light)
            Spacer(modifier = Modifier.height(8.dp))
            Text(message, fontSize = 36.sp)
        }
    }
}
```

<video width="320" height="320" autoplay muted loop>
  <source src="https://i.imgur.com/dEqERF5.mp4" type="video/mp4">
</video>

```
🔴 Composition
🔴 DisposableEffect
🔴 SideEffect
🔴 LaunchedEffect
---Click---------------------
---Click---------------------
🟢 Composition
🔴 onDispose
🟢 DisposableEffect
🟢 SideEffect
🟢 LaunchedEffect
```

<div class="insight">
<h3>✨ Insights</h3>

<p>Composition and Effects are skipped when the inputs don't change!</p>
</div>

Just like [the documentation says](https://developer.android.com/jetpack/compose/lifecycle#skipping). But what does "inputs not changing" really mean? Let's find out.

Instead of passing in a `String`, let's create our own `class`

```kotlin{4-5}
private class Light(val emoji: String)
```
Update the `TrafficLight` and `ReadySetGo` to use `Light` instead of a `String`

```kotlin{15,23,32,40,50}
@Composable
fun ReadySetGoClass() {
    var count by remember { mutableStateOf(1) }

    Box(
        contentAlignment = Alignment.Center,
        modifier = Modifier
            .fillMaxSize()
            .clickable {
                log("---Click---------------------")
                count++
            },
    ) {
        val step = count % 3
        val light = if (step == 0) Light("🟢") else Light("🔴")
        val message = when (step) {
            1 -> "Ready"
            2 -> "Set"
            0 -> "GO!"
            else -> "Uh Oh!"
        }
        Column(horizontalAlignment = Alignment.CenterHorizontally) {
            TrafficLight(light)
            Spacer(modifier = Modifier.height(8.dp))
            Text(message, fontSize = 36.sp)
        }
    }
}

@Composable
private fun TrafficLight(
    light: Light,
    modifier: Modifier = Modifier
) {
    val lightEmoji = light.emoji
    log("$lightEmoji Composition")

    Text(lightEmoji, fontSize = 120.sp, modifier = modifier)

    DisposableEffect(light) {
        log("$lightEmoji DisposableEffect")

        onDispose {
            log("$lightEmoji onDispose")
        }
    }
    SideEffect {
        log("$lightEmoji SideEffect")
    }
    LaunchedEffect(light) {
        log("$lightEmoji LaunchedEffect")
    }
}
```

Here are the logs after the change

```
🔴 Composition
🔴 DisposableEffect
🔴 SideEffect
🔴 LaunchedEffect
---Click---------------------
🔴 Composition
🔴 onDispose
🔴 DisposableEffect
🔴 SideEffect
🔴 LaunchedEffect
---Click---------------------
🟢 Composition
🔴 onDispose
🟢 DisposableEffect
🟢 SideEffect
🟢 LaunchedEffect
```

Well what happened there!

Our `Light` doesn't implement `.equals()`, the default implementation returns true only if they are the same instances. But we are creating a new instance every time. So compose runtime sees these as different inputs.

Let's add a log to equals

```kotlin{4-5}
private class Light(val emoji: String) {
    override fun equals(other: Any?): Boolean {
        if (other is Light) {
            val result = super.equals(other)
            log("$emoji.equals(${other.emoji}) = $result")
            return result
        }
        return super.equals(other)
    }
}
```
Haven't changed the implementation yet. Just added a log.

The same logs as above but with `equals()`

```
🔴 Composition
🔴 DisposableEffect
🔴 SideEffect
🔴 LaunchedEffect
---Click---------------------
🔴.equals(🔴) = false
🔴 Composition
🔴.equals(🔴) = false
🔴.equals(🔴) = false
🔴 onDispose
🔴 DisposableEffect
🔴 SideEffect
🔴 LaunchedEffect
---Click---------------------
🔴.equals(🟢) = false
🟢 Composition
🔴.equals(🟢) = false
🔴.equals(🟢) = false
🔴 onDispose
🟢 DisposableEffect
🟢 SideEffect
🟢 LaunchedEffect
```

So compose runtime compared the inputs. It observed that they are different (`.equals()` returned `false`), so ran the composable with the new input. It then compared the inputs again, to see if it has to run the `DisposableEffect` and the `LaunchedEffect` and ran them again because it received `false`.

Let's implement `equals()`

```kotlin{4}
private class Light(val emoji: String) {
    override fun equals(other: Any?): Boolean {
        if (other is Light) {
            val result = emoji == other.emoji
            log("$emoji.equals(${other.emoji}) = $result")
            return result
        }
        return super.equals(other)
    }
}
```

```
🔴 Composition
🔴 DisposableEffect
🔴 SideEffect
🔴 LaunchedEffect
---Click---------------------
🔴.equals(🔴) = true
---Click---------------------
🔴.equals(🟢) = false
🟢 Composition
🔴.equals(🟢) = false
🔴.equals(🟢) = false
🔴 onDispose
🟢 DisposableEffect
🟢 SideEffect
🟢 LaunchedEffect
```
Back to normal.

Let's summarize all the insights

<div class="insight">
<h3>✨ Insights</h3>
<p>🟩 When a <code>Composable</code> is added to the UI:</p>

<ul>
<li>Composition (our <code>@Composable</code> function) runs first</li>
<li>Then all the <code>DisposableEffect</code>s in the order present in the code</li>
<li>Then all the <code>SideEffect</code>s in the order present in the code</li>
<li>and then all the <code>LaunchedEffect</code>s in the order present in the code</li>
</ul>

<p>🗑️  When a <code>Composable</code> is removed from the UI:</p>

<ul>
<li>All the <code>onDispose</code>s run in <strong>reverse order</strong> as present in the code</li>
</ul>

<p>🔀 When a composable is being replaced with another or recomposed with the new state:</p>
<ul>
<li>Composition of the <i>incoming composable</i>* runs before the <i>outgoing composable</i>* disposes (<code>onDispose</code>)</li>
<li><code>DisposableEffect</code> of the <i>incoming composable</i>* runs after the <i>outgoing composable</i>* disposes (<code>onDispose</code>)</li>
</ul>

<p>💫 When animated, the outgoing composable is disposed only after the animation is complete. Hence the incoming composable's <code>DisposableEffect</code>s run before the the outgoing composable disposes.</p>

<p>🚫 Composition and Effects are skipped when the inputs don't change. Inputs are compared using their <code>equals()</code> method</p>
</div>

