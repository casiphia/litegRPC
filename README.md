# LitegRPC

**LitegRPC** is a lightweight and extensible gRPC middleware framework for Go. It simplifies interceptor management and allows seamless integration of logging, authentication, and custom middleware with minimal overhead.

## Features
- **Middleware Management** – Easily register and manage unary and stream interceptors.
- **Extensibility** – Supports custom middleware for logging, authentication, tracing, and more.
- **Lightweight** – Minimal dependencies, designed for high performance.
- **Plug & Play** – Integrates smoothly with existing gRPC services.

## Installation
```sh
go get github.com/your-org/LitegRPC
```

## Quick Start

### 1. Create a gRPC Server with Middleware
```go
package main

import (
	"log"
	"net"
	"google.golang.org/grpc"
	"github.com/your-org/LitegRPC/middleware"
	"github.com/your-org/LitegRPC/middlewares"
	pb "path/to/protobuf"
)

// Service implementation
type myService struct {
	pb.UnimplementedMyServiceServer
}

func (s *myService) MyMethod(ctx context.Context, req *pb.MyRequest) (*pb.MyResponse, error) {
	return &pb.MyResponse{Message: "Hello, " + req.Name}, nil
}

func main() {
	listener, err := net.Listen("tcp", ":50051")
	if err != nil {
		log.Fatalf("Failed to listen: %v", err)
	}

	// Initialize middleware manager
	mm := middleware.NewMiddlewareManager()
	mm.UseUnary(middlewares.LoggingMiddleware)
	mm.UseUnary(middlewares.AuthMiddleware)

	// Create gRPC server with interceptors
	grpcServer := grpc.NewServer(
		grpc.UnaryInterceptor(mm.BuildUnaryInterceptor()),
	)

	// Register service
	pb.RegisterMyServiceServer(grpcServer, &myService{})

	log.Println("gRPC Server is running on port 50051...")
	if err := grpcServer.Serve(listener); err != nil {
		log.Fatalf("Failed to serve: %v", err)
	}
}
```

### 2. Define Middleware
#### Logging Middleware
```go
package middlewares

import (
	"context"
	"log"
	"google.golang.org/grpc"
)

func LoggingMiddleware(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (interface{}, error) {
	log.Printf("Request - Method: %s, Payload: %+v", info.FullMethod, req)
	resp, err := handler(ctx, req)
	log.Printf("Response - Method: %s, Response: %+v, Error: %v", info.FullMethod, resp, err)
	return resp, err
}
```

#### Authentication Middleware
```go
package middlewares

import (
	"context"
	"google.golang.org/grpc"
	"google.golang.org/grpc/metadata"
	"google.golang.org/grpc/status"
	"google.golang.org/grpc/codes"
)

func AuthMiddleware(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (interface{}, error) {
	md, ok := metadata.FromIncomingContext(ctx)
	if !ok {
		return nil, status.Errorf(codes.Unauthenticated, "Missing metadata")
	}
	tokens := md.Get("authorization")
	if len(tokens) == 0 || tokens[0] != "valid-token" {
		return nil, status.Errorf(codes.Unauthenticated, "Invalid token")
	}

	return handler(ctx, req)
}
```

## License
This project is licensed under the MIT License.

