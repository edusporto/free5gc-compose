FROM golang:1.18.10-alpine AS builder

WORKDIR /build

# Copy the Go module files first to leverage Docker layer caching
COPY ./mnc_NWDAF/nwdaf/go.mod ./
COPY ./mnc_NWDAF/nwdaf/go.sum ./

RUN go mod download

COPY ./mnc_NWDAF/nwdaf/ .

RUN CGO_ENABLED=0 GOOS=linux go build -o nwdaf nwdaf.go


FROM alpine:3.15

LABEL description="NWDAF for Free5GC open source 5G Core Network"

WORKDIR /free5gc

RUN mkdir -p config/

COPY --from=builder /build/nwdaf .

RUN mkdir -p config/ log/ cert/

# Copy executable and default certs
# COPY --from=builder /free5gc/${F5GC_MODULE} ./
# COPY --from=builder /free5gc/cert/${F5GC_MODULE}.pem ./cert/
# COPY --from=builder /free5gc/cert/${F5GC_MODULE}.key ./cert/

# Config files volume
VOLUME [ "/free5gc/config" ]

# Certificates (if not using default) volume
VOLUME [ "/free5gc/cert" ]
