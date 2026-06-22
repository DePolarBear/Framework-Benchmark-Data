# Framework Benchmark Data — FastAPI vs Django (x86 vs ARM)

Cross-architecture performance comparison of the **FastAPI** and **Django** web frameworks on **x86-64** and **ARM** hardware. The study measures HTTP throughput, latency distribution, resource usage, energy efficiency, and WebSocket round-trip latency under increasing concurrency.

This is an extension of an earlier study (v1) that compared the two frameworks on ARM only. Version v2 adds a second architecture (x86), a *tuned* Django variant, energy metrics, and a more accurate latency-measurement methodology.

## Test setup

**Server platforms:**

| Platform | CPU | Architecture |
|----------|-----|--------------|
| x86 server | AMD Ryzen 7 (8 c / 16 t) | x86-64 |
| ARM server | ARM Cortex-A76 (4 cores) | AArch64 |

**Frameworks:** FastAPI · Django (default middleware) · Django (tuned, no middleware) — all served via Uvicorn (12 workers).

**Endpoints:** `ping`, `hello`, `echo` (POST), `heavy` (CPU-bound), `ws` (WebSocket echo).

Load was generated from a separate client over wired gigabit Ethernet.

## Key results

HTTP throughput (requests/s, `ping` endpoint):

| Platform | FastAPI | Django tuned | Django default |
|----------|--------:|-------------:|---------------:|
| x86 | 27,898 | 8,115 | 3,227 |
| ARM | 4,761 | 1,415 | 549 |

**Main findings:**

1. **FastAPI leads on both architectures** — ~8.6x higher throughput than default Django, ~3.4x than tuned. The ratio is platform-independent.
2. **x86 vs ARM: a consistent ~5.5x gap** across all configurations.
3. **Middleware overhead costs ~2.5x** — removing middleware (tuned) roughly doubles to triples Django throughput.
4. **Energy efficiency is comparable** — x86 is ~6x faster but draws ~6x more power; per watt, both architectures are nearly equal.
5. **WebSocket: FastAPI scales better under high concurrency** — comparable up to 500 connections, but at 1,000 concurrent connections FastAPI keeps p95 round-trip at ~2 ms (x86) / ~5 ms (ARM) while Django rises to 8-23 ms.

## Repository structure

```
Framework Benchmark Data/
├── HTTP/              # HTTP results (summary + latency percentiles per config)
│   ├── ASUS_FASTAPI_summary.csv
│   ├── ASUS_DJANGO_summary.csv
│   ├── ASUS_DJANGO_TUNED_summary.csv
│   ├── RASPI_*_summary.csv
│   └── *_latency_percentiles.csv
├── WebSocket/         # WebSocket round-trip results per config
│   └── *_ws_summary.csv
└── console output/    # raw run logs
```

Naming convention: `PLATFORM_FRAMEWORK_type.csv` (e.g. `ASUS_FASTAPI_summary.csv`).

## Data columns

**HTTP summary** — `endpoint, method, requests_per_sec, success_ratio, latency_mean_ms, latency_max_ms, total_requests`

**HTTP latency percentiles** — `endpoint, p50_ms, p90_ms, p95_ms, p99_ms`

**WebSocket summary** — `framework, connections, msgs_per_s, rtt_avg_ms, rtt_p50_ms, rtt_p90_ms, rtt_p95_ms` (round-trip latency in milliseconds)

## Tools

| Tool | Purpose |
|------|---------|
| [vegeta](https://github.com/tsenart/vegeta) | HTTP load testing (constant rate, latency percentiles) |
| [k6](https://k6.io/) | WebSocket load testing (round-trip latency, concurrency) |
| [psutil](https://github.com/giampaolo/psutil) | CPU / RAM monitoring |

## Notes

- **Power on ARM is estimated**, not measured — the ARM board has no hardware power sensor. Efficiency figures for ARM use a 7-9 W estimate based on independent measurements and should be read as approximate.
- **CPU saturation differs** — the ARM server ran at 97-99% CPU (CPU-bound), the x86 server at ~80% (limited elsewhere), so x86 throughput may be a lower bound.
- Latency values use constant-rate load generation (coordinated-omission correction), so absolute values are higher but more representative than naive tools.
