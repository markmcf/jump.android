# Android Social Sign-In Only Integration

This guide describes integrating Social Sign-In or Social Sharing with your app, but not implmenting Janrain’s User Registration features. For a description of integration steps
for all of JUMP see `Jump_Integration_Guide.md`.

## The Big Picture

1. Gather you configuration details.
2. Declare the library project dependency.
3. Declare the library required activities permissions and add to your `AndroidManifest.xml` file.
4. Add library required activities permissions and to your `AndroidManifest.xml` file.
5. Import and initialize the library.
6. Begin authentication by calling one of the two `show...Dialog methods`.
7. Receive your authentication token URL's response in `JREngageDelegate#jrAuthenticationDidReachTokenUrl()`

## Gather Configuration Details


1.  Sign in to the Janrain Dashboard – https://dashboard.janrain.com
2.  Click the Engage icon (Looks like talk balloons).
3.  You get a new page. In the lower left panel, under Widgets and SKDs, click Android.
4.  On the new page, near the top, click Sign-In Providers. Then go to the relevant provider guide in the Provider Setup Guide and follow the instructions.
5.  When you finish configuring all of the Identity Providers that you want, click Save, then click Next.
6.  On the next page, you see a list of providers on the left. Double click the ones you want to appear in your app.
7.  When you finish, click Save.

    Retrieve your 20-character Engage application ID from the Dashboard. (From the Provider page of the dashboard, click the name of your application in the bread crumbs at the top of the page. Go to the Settings panel, and click the pencil icon. Your ID and secrets are on the right.)


## Declare the Library Dependency

We recommend that you use IntelliJ or Android Studio, because it’s easy. In either case, import the JUMP module and add a module dependency to it.

If you are using Eclipse, see the Eclipse_Import_Guide.md. 

If you are using the ant build tool chain the library just use
`android update project -p path/to/your/prj -l path/to/jump.android/Jump`.

## Declare the JUMP Activities

Ensure the declaration of the `android.permission.INTERNET` permission in your `<uses-permission>` element,
and copy from `.../Jump/AndroidManifest.xml`, adding the following two `<activity>` XML elements, and to your
project's `AndroidManifest.xml` file:

    <manifest xmlns:android="http://schemas.android.com/apk/res/android" ... >

      <uses-permission android:name="android.permission.INTERNET" />
      <uses-sdk android:minSdkVersion="4" android:targetSdkVersion="13" />

      ...

      <application ... >

      ...


        <!-- The following activities are for the Janrain Engage for Android library -->
        <!-- This activity must have a dialog theme such as Theme.Holo.Dialog, or
            Theme.Dialog, etc.

            Using android:theme="@style/Theme.Janrain.Dialog" will result in Theme.Dialog on API 4-10 and
            Theme.Holo.DialogWhenLarge.NoActionBar on API 11+
        -->
        <activity
            android:name="com.janrain.android.engage.ui.JRFragmentHostActivity"
            android:configChanges="orientation|screenSize"
            android:theme="@style/Theme.Janrain.Dialog.Light"
            android:windowSoftInputMode="adjustResize|stateHidden"
            />

        <!-- This activity must have a normal (non-dialog) theme such as Theme, Theme.Light, Theme.Holo, etc.

            Using android:theme="@style/Theme.Janrain" or "@style/Theme.Janrain.Light" will result in
            Theme (or Theme.Light) on API 4-10 and
            Theme.Holo (or Theme.Holo.Light) on API 11+
        -->
        <activity
            android:name="com.janrain.android.engage.ui.JRFragmentHostActivity$Fullscreen"
            android:configChanges="orientation|screenSize"
            android:theme="@style/Theme.Janrain.Light"
            android:windowSoftInputMode="adjustResize|stateHidden"
            />

      ...

      </application>

    </manifest>

Note: If you wish to target a version of Android lower than 13 (which is 3.2) you *can*. To do so, change the
`android:targetSdkVersion`, to your desired deployment target. _You must still build against API 13+
even when targeting a lower API level._ The build SDK used when compiling your project is defined by your
project's `local.properties`. Use `android list target` to get a list of targets available in your installation of
the Android SDK. Use `android update project -p path/to/project -t target_name_or_target_installation_id` to
update the build target SDK for your project. (This does *not* affect your project's
`minSdkVersion` or `targetSdkVersion`.)

