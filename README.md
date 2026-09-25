# Lazers Go SDK

Private server-side Go SDK for Signal Protocol v1. Requires Go 1.22+ and only uses the standard library.

```go
client, err := lazers.NewClient(lazers.Config{APIKey: "...", Endpoint: "https://api.lazers.app", ProjectID: "project_1", Environment: "production", ServiceName: "api"})
client.CaptureError(err, lazers.CaptureContext{Actor: map[string]any{"type":"user", "id":"user_1"}})
_ = client.Close(context.Background())
```

Capture methods enqueue and return without delivery errors. A single worker owns delivery; it safely handles concurrent captures, drops the oldest item on capacity overflow, and `Flush(ctx)` waits for its queue or context cancellation. `Close(ctx)` flushes then stops the worker.

## HTTP server example

See [`examples/http/main.go`](examples/http/main.go): it initializes one client, captures panic recovery with a request actor, and calls `Close` during graceful shutdown.

Gin, Fiber, and Echo use the same pattern: create one process-level client, recover/report errors through their error middleware, pass only stable actor IDs, and call `Close` from the server shutdown hook. No framework middleware package is shipped yet.

## Security

Server-side only. Never expose the secret project API key in browser code, Flutter/mobile, frontend bundles, or any client application. These platforms need future client-safe credentials/DSNs.

## Validation

Run `go fmt ./...`, `go vet ./...`, `go test ./...`, and `go test -race ./...` from this directory. This workspace currently has no Go toolchain, so those commands could not be run here.