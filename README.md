# Google Analytics for We chat mini program (Android SDK)

English Translation of the github repo - https://github.com/rchunping/wxapp-google-analytics

The [Measurement Protocol](https://developers.google.com/analytics/devguides/collection/protocol/v1/reference) is fully implemented, and the API interface is highly consistent with [Google Analytics for Android](https://developers.google.com/analytics/devguides/collection/android/v4/)

## Quick start

### 1. Google Analytics settings

The new version can't directly create mobile app type media resources, because the mobile app needs to connect to the mobile project in Firebase.

The solution is to create a property, select a website, and when the creation is complete, create a new data view. At this time, you can select the mobile application and delete the default view. This creation does not need to be associated with a Firebase project. For details, see [Add Google Analytics to Android app without Firebase
](https://stackoverflow.com/questions/45853012/add-google-analytics-to-android-app-without-firebase) for detailed steps.

#### 2. Add a ga.js file to your WeChat applet project
If you use the [WePY](https://github.com/Tencent/wepy) or [mpvue](https://github.com/Meituan-Dianping/mpvue) framework to develop applets, you can also install them via npm.

```bash
npm install git+https://github.com/rchunping/wxapp-google-analytics.git#v1.2.1
```

### 3. Set the request legal domain name in the WeChat applet background

Because the www.google-analytics.com domain name is not filed and cannot be added to the request legal domain name of the applet, you need to prepare a filed domain name for data forwarding, such as ga-proxy.example.com.

How to configure forwarding see [issue 4](https://github.com/rchunping/wxapp-google-analytics/issues/4)）**Change this**

> **Reminder:** Legal domain names can only be set 5 times a month. If it is just a local development test, you can set it first, as long as you don't check the domain name and TLS version of the development environment in the development tool, and then set the legal domain name before submitting the review.

### 4. Modify in the framework `app.js`
```js
var ga = require('path/to/ga.js');
var GoogleAnalytics = ga.GoogleAnalytics;
App({
    // ...
    tracker: null,
    getTracker: function () {
        if (!this.tracker) {
            // Initialize GoogleAnalytics Tracker
            this.tracker = GoogleAnalytics.getInstance(this)
                            .setAppName('Applet name')
                            .setAppVersion('Applet version number')
                            .newTracker('UA-XXXXXX-X'); //Replace with your Tracking ID
           
            //Use your own legal domain name for tracking data forwarding
            this.tracker.setTrackerServer("https://ga-proxy.example.com"); 
        }
        return this.tracker;
    },
    // ...
})
```

### 5. Find a `Page` to try out the simple ScreenView statistics.
```js
var ga = require('path/to/ga.js');
var HitBuilders = ga.HitBuilders;
Page({
    // ...

    // 一Processing ScreenView in onShow()
    onShow: function(){
        // Get that Tracker instance
        var t = getApp().getTracker();
        t.setScreenName('This is my first page');
        t.send(new HitBuilders.ScreenViewBuilder().build());
    },
    // ...
})
```

### 6. Go to the Google Stats background to see it in real time > Overview
If everything is ok, you will see the data in a few seconds.

## Code example

### Tracker
The tracker is used to collect data and send it to the Google Stats server. The tracker corresponds to the media resources counted by Google. You can create multiple trackers to correspond to different media resources.

```js
var ga = require('path/to/ga.js');
var GoogleAnalytics = ga.GoogleAnalytics;

var app = getApp(); //Get the App instance of the WeChat applet

// Initialize GoogleAnalytics
var gaInstance = GoogleAnalytics.getInstance(app);
gaInstance.setAppName('Applet name'); // Set the APP name
gaInstance.setAppVersion('Applet version number'); //Set the APP version number, [optional]

// Create a Tracker
var tracker = gaInstance.newTracker('UA-XXXXXX-X'); // The parameter is the tracking ID in the Google analytics property. (Tracking ID)

//Use your own legal domain name for tracking data forwarding
tracker.setTrackerServer("https://ga-proxy.example.com"); 
```

In most cases we only need one tracker, so it is recommended to share a tracker globally in `app.js`:

```js
// app.js
var ga = require('path/to/ga.js');
var GoogleAnalytics = ga.GoogleAnalytics;
App({
    // ...
    tracker: null,
    getTracker: function () {
        if (!this.tracker) {
            // Initialize GoogleAnalytics Tracker
            this.tracker = GoogleAnalytics.getInstance(this)
                            .setAppName('Applet name')
                            .setAppVersion('Applet version number')
                            .newTracker('UA-XXXXXX-X'); 

            //Use your own legal domain name for tracking data forwarding
            this.tracker.setTrackerServer("https://ga-proxy.example.com")
        }
        return this.tracker;
    },
    // ...
})
```

The use of the tracker (usually in the `Page` logic)


```js
// /pages/index/index.js

Page({
    // ...
    onShow: function(){

        // Get global tracker
        var t = getApp().getTracker();

        // This screen name will be used for all subsequent matching data.
        t.setScreenName('这是首页');

        // t.send(Hit) Reporting data
    },
    // ...
})


```


### Matching build HitBuilder

To configure all the parameters needed for a match, `HitBuilder` provides some common methods that are needed for all match types.

Generally do not need to use `HitBuilder` directly, please use the following `ScreenViewBuilder`, `EventBuilder`, `ExceptionBuilder`, `TimingBuilder` according to the actual match type.

### Screen ScreenView

The screen represents what the user viewed within your applet.

```js
var ga = require('path/to/ga.js');
var HitBuilders = ga.HitBuilders;
Page({
    // ...

    // Usually handle ScreenView in onShow()
    onShow: function(){
        // Get that Tracker instance
        var t = getApp().getTracker();
        t.setScreenName('Current screen name'); 
        t.send(new HitBuilders.ScreenViewBuilder().build());
    },
    // ...
})
```
Support for custom dimensions and metrics (you need to pre-define in the Google Analytics background)

```js
t.send(new HitBuilders.ScreenViewBuilder()
    .setCustomDimension(1,"Dimension 1")
    .setCustomDimension(2,"Dimension 2")
    .setCustomMetric(1,100.35)
    .setCustomMetric(2,200).build());
```

> **Reminder:** Custom dimensions and metrics can be set on all `HitBuilder`.

### Event

Events can help you measure how users interact with interactive components in your applet, such as button clicks.
Each event consists of 4 fields: **Category**, **Action**, **Label** and **Value**. The two parameters ** category** and **action** are required.

```js
t.send(new HitBuilders.EventBuilder()
    .setCategory('video')
    .setAction('Click')
    .setLabel('Play') // Optional
    .setValue(1).build()); // Optional
```

### Crash and Exception Exception

You can count the captured exception information in the applet.

```js
t.send(new HitBuilders.ExceptionBuilder()
    .setDescription('Exception description information')  
    .setFatal(false).build()); // Optional, whether it is a serious error, the default is true
```

### User Timing Timing

The timer has four parameters: **Category**, **Value**, **Name**, **Label**. Where **category** and **value** are required, and the unit of **value** is milliseconds.

```js
t.send(new HitBuilders.TimingBuilder()
    .setCategory('Timer')
    .setValue(63000)
    .setVariable('User registration')
    .setLabel('Form').build());
```

### Session Management

The default session duration is 30 minutes and can be set at the Google Analytics property level. You can also start a new session manually by using the `setNewSession` method when sending a match.

```js
// Start a new session with the hit.
t.send(new HitBuilders.ScreenViewBuilder()
    .setNewSession()
    .build());
```

> **Reminder: ** All `HitBuilder` supports manual start of new sessions.

### User ID (cross-application, cross-device tracking)

The User ID is used for the same user tracking across applications and across devices. For example, you can set the applet user's <del>`OpenID` or </del> `UnionID` to the User ID.

> ** You need to enable User ID tracking in the media resources in the Google Analytics background. **

To send a User ID, use the `Measurement Protocol & Number Syntax` and `&uid` parameter names to set the `userId` field, as shown in the following example:
```js
  // You only need to set User ID on a tracker once. By setting it on the
  // tracker, the ID will be sent with all subsequent hits.
  t.set("&uid", '12345');

  // This hit will be sent with the User ID value and be visible in
  // User-ID-enabled views (profiles).
  t.send(new HitBuilders.EventBuilder()
      .setCategory("UX")
      .setAction("User Sign In")
      .build());
```

The above code uses the `&uid` parameter, all the parameters of the `Measurement Protocol & Number Syntax ` can go to [Measurement Protocol Parameter Reference] (https://developers.google.com/analytics/devguides/collection/protocol/v1/parameters ) View.

> **Reminder: ** These `&uid` parameters can be set at the `Tracker` level or on a single match of `HitBuilder`.


## E-commerce activities related

Check out [Measuring E-Commerce Events] (doc/ecommerce.md)

## Campaign and traffic source attribution

You can use the `setCampaignParamsFromUrl` method to set campaign parameters directly in the match builder `HitBuilder` to attribute user activity in a series of sessions to a specific referral traffic source or marketing campaign:

```js
// Set screen name.
t.setScreenName(screenName);

// In this example, campaign information is set using
// a url string with Google Analytics campaign parameters.
// Note: This is just an example, the URL? The previous part is actually useless, mainly the analysis of the utm_XXXXX series parameters.
//
var campaignUrl = "http://example.com/index.html?" +
    "utm_source=email&utm_medium=email_marketing&utm_campaign=summer" +
    "&utm_content=email_variation_1";

// Campaign data sent with this hit.
t.send(new HitBuilders.ScreenViewBuilder()
    .setCampaignParamsFromUrl(campaignUrl)
    .build()
);
```

> **Important Reminder:** If you want to track new users brought by your ads, make sure that `setCampaignParamsFromUrl` is applied to the first match sent by this new user.

You can also set it via `setCampaignParamsOnNextHit` on the tracker:

```js
t.setScreenName(screenName);
var campaignUrl = "http://example.com/index.html?" +
    "utm_source=email&utm_medium=email_marketing&utm_campaign=summer" +
    "&utm_content=email_variation_1";
t.setCampaignParamsOnNextHit(campaignUrl); // The next sent match will carry these parameters

t.send(new HitBuilders.ScreenViewBuilder().build());
```

### Tracking the scene value of the WeChat applet

The scene value of the WeChat applet can be used as a traffic source for tracking. To track scene values you need to handle in `App``onLaunch`

```js
// app.js
var ga = require('path/to/ga.js');
var CampaignParams = ga.CampaignParams;

App({
    //...
    onLaunch: function(options) {
        if (options && options.scene) {
            var campaignUrl = CampaignParams.buildFromWeappScene(options.scene).toUrl();
            var t = getApp().getTracker();
            t.setCampaignParamsOnNextHit(campaignUrl);

            // The next sent match will bring WeChat scene information
            // t.send(Hit) 
        }
    },
    //...
})
```
`CampaignParams.buildFromWeappScene` will convert the scene values into `utm_source` and `utm_medium` parameters for tracking.

The report seen by Google Stats is similar to [such] (#scene-example)

### Tracking QR code parameters of WeChat applet

Each WeChat applet can set up a QR code of up to 100,000 custom parameters. The following describes how to track the promotion effect of each QR code.

> The QR code of the WeChat applet custom parameters needs to be generated by the API. Refer to [WeChat Document] (https://mp.weixin.qq.com/debug/wxadoc/dev/api/qrcode.html).

Suppose you use the `path` that is used to generate the QR code.

```js
{"path": "pages/index/index?utm_source=Coffee%20Bar&utm_medium=qrcode", "width": 430}

// path It's usually the entry page of your applet, or it's one of the pages registered on the app.
```

Use `CampaignParams.parseFromPageOptions` to identify the ad parameters in the QR code in `onLoad` corresponding to `Page`:

```js
// pages/index/index.js
var ga = require('path/to/ga.js');
var CampaignParams = ga.CampaignParams;

Page({
    //...
    onLoad: function(options) {
        var t = getApp().getTracker();
        // Parse the utm_xxxxxx parameter in the options to generate an ad connection URL
        var campaignUrl = CampaignParams.parseFromPageOptions(options).toUrl();
        t.setCampaignParamsOnNextHit(campaignUrl);

        // The next sent match will be accompanied by the ad source information.
        // t.send(Hit) 
    },
    //...
})
```

If the argument to `path` is not a campaign parameter starting with `utm_`, then calling `parseFromPageOptions` requires passing in a second argument to specify the mapping of the parameter name:

```js
{"path" : "pages/index/index?var1=Coffee%20Bar&var2=Scan%20Qrcode", "width": 430}


// After scanning the code into the applet
Page({
    onLoad: funciton(options) {
        // At this point options is like this
        // { "var1" : "Coffee Bar", "var2" : "Scan Qrcode" }
        // Need to specify a mapping relationship with utm_xxx
        var map = {
            "var1" : "utm_source", //Correspond to var1 to utm_source
            "var2" : "utm_medium"
        };

        var campaignUrl = CampaignParams.parseFromPageOptions(options, map).toUrl();
        var t = getApp().getTracker();
        t.setCampaignParamsOnNextHit(campaignUrl);
    }
})

```

Campaign parameter list

| Parameter | Description | Example |
| --- | --- | --- |
| `utm_source` | Campaign source to identify specific search engines, newsletters, or other sources | utm_source=google |
| `utm_medium` | Campaign medium for identifying emails or media with cost-per-click (CPC) ads | utm_medium=cpc |
| `utm_term` | Campaign words for paid search, keywords for ads | utm_term=running+shoes |
| `utm_content` | Campaign content for A/B testing and content targeting ads to distinguish between different ads or links to the same URL | utm_content=logolink<br/>utm_content=textlink |
| `utm_campaign` | Campaign name for keyword analysis to identify specific product promotions or strategic campaigns | utm_campaign=spring_sale |

See all [campaign parameters] (https://developers.google.com/analytics/devguides/collection/android/v4/campaigns#campaign-params).

## API Reference

All interfaces directly refer to the Google Analytics SDK for Android, you can directly view the corresponding API documentation.

[GoogleAnalytics] (doc/GoogleAnalytics.md)

Tracker

* [Tracker] (https://developers.google.com/android/reference/com/google/android/gms/analytics/Tracker.html#enableAdvertisingIdCollection(boolean))

Matching build

* [HitBuilder] (https://developers.google.com/android/reference/com/google/android/gms/analytics/HitBuilders.HitBuilder)
* [ScreenViewBuilder] (https://developers.google.com/android/reference/com/google/android/gms/analytics/HitBuilders.ScreenViewBuilder)
* [EventBuilder] (https://developers.google.com/android/reference/com/google/android/gms/analytics/HitBuilders.EventBuilder)
* [ExceptionBuilder](https://developers.google.com/android/reference/com/google/android/gms/analytics/HitBuilders.ExceptionBuilder)
* [TimingBuilder] (https://developers.google.com/android/reference/com/google/android/gms/analytics/HitBuilders.TimingBuilder)

Enhanced E-Commerce API

* [Product](https://developers.google.com/android/reference/com/google/android/gms/analytics/ecommerce/Product)
* [ProductAction](https://developers.google.com/android/reference/com/google/android/gms/analytics/ecommerce/ProductAction)
* [Promotion] (https://developers.google.com/android/reference/com/google/android/gms/analytics/ecommerce/Promotion)

## Features

* Full implementation [Measurement Protocol] (https://developers.google.com/analytics/devguides/collection/protocol/v1/reference)
* Support multiple batch data matching report
* Because WeChat applet only supports 10 `wx.request` concurrency, in order not to affect the network request of business data, the data is reported in order, occupying at most one `wx.request`

## Known issues

1. Because the system information that WeChat applet can obtain is very limited, you will not be able to get the user characteristics information of 'age`, `gender`, `interest`.
2. Display ads are invalid
3. There is an error in the screen resolution. Get the `windowWidth`, `windowHeight`, `pixelRatio` from the WeChat applet `wx.getSystemInfo`.

> **Other possible issues**
>
> If the console shows `"ga:failed"`, the content is `errMsg:"request:fail The TLS version required by the applet must be greater than or equal to 1.2".
> I encountered this problem when I turned over the wall, and it was normal when I did not turn over the wall. Please check the development environment does not verify the requested domain name and TLS version in the development tool.
>
> ``ga:failed"` will appear on the iPhone, the content is `errMsg: "request:fail response data convert to utf8 fail"`
> This is a WeChat applet interface bug. The return content is always treated as json data. In fact, the data has been successfully sent to Google. The returned content is a 1x1 gif image. Does not affect data collection (see [issue 1] (https://github.com/rchunping/wxapp-google-analytics/issues/1)).
>
> You can find the information of "ga.****" through the console.

## References

Protocol [Measurement Protocol] (https://developers.google.com/analytics/devguides/collection/protocol/v1/reference)
Interface reference [Google Analytics for Android] (https://developers.google.com/analytics/devguides/collection/android/v4/)
[Campaign parameters] (https://developers.google.com/analytics/devguides/collection/android/v4/campaigns#campaign-params)












































































