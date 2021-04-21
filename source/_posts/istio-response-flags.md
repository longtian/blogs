---
title: Istio 请求响应标志
date: 2021-04-21 15:40:35
tags:
  - istio
---

整理了一下 Istio 的 Response Flags

[介绍](https://www.envoyproxy.io/docs/envoy/latest/configuration/observability/access_log/usage#config-access-log-format-response-flags)

[源码](https://github.com/envoyproxy/envoy/blob/v1.18.2/source/common/stream_info/utility.h#L21)

协议       | 缩写   | 备注                              | 备注
----------|-------|-----------------------------------|---------------------------
HTTP      | DC    | DOWNSTREAM_CONNECTION_TERMINATION | Downstream connection termination.
HTTP	  | DI    | DELAY_INJECTED                    | The request processing was delayed for a period specified via fault injection.
HTTP	  | DPE   | DOWNSTREAM_PROTOCOL_ERROR         | The downstream request had an HTTP protocol error.
HTTP	  | FI    | FAULT_INJECTED                    | The request was aborted with a response code specified via fault injection.
HTTP	  | IH    | INVALID_ENVOY_REQUEST_HEADERS     | The request was rejected because it set an invalid value for a strictly-checked header in addition to 400 response code.
HTTP	  | LH    | FAILED_LOCAL_HEALTH_CHECK         | Local service failed health check request in addition to 503 response code.
HTTP	  | LR    | LOCAL_RESET                       | Connection local reset in addition to 503 response code.
HTTP/TCP  | NC    | NO_CLUSTER_FOUND                  | Upstream cluster not found.
HTTP/TCP  | NR    | NO_ROUTE_FOUND                    | No route configured for a given request in addition to 404 response code, or no matching filter chain for a downstream connection.
HTTP      | RL    | RATE_LIMITED                      | The request was ratelimited locally by the HTTP rate limit filter in addition to 429 response code.
HTTP	  | RLSE  | RATELIMIT_SERVICE_ERROR           | The request was rejected because there was an error in rate limit service.
HTTP	  | SI    | STREAM_IDLE_TIMEOUT               | Stream idle timeout in addition to 408 response code.
HTTP      | UAEX  | UNAUTHORIZED_EXTERNAL_SERVICE     | The request was denied by the external authorization service.
HTTP      | UC    | UPSTREAM_CONNECTION_TERMINATION   | Upstream connection termination in addition to 503 response code.
HTTP/TCP  | UF    | UPSTREAM_CONNECTION_FAILURE       | Upstream connection failure in addition to 503 response code.
HTTP/TCP  | UH    | NO_HEALTHY_UPSTREAM               | No healthy upstream hosts in upstream cluster in addition to 503 response code.
HTTP      | UMSDR | UPSTREAM_MAX_STREAM_DURATION_REACHED | The upstream request reached to max stream duration.
HTTP/TCP  |	UO    | UPSTREAM_OVERFLOW                 | Upstream overflow (circuit breaking) in addition to 503 response code.
HTTP	  | UPE	  | UPSTREAM_PROTOCOL_ERROR           | The upstream response had an HTTP protocol error.
HTTP	  | UR	  | UPSTREAM_REMOTE_RESET             | Upstream remote reset in addition to 503 response code.
HTTP/TCP  | URX   | UPSTREAM_RETRY_LIMIT_EXCEEDED     | The request was rejected because the upstream retry limit (HTTP) or maximum connect attempts (TCP) was reached.
HTTP	  | UT	  | UPSTREAM_REQUEST_TIMEOUT          | Upstream request timeout in addition to 504 response code.
