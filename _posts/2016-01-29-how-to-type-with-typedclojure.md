---
layout: post
title:  "How to type with TypedClojure?"
categories: clojure typedclojure
---

When I was working on [cats.typed](https://github.com/kazuhiro/cats.typed) I quite often was wondering if there is something like Rosetta Stone for [TypedClojure](http://typedclojure.org/). A place where I could find type declarations in Scala or Java, and corresponding declarations for Clojure. As I have not found such place, so I was inspired to write this post. It contains only few basic cases, yet, it still could be useful to make the first steps.

# Function with no arguments and no return value
Java
{% highlight java %}
public void printlnString() {
  System.out.println("void");
}
{% endhighlight %}

Scala
{% highlight scala %}
def printlnString(): Unit = {
  println("void")
}
{% endhighlight %}

Clojure
{% highlight clojure %}
(t/ann println-string [-> (t/Value nil)])
(defn println-string
  []
  (println "void"))
{% endhighlight %}

# Function with a single argument and return value
Java
{% highlight java %}
public Integer stringToInt(String a) {
  return Integer.parse(a);
}
{% endhighlight %}

Scala
{% highlight scala %}
def stringToInt(a: String): Int = {
  a.toInt
}
{% endhighlight %}

Clojure
{% highlight clojure %}
(t/ann string-to-int [String -> Integer])
(defn string-to-int
  [a]
  (Integer/parseInt a))
{% endhighlight %}

# Function with two arguments and return value
Java
{% highlight java %}
public Long sumArgsLength(String a, String b) {
  return Long.valueOf(a.length() + b.length());
}
{% endhighlight %}

Scala
{% highlight scala %}
def sumArgsLength(a: String, b: String): Long = {
  (a.length + b.length).toLong
}
{% endhighlight %}

Clojure
{% highlight clojure %}
(t/ann sum-args-length [String String -> Long])
(defn sum-args-length
  [^String a ^String b]
  (long
   (+ (.length a) (.length b))))
{% endhighlight %}

# Function with typed argument and return value
Java
{% highlight java %}
class Custom<T> {
  T name;
}

public String getCustomName(Custom<String> custom) {
  return custom.name;
}
{% endhighlight %}

Scala
{% highlight scala %}
case class Custom[T](name: T)

def getCustomName(custom: Custom[String]): String = {
  custom.name
}
{% endhighlight %}

Clojure
{% highlight clojure %}
(t/ann-record [[type :variance :covariant]]
              Custom
              [name :- type])
(defrecord Custom [name])

(t/ann get-custom-name [(Custom String) -> String])
(defn get-custom-name [custom] (:name custom))
{% endhighlight %}
