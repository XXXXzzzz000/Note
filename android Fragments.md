# Fragments

## In this document

1. [Design Philosophy](https://developer.android.com/guide/components/fragments.html#Design)
2. Creating a Fragment
   1. [Adding a user interface](https://developer.android.com/guide/components/fragments.html#UI)
   2. [Adding a fragment to an activity](https://developer.android.com/guide/components/fragments.html#Adding)
3. [Managing Fragments](https://developer.android.com/guide/components/fragments.html#Managing)
4. [Performing Fragment Transactions](https://developer.android.com/guide/components/fragments.html#Transactions)
5. Communicating with the Activity
   1. [Creating event callbacks to the activity](https://developer.android.com/guide/components/fragments.html#EventCallbacks)
   2. [Adding items to the App Bar](https://developer.android.com/guide/components/fragments.html#ActionBar)
6. Handling the Fragment Lifecycle
   1. [Coordinating with the activity lifecycle](https://developer.android.com/guide/components/fragments.html#CoordinatingWithActivity)
7. [Example](https://developer.android.com/guide/components/fragments.html#Example)

## Key classes

1. `Fragment`
2. `FragmentManager`
3. `FragmentTransaction`

## See also

1. [Building a Dynamic UI with Fragments](https://developer.android.com/training/basics/fragments/index.html)
2. [Handling Lifecycles with Lifecycle-Aware Components](https://developer.android.com/topic/libraries/architecture/lifecycle.html)
3. [Guide to App Architecture](https://developer.android.com/topic/libraries/architecture/guide.html)
4. [Supporting Tablets and Handsets](https://developer.android.com/guide/practices/tablets-and-handsets.html)

A `Fragment` represents a behavior or a portion of user interface in an `Activity`. You can combine multiple fragments in a single activity to build a multi-pane UI and reuse a fragment in multiple activities. You can think of a fragment as a modular section of an activity, which has its own lifecycle, receives its own input events, and which you can add or remove while the activity is running (sort of like a "sub activity" that you can reuse in different activities).

A fragment must always be embedded in an activity and the fragment's lifecycle is directly affected by the host activity's lifecycle. For example, when the activity is paused, so are all fragments in it, and when the activity is destroyed, so are all fragments. However, while an activity is running (it is in the *resumed* [lifecycle state](https://developer.android.com/guide/components/activities.html#Lifecycle)), you can manipulate each fragment independently, such as add or remove them. When you perform such a fragment transaction, you can also add it to a back stack that's managed by the activity—each back stack entry in the activity is a record of the fragment transaction that occurred. The back stack allows the user to reverse a fragment transaction (navigate backwards), by pressing the *Back* button.

When you add a fragment as a part of your activity layout, it lives in a `ViewGroup` inside the activity's view hierarchy and the fragment defines its own view layout. You can insert a fragment into your activity layout by declaring the fragment in the activity's layout file, as a `<fragment>` element, or from your application code by adding it to an existing `ViewGroup`. However, a fragment is not required to be a part of the activity layout; you may also use a fragment without its own UI as an invisible worker for the activity.

This document describes how to build your application to use fragments, including how fragments can maintain their state when added to the activity's back stack, share events with the activity and other fragments in the activity, contribute to the activity's action bar, and more.

For information about handling lifecycles, including guidance about best practices, see [Handling Lifecycles with Lifecycle-Aware Components](https://developer.android.com/topic/libraries/architecture/lifecycle.html). For broader guidance about app architecture, see [Guide to App Architecture](https://developer.android.com/topic/libraries/architecture/guide.html).

## Design Philosophy

------

Android introduced fragments in Android 3.0 (API level 11), primarily to support more dynamic and flexible UI designs on large screens, such as tablets. Because a tablet's screen is much larger than that of a handset, there's more room to combine and interchange UI components. Fragments allow such designs without the need for you to manage complex changes to the view hierarchy. By dividing the layout of an activity into fragments, you become able to modify the activity's appearance at runtime and preserve those changes in a back stack that's managed by the activity. They are now widely available through the [fragment support library](https://developer.android.com/topic/libraries/support-library/packages.html#v4-fragment).

For example, a news application can use one fragment to show a list of articles on the left and another fragment to display an article on the right—both fragments appear in one activity, side by side, and each fragment has its own set of lifecycle callback methods and handle their own user input events. Thus, instead of using one activity to select an article and another activity to read the article, the user can select an article and read it all within the same activity, as illustrated in the tablet layout in figure 1.

You should design each fragment as a modular and reusable activity component. That is, because each fragment defines its own layout and its own behavior with its own lifecycle callbacks, you can include one fragment in multiple activities, so you should design for reuse and avoid directly manipulating one fragment from another fragment. This is especially important because a modular fragment allows you to change your fragment combinations for different screen sizes. When designing your application to support both tablets and handsets, you can reuse your fragments in different layout configurations to optimize the user experience based on the available screen space. For example, on a handset, it might be necessary to separate fragments to provide a single-pane UI when more than one cannot fit within the same activity.

**Figure 1.** An example of how two UI modules defined by fragments can be combined into one activity for a tablet design, but separated for a handset design.

For example—to continue with the news application example—the application can embed two fragments in *Activity A*, when running on a tablet-sized device. However, on a handset-sized screen, there's not enough room for both fragments, so *Activity A* includes only the fragment for the list of articles, and when the user selects an article, it starts *Activity B*, which includes the second fragment to read the article. Thus, the application supports both tablets and handsets by reusing fragments in different combinations, as illustrated in figure 1.

For more information about designing your application with different fragment combinations for different screen configurations, see the guide to [Supporting Tablets and Handsets](https://developer.android.com/guide/practices/tablets-and-handsets.html).

## Creating a Fragment

------

**Figure 2.** The lifecycle of a fragment (while its activity is running).

To create a fragment, you must create a subclass of `Fragment` (or an existing subclass of it). The `Fragment` class has code that looks a lot like an `Activity`. It contains callback methods similar to an activity, such as `onCreate()`, `onStart()`, `onPause()`, and `onStop()`. In fact, if you're converting an existing Android application to use fragments, you might simply move code from your activity's callback methods into the respective callback methods of your fragment.

Usually, you should implement at least the following lifecycle methods:

- `onCreate()`

  The system calls this when creating the fragment. Within your implementation, you should initialize essential components of the fragment that you want to retain when the fragment is paused or stopped, then resumed.

- `onCreateView()`

  The system calls this when it's time for the fragment to draw its user interface for the first time. To draw a UI for your fragment, you must return a `View` from this method that is the root of your fragment's layout. You can return null if the fragment does not provide a UI.

- `onPause()`

  The system calls this method as the first indication that the user is leaving the fragment (though it does not always mean the fragment is being destroyed). This is usually where you should commit any changes that should be persisted beyond the current user session (because the user might not come back).

Most applications should implement at least these three methods for every fragment, but there are several other callback methods you should also use to handle various stages of the fragment lifecycle. All the lifecycle callback methods are discussed in more detail in the section about [Handling the Fragment Lifecycle](https://developer.android.com/guide/components/fragments.html#Lifecycle).

There are also a few subclasses that you might want to extend, instead of the base `Fragment`class:

- `DialogFragment`

  Displays a floating dialog. Using this class to create a dialog is a good alternative to using the dialog helper methods in the `Activity` class, because you can incorporate a fragment dialog into the back stack of fragments managed by the activity, allowing the user to return to a dismissed fragment.

- `ListFragment`

  Displays a list of items that are managed by an adapter (such as a `SimpleCursorAdapter`), similar to `ListActivity`. It provides several methods for managing a list view, such as the `onListItemClick()` callback to handle click events.

- `PreferenceFragmentCompat`

  Displays a hierarchy of `Preference` objects as a list, similar to `PreferenceActivity`. This is useful when creating a "settings" activity for your application.

### Adding a user interface

A fragment is usually used as part of an activity's user interface and contributes its own layout to the activity.

To provide a layout for a fragment, you must implement the `onCreateView()` callback method, which the Android system calls when it's time for the fragment to draw its layout. Your implementation of this method must return a `View` that is the root of your fragment's layout.

**Note:** If your fragment is a subclass of `ListFragment`, the default implementation returns a `ListView` from `onCreateView()`, so you don't need to implement it.

To return a layout from `onCreateView()`, you can inflate it from a [layout resource](https://developer.android.com/guide/topics/resources/layout-resource.html) defined in XML. To help you do so, `onCreateView()` provides a`LayoutInflater` object.

For example, here's a subclass of `Fragment` that loads a layout from the `example_fragment.xml` file:

```
public static class ExampleFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.example_fragment, container, false);
    }
}
```

### Creating a layout

In the sample above, `R.layout.example_fragment` is a reference to a layout resource named `example_fragment.xml` saved in the application resources. For information about how to create a layout in XML, see the [User Interface](https://developer.android.com/guide/topics/ui/index.html) documentation.

The `container` parameter passed to `onCreateView()` is the parent `ViewGroup` (from the activity's layout) in which your fragment layout will be inserted. The `savedInstanceState` parameter is a `Bundle` that provides data about the previous instance of the fragment, if the fragment is being resumed (restoring state is discussed more in the section about [Handling the Fragment Lifecycle](https://developer.android.com/guide/components/fragments.html#Lifecycle)).

The `inflate()` method takes three arguments:

- The resource ID of the layout you want to inflate.
- The `ViewGroup` to be the parent of the inflated layout. Passing the `container` is important in order for the system to apply layout parameters to the root view of the inflated layout, specified by the parent view in which it's going.
- A boolean indicating whether the inflated layout should be attached to the `ViewGroup` (the second parameter) during inflation. (In this case, this is false because the system is already inserting the inflated layout into the `container`—passing true would create a redundant view group in the final layout.)

Now you've seen how to create a fragment that provides a layout. Next, you need to add the fragment to your activity.

### Adding a fragment to an activity

Usually, a fragment contributes a portion of UI to the host activity, which is embedded as a part of the activity's overall view hierarchy. There are two ways you can add a fragment to the activity layout:

- Declare the fragment inside the activity's layout file.

  In this case, you can specify layout properties for the fragment as if it were a view. For example, here's the layout file for an activity with two fragments:

  ```
  <?xml version="1.0" encoding="utf-8"?>
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:orientation="horizontal"
      android:layout_width="match_parent"
      android:layout_height="match_parent">
      <fragment android:name="com.example.news.ArticleListFragment"
              android:id="@+id/list"
              android:layout_weight="1"
              android:layout_width="0dp"
              android:layout_height="match_parent" />
      <fragment android:name="com.example.news.ArticleReaderFragment"
              android:id="@+id/viewer"
              android:layout_weight="2"
              android:layout_width="0dp"
              android:layout_height="match_parent" />
  </LinearLayout>
  ```

  The `android:name` attribute in the `<fragment>` specifies the `Fragment` class to instantiate in the layout.

  When the system creates this activity layout, it instantiates each fragment specified in the layout and calls the `onCreateView()` method for each one, to retrieve each fragment's layout. The system inserts the `View` returned by the fragment directly in place of the `<fragment>` element.

  **Note:** Each fragment requires a unique identifier that the system can use to restore the fragment if the activity is restarted (and which you can use to capture the fragment to perform transactions, such as remove it). There are two ways to provide an ID for a fragment:

  - Supply the `android:id` attribute with a unique ID.
  - Supply the `android:tag` attribute with a unique string.

- Or, programmatically add the fragment to an existing `ViewGroup`.

  At any time while your activity is running, you can add fragments to your activity layout. You simply need to specify a `ViewGroup` in which to place the fragment.

  To make fragment transactions in your activity (such as add, remove, or replace a fragment), you must use APIs from `FragmentTransaction`. You can get an instance of `FragmentTransaction` from your `FragmentActivity` like this:

  ```
  FragmentManager fragmentManager = getSupportFragmentManager();
  FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
  ```

  You can then add a fragment using the `add()` method, specifying the fragment to add and the view in which to insert it. For example:

  ```
  ExampleFragment fragment = new ExampleFragment();
  fragmentTransaction.add(R.id.fragment_container, fragment);
  fragmentTransaction.commit();
  ```

  The first argument passed to `add()` is the `ViewGroup` in which the fragment should be placed, specified by resource ID, and the second parameter is the fragment to add.

  Once you've made your changes with `FragmentTransaction`, you must call `commit()` for the changes to take effect.

#### Adding a fragment without a UI

The examples above show how to add a fragment to your activity in order to provide a UI. However, you can also use a fragment to provide a background behavior for the activity without presenting additional UI.

To add a fragment without a UI, add the fragment from the activity using `add(Fragment, String)` (supplying a unique string "tag" for the fragment, rather than a view ID). This adds the fragment, but, because it's not associated with a view in the activity layout, it does not receive a call to `onCreateView()`. So you don't need to implement that method.

Supplying a string tag for the fragment isn't strictly for non-UI fragments—you can also supply string tags to fragments that do have a UI—but if the fragment does not have a UI, then the string tag is the only way to identify it. If you want to get the fragment from the activity later, you need to use `findFragmentByTag()`.

For an example activity that uses a fragment as a background worker, without a UI, see the `FragmentRetainInstance.java` sample, which is included in the SDK samples (available through the Android SDK Manager) and located on your system as`<sdk_root>/APIDemos/app/src/main/java/com/example/android/apis/app/FragmentRetainInstance.java`.

## Managing Fragments

------

To manage the fragments in your activity, you need to use `FragmentManager`. To get it, call `getSupportFragmentManager()` from your activity.

Some things that you can do with `FragmentManager` include:

- Get fragments that exist in the activity, with `findFragmentById()` (for fragments that provide a UI in the activity layout) or `findFragmentByTag()` (for fragments that do or don't provide a UI).
- Pop fragments off the back stack, with `popBackStack()` (simulating a *Back* command by the user).
- Register a listener for changes to the back stack, with `addOnBackStackChangedListener()`.

For more information about these methods and others, refer to the `FragmentManager` class documentation.

As demonstrated in the previous section, you can also use `FragmentManager` to open a `FragmentTransaction`, which allows you to perform transactions, such as add and remove fragments.

## Performing Fragment Transactions

------

A great feature about using fragments in your activity is the ability to add, remove, replace, and perform other actions with them, in response to user interaction. Each set of changes that you commit to the activity is called a transaction and you can perform one using APIs in `FragmentTransaction`. You can also save each transaction to a back stack managed by the activity, allowing the user to navigate backward through the fragment changes (similar to navigating backward through activities).

You can acquire an instance of `FragmentTransaction` from the `FragmentManager` like this:

```
FragmentManager fragmentManager = getSupportFragmentManager();
FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
```

Each transaction is a set of changes that you want to perform at the same time. You can set up all the changes you want to perform for a given transaction using methods such as `add()`, `remove()`, and `replace()`. Then, to apply the transaction to the activity, you must call `commit()`.

Before you call `commit()`, however, you might want to call `addToBackStack()`, in order to add the transaction to a back stack of fragment transactions. This back stack is managed by the activity and allows the user to return to the previous fragment state, by pressing the *Back* button.

For example, here's how you can replace one fragment with another, and preserve the previous state in the back stack:

```
// Create new fragment and transaction
Fragment newFragment = new ExampleFragment();
FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();

// Replace whatever is in the fragment_container view with this fragment,
// and add the transaction to the back stack
transaction.replace(R.id.fragment_container, newFragment);
transaction.addToBackStack(null);

// Commit the transaction
transaction.commit();
```

In this example, `newFragment` replaces whatever fragment (if any) is currently in the layout container identified by the `R.id.fragment_container` ID. By calling `addToBackStack()`, the replace transaction is saved to the back stack so the user can reverse the transaction and bring back the previous fragment by pressing the *Back* button.

`FragmentActivity` will automatically retrieve fragments from the back stack via `onBackPressed()`.

If you add multiple changes to the transaction (such as another `add()` or `remove()`) and call `addToBackStack()`, then all changes applied before you call `commit()` are added to the back stack as a single transaction and the *Back* button will reverse them all together.

The order in which you add changes to a `FragmentTransaction` doesn't matter, except:

- You must call `commit()` last
- If you're adding multiple fragments to the same container, then the order in which you add them determines the order they appear in the view hierarchy

If you do not call `addToBackStack()` when you perform a transaction that removes a fragment, then that fragment is destroyed when the transaction is committed and the user cannot navigate back to it. Whereas, if you do call `addToBackStack()` when removing a fragment, then the fragment is *stopped*and will be resumed if the user navigates back.

**Tip:** For each fragment transaction, you can apply a transition animation, by calling `setTransition()` before you commit.

Calling `commit()` does not perform the transaction immediately. Rather, it schedules it to run on the activity's UI thread (the "main" thread) as soon as the thread is able to do so. If necessary, however, you may call `executePendingTransactions()` from your UI thread to immediately execute transactions submitted by `commit()`. Doing so is usually not necessary unless the transaction is a dependency for jobs in other threads.

**Caution:** You can commit a transaction using `commit()` only prior to the activity [saving its state](https://developer.android.com/guide/components/activities.html#SavingActivityState) (when the user leaves the activity). If you attempt to commit after that point, an exception will be thrown. This is because the state after the commit can be lost if the activity needs to be restored. For situations in which its okay that you lose the commit, use `commitAllowingStateLoss()`.

## Communicating with the Activity

------

Although a `Fragment` is implemented as an object that's independent from an `FragmentActivity` and can be used inside multiple activities, a given instance of a fragment is directly tied to the activity that contains it.

Specifically, the fragment can access the `FragmentActivity` instance with `getActivity()` and easily perform tasks such as find a view in the activity layout:

```
View listView = getActivity().findViewById(R.id.list);
```

Likewise, your activity can call methods in the fragment by acquiring a reference to the `Fragment` from `FragmentManager`, using `findFragmentById()` or `findFragmentByTag()`. For example:

```
ExampleFragment fragment = (ExampleFragment) getFragmentManager().findFragmentById(R.id.example_fragment);
```

### Creating event callbacks to the activity

In some cases, you might need a fragment to share events with the activity. A good way to do that is to define a callback interface inside the fragment and require that the host activity implement it. When the activity receives a callback through the interface, it can share the information with other fragments in the layout as necessary.

For example, if a news application has two fragments in an activity—one to show a list of articles (fragment A) and another to display an article (fragment B)—then fragment A must tell the activity when a list item is selected so that it can tell fragment B to display the article. In this case, the `OnArticleSelectedListener` interface is declared inside fragment A:

```
public static class FragmentA extends ListFragment {
    ...
    // Container Activity must implement this interface
    public interface OnArticleSelectedListener {
        public void onArticleSelected(Uri articleUri);
    }
    ...
}
```

Then the activity that hosts the fragment implements the `OnArticleSelectedListener` interface and overrides `onArticleSelected()` to notify fragment B of the event from fragment A. To ensure that the host activity implements this interface, fragment A's `onAttach()` callback method (which the system calls when adding the fragment to the activity) instantiates an instance of `OnArticleSelectedListener` by casting the `Activity` that is passed into `onAttach()`:

```
public static class FragmentA extends ListFragment {
    OnArticleSelectedListener mListener;
    ...
    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        try {
            mListener = (OnArticleSelectedListener) context;
        } catch (ClassCastException e) {
            throw new ClassCastException(context.toString() + " must implement OnArticleSelectedListener");
        }
    }
    ...
}
```

If the activity has not implemented the interface, then the fragment throws a `ClassCastException`. On success, the `mListener` member holds a reference to activity's implementation of `OnArticleSelectedListener`, so that fragment A can share events with the activity by calling methods defined by the `OnArticleSelectedListener` interface. For example, if fragment A is an extension of `ListFragment`, each time the user clicks a list item, the system calls `onListItemClick()` in the fragment, which then calls `onArticleSelected()` to share the event with the activity:

```
public static class FragmentA extends ListFragment {
    OnArticleSelectedListener mListener;
    ...
    @Override
    public void onListItemClick(ListView l, View v, int position, long id) {
        // Append the clicked item's row ID with the content provider Uri
        Uri noteUri = ContentUris.withAppendedId(ArticleColumns.CONTENT_URI, id);
        // Send the event and Uri to the host activity
        mListener.onArticleSelected(noteUri);
    }
    ...
}
```

The `id` parameter passed to `onListItemClick()` is the row ID of the clicked item, which the activity (or other fragment) uses to fetch the article from the application's `ContentProvider`.

More information about using a content provider is available in the [Content Providers](https://developer.android.com/guide/topics/providers/content-providers.html) document.

### Adding items to the App Bar