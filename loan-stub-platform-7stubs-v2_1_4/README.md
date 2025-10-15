
# Loan Stub Platform (7 Services) – v2.1.4

**What's new**
- Collector exports to **Jaeger via OTLP (gRPC 4317)** and prints spans via **debug exporter**.
- Services send spans to Collector at **`otel-collector:4317`**.
- Responses now include **`traceId`** so you can copy-paste into Jaeger.

## Run
```bash
docker compose up -d --build
# Watch the RS setup until it prints "PRIMARY ready"
docker compose logs -f mongo-rs-setup
```

## Readiness
```bash
for p in 7001 7002 7003 7004 7005 7006 7007; do
  echo "----- $p"; curl -s http://localhost:$p/ready; echo; done
```

## Trigger traces (use POST + plain `&`)
```bash
# Happy path chain
curl -s -X POST "http://localhost:7001/process?loanId=L-10001&amount=50000&fail=false" | jq .

# Failure at Credit Score
curl -s -X POST "http://localhost:7003/process?loanId=L-FAIL01&amount=10000&fail=true" | jq .
```

## View traces & get the **end-to-end trace ID**
- **From API response**: The JSON includes `traceId`. That is the end-to-end ID for the whole trace.
- **In Jaeger**: Open http://localhost:16686 → Search Service = `loan-application-service`, Lookback = Last 1h, or directly open:
  - `http://localhost:16686/trace/<TRACE_ID>`
- **From Collector logs** (debug exporter):
```bash
docker compose logs -f otel-collector | egrep -i "ResourceSpans|trace_id|Span|export"
```

## Notes
- Jaeger accepts OTLP natively on 4317/4318. We export via OTLP from the Collector.
- Each stub uses OTel FastAPI + Requests instrumentation; W3C context is propagated downstream.
