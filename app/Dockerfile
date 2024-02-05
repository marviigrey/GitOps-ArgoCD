FROM golang:latest

WORKDIR /app
COPY go.mod ./
RUN go mod download

COPY *.go ./

RUN CGO_ENABLED=0 GOOS=linux go build -o main .

CMD ["./main"]
