---
layout: post
title: Material Design Animations & Transitions
comments: true
twitter: true
---

[Android Transition Framework][transition-framework] can be used for **three** main things:

1. Animate activity layout content when transitioning from one activity to another.
2. Animate shared elements (Hero views) in transitions between activities.
3. Animate view changes within same activity.


## 1. Transitions between Activities

Animate existing activity layout **content**

![A Start B][transition_a_to_b]

When transitioning from `Activity A` to `Activity B` content layout is animated according to defined transition. There are three predefined transitions available on `android.transition.Transition` you can use: **Explode**, **Slide** and **Fade**. 
All these transitions track changes to the visibility of target views in activity layout and animate those views to follow transition rules.

[Explode][explode_link] | [Slide][slide_link] | [Fade][fade_link]
--- | --- | ---
![transition_explode] | ![transition_slide] | ![transition_fade]






You can define these transitions **declarative** using XML or **programatically**. 

### FADE SAMPLE
#### Declarative XML
Transitions are defined on XML files in `res/transition`

> res/transition/activity_fade.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<fade xmlns:android="http://schemas.android.com/apk/res/
    android:duration="1000"/>

```

> res/transition/activity_slide.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<slide xmlns:android="http://schemas.android.com/apk/res/
    android:duration="1000"/>

```

To use these transitions you need to inflate them using `TransitionInflater`

> MainActivity.java
 
```java
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_transition);
        setupWindowAnimations();
    }

    private void setupWindowAnimations() {
        Slide slide = TransitionInflater.from(this).inflateTransition(R.transition.activity_slide);
        getWindow().setExitTransition(slide);
    }

```

> TransitionActivity.java
 
```java
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_transition);
        setupWindowAnimations();
    }

    private void setupWindowAnimations() {
        Fade fade = TransitionInflater.from(this).inflateTransition(R.transition.activity_fade);
        getWindow().setEnterTransition(fade);
    }

```

#### Programatically 

> MainActivity.java
 
```java
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_transition);
        setupWindowAnimations();
    }

    private void setupWindowAnimations() {
        Slide slide = new Slide();
        slide.setDuration(1000);
        getWindow().setExitTransition(slide);
    }

```

> TransitionActivity.java
 
```java
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_transition);
        setupWindowAnimations();
    }

    private void setupWindowAnimations() {
        Fade fade = new Fade();
        fade.setDuration(1000);
        getWindow().setEnterTransition(fade);
    }

```

#### Any of those produce this result:

![transition_fade]


### What is happening step by step:

1. Activity A starts Activity B

2. Transition Framework finds A Exit Transition (slide) and apply it to all visible views.
3. Transition Framework finds B Enter Transition (fade) and apply it to all visible views.
4. **On Back Pressed** Transition Framework executes Enter and Exit reverse animations respectively (If we had defined output `returnTransition` and `reenterTransition`, these have been executed instead) 

### ReturnTransition & ReenterTransition

Return and Reenter Transitions are the reverse animations for Enter and Exit respectively.

  * EnterTransition <--> ReturnTransition
  * ExitTransition <--> ReenterTransition

If Return or Reenter are not defined, Android will execute a reversed version of Enter and Exit Transitions. But if you do define them, you can have different transitions for entering and exiting an activity.

![b back a][transition_b_to_a]

We can modify previous Fade sample and define a `ReturnTransition` for `TransitionActivity`, in this case, a Slide transition. This way, when returning from B to A, instead of seeing a Fade out (reversed Enter Transition) we will see a Slide out transition
 
> TransitionActivity.java
 
```java
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_transition);
        setupWindowAnimations();
    }

    private void setupWindowAnimations() {
        Fade fade = new Fade();
        fade.setDuration(1000);
        getWindow().setEnterTransition(fade);
        
        Slide slide = new Slide();
        fade.setDuration(1000);
        getWindow().setReturnTransition(slide);        
    }

```
Compare both cases

Without Return Transition | With Return Transition 
--- | --- 
![transition_fade] | ![transition_fade2] 


## 2. Shared elements between Activities

The idea behind this is having two different views in two different layouts and link them somehow with an animation.

Transition framework will then do _whatever animations it consider necessary_ to show the user a transition from one view to another.

Keep this always in mind: the view **is not really moving** from one layout to another. They are two independent views.


![A Start B with shared][shared_element]


### a) Enable Window Content Transition

This is something you need to setup once on your app `styles.xml`.

> values/styles.xml

```xml
<style name="MaterialAnimations" parent="@style/Theme.AppCompat.Light.NoActionBar">
    ...
    <item name="android:windowContentTransitions">true</item
    ...
</style>
```

