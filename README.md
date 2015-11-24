# Material design	

Material design was added to Android 5.0 the sdk 21 and above

###AppTheme and styles:
In the manifest we set the Application Theme
```xml
<application
  android:allowBackup="true"
  android:icon="@mipmap/ic_launcher"
  android:label="@string/app_name"
  android:theme="@style/AppTheme">
```
The Theme is defined in values/theme.xml
## RecyclerView and CardView
The activity_main.xml layout contains a **RecyclerView **and a Toolbar. The toolbar has an *android:transitionName* that sets the common element in a transition.
MainActivity onCreate()

### LayoutManager
We add a LayoutManager, that is responsible for positioning individual views in the RecyclerView and to establish the recycling policy. 
[http://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html](http://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html)
Actually it is a *StaggeredGridLayoutManager* that allows to have items in a list as well as in a grid and also switch between them.

### CardView
The view items for the RecyclerView are going to be **CardView**s. See the layout/row_places.xml file. Observe that we define the following attributes for the view:
 ```
 card_view:cardCornerRadius 
 card_view:cardElevation 
```
See that to the LinearLayout called mainHolder, we’ve added **android:background=?android:selectableItemBackground** that sets the ripple effect when the cards are touched. See next figure for an example of the effect.

![Ripple effect](https://cloud.githubusercontent.com/assets/1186025/11369025/3170af0e-92bd-11e5-92ec-2c32976a43e5.png)

### The Adapter for the RecyclerView

[http://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html](http://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html)

The adapter must implement the **Holder pattern **(it was optional in the ListView) that is useful for avoiding calling the findViewById for each single list item that is loaded in the RecyclerView.

The Adapter must override the following methods
```
onCreateViewHolder
onBindViewHolder
getItemCount
```
### Capturing the clicks on the RecyclerView items to go to the detail view

Unlike the ListView, the RecyclerView does not provide a OnItemClick interface and so we’ll need to implement one in the Adapter. We’ll do that in the following way:

* The ViewHolder will implement the View.OnClickListener interface to handle the click on the list/grid item

* The Adapter will define a OnItemClickListener interface with the method onItemClick(), and define a local attribute to store one object implementing the interface. It will also define the public method setOnItemClickListener() for the listener to be set.

* When handling the onClick() event from the View.OnClickListener it will call the onItemClick() method of the OnItemClickListener it has been set before.

* The MainActivity will create a OnClickItemListener and set it to the RecyclerAdapter 

## Switch between List and Grid views

The file menu/menu_main.xml contains an item with the id "action_toggle" when it is pressed we want our RecyclerView to switch between a list and a more compact grid. To do so we just need to change the **spanCount **of the **StaggedLayoutManager **we set to the RecyclerView in the MainActivity.   

**mStaggeredLayoutManager**.setSpanCount(2);

## Palette API: cool colors

We want the namePlaceHolder in our CardView to have as background color the one that predominates in the image of the card. To do so we need to dynamically determine the color analysing the image. We will use the BitmapFactory and the Palette API 

[http://developer.android.com/reference/android/graphics/BitmapFactory.html](http://developer.android.com/reference/android/graphics/BitmapFactory.html)

[http://developer.android.com/reference/android/support/v7/graphics/Palette.html](http://developer.android.com/reference/android/support/v7/graphics/Palette.html)

The Palette is generated asynchronously in a different thread. Once it’s been generated we ask for the muted color and set it to the image title background.

## Material design: Animations 

The aim of the animations is to give information to the user about the interface by giving more context to the elements shown and by explaining how the user can interact with the UI elements

### Ripple effect

The Material design defines a pattern called the Floating Action Button (FAB). We are going to implement it in the DetailActivity. See that the layout/activity_detail.xml file has a ImageButton with the android:background="@drawable/btn_background. This button is going to be our main activity action button. 

The background file, see /drawable/btn_background.xml, defines a ripple effect. The file defines a background color for the button, and a shape and color for the ripple effect.

### Reveal animation

We want our users to add notes of what they want to do in a given place. To do that the activity_detail.xml already has an EditText that is invisible by default (see the onCreate() at the DetailActivity). When the user presses the Action Button we want the EditText to appear with a cool animation. The animation is triggered in methods **revealEditText **and **hideEditText. **

```java
private void revealEditText(LinearLayout view) {
 int cx = view.getRight() - 30;
 int cy = view.getBottom() - 60;
 int finalRadius = Math.max(view.getWidth(), view.getHeight());
 Animator anim = ViewAnimationUtils.createCircularReveal(view, cx, cy, 0, finalRadius);
 view.setVisibility(View.VISIBLE);
 isEditTextVisible = true;
 anim.start();
}
```

We add a animation to the view that holds the edit text. This animation is performed on a separated thread and cannot be stopped and resumed, not even re-used.

[http://developer.android.com/reference/android/view/ViewAnimationUtils.html](http://developer.android.com/reference/android/view/ViewAnimationUtils.html)

See that we are creating a circular review animation that simulates that the animation is begun near the action button.
```java
private void hideEditText(final LinearLayout view) {
 int cx = view.getRight() - 30;
 int cy = view.getBottom() - 60;
 int initialRadius = view.getWidth();

 Animator anim = ViewAnimationUtils.*createCircularReveal*(view, cx, cy, initialRadius, 0);
 anim.addListener(**new **AnimatorListenerAdapter() {

   @Override
   public void onAnimationEnd(Animator animation) {
     super.onAnimationEnd(animation);
     view.setVisibility(View.INVISIBLE);
   }
 });

isEditTextVisible = false;
anim.start();
}
```
The hideEditText want’s to hide the edit text. So we creates the reverse animation and start it. Tought, in this case we want the EditText to be invisible when the animation ends. So we added and **AnimationListenerAdapter **that overrides the **onAnimationEnd **in order to make the view invisible again. 

[http://developer.android.com/reference/android/animation/AnimatorListenerAdapter.html](http://developer.android.com/reference/android/animation/AnimatorListenerAdapter.html)

### Animate the Action Button

When the button is pressed we want to animate it. Basically we want the + sign to morph to a checkmark sign. See that the DetailActivity sets the button background to icn_morp and to background_morph_reverse depending of its state.

The first file looks like this:
```xml
<?xml version="1.0" encoding="utf-8"?>
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
 android:drawable="@drawable/icn_add">

 <target
   android:name="sm_vertical_line"
   android:animation="@anim/path_morph"/>

 <target
   android:animation="@anim/path_morph_lg"
   android:name="lg_vertical_line"/>

 <target
   android:animation="@anim/fade_out"
   android:name="horizontal_line"/>

</animated-vector>
```
This file assigns a different animation to each line of the button’s plus sign. The animation morphes the sign to the final checkmark. See the following links for information about vector path and animations:

[http://www.w3.org/TR/SVG11/paths.html#PathData](http://www.w3.org/TR/SVG11/paths.html#PathData)

[http://developer.android.com/training/material/animations.html#AnimVector](http://developer.android.com/training/material/animations.html#AnimVector)


In the onClickListener of the action button we add the following code in order to animate the button when is pressed. Observe in the actual code that the animation is toggled depending on its state:
```java
mAddButton.setImageResource(R.drawable.icn_morp);
mAnimatable = (Animatable) (mAddButton).getDrawable();
mAnimatable.start();
```
### Activity Transitions

It may be useful to make an animation to transition between activities or fragments. This animations can reveal users information about the app usability. There are three predefined transitions: Explode, Slide and Fade. These transition can be set as exit or enter. In our example we use the Fade transition when entering to the detail transition. When going from MainActivity to DetailActivity the Fade animation will be applied as entering transition to all the visible Views of the DetailActivity. When the back button is pressed and the app goes from the DetaiActivit to MainActivity the Fade transition animation is played reversely.  

This is the code:
```java
Transition fade = new Fade();
fade.excludeTarget(android.R.id.navigationBarBackground, true);
fade.excludeTarget(android.R.id.statusBarBackground, true);
getWindow().setEnterTransition(fade);
```
### Activity Transitions with shared elements

Activity and fragment transitions with shared elements are useful to give context and show continuity between UI elements across different views.

Between the List View and the detail view we are going to transition the following elements:

* The place image

* The title 

* The background area of the title

* The toolbar

The elements we want to transition from one Activity to the other must share the same android:transitionName property in the layout files. In our case the files are row_places.xml and activity_detail.xml with two shared elements ImageView and LinearLayout (placeNameHolder) 
```xml
<ImageView
 android:id="@+id/placeImage"
 android:layout_width="match_parent"
 android:layout_height="220dp"
 android:scaleType="centerCrop"
 android:transitionName="tImage"/>

<LinearLayout
 android:id="@+id/placeNameHolder"
 android:layout_width="match_parent"
 android:layout_height="60dp"
 android:background="@color/primary_dark"
 android:transitionName="tNameHolder">
```
Now we need to make the scene transition animation when going from the main activity and the detail activity. To do so we need the following code in the onItemClickListener method in the MainActivity:
```java
Pair<View, String> imagePair = Pair.create((View) placeImage, "tImage");
Pair<View, String> holderPair = Pair.create((View) placeNameHolder, "tNameHolder");
Pair<View, String> navPair = Pair.create(navigationBar, Window.NAVIGATION_BAR_BACKGROUND_TRANSITION_NAME);
Pair<View, String> statusPair = Pair.create(statusBar, Window.STATUS_BAR_BACKGROUND_TRANSITION_NAME);
Pair<View, String> toolbarPair = Pair.create((View)toolbar, "tActionBar");

ActivityOptionsCompat options = ActivityOptionsCompat.makeSceneTransitionAnimation(MainActivity.this, imagePair, holderPair, navPair, statusPair, toolbarPair);
ActivityCompat.startActivity(MainActivity.this, transitionIntent, options.toBundle());
```
In code above we created a pair for each view element we want to transition. Each pair containing the view from where the transition begins and its transitionName. Then we make the scene transition animation with the context and all the pairs. Finally we start the DetailActivity as usual but adding the scene transition animation to the method’s parameters.

### Animating the FAB transition

We want the FAB button to add elements in the todo list to fade in and out. To do so we need to add the following code in the DetailActivity, onCreate()
```java
getWindow().getEnterTransition().addListener(**new **TransitionAdapter() {
 @Override
 public void onTransitionEnd(Transition transition) {
     mAddButton.animate().alpha(1.0f);
     getWindow().getEnterTransition().removeListener(**this**);
 }
});
```
The listener we added to the enter transition is triggered when the transition ends. At this moment the button will fade in going from alpha 0 to alpha 1 (from totally transparent to opaque). Note that the ImageButton in activity_datail.xml has the alpha set to 0.0 

## Interesting links

The example presented here was taken from: [http://www.raywenderlich.com/103367/material-design](http://www.raywenderlich.com/103367/material-design)

Google documentation [https://www.google.com/design/spec/material-design/introduction.html](https://www.google.com/design/spec/material-design/introduction.html)

Material Icons: [https://www.google.com/design/icons/](https://www.google.com/design/icons/)

Android assets studio: [https://romannurik.github.io/AndroidAssetStudio/](https://romannurik.github.io/AndroidAssetStudio/)

This is a very nice tutorial. More on animations: [https://github.com/lgvalle/Material-Animations](https://github.com/lgvalle/Material-Animations)