## Import and Initialize

Import the following classes:

    import com.janrain.android.engage.JREngage;
    import com.janrain.android.engage.JREngageDelegate;
    import com.janrain.android.engage.JREngageError;
    import com.janrain.android.engage.net.async.HttpResponseHeaders;
    import com.janrain.android.engage.types.JRActivityObject;
    import com.janrain.android.engage.types.JRDictionary;

Interaction begins by calling the `JREngage.initInstance` method, which returns the `JREngage` object:

    private static final String ENGAGE_APP_ID = "";
    private static final String ENGAGE_TOKEN_URL = "";
    private JREngage mEngage;
    private JREngageDelegate mEngageDelegate = ...;

    ...

    mEngage = JREngage.initInstance(this.getApplicationContext(), ENGAGE_APP_ID, ENGAGE_TOKEN_URL, this);

The [initInstance](http://janrain.github.com/engage.android/docs/html/classcom_1_1janrain_1_1android_1_1engage_1_1_j_r_engage.html#a469d808d2464c065bc16dedec7a2cc23)
takes four arguments, `context`, `appId`, `tokenUrl`, and `delegate`:

- `context` — Your Android application Context.
- `appId` — Your Engage application ID (found on the Engage Dashboard).
- `tokenUrl` — Your web server's authentication token URL.
- `delegate` — An implementation of the `JREngageDelegate` interface through which you will receive callbacks
  from the library.

### Choosing your (Social Sign-In) Engage Delegate Class

Select a class you will use to receive callbacks from the Engage library (Social Sign-In, Social Sharing, or both). This is called your Engage
delegate. The delegate should be a singleton object. A good place to start is your app's Android Application
class, if you have one. Activities which are always at the root of the back stack can be acceptable choices.
Avoid using Activities which will be short lived.

## Social Sign-In

An Engage authentication is meaningful in the context of authenticating your mobile app *to* something.
If you are unsure of what your users should be authenticating to, then Janrain User Registration may be a great choice.
(User Registration signs authenticating users into your copy of their social profile, affording your mobile app a
place to store and retrieve data from, and a pier from which to build out a service.)

Once the `JREngage` object has been initialized, start authentication by calling
[showAuthenticationDialog](http://janrain.github.com/engage.android/docs/html/classcom_1_1janrain_1_1android_1_1engage_1_1_j_r_engage.html#a0de1aa16e951a1b62e2ef459b1596e83)
method:

    mEngage.showAuthenticationDialog();

You will receive your authentication token URL's response in the `jrAuthenticationDidReachTokenUrl` method.
When received you will have access to the body of the response, as well as the headers, which frequently
contain session cookies which you should use to coordinate the app's session with your web server. Parsing your
authentication token URL's response for session establishing information, or retrieving session cookies from
the header, is your app's responsibility.

For guidance implementing your web-server's authentication token URL, see Authentication-Token-URL.md.

## Social Sharing

If you want to share an activity, first create an instance of the
[`JRActivityObject`](http://janrain.github.com/engage.android/docs/html/classcom_1_1janrain_1_1android_1_1engage_1_1types_1_1_j_r_activity_object.html):

    String activityText = "added JREngage to her Android application!";
    String activityLink = "http://janrain.com";

    JRActivityObject jrActivity = new JRActivityObject(activityText, activityLink);

Populate the new object with information about the activity being shared by. Here, we add a Facebook "action link:"

    activityObject.addActionLink(new JRActionLink("Download the Quick Share demo!",
          "https://market.android.com/details?id=com.janrain.android.quickshare");

Then pass the activity to the
[`showSocialPublishingDialogWithActivity`](http://janrain.github.com/engage.android/docs/html/classcom_1_1janrain_1_1android_1_1engage_1_1_j_r_engage.html#aef1ecf0e43afeed0eb0a779c67eff285)
method:

    mEngage.showSocialPublishingDialogWithActivity(jrActivity);

## FAQ, Troubleshooting, Getting Help

The Android and iOS native library community forums are available here:
https://community.janrain.com/forums/20472248-Mobile-Android