Here you can also specity default enter, exit and shared element transitions for the whole app if you want

```
<style name="MaterialAnimations" parent="@style/Theme.AppCompat.Light.NoActionBar">
    ...
    <!-- specify enter and exit transitions -->
    <item name="android:windowEnterTransition">@transition/explode</item>
    <item name="android:windowExitTransition">@transition/explode</item>

    <!-- specify shared element transitions -->
    <item name="android:windowSharedElementEnterTransition">@transition/changebounds</item>
    <item name="android:windowSharedElementExitTransition">@transition/changebounds</item>
    ...
</style>
```



### b) Define a common transition name

To make the trick you need to give both, origin and target views, the same **`android:transitionName`**. They may have different ids or properties, but `transitionName` must be the same.

> layout/activity_a.xml

```xml
<ImageView
        android:id="@+id/small_blue_icon"
        style="@style/MaterialAnimations.Icon.Small"
        android:src="@drawable/circle"
        android:transitionName="@string/blue_name" />
```

> layout/activity_b.xml

```xml
<ImageView
        android:id="@+id/big_blue_icon"
        style="@style/MaterialAnimations.Icon.Big"
        android:src="@drawable/circle"
        android:transitionName="@string/blue_name" />
```

### c) Start an activity with a shared element 

Use the ActivityOptions.makeSceneTransitionAnimation() method to define shared element origin view and transition name.

> MainActivity.java

```java

squareBlue.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Intent i = new Intent(MainActivity.this, SharedElementActivity.class);

        View sharedView = blueIconImageView;
        String transitionName = getString(R.string.blue_name);

        ActivityOptions transitionActivityOptions = ActivityOptions.makeSceneTransitionAnimation(MainActivity.this, sharedView, transitionName);
        startActivity(i, transitionActivityOptions.toBundle());
    }
});

```


Just that code will produce this beautiful transition animation:

