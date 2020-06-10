# Elasticsearch docker building

## Requirement

- Docker

- Docker buildx (if you need to build it on AMD64 platform)

## Build

Use following command to build

```bash
docker buildx build . -t kopkop/elasticsearch --platform linux/arm64
```
