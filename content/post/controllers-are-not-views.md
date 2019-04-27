---
title: "Controllers Aren't Views"
date: 2019-04-25T21:49:35+10:00
blurb: "Many treat Conductor's Controller class as a view, but I think that has issues."
draft: false
---

I don't make it a [secret](https://chrishorner.codes/presentation/conductor-architecture-discussion/) that I'm a big fan
of [Conductor](https://github.com/bluelinelabs/Conductor). I feel it's a fantastic alternative to Android's `Fragment`
API when it comes to building single `Activity` apps. It provides developers with a simpler life cycle, easy back-stack
management, and clean separation of concerns for managing transitions between screens.

For a while now, I've noticed many developers seem compelled to hold references to `View` objects in their `Controller` classes. This seems to be a habit they've adopted from their days working with Fragments where it's common to hold references to sub-Views in member variables, often with [ButterKnife](https://jakewharton.github.io/butterknife/).

```kotlin
class SomeFragment : Fragment {
  @BindView(R.id.title) lateinit var title: TextView
  @BindView(R.id.subtitle) lateinit var subtitle: TextView
}
```

The Conductor documentation reaffirms that they should think about a Controller as a replacement for Fragment:

> Think of it as a lighter-weight and more predictable Fragment alternative with an easier to manage lifecycle.

However I feel this wasn't a great idea with Fragments, and it's still not a great idea with Controllers. My argument
being that Controllers and Fragments have different life cycles to the Views they house, and will likely outlive them.
If you navigate to another Controller via `router.pushController()`, the Controller you came from won't have its
instance destroyed, but its View will be.

This means that when returning to our original Controller, we'll be creating an entirely new View. Any previous bindings
we created will be for our now dead View and its sub-Views, creating a memory leak.

To get around this issue, some choose to nullify these references using something like ButterKnife's `unbind()` utility.

```kotlin
override fun onDestroyView() {
  unbinder.unbind()
}
```

For Fragments, some make use of Kotlin's
[synthetic imports](https://kotlinlang.org/docs/tutorials/android-plugin.html#importing-synthetic-properties). This
allows the developer to reference Views by their IDs, without having to perform any binding or `findViewById()` calls.

```kotlin
override fun onActivityCreated(savedInstanceState: Bundle?) {
  title.text = getTitleTextSomehow()
}
```

On the surface this seems great; we're writing less code and the default implementation should clear references for us
when the View goes out of scope. (Under the hood a `HashMap` is created to cache all the necessary `findViewById()`
calls).

However both of these approaches rely on you only attempting to access these references at the correct point in the Controller/Fragment's life cycle. Attempting to access them at the wrong time will result in a crash.

Because of this, I think it's worth embracing the fact that neither a Fragment nor a Controller should be considered a
View. I don't mean _View_ in the sense of Android's `View` class, but rather _View_ in your architecture's MV-Whatever.
(Whether that be Model-View-Presenter, Model-View-ViewModel, Model-View-Intent, or whatever).

I think Conductor's Controllers are excellent for:

 - Bridging data from your model to your view
 - Representing a node in your navigation graph

After that, it's worth letting your Android `View` classes be your _View_. If you follow this approach, you completely
circumvent the issue of accessing your sub-Views at the incorrect point in the life cycle. Conductor even gives you two
perfect callbacks for your binding and unbinding to occur.

```kotlin
class SomeController : Controller {

  override fun onAttach(view: View) {
    // Push state into your view here.

  }

  override fun onDetach(view: View) {
    // Stop pushing state into your view here.

  }
}
```

You've got a non-null, life-cycle-timing correct `View` reference to work with. How you push state into your View is
entirely up to you. I've found that creating some kind of _Presenter_ can be useful.

```kotlin
class SomePresenter(view: View) : Consumer<State> {

  private val title: TextView = view.findViewById(R.id.title)
  private val subtitle: TextView = view.findViewById(R.id.subtitle)

  override fun accept(state: State) {
    // Modify sub-views as new states are received.

  }
}
```

This lets your hold non-null, read only references to sub-Views that will get garbage collected at the correct time
(so long as you manage what's holding on to the `Presenter`). Here's an example using RxJava to be trendy:

```kotlin
class SomeController : Controller {

  private var stateDisposable = Disposables.disposed()

  override fun onAttach(view: View) {
    val presenter = SomePresenter(view)
    stateStream.subscribe(presenter)
  }

  override fun onDetach(view: View) {
    disposable.dispose()
  }
}
```

I know many will say that
[Lifecycle-Aware Components](https://developer.android.com/topic/libraries/architecture/lifecycle) and
[Data Binding](https://developer.android.com/topic/libraries/data-binding) already provide a solution for this problem,
however I feel they end up creating more mess in order to do so. That's probably the topic for another post, so I'll end
things here for now!