![a to b with shared element](https://raw.githubusercontent.com/lgvalle/Material-Animations/master/screenshots/transition-shared-elements.gif)

As you can see, Transition framework is creating and executing an animation to create the illusion that the view is moving and changing shape.

To proof the blue square view is not really _moving_ we can do this quick exercise: change transitioName in DetailsActivity from Big Blue Square to the Title Text above it.

```xml
<TextView
        android:layout_width="wrap_content"
        android:text="Activity Detail 2"
        style="@style/Base.TextAppearance.AppCompat.Large"
        android:layout_centerHorizontal="true"
        android:transitionName="@string/square_blue_name"
        android:layout_above="@+id/big_square_blue"
        android:layout_height="wrap_content" />
```

If we now execute the app we have the same behaviour but targeting a different view:

![a to b with shared element - 2](https://raw.githubusercontent.com/lgvalle/Material-Animations/master/screenshots/transition-shared-elements2.gif)        


## 3. Animate view layout elements

Transition framework can also be used to animate element changes within current activity layout. 

Transitions happen between scenes. An scene defines a static state of our UI. You can do complex things regarding _scenes_ but I want to keep this example as **simple as possible**. 

If you want to know more about scenes I recomend you check [this video by Chet Hasse] 
(https://www.youtube.com/watch?v=S3H7nJ4QaD8)

In this example I'm going to use the easier way to animate layout changes inside an Activity layout:

```java
TransitionManager.beginDelayedTransition(sceneRoot);
```

With just this line of code we are telling the framework we are going to perform some UI changes that it will need to animate.

After that we made the changes on our UI elements:

```java
setViewWidth(squareRed, 500);
setViewWidth(squareBlue, 500);
setViewWidth(squareGreen, 500);
setViewWidth(squareYellow, 500);
```

This will change those views width attribute to make it larger. That will trigger a `layoutMeasure`. At that point the Transition framework will record start and ending values and will create an animation to transition from one to another.

```java
 squareGreen.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                TransitionManager.beginDelayedTransition(sceneRoot);
                setViewWidth(squareRed, 500);
                setViewWidth(squareBlue, 500);
                setViewWidth(squareGreen, 500);
                setViewWidth(squareYellow, 500);
            }
        });
    }

    private void setViewWidth(View view, int x) {
        ViewGroup.LayoutParams params = view.getLayoutParams();
        params.width = x;
        view.setLayoutParams(params);
    }
```
    
![a to b with shared element - 2](https://raw.githubusercontent.com/lgvalle/Material-Animations/master/screenshots/scene-transition.gif)   


## 4. (Bonus) Shared elements + Circular Reveal
Circular Reveal is just an animation to show or hide a group of UI elements. It is available since API 21 in `ViewAnimationUtils` class. 

In this example I'm going to demostrate how can you make use of Shared Element Transition and Circular Reveal Animation to smoothly switch UI context.

![shared+circularreveal](https://raw.githubusercontent.com/lgvalle/Material-Animations/master/screenshots/example3.gif)   

### Enter Animation
What is happening step by step is:

* Shared orange box is transitioning from `MainActivity` to `DetailsActivity`.
* `DetailsActivity` background viewgroup visibility starts as `INVISIBLE`.

```xml
 <RelativeLayout
        android:layout_width="match_parent"
        android:id="@+id/backgroundViewGroup"
        android:visibility="invisible"
        ...
```        
* After `SharedElementEnterTransition` ends a `CircularReveal` animation takes place making the background viewgroup visible.

```java
        Transition enterTransition = getWindow().getSharedElementEnterTransition();
        enterTransition.addListener(new Transition.TransitionListener() {
            @Override
            public void onTransitionStart(Transition transition) {}

            @Override
            public void onTransitionEnd(Transition transition) {
                animateRevealShow(bgViewGroup);
            }

            @Override
            public void onTransitionCancel(Transition transition) {}

            @Override
            public void onTransitionPause(Transition transition) {}

            @Override
            public void onTransitionResume(Transition transition) {}
        });
```

### Exit Animation
On exit transition steps are:

* `SharedElementReturnTransition` is delayed 1 second.

```java
        Transition sharedElementReturnTransition = getWindow().getSharedElementReturnTransition();
        sharedElementReturnTransition.setStartDelay(ANIM_DURATION);
```

* `ReturnTransition` duration is setted to 1 second. Have in mind this are two **different** transitions.

```java
        Transition returnTransition = getWindow().getReturnTransition();
        returnTransition.setDuration(ANIM_DURATION);
```


* On `ReturnTransition` start a `CircularReveal` animation takes place hiding the background viewgroup.

```java
        returnTransition.addListener(new Transition.TransitionListener() {
            @Override
            public void onTransitionStart(Transition transition) {
                animateRevealHide(bgViewGroup);
            }

            @Override
            public void onTransitionEnd(Transition transition) {}

            @Override
            public void onTransitionCancel(Transition transition) {}

            @Override
            public void onTransitionPause(Transition transition) {}

            @Override
            public void onTransitionResume(Transition transition) {}
        });
```


* After 1 second, `CircularReveal` has finished and `SharedElementReturnTransition` gets executed producing orange box animation.


# Sample source code

**[https://github.com/lgvalle/Material-Animations](https://github.com/lgvalle/Material-Animations/)**


# More information

  * Alex Lockwood posts about transitions in Lollipop. A great in deep into this topic: [http://www.androiddesignpatterns.com/2014/12/activity-fragment-transitions-in-android-lollipop-part1.html](http://www.androiddesignpatterns.com/2014/12/activity-fragment-transitions-in-android-lollipop-part1.html)
  * Very complete repository with examples by Saul Molinero: [https://github.com/saulmm/Android-Material-Examples](https://github.com/saulmm/Android-Material-Examples)
  * Chet Hasse video explaining Transition framework: [https://www.youtube.com/watch?v=S3H7nJ4QaD8](https://www.youtube.com/watch?v=S3H7nJ4QaD8)



[transition-framework]: https://developer.android.com/training/transitions/overview.html

[explode_link]: https://developer.android.com/reference/android/transition/Explode.html
[fade_link]: https://developer.android.com/reference/android/transition/Fade.html
[slide_link]: https://developer.android.com/reference/android/transition/Slide.html

[transition_explode]: https://raw.githubusercontent.com/lgvalle/Material-Animations/master/screenshots/transition_explode.gif
[transition_slide]: https://raw.githubusercontent.com/lgvalle/Material-Animations/master/screenshots/transition_slide.gif
[transition_fade]: https://raw.githubusercontent.com/lgvalle/Material-Animations/dev/screenshots/transition_fade.gif
[transition_fade2]: https://raw.githubusercontent.com/lgvalle/Material-Animations/dev/screenshots/transition_fade2.gif
[transition_a_to_b]: https://raw.githubusercontent.com/lgvalle/Material-Animations/dev/screenshots/transition_A_to_B.png
[transition_b_to_a]: https://raw.githubusercontent.com/lgvalle/Material-Animations/dev/screenshots/transition_B_to_A.png

[shared_element]: https://raw.githubusercontent.com/lgvalle/Material-Animations/dev/screenshots/shared_element.png
