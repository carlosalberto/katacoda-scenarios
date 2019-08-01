Let's create an instance of a real tracer, such as Jaeger (http://github.com/uber/jaeger-client-java). Our `pom.xml` already imports Jaeger:

```xml
<dependency>
    <groupId>com.lightstep.tracer</groupId>
    <artifactId>lightstep-tracer-jre</artifactId>
    <version>0.16.4</version>
</dependency>
<dependency>
   <groupId>com.lightstep.tracer</groupId>
   <artifactId>tracer-okhttp</artifactId>
   <version>0.17.2</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.25</version>
</dependency>
```

We need to add some imports:

<pre class="file" data-target="clipboard">
import io.opentracing.Span;
import io.opentracing.Tracer;
</pre>

And we define a helper function that will create a tracer.

<pre class="file" data-target="clipboard">
public static Tracer initTracer(String component) {
	Tracer tracer = new com.lightstep.tracer.jre.JRETracer(
		 new com.lightstep.tracer.shared.Options.OptionsBuilder()
                    .withComponentName(component)
		    .withAccessToken("{your_access_token}")
		    .build()
	);
        return tracer;
}
</pre>

To use this instance, let's change the main function:

<pre class="file" data-target="clipboard">
Tracer tracer = initTracer("hello-world");
new Hello(tracer).sayHello(helloTo);
tracer.close();
</pre>

Note that we are passing a string `hello-world` to the init method. It is used to mark all spans emitted by the tracer as originating from a `hello-world` service.

This is how our `Hello` class looks like now:

<pre class="file" data-filename="java/src/main/java/lesson01/exercise/Hello.java" data-target="replace">package lesson01.exercise;

import io.opentracing.Tracer;
import io.opentracing.Span;

public class Hello {

    private final io.opentracing.Tracer tracer;

    private Hello(io.opentracing.Tracer tracer) {
        this.tracer = tracer;
    }

    private void sayHello(String helloTo) {
        Span span = tracer.buildSpan("say-hello").startManual();

        String helloStr = String.format("Hello, %s!", helloTo);
        System.out.println(helloStr);

        span.finish();
    }

    public static Tracer initTracer(String component) {
	Tracer tracer = new com.lightstep.tracer.jre.JRETracer(
		 new com.lightstep.tracer.shared.Options.OptionsBuilder()
                    .withComponentName(component)
		    .withAccessToken("{your_access_token}")
		    .build()
	);
        return tracer;
    }

    public static void main(String[] args) {
        if (args.length != 1) {
            throw new IllegalArgumentException("Expecting one argument");
        }
        String helloTo = args[0];

        Tracer tracer = initTracer("hello-world");
        new Hello(tracer).sayHello(helloTo);
        tracer.close();

    }
}</pre>

Running the program now, we see a span logged. Try it out: `./run.sh lesson01.exercise.Hello Donut`{{execute}}.
