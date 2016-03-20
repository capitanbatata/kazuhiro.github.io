---
layout: post
title:  "ActorLogging vs. scala-logging - what is the difference?"
categories: scala akka logging
---

It is natural to use [ActorLogging](http://doc.akka.io/api/akka/snapshot/akka/actor/ActorLogging.html) when you need to provide a logger instance inside your Actor. Yet from time to time you might see instead logging traits (ie. _LazyLogging_ or _StrictLogging_) from [scala-logging](https://github.com/typesafehub/scala-logging). Usually, that could be seen in the code where [Actor Model](https://en.wikipedia.org/wiki/Actor_model) meets [Dataflow](https://en.wikipedia.org/wiki/Dataflow) or [Reactive Streams](http://www.reactive-streams.org/). Can we use those two solutions interchangeably? Let's look into details.

*ActorLogging* is deeply integrated with Akka. By default, it provides a [Mapped Diagnostic Context](https://en.wikipedia.org/wiki/MDC) (see [DiagnosticActorLogging](http://doc.akka.io/api/akka/snapshot/akka/actor/DiagnosticActorLogging.html)) with a bunch of useful information, including a full path of the actor in the actor tree. Also, some of the logging settings are shared with Akka configuration, so if you change for instance _akka.loglevel_ in your _application.conf_ it will have an effect on logs reported by your actor. It is also important to note that _log_ instance provided by ActorLogging is supposed to be used inside an actor, so do not propagate it to deferred computations (ie. Futures).

*scala-logging* is a more general solution. Having no connections to Akka, it cannot provide a proper context of the actor. Also, logging configuration is completely separated from Akka settings. There is no need to protect _logger_ instance, so you can pass it whenever you need it.

To sum up, whenever logging occurs inside an Actor body - use ActorLogging to keep the context of the logging information. In all other places, use scala-logging.

Here is a simple application which shows a difference between discussed logging using default configurations:

{% highlight scala %}
import akka.actor._
import com.typesafe.scalalogging._

case object DummyMessage

class DummyActor extends Actor with ActorLogging with StrictLogging {
  override def receive = {
    case DummyMessage =>
      log.info("I am ActorLogging")
      logger.info("I am StrictLogging")
  }
}

object Main extends App {
  import scala.concurrent.ExecutionContext.Implicits.global
  val system = ActorSystem("main")

  val dummy = system.actorOf(Props(new DummyActor))
  dummy ! DummyMessage
}
{% endhighlight %}

{% highlight bash %}
sbt run Main
{% endhighlight %}

{% highlight bash %}
[info] Running Main
[INFO] [03/20/2016 10:41:58.901] [main-akka.actor.default-dispatcher-2] [akka://main/user/$a] I am ActorLogging
10:41:58.906 [main-akka.actor.default-dispatcher-2] INFO  DummyActor - I am StrictLogging
{% endhighlight %}
