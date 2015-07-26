---
layout: post
title: How ButterKnife actually works?
---

You all know ButterKnife: the brilliant annotation processing library to bind views and methods for Android.

http://jakewharton.github.io/butterknife/

But, how it works? Most people think it just add new methods to our classes saving us a quite a lot of boiler plate code. 

You may be surprised to discover that is not actually happening at all :)

First, you need a quick overview of how annotation processing works in java.

##Java Annotation Processing

Annotation processing is a tool build in javac for scanning and processing annotations at compile time.

You can define your own annotations and a processor to handle them.

  * Annotations are scanned and processed at **compile time**.

  * An Annotation Processor reads java code, process its annotations and **generate java code in response**. 
  * This java code will then be compiled again as a regular java class. 

  * An Annotation Processor **can not change** an existing input java class. Neither adding or modifiying methods.

##ButterKnife workflow

When you compile your Android project [ButterKnife Annotations Processor](https://github.com/JakeWharton/butterknife/blob/master/butterknife/src/main/java/butterknife/internal/ButterKnifeProcessor.java) `process` method is executed doing the following:

  * First, it scans all java classes looking for ButterKnife annotations: `@InjectView`, `@OnClick`, etc.
  
  * When it find a class with any of these annotations it creates a file called: `<className>$$ViewInjector.java`
  
  * This new ViewInjector class contains all the neccessary methods to handle annotation logic: `findViewById`, `view.setOnClickListener`, etc.
  
  * Finally, during execution, when we call `ButterKnife.inject(this)` each ViewInjector `inject` method is called.

##Java Compiler

At [OpenJDK](http://openjdk.java.net/groups/compiler/doc/compilation-overview/index.html) you can read an excellent overview of how Java compiler works. 

This chart summarizes very well the part that interests us:


##ButterKnife Example

For the same sample code you can find at https://github.com/JakeWharton/butterknife

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
    target.user = finder.castView(view, 2131361865, "field 'user'");
    view = finder.findRequiredView(source, 2131361868, "field 'pass'");
    target.pass = finder.castView(view, 2131361868, "field 'pass'");

    view = finder.findRequiredView(source, 2131361874, "field 'submit' and method 'submit'");
    view.setOnClickListener(
      new butterknife.internal.DebouncingOnClickListener() {
        @Override public void doClick(android.view.View p0) {
          target.submit();
        }
      });
  }

  @Override public void reset(T target) {
    target.user = null;
    target.pass = null;
  }
}
```
 
Then, during execution time, when we call `ButterKnife.bind(this);` what happens is:

  * ButterKnife calls `findViewBinderForClass(ExampleActivity.class)` finding `ExampleActivity$$ViewBinder.java`
  * `ExampleActivity$$ViewBinder.bind()` is executed, finding and casting views and setting them into `ExampleActivity.class` attributes, which are `public`
  * `onClickListeners` for views are setted as a wrapper to execute target defined method to handle clicks (annotated with `@OnClick`)

This is why annotated attributes and methods **must** be public: ButterKnife needs to be able to access them from a separate class.  
