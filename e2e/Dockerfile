FROM golang:1.23.2-bookworm@sha256:2341ddffd3eddb72e0aebab476222fbc24d4a507c4d490a51892ec861bdb71fc as builder

ENV KUBECTL_VERSION="v1.28.3"
ENV HELM_VERSION="v3.13.1"

RUN go install github.com/onsi/ginkgo/v2/ginkgo@v2.1.6
RUN wget -q https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl -O /usr/local/bin/kubectl && \
    chmod +x /usr/local/bin/kubectl && \
    wget -q https://get.helm.sh/helm-${HELM_VERSION}-linux-amd64.tar.gz -O - | tar -xzO linux-amd64/helm > /usr/local/bin/helm && \
    chmod +x /usr/local/bin/helm

WORKDIR /usr/src/app
COPY go.mod go.sum ./
RUN go mod download && go mod verify
COPY . .
WORKDIR /usr/src/app/e2e
RUN make e2e-bin

FROM alpine:3.20.3@sha256:beefdbd8a1da6d2915566fde36db9db0b524eb737fc57cd1367effd16dc0d06d
RUN apk add -U --no-cache \
    ca-certificates \
    bash \
    curl \
    tzdata \
    libc6-compat \
    openssl

COPY --from=builder /go/bin/ginkgo /usr/local/bin/
COPY --from=builder /usr/local/bin/kubectl /usr/local/bin/
COPY --from=builder /usr/local/bin/helm /usr/local/bin/

COPY --from=builder /usr/src/app/e2e/entrypoint.sh                   /entrypoint.sh
COPY --from=builder /usr/src/app/e2e/suites/provider/provider.test   /provider.test
COPY --from=builder /usr/src/app/e2e/suites/argocd/argocd.test       /argocd.test
COPY --from=builder /usr/src/app/e2e/suites/flux/flux.test           /flux.test
COPY --from=builder /usr/src/app/e2e/suites/generator/generator.test /generator.test
COPY --from=builder /usr/src/app/e2e/k8s                             /k8s

CMD [ "/entrypoint.sh" ]
