---
created: 2026-03-22T14:17:16+08:00
---

# proto

### protoc

protoc --version

### protoc-gen-go

protoc-gen-go --version

```bash
protoc --proto_path=./internal --proto_path=./third_party --go_out=paths=source_relative:./internal internal/conf/conf.proto
```

