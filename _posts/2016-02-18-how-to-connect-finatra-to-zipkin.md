---
layout: post
title:  "How to connect Finatra to Zipkin? "
categories: scala finagle finatra zipkin
---

[Finatra](https://twitter.github.io/finatra/) is a HTTP framework built on top of [Finagle](https://twitter.github.io/finagle/). Thanks to that, developers can use a wide range of built-in features, particularly, request tracing using [Zipkin](http://zipkin.io/). However, this outstanding functionality is neither described in the documentation nor in code examples (at least, I haven't found it). So how to connect those two?

First of all, we have to setup [Zipkin](http://zipkin.io/) using Docker:
{% highlight bash %}
git clone https://github.com/openzipkin/docker-zipkin
cd docker-zipkin
docker-compose up
{% endhighlight %}
This will setup three containers - [Mysql](https://www.mysql.com/) storage, zipkin dependencies, and zipkin itself.
{% highlight bash %}
docker ps --format "{% raw %}{{.Image}}{% endraw %}"

# output
openzipkin/zipkin-dependencies
openzipkin/zipkin
openzipkin/zipkin-mysql
{% endhighlight %}

Now it is time to run [Finatra](https://twitter.github.io/finatra/) demo application and connect it to [Zipkin](http://zipkin.io/):
{% highlight bash %}
git clone https://github.com/twitter/finatra.git
cd finatra
sbt "helloWorld/runMain com.twitter.hello.HelloWorldServerMain
-admin.port=:8880
-Dzipkin.host=localhost:9410
-Dtracing.debugTrace=true
-Dzipkin.initialSampleRate=1.0
-DtracingEnabled=true"
{% endhighlight %}
Why there is need for so many flags? Well, let's have a closer look at them:

* **admin.port** to omit collision with [Zipkin](http://zipkin.io/)'s web interface admin panel
* **zipkin.host** points to our [Zipkin](http://zipkin.io/) collector
* **tracing.debugTrace** to see if traces are sent to collector
* **tracing.initialSampleRate** to trace all requests (otherwise only n-th lucky request will go to collector)
* **tracingEnabled** to enable tracing

{% highlight bash %}
[info] Running com.twitter.hello.HelloWorldServerMain -admin.port=:8880 -Dipkin.host=localhost:9410 -Dtracing.debugTrace=true -Dinit
ialSampleRate=1.0 -DtracingEnabled=true
Feb 17, 2016 4:12:10 PM com.twitter.finagle.http.HttpMuxer$$anonfun$4 apply
INFO: HttpMuxer[/admin/metrics.json] = com.twitter.finagle.stats.MetricsExporter(<function1>)
Feb 17, 2016 4:12:10 PM com.twitter.finagle.http.HttpMuxer$$anonfun$4 apply
INFO: HttpMuxer[/admin/per_host_metrics.json] = com.twitter.finagle.stats.HostMetricsExporter(<function1>)
I 0217 15:12:10.259 THREAD44:  => com.twitter.server.handler.AdminRedirectHandler
I 0217 15:12:10.259 THREAD44: /admin => com.twitter.server.handler.SummaryHandler
I 0217 15:12:10.260 THREAD44: /admin/contention => com.twitter.finagle.Filter$$anon$1
I 0217 15:12:10.261 THREAD44: /admin/lint => com.twitter.server.handler.LintHandler
I 0217 15:12:10.260 THREAD44: /admin/server_info => com.twitter.finagle.Filter$$anon$1
I 0217 15:12:10.261 THREAD44: /admin/lint.json => com.twitter.server.handler.LintHandler
I 0217 15:12:10.265 THREAD44: /admin/threads => com.twitter.server.handler.ThreadsHandler
I 0217 15:12:10.266 THREAD44: /admin/threads.json => com.twitter.server.handler.ThreadsHandler
I 0217 15:12:10.266 THREAD44: /admin/announcer => com.twitter.finagle.Filter$$anon$1
I 0217 15:12:10.266 THREAD44: /admin/pprof/profile => com.twitter.server.handler.ProfileResourceHandler
I 0217 15:12:10.266 THREAD44: /admin/dtab => com.twitter.finagle.Filter$$anon$1
I 0217 15:12:10.266 THREAD44: /admin/pprof/heap => com.twitter.server.handler.HeapResourceHandler
I 0217 15:12:10.267 THREAD44: /admin/shutdown => com.twitter.server.handler.ShutdownHandler
I 0217 15:12:10.267 THREAD44: /admin/events => com.twitter.server.handler.EventsHandler
I 0217 15:12:10.267 THREAD44: /admin/events/record/ => com.twitter.server.handler.EventRecordingHandler
I 0217 15:12:10.266 THREAD44: /admin/ping => com.twitter.server.handler.ReplyHandler
I 0217 15:12:10.267 THREAD44: /admin/tracing => com.twitter.server.handler.TracingHandler
I 0217 15:12:10.268 THREAD44: /admin/metrics => com.twitter.server.handler.MetricQueryHandler
I 0217 15:12:10.268 THREAD44: /admin/clients/ => com.twitter.server.handler.ClientRegistryHandler
I 0217 15:12:10.268 THREAD44: /admin/servers/ => com.twitter.server.handler.ServerRegistryHandler
I 0217 15:12:10.268 THREAD44: /admin/files/ => com.twitter.server.handler.ResourceHandler
I 0217 15:12:10.268 THREAD44: /admin/logging => com.twitter.server.handler.LoggingHandler
I 0217 15:12:10.266 THREAD44: /admin/pprof/contention => com.twitter.server.handler.ProfileResourceHandler
I 0217 15:12:10.269 THREAD44: /admin/registry.json => com.twitter.server.handler.RegistryHandler
I 0217 15:12:10.269 THREAD44: /favicon.ico => com.twitter.server.handler.ResourceHandler
I 0217 15:12:10.277 THREAD44: Serving admin http on 0.0.0.0/0.0.0.0:8880
I 0217 15:12:10.324 THREAD44: Finagle version 6.33.0 (rev=21d0ee8b5070b735eda5c84d7aa6fbf1ba7b1635) built at 20160203-202859
{% endhighlight %}

Hope it helps!
