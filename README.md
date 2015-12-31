# Stetho [![Build Status](https://travis-ci.org/facebook/stetho.svg?branch=master)](https://travis-ci.org/facebook/stetho)
　2015年　stetho 是开源大户 Facebook 提供的一款 Android 调试神器，可以在 Chrome 上像调试 Web App 一样，调试 Android Native App, 听起来就很酷，但感觉只是刚发布时红过一阵子。
[Stetho](https://facebook.github.io/stetho) is a sophisticated debug bridge for Android applications. When enabled,
developers have access to the Chrome Developer Tools feature natively part of
the Chrome desktop browser. Developers can also choose to enable the optional
`dumpapp` tool which offers a powerful command-line interface to application
internals.

Once you complete the set-up instructions below, just start your app and point
your laptop browser to `chrome://inspect`.  Click the "Inspect" button to
begin.

## Set-up

### Download
Download [the latest JARs](https://github.com/facebook/stetho/releases/latest) or grab via Gradle:
```groovy
compile 'com.facebook.stetho:stetho:1.2.0'
```
or Maven:
```xml
<dependency>
  <groupId>com.facebook.stetho</groupId>
  <artifactId>stetho</artifactId>
  <version>1.2.0</version>
</dependency>
```

Only the main `stetho` dependency is strictly required; however, you may also wish to use one of the network helpers:

```groovy
compile 'com.facebook.stetho:stetho-okhttp:1.2.0'
```
or:
```groovy
compile 'com.facebook.stetho:stetho-urlconnection:1.2.0'
```

You can also enable a JavaScript console with:

```groovy
compile 'com.facebook.stetho:stetho-js-rhino:1.2.0'
```
For more details on how to customize the JavaScript runtime see [stetho-js-rhino](stetho-js-rhino/).

### Putting it together
Integrating with Stetho is intended to be seamless and straightforward for
most existing Android applications.  There is a simple initialization step
which occurs in your `Application` class:

```java
public class MyApplication extends Application {
  public void onCreate() {
    super.onCreate();
    Stetho.initializeWithDefaults(this);
  }
}
```

This brings up most of the default configuration but does not enable some
additional hooks (most notably, network inspection).  See below for specific
details on individual subsystems.

### Enable network inspection
If you are using the popular [OkHttp](https://github.com/square/okhttp)
library at the 2.2.x+ release, you can use the
[Interceptors](https://github.com/square/okhttp/wiki/Interceptors) system to
automatically hook into your existing stack.  This is currently the simplest
and most straightforward way to enable network inspection:

```java
OkHttpClient client = new OkHttpClient();
client.networkInterceptors().add(new StethoInterceptor());
```

If you are using `HttpURLConnection`, you can use `StethoURLConnectionManager`
to assist with integration though you should be aware that there are some
caveats with this approach.  In particular, you must explicitly add
`Accept-Encoding: gzip` to the request headers and manually handle compressed
responses in order for Stetho to report compressed payload sizes.

See the [`stetho-sample` project](stetho-sample) for more details.

## Going further

### Custom dumpapp plugins
Custom plugins are the preferred means of extending the `dumpapp` system and
can be added easily during configuration.  Simply replace your configuration
step as such:

```java
Stetho.initialize(Stetho.newInitializerBuilder(context)
    .enableDumpapp(new DumperPluginsProvider() {
      @Override
      public Iterable<DumperPlugin> get() {
        return new Stetho.DefaultDumperPluginsBuilder(context)
            .provide(new MyDumperPlugin())
            .finish();
      }
    })
    .enableWebKitInspector(Stetho.defaultInspectorModulesProvider(context))
    .build())
```

See the [`stetho-sample` project](stetho-sample) for more details.

## Improve Stetho!
See the [CONTRIBUTING.md](CONTRIBUTING.md) file for how to help out.

## License
Stetho is BSD-licensed. We also provide an additional patent grant.
