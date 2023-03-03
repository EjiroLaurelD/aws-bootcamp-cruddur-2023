# Week 2 — Distributed Tracing

I signed up on honeycomb and created an environment called bootcamp
I got my api key and i exported it as an environment variable 
export HONEYCOMB_API_KEY=redacted
gp env HONEYCOMB_API_KEY=redacted

I configured OTEL (open telemetry) to send to honeycomb
      ```
      OTEL_SERVICE_NAME: "backend-flask"
      OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
      OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
      ```
 
I Installed these packages to instrument a Flask app with OpenTelemetry on my terminal:
```
pip install opentelemetry-api \
    opentelemetry-sdk \
    opentelemetry-exporter-otlp-proto-http \
    opentelemetry-instrumentation-flask \
    opentelemetry-instrumentation-requests
```

I added the following to my requirement.txt file
```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```

I installed the dependencies using 
`pip install -r requirements.txt` 

I added the following to apps.py from Honeycomb
```
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
```

```
# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)
```

```
# Initialize automatic instrumentation with Flask
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```


I ran npm install nmp i
then docker compose up


I copied the codes from https://docs.honeycomb.io/getting-data-in/opentelemetry/python/  to create spans in home.activities file

![honeycomb-trace](assets/trace.png)  
![honeycomb-span](assets/span.png)  

cd into back-end flask , create xray.json file 

```
{
  "SamplingRule": {
      "RuleName": "Cruddur",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "Cruddur",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}
```
 Then I created xray on the terminal using code below
 ```
  xray create-group \
   --group-name "Cruddur" \
 --filter-expression "service(\"backend-flask\")"
```

 aws cli on my terminal and got this output
i got this output

![aws-tray](assets/xray.png)  

I created sampling rules with the code
`aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json`

I added  xray-daemon to my docker compose file 
```
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "us-east-1"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
```