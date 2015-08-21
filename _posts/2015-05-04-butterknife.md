---
layout: post
title: How ButterKnife actually works?
comments: true
twitter: false
---

You all know [ButterKnife](http://jakewharton.github.io/butterknife/): the brilliant annotation processing library to bind views and methods for Android by [@JakeWharton](https://twitter.com/JakeWharton)

But, *how does it work*? Most people think by just adding new methods to our classes, saving us a quite a lot of boiler plate code.

Well, it’s definitely saving us some code, but you may be surprised to discover that is **not changing our classes at all**.

Want to find out? First, you need a quick overview of how annotation processing works in java.

##Java Annotation Processing

> Annotation processing is a tool build in javac for scanning and processing annotations at compile time.

You can define your own annotations and a custom processor to handle them.

  * Annotations are scanned and processed at **compile time**.

  * An Annotation Processor reads java code, process its annotations and **generate java code in response**. 
  
  * Generated java code is then compiled again as a regular java class

  * An Annotation Processor **can not change** an existing input java class. Neither adding or modifiying methods.


###Java Compiler

At [OpenJDK][link2] you can read an excellent overview of how Java compiler works. 

This chart summarizes very well the part that interests us:

![java compiler][image-java-compiler]
*Java compiling process overview*

##ButterKnife workflow

When you compile your Android project [ButterKnife Annotations Processor][link1] `process` method is executed, doing the following:

  * First, it scans all java classes looking for ButterKnife annotations: `@InjectView`, `@OnClick`, etc.
  
  * When it find a class with any of these annotations it creates a file called: `<className>$$ViewInjector.java`
  
  * This new ViewInjector class contains all the neccessary methods to handle annotation logic: `findViewById`, `view.setOnClickListener`, etc.
  
  * Finally, during execution, when we call `ButterKnife.inject(this)` each ViewInjector `inject` method is called.




##Sample

For the sample code you can find at [https://github.com/JakeWharton/butterknife](https://github.com/JakeWharton/butterknife) this is what happens underneath:

![butterknife sample][image-view-bind]

> ExampleActivity.java

```java
class ExampleActivity extends Activity {
  @FindView(R.id.user) EditText username;
  @FindView(R.id.pass) EditText password;

  @OnClick(R.id.submit) void submit() {
    // TODO call server...
  }

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    ButterKnife.bind(this);
    // TODO Use fields...
  }
}
```

During compile time this java class will be generated:

> ExampleActivity$$ViewBinder.java

```java
public class ExampleActivity$$ViewBinder<T extends com.lgvalle.samples.ui.ExampleActivity> implements ViewBinder<T> {
  @Override public void bind(final Finder finder, final T target, Object source) {
    View view;
    view = finder.findRequiredView(source, 2131361865, "field 'user'");
    target.username = finder.castView(view, 2131361865, "field 'user'");
    view = finder.findRequiredView(source, 2131361868, "field 'pass'");
    target.password = finder.castView(view, 2131361868, "field 'pass'");

    view = finder.findRequiredView(source, 2131361874, "field 'submit' and method 'submit'");
    view.setOnClickListener(
      new butterknife.internal.DebouncingOnClickListener() {
        @Override public void doClick(android.view.View p0) {
          target.submit();
        }
      });
  }

  @Override public void reset(T target) {
    target.username = null;
    target.password = null;
  }
}
```
 
Then, during execution time, when we call `ButterKnife.bind(this);` what happens is:

  * ButterKnife calls `findViewBinderForClass(ExampleActivity.class)` finding `ExampleActivity$$ViewBinder.java`
  * `ExampleActivity$$ViewBinder.bind()` is executed, finding and casting views and setting them into `ExampleActivity.class` attributes, which are `public`
  * `onClickListeners` for views are setted up as a wrapper to execute target defined method to handle clicks (annotated with `@OnClick`)

This is why annotated attributes and methods **must** be public: ButterKnife needs to be able to **access them from a separate class**.  


##More info

If you want to know more about Java Annotation Processing, this three post help me out a lot when writing this post:

  * [Playing with Java annotation processing][ref1]
  * [Annotation Processing 101][ref2]
  * [Java Custom Annotations Example][ref3]

[image-java-compiler]: https://raw.githubusercontent.com/lgvalle/lgvalle.github.io/master/public/images/butterknife-java-compiler.png "How Java compiler works"
[image-view-bind]: https://raw.githubusercontent.com/lgvalle/lgvalle.github.io/master/public/images/butterknife-viewbind.png "View bind"
[link1]: https://github.com/JakeWharton/butterknife/blob/master/butterknife/src/main/java/butterknife/internal/ButterKnifeProcessor.java
[link2]: http://openjdk.java.net/groups/compiler/doc/compilation-overview/index.html
[ref1]: http://programmaticallyspeaking.com/playing-with-java-annotation-processing.html "Playing with Java annotation processing"
[ref2]: http://hannesdorfmann.com/annotation-processing/annotationprocessing101/ "Annotation Processing 101"
[ref3]: http://www.mkyong.com/java/java-custom-annotations-example/ "Java Custom Annotations Example"


<br/>
<a href="https://twitter.com/share" class="twitter-share-button" data-url="http://lgvalle.github.io/2015/05/04/butterknife/" data-via="lgvalle" data-size="large">Tweet</a>
