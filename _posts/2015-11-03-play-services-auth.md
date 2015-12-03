---
layout: post
title: Google Plus authentication on Android without permissions
comments: true
twitter: false
---

You were probably using G+ to sign in into your app.

Maybe you were also requesting a [code to authenticate your backend](https://developers.google.com/identity/sign-in/android/backend-auth) server too.

And for this you were requesting these **two permissions**:

```xml
<!-- Google Plus login -->
<uses-permission android:name="android.permission.GET_ACCOUNTS" />
<uses-permission android:name="android.permission.USE_CREDENTIALS" />
```    

If that's your case I have good news: **you don't need them anymore**.

With [Google Play Services 8.3](http://android-developers.blogspot.co.uk/2015/11/whats-new-in-google-play-services-83.html) you can use `Auth.GoogleSignInApi` to authenticate your user (and your backend) without requesting access to her accounts or contacts.

Here's how to **migrate** from old system to the new one:

_(If you are starting over for the first time you may want to look here for the whole process: [Start Integrating Google Sign-In into Your Android App](https://developers.google.com/identity/sign-in/android/sign-in]))_


### 1. Add dependency

```groovy
dependencies {
        compile 'com.google.android.gms:play-services-auth:8.3.0'
    }
```    

### 2. Configure Google Sign-In and GoogleApiClient

```java
GoogleSignInOptions gso = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
        .requestEmail()
        .build();      
```

> If you also need to authenticate your server:

```
GoogleSignInOptions gso = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
                .requestServerAuthCode(YOUR_SERVER_CLIENT_ID))
                .build();
```                

### 3. Create GoogleApiClient

```java
 mGoogleApiClient = new GoogleApiClient.Builder(getActivity())
                .enableAutoManage(getActivity(), this)
                .addApi(Auth.GOOGLE_SIGN_IN_API, gso)
                .build();
```


### 4. Start Sign-In

```java
private void signIn() {
    Intent signInIntent = Auth.GoogleSignInApi.getSignInIntent(mGoogleApiClient);
    startActivityForResult(signInIntent, RC_SIGN_IN);
}
```

At this point a native dialog will prompt to ask the user to select an account.

### 5. Capture Activity Result    

```java
@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == RC_SIGN_IN && resultCode == Activity.RESULT_OK) {
        GoogleSignInResult result = Auth.GoogleSignInApi.getSignInResultFromIntent(data);
        handleSignInResult(result);
    }
}

private void handleSignInResult(GoogleSignInResult result) {
    if (result.isSuccess()) {
        // Signed in successfully
        GoogleSignInAccount acct = result.getSignInAccount();
        updateUI(acct.getDisplayName())
        
        // If you are also authenticating your server:
        String code = acct.getServerAuthCode();
        api.loginWithGoogle(code);
    }
}
```

All your previous G+ auth flow code can be safely thrown to the bin.