# Lazers Go SDK

Server-side Go SDK for Lazers Signal Protocol v1. Requires Go 1.22+ and uses only the standard library.

Repository: [https://github.com/Emmydapson/lazers](https://github.com/Emmydapson/lazers)

> **Note on Pilot Ownership:** This package is currently maintained under Emmanuel Ilori's repository (`github.com/Emmydapson/lazers`) as a nested monorepo module. Future releases may migrate to dedicated organization ownership.

## Installation

```bash
go get github.com/Emmydapson/lazers/packages/go-sdk@v0.1.0
```

For nested monorepo modules, Go module semantic version tags follow the standard format:
`packages/go-sdk/v0.1.0`.

## Quickstart

```go
package main

import (
	"context"
	"os"

	lazers "github.com/Emmydapson/lazers/packages/go-sdk"
)

func main() {
	client, err := lazers.NewClient(lazers.Config{
		APIKey:      os.Getenv("LAZERS_API_KEY"),
		Endpoint:    "https://api.lazers.app",
		ProjectID:   "project_1",
		Environment: "production",
		ServiceName: "api",
	})
	if err != nil {
		panic(err)
	}

	client.CaptureError(err, lazers.CaptureContext{
		Actor: map[string]any{"type": "user", "id": "user_1"},
	})

	_ = client.Close(context.Background())
}
```

Capture methods enqueue and return without delivery errors. A single worker owns delivery; it safely handles concurrent captures, drops the oldest item on capacity overflow, and `Flush(ctx)` waits for its queue or context cancellation. `Close(ctx)` flushes then stops the worker.

## HTTP server example

See [`examples/http/main.go`](examples/http/main.go): it initializes one client, captures panic recovery with a request actor, and calls `Close` during graceful shutdown.

Gin, Fiber, and Echo use the same pattern: create one process-level client, recover/report errors through their error middleware, pass only stable actor IDs, and call `Close` from the server shutdown hook. No framework middleware package is shipped yet.

## Security

Server-side only. Never expose the secret project API key in browser code, Flutter/mobile, frontend bundles, or any client application. These platforms need future client-safe credentials/DSNs.

## Validation

Run `go fmt ./...`, `go vet ./...`, `go test ./...`, and `go test -race ./...` from this directory.
