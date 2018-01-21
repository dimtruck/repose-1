# Repose Java Support Library

## What

Use this dependency to integrate [Opentracing](http://opentracing.io/) support into your application/service fronted by [Repose](http://www.openrepose.org/).

## How

Add this dependency to your build file (Gradle, Maven, Ivy, etc.).  To extract the span context, use the following code snippet:

```
// OpenTracing hotness
import org.openrepose.contrib.opentracing.TraceGUIDExtractor;

...

public static final String TRACE_GUID = "X-Trans-Id";

// set up your opentracingservice configuration elsewhere
@Autowired
OpenTracingService openTracingService;

...

// Extract existing span context
SpanContext context = openTracingService.get().getGlobalTracer().extract(
    Format.Builtin.TEXT_MAP, new TraceGUIDExtractor(wrappedRequest.getHeader(TRACE_GUID)));

Tracer.SpanBuilder spanBuilder;

// check if context exists.  If not, start a new one (root span)
if (context == null)
    spanBuilder = openTracingService.get().getGlobalTracer()
        .buildSpan("my-application")
        .withTag(Tags.SPAN_KIND.getKey(), Tags.SPAN_KIND_CLIENT);
else
    spanBuilder = openTracingService.get().getGlobalTracer()
        .buildSpan("my-application")
        .asChildOf(context)
        .withTag(Tags.SPAN_KIND.getKey(), Tags.SPAN_KIND_CLIENT);

// Start the new span
activeSpan = spanBuilder.startActive();
```

To inject the above span for subsequent requests:

```
// OpenTracing hotness
import org.openrepose.contrib.opentracing.TraceGUIDInjector;
import org.springframework.http.HttpHeaders;


...

public static final String TRACE_GUID = "X-Trans-Id";

// set up your opentracingservice configuration elsewhere
@Autowired
OpenTracingService openTracingService;

...

HttpHeaders httpHeaders = new HttpHeaders();
httpHeaders.set("Content-Type", "application/json");
httpHeaders.set("Accept", "application/json");

openTracingService.getGlobalTracer().inject(
    activeSpan.context(), Format.Builtin.TEXT_MAP,
    new TraceGUIDInjector(httpHeaders, TRACE_GUID));

```

## Why

Repose utilizes `X-TRANS-ID` header to wrap all tracing/debugging details of a request.  It's a nice wrapper that can be utilized not jsut for OpenTracing but fits very well for this use case.  In fact, if your application does not need to instrument anything inside of the app, you can simply pass `X-TRANS-ID` header to the next request and if that service is also fronted by Repose, the extraction/injection will occur automatically.

**Exception**: Exception to this are any user interface applications or applications that do not use Repose as a proxy.  In this case, this wrapper will help in injecting the span information in the proper headers.
