# Week 2 — Distributed Tracing

- I signed up on honeycomb and created an environment called bootcamp
- I got my api key and i exported it as an environment variable 
- export HONEYCOMB_API_KEY=redacted
- gp env HONEYCOMB_API_KEY=redacted

- I configured OTEL (open telemetry) to send to honeycomb
```
      OTEL_SERVICE_NAME: "backend-flask"
      OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
      OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
```
 
- I Installed these packages on my terminal to instrument a Flask app with OpenTelemetry:
```
pip install opentelemetry-api \
    opentelemetry-sdk \
    opentelemetry-exporter-otlp-proto-http \
    opentelemetry-instrumentation-flask \
    opentelemetry-instrumentation-requests
```
  
- I added the following to my requirement.txt file
```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```

- I installed the dependencies using 
`pip install -r requirements.txt` 

- I added the following to apps.py from Honeycomb
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


- I ran npm install nmp i then docker compose up


- I copied the codes from https://docs.honeycomb.io/getting-data-in/opentelemetry/python/  to create spans in home.activities file

![honeycomb-trace](assets/trace.png)  
![honeycomb-span](assets/spans.png)  


#### AWS XRAY
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

 - Then I created xray on the terminal using code below 

 ```
  xray create-group \
   --group-name "Cruddur" \
 --filter-expression "service(\"backend-flask\")"
```
![aws-tray](assets/xray.png)  

- I created sampling rules with the code
`aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json`

- I added  xray-daemon to my docker compose file 
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
- I also added these environment variables to my docker compose file
```
      AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
      AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```
- I edited the app.py file to include xray

```
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='Cruddur', dynamic_naming=xray_url)
XRayMiddleware(app, xray_recorder)
```
- I ran docker compose up and xray traces was successfully sent
![aws-tray](assets/traces.png)  
![aws-tray](assets/trace1.png)  
![aws-tray](assets/tracemap.png)  
![aws-tray](assets/tracemap2.png)  
