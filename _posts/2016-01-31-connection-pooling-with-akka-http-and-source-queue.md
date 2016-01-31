---
layout: post
title:  "Connection pooling in akka-http with source queue"
categories: scala akka akka-http akka-streams
---
The first line solution to consume HTTP-based services in Scala world is to use [akka-http](http://doc.akka.io/docs/akka-stream-and-http-experimental/2.0.2/). From the three provided _levels_ of the API, a [Host-Level Client-Site API](http://doc.akka.io/docs/akka-stream-and-http-experimental/2.0.2/scala/http/client-side/connection-level.html) is a quite natural choice for service clients. Basically, it just provides a pool of connections to specified HTTP service. Here is an example code from akka documentation which shows how we could use akka-http to send a single request to our service:

{% highlight scala %}
import akka.http.scaladsl.Http
import akka.http.scaladsl.model._
import akka.stream.ActorMaterializer
import akka.stream.scaladsl._

import scala.concurrent.Future
import scala.util.Try

implicit val system = ActorSystem()
implicit val materializer = ActorMaterializer()

// construct a pool client flow with context type `Int`
val poolClientFlow = Http().cachedHostConnectionPool[Int]("akka.io")
val responseFuture: Future[(Try[HttpResponse], Int)] =
  Source.single(HttpRequest(uri = "/") -> 42)
    .via(poolClientFlow)
    .runWith(Sink.head)
{% endhighlight %}

As it is a great example of akka-http usage it could be easily overused by developers. Using this approach, we create a new flow instance each time we want to send a request. This is the cause of `"Exceeded configured max-open-requests value of ..."` exception if we try to spawn more parallel requests then configured the `max-open-requests` value. To prevent resource starvation, we can introduce `Source.queue` instead of `Source.single`. This way we can create a single flow to which we will enqueue all our requests:

{% highlight scala %}
import akka.actor.ActorSystem
import akka.http.scaladsl.Http
import akka.http.scaladsl.model.{HttpRequest, HttpResponse}
import akka.stream.scaladsl.{Keep, Sink, Source}
import akka.stream.{ActorMaterializer, OverflowStrategy}

import scala.concurrent.duration._
import scala.concurrent.{Future, Await, Promise}
import scala.util.{Failure, Success}

import scala.concurrent.ExecutionContext.Implicits.global

implicit val system = ActorSystem("main")
implicit val materializer = ActorMaterializer()

val pool = Http().cachedHostConnectionPool[Promise[HttpResponse]](host = "google.com", port = 80)
val queue = Source.queue[(HttpRequest, Promise[HttpResponse])](10, OverflowStrategy.dropNew)
  .via(pool)
  .toMat(Sink.foreach({
     case ((Success(resp), p)) => p.success(resp)
    case ((Failure(e), p)) => p.failure(e)
  }))(Keep.left)
  .run


val promise = Promise[HttpResponse]
val request = HttpRequest(uri = "/") -> promise

val response = queue.offer(request).flatMap(buffered => {
  if (buffered) promise.future
  else Future.failed(new RuntimeException())
})

Await.ready(response, 3 seconds)
{% endhighlight %}

I hope this will help to develop more reliable akka-http services.
