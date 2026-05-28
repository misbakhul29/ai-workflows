---
name: grpc-gateway
description: Automate the creation of a high-performance gRPC-Gateway in Go with REST to gRPC transpilation, Clean Architecture, auto-generated Swagger UI, and Rate Limiting.
---

# Go gRPC-Gateway Microservices Builder

**Trigger Keywords**: "buatkan grpc-gateway", "setup microservice golang gRPC", "api gateway golang", "rest ke grpc go".

## Tujuan
Workflow ini dirancang untuk memandu pembuatan proyek API Gateway berkinerja tinggi menggunakan Go dan `gRPC-Gateway v2`. Proyek akan menerjemahkan REST HTTP/JSON request menjadi gRPC call ke backend microservices, dilengkapi dengan Swagger UI dinamis dan middleware keamanan (Rate Limiting).

## Standar Arsitektur & Aturan
1. **Clean Architecture**: Pisahkan layer `cmd` (entry point), `internal` (logic/middleware), dan `proto` (definisi kontrak).
2. **Stateless Gateway**: Gateway tidak boleh menyimpan state dan hanya bertugas me-routing HTTP ke gRPC.
3. **No Direct HTTP Handlers for Microservices**: Semua service menggunakan gRPC murni. HTTP ditangani sepenuhnya oleh gRPC-Gateway.
4. **Structured Logging**: Gunakan logger terstruktur (misal: Uber Zap) untuk kemudahan pelacakan (*observability*).

## Struktur Direktori yang Disarankan
```text
├── cmd
│   ├── dev/gen_pb          # Generator protobuf & Swagger JSON otomatis
│   ├── gateway             # Entry point API Gateway (HTTP)
│   └── services/<name>     # Entry point untuk setiap gRPC Microservice
├── internal
│   ├── config              # Pemuat environment (.env)
│   ├── logger              # Wrapper logger (Zap)
│   ├── middleware          # Middleware gateway (Rate Limiter, CORS)
│   └── services
│       └── api             # Implementasi fungsi server gRPC
├── pb                      # Berkas hasil generate (jangan diedit manual)
├── proto                   # Definisi .proto dan konfigurasi swagger (docs.json)
└── third_party             # Dependensi ekstensi proto (Google APIs)
```

## Langkah Implementasi (Workflow)

### Langkah 1: Inisialisasi Proyek & Dependensi
1. Jalankan inisialisasi module: `go mod init <module-name>`
2. Unduh dependensi utama:
   ```bash
   go get github.com/grpc-ecosystem/grpc-gateway/v2/runtime
   go get google.golang.org/grpc
   go get google.golang.org/protobuf
   go get go.uber.org/zap
   go get github.com/joho/godotenv
   ```

### Langkah 2: Setup Third-Party Proto (Google APIs)
gRPC-Gateway membutuhkan ekstensi proto dari Google untuk mapping HTTP.
1. Buat direktori `third_party/google/api`
2. Unduh file proto yang wajib:
   ```bash
   curl -sSL https://raw.githubusercontent.com/googleapis/googleapis/master/google/api/annotations.proto -o third_party/google/api/annotations.proto
   curl -sSL https://raw.githubusercontent.com/googleapis/googleapis/master/google/api/http.proto -o third_party/google/api/http.proto
   ```

### Langkah 3: Penulisan Berkas Proto
1. Buat file konvensi utama (contoh: `proto/user.proto`).
2. Pastikan menyertakan import berikut:
   ```protobuf
   import "google/api/annotations.proto";
   import "protoc-gen-openapiv2/options/annotations.proto";
   ```
3. Definisikan mapping HTTP menggunakan `option (google.api.http)` pada setiap RPC.

### Langkah 4: Skrip Generator Otomatis (gen_pb)
Buat file `cmd/dev/gen_pb/main.go` yang akan mengeksekusi `protoc` secara terprogram via `os/exec`.
Parameter wajib `protoc` yang perlu ditambahkan:
- `-I.`, `-Ithird_party`, dan include path dari modul `grpc-gateway`.
- `--go_out=.`, `--go-grpc_out=.`, `--grpc-gateway_out=.`
- `--openapiv2_out=allow_merge=true,merge_file_name=docs:proto`

### Langkah 5: Implementasi Microservice (Backend gRPC)
1. Buat struct server di `internal/services/api/<nama_service>.go` yang mengimplementasikan interface generated server.
2. Buat entry point server gRPC di `cmd/services/<nama_service>/main.go` yang mem-binding listener TCP (misal port 50051) ke server gRPC.

### Langkah 6: Implementasi API Gateway (HTTP)
Buat entry point gateway di `cmd/gateway/main.go`:
1. Buat gRPC koneksi ke microservice (menggunakan `grpc.DialContext`).
2. Buat instance `runtime.NewServeMux()`.
3. Daftarkan service ke mux via fungsi register (misal `pb.RegisterUserServiceHandler`).
4. (Opsional) Tambahkan HTTP handler terpisah di dalam Mux utama untuk men-serve berkas statis `docs.json` dan merender Swagger UI.
5. Bungkus mux utama dengan middleware (Rate Limit, CORS, Logger).
6. Jalankan server HTTP (misal port 8080).

### Langkah 7: Pengujian & Verifikasi
1. Jalankan `go run ./cmd/dev/gen_pb/` untuk meng-generate proto.
2. Jalankan microservice `go run ./cmd/services/<nama_service>/`.
3. Jalankan gateway `go run ./cmd/gateway/`.
4. Uji via curl: `curl -X 'GET' 'http://localhost:8080/v1/resource'`.

## Checklist Kualitas Agen
Agen harus memverifikasi hal berikut sebelum menyelesaikan tugas:
- [ ] Protoc berhasil digenerate tanpa error import "google.api.http".
- [ ] Gateway server memproses Rate Limit dengan benar (Return `429 Too Many Requests`).
- [ ] Swagger UI merender definisi proto dengan benar pada endpoint `/docs/`.
- [ ] Tidak ada HTTP routing yang bypass gateway untuk endpoint gRPC.
