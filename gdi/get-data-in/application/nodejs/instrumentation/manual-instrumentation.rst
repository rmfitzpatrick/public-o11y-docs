.. _nodejs-manual-instrumentation:

********************************************************************
Manually instrument Node applications for Splunk Observability Cloud
********************************************************************

.. meta::
   :description: Manually instrument your Node application when you need to add custom attributes to spans or want to manually generate spans and metrics. Keep reading to learn how to manually instrument your Node application for Splunk Observability Cloud.

Instrumenting applications automatically using the agent of the Splunk Distribution of OpenTelemetry Node covers most needs. Manually instrumenting your application is only necessary when, for example, you need to add custom attributes to spans or need to manually generate spans.

.. note:: Manual OTel instrumentation is fully compatible with Splunk automatic Node.js instrumentation and is fully supported by Splunk.

.. _nodejs-otel-custom-traces:

Custom traces
=====================================

To send custom traces to Splunk Observability Cloud, add the required dependencies:

.. code-block:: javascript

   const { start } = require('@splunk/otel');
   const opentelemetry = require('@opentelemetry/api');
   const { Resource } = require('@opentelemetry/resources');
   const {
      SemanticResourceAttributes,
   } = require('@opentelemetry/semantic-conventions');
   const { ConsoleSpanExporter } = require('@opentelemetry/sdk-trace-base');

   // All fields are optional.
   start({
      // Takes preference over OTEL_SERVICE_NAME environment variable
      serviceName: 'my-service',
      tracing: {
         spanExporterFactory: () => [new ConsoleSpanExporter()],
         tracerConfig: { resource: new Resource({ [SemanticResourceAttributes.SERVICE_VERSION]: '0.1.0' }) }
      },
   });

   const resource = Resource.default().merge(
      new Resource({
         [SemanticResourceAttributes.SERVICE_NAME]: 'service-name-here',
         [SemanticResourceAttributes.SERVICE_VERSION]: '0.1.0',
      }),
   );

   const exporter = new ConsoleSpanExporter();
   const processor = new BatchSpanProcessor(exporter);
   provider.addSpanProcessor(processor);

   provider.register();

Create a tracer
----------------------------------------------------

Create or acquire a tracer anywhere in your application:

.. code-block:: javascript

   const tracer = opentelemetry.trace.getTracer(
      // Uniquely identify instrumentation scope
      '<name-of-scope>',
      '<version of scope>',
   );

Create spans
---------------------------------------------

After you've created a tracer, create spans. For example:

.. code-block:: javascript

   function rollTheDice(rolls, min, max) {
      // Creates a span
      return tracer.startActiveSpan('rollTheDice', (span) => {
         const result = [];
         for (let i = 0; i < rolls; i++) {
            result.push(rollOnce(min, max));
         }
         // Ends the span
         span.end();
         return result;
      });
   }

.. note:: For more examples of manual instrumentation, see :new-page:`Manual instrumentation <https://opentelemetry.io/docs/instrumentation/js/manual/>` in the OpenTelemetry official documentation.


.. _nodejs-otel-custom-metrics:

Custom metrics
=====================================

To send custom application metrics to Splunk Observability Cloud, add ``@opentelemetry/api-metrics`` to your dependencies:

.. code-block:: javascript

   const { start } = require('@splunk/otel');
   const { Resource } = require('@opentelemetry/resources');
   const { metrics } = require('@opentelemetry/api-metrics');

   // All fields are optional.
   start({
     // Takes preference over OTEL_SERVICE_NAME environment variable
     serviceName: 'my-service',
     metrics: {
       // The suggested resource is filled in using OTEL_RESOURCE_ATTRIBUTES
       resourceFactory: (suggestedResource: Resource) => {
         return suggestedResource.merge(new Resource({
           'my.property': 'xyz',
           'build': 42,
         }));
       },
       exportIntervalMillis: 1000, // default: 5000
       // The default exporter used is OTLP over gRPC
       endpoint: 'http://collector:4317',
     },
   });

   const meter = metrics.getMeter('my-meter');
   const counter = meter.createCounter('clicks');
   counter.add(3);

Set up custom metric readers and exporters
----------------------------------------------------

You can provide custom exporters and readers using the ``metricReaderFactory`` setting.

.. caution:: Usage of ``metricReaderFactory`` invalidates the ``exportInterval`` and ``endpoint`` settings.

The following example shows how to provide a custom exporter:

.. code-block:: javascript

   const { start } = require('@splunk/otel');
   const { PrometheusExporter } = require('@opentelemetry/exporter-prometheus');
   const { OTLPMetricExporter } = require('@opentelemetry/exporter-metrics-otlp-http');
   const { PeriodicExportingMetricReader } = require('@opentelemetry/sdk-metrics-base');

   start({
     serviceName: 'my-service',
     metrics: {
       metricReaderFactory: () => {
         return [
           new PrometheusExporter(),
           new PeriodicExportingMetricReader({
             exportIntervalMillis: 1000,
             exporter: new OTLPMetricExporter({ url: 'http://localhost:4318' })
           })
         ]
       },
     },
   });

Select the type of aggregation temporality
--------------------------------------------

Aggregation temporality describes how data is reported over time.

You can define two different aggregation temporalities:

- ``AggregationTemporality.CUMULATIVE``: Cumulative metrics, such as counters and histograms, are continuously summed together from a given starting point, which in this case is set with the call to ``start``. This is the default temporality.
- ``AggregationTemporality.DELTA``: Metrics are summed together relative to the last metric collection step, which is set by the export interval.

To configure aggregation temporality in your custom metrics, use ``AggregationTemporality`` as in the example:

.. code-block:: javascript

   const { start } = require('@splunk/otel');
   const { OTLPMetricExporter } = require('@opentelemetry/exporter-metrics-otlp-grpc');
   const { AggregationTemporality, PeriodicExportingMetricReader } = require('@opentelemetry/sdk-metrics-base');

   start({
     serviceName: 'my-service',
     metrics: {
       metricReaderFactory: () => {
         return [
           new PeriodicExportingMetricReader({
             exporter: new OTLPMetricExporter({
               temporalityPreference: AggregationTemporality.DELTA
             })
           })
         ]
       },
     },
   });

For more information on aggregation temporality, see :new-page:`https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/metrics/data-model.md#sums <https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/metrics/data-model.md#sums>` on GitHub.
