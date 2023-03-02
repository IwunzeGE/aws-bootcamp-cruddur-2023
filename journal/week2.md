# Week 2 — Distributed Tracing

## Honeycomb

- Create a honeycomb.io account
- Create an environment and grab the API keys

![create environment](https://user-images.githubusercontent.com/110903886/222331034-c27f04cb-657f-4a26-a0f7-6335ec85b0cb.png)

- Set your ENV variables

```
export HONEYCOMB_API_KEY="Your API Key from the created environment"
export HONEYCOMB_SERVICE_NAME="Cruddur"
gp env HONEYCOMB_API_KEY=""
gp env HONEYCOMB_SERVICE_NAME="Cruddur"

```

![set env variables](https://user-images.githubusercontent.com/110903886/222331867-4eb99821-951d-4ee3-ba58-0544707ae39a.png)

- Add the following Env Vars to `backend-flask` in docker compose

```
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
OTEL_SERVICE_NAME: "${HONEYCOMB_SERVICE_NAME}"
```

- Add the following files to our requirements.txt

```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```

We'll install these dependencies:
`cd backend-flask`

`pip install -r requirements.txt`

- Initialize by adding to your `app.py` file

```
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)

# Initialize automatic instrumentation with Flask
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```

- Docker compose up and hit some api endpoint on the backend 

- Check your honecomb datasets to see if there is data

![honeycomb data](https://user-images.githubusercontent.com/110903886/222351330-39aa902b-ea00-4a10-9768-fe942ec4d237.png)

- We could run a trace for each api call recorded and see more details

![trace api](https://user-images.githubusercontent.com/110903886/222353091-ddda215f-3cac-4903-b389-24274cf03ff7.png)

## X-Ray
### Instrument AWS X-Ray for Flask

```
export AWS_REGION="us-east-1"
gp env AWS_REGION="us-east-1"
```

- Add to the requirements.txt
```
aws-xray-sdk
```

- Install pythonpendencies

`pip install -r requirements.txt`

- Add to app.py

```
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='backend-flask', dynamic_naming=xray_url)
XRayMiddleware(app, xray_recorder)
```

### Setup AWS X-Ray Resources

- Add aws/json/xray.json

```
{
  "SamplingRule": {
      "RuleName": "Cruddur",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "backend-flask",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}
```

- Create an Xray group

```
FLASK_ADDRESS="https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"backend-flask\")"
```

![create xray group](https://user-images.githubusercontent.com/110903886/222570339-58b70e6a-655f-4f72-82cd-8634ce35aa71.png)

![create xray group 2](https://user-images.githubusercontent.com/110903886/222570542-75343417-664b-42ed-81a0-e2ba049c5f06.png)

- Create a sampling rule

```
aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
```

![sampling rule 0](https://user-images.githubusercontent.com/110903886/222570957-e4873744-1d82-4d40-b24a-1329e54e0078.png)

![sampling rule](https://user-images.githubusercontent.com/110903886/222570615-ef67b475-d957-4ffe-baee-b80f8a17e12f.png)

