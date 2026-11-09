# from: golang:1.26-alpine
# WORKDIR /app
# COPY go.mod go.sum ./
# RUN go mod download
# COPY . .
# RUN go build -o server .

# FROM golang:1.22-alpine AS builder
# WORKDIR /app
# COPY . .
# RUN go build -o server .

# FROM alpine
# COPY --from=builder /app/server /server
# EXPOSE 8080
# CMD ["/server"]

# build server
FROM golang:1.27-bookworm AS server-builder

WORKDIR /app

COPY fider/go.mod fider/go.sum ./

RUN go mod download 

COPY fider/ .

RUN make build-server

# Build UI
FROM node:24-bookworm AS ui-builder

WORKDIR /app

COPY fider/package.json fider/package-lock.json ./

RUN npm ci

COPY fider/ .

RUN make build-ssr
RUN make build-ui  

#runtime step
FROM gcr.io/distroless/cc-debian12:nonroot

WORKDIR /app 

COPY --from=server-builder /app/migrations ./migrations
COPY --from=server-builder /app/views ./views
COPY --from=server-builder /app/locale ./locale
COPY --from=server-builder /app/LICENSE ./LICENSE
COPY --from=server-builder /app/fider ./fider
COPY --from=server-builder /app/static  ./static

COPY --from=ui-builder /app/favicon.png /app
COPY --from=ui-builder /app/dist /app/dist
COPY --from=ui-builder /app/robots.txt /app
COPY --from=ui-builder /app/ssr.js /app

EXPOSE 3000

HEALTHCHECK --timeout=5s CMD [ "./fider" "ping"]

ENTRYPOINT [ "./fider" ]