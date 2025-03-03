TEST?=$$(go list ./...)
GOFMT_FILES?=$$(find . -name '*.go')
WEBSITE_REPO=github.com/hashicorp/terraform-website
PKG_NAME=cloudflare
VERSION=$(shell git describe --tags --always)
DEV_VERSION=99.0.0
CLOUDFLARE_GO_VERSION?=master

default: build

build: vet fmtcheck
	go install -ldflags="-X github.com/cloudflare/terraform-provider-cloudflare/main.version=$(VERSION)"

sweep:
	@echo "WARNING: This will destroy infrastructure. Use only in development accounts."
	go test $(TEST) -v -sweep=$(SWEEP) $(SWEEPARGS)

test: fmtcheck
	go test -i $(TEST) || exit 1
	echo $(TEST) | \
		xargs -t -n4 go test $(TESTARGS) -timeout=30s -parallel=4 -race

testacc: fmtcheck
	TF_ACC=1 go test $(TEST) -v $(TESTARGS) -timeout 120m -parallel 1

lint: tools terraform-provider-lint golangci-lint

terraform-provider-lint: tools
	$$(go env GOPATH)/bin/tfproviderlintx \
	 -R003=false \
	 -R012=false \
	 -S006=false \
	 -S014=false \
	 -S020=false \
	 -S022=false \
	 -S023=false \
	 -AT001=false \
	 -AT002=false \
	 -AT003=false \
	 -AT006=false \
	 -AT012=false \
	 -R013=false \
	 -XAT001=false \
	 -XR001=false \
	 -XR003=false \
	 -XR004=false \
	 -XS001=false \
	 -XS002=false \
	 ./internal/provider/

vet:
	@echo "==> Running go vet ."
	@go vet ./... ; if [ $$? -ne 0 ]; then \
		echo ""; \
		echo "Vet found suspicious constructs. Please check the reported constructs"; \
		echo "and fix them if necessary before submitting the code for review."; \
		exit 1; \
	fi

fmt:
	gofmt -w $(GOFMT_FILES)

test-compile:
	@if [ "$(TEST)" = "./..." ]; then \
		echo "ERROR: Set TEST to a specific package. For example,"; \
		echo "  make test-compile TEST=./$(PKG_NAME)"; \
		exit 1; \
	fi
	go test -c $(TEST) $(TESTARGS)

website:
ifeq (,$(wildcard $(GOPATH)/src/$(WEBSITE_REPO)))
	echo "$(WEBSITE_REPO) not found in your GOPATH (necessary for layouts and assets), get-ting..."
	git clone https://$(WEBSITE_REPO) $(GOPATH)/src/$(WEBSITE_REPO)
endif
	ln -sf ../../../../ext/providers/cloudflare/website/docs $(GOPATH)/src/github.com/hashicorp/terraform-website/content/source/docs/providers/cloudflare
	ln -sf ../../../ext/providers/cloudflare/website/cloudflare.erb $(GOPATH)/src/github.com/hashicorp/terraform-website/content/source/layouts/cloudflare.erb
	@$(MAKE) -C $(GOPATH)/src/$(WEBSITE_REPO) website-provider PROVIDER_PATH=$(shell pwd) PROVIDER_NAME=$(PKG_NAME)

website-test:
ifeq (,$(wildcard $(GOPATH)/src/$(WEBSITE_REPO)))
	echo "$(WEBSITE_REPO) not found in your GOPATH (necessary for layouts and assets), get-ting..."
	git clone https://$(WEBSITE_REPO) $(GOPATH)/src/$(WEBSITE_REPO)
endif
	ln -sf ../../../../ext/providers/cloudflare/website/docs $(GOPATH)/src/github.com/hashicorp/terraform-website/content/source/docs/providers/cloudflare
	ln -sf ../../../ext/providers/cloudflare/website/cloudflare.erb $(GOPATH)/src/github.com/hashicorp/terraform-website/content/source/layouts/cloudflare.erb
	@$(MAKE) -C $(GOPATH)/src/$(WEBSITE_REPO) website-provider-test PROVIDER_PATH=$(shell pwd) PROVIDER_NAME=$(PKG_NAME)

clean-dev:
	@echo "==> Removing development version ($(DEV_VERSION))"
	@rm -f terraform-provider-cloudflare_$(DEV_VERSION)

build-dev: clean-dev
	@echo "==> Building development version ($(DEV_VERSION))"
	go build -gcflags="all=-N -l" -o terraform-provider-cloudflare_$(DEV_VERSION)

generate-changelog:
	@echo "==> Generating changelog..."
	@sh -c "'$(CURDIR)/scripts/generate-changelog.sh'"

golangci-lint:
	@golangci-lint run ./$(PKG_NAME)/... --config .golintci.yml

tools:
	@echo "==> Installing development tooling..."
	go generate -tags tools tools/tools.go

deps-check:
	@echo "==> Checking source code with go mod tidy..."
	@go mod tidy
	@git diff --exit-code -- go.mod go.sum || \
		(echo; echo "Unexpected difference in go.mod/go.sum files. Run 'go mod tidy' command or revert any go.mod/go.sum changes and commit."; exit 1)

update-go-client:
	@echo "==> Updating the cloudflare-go client to $(CLOUDFLARE_GO_VERSION)"
	go get github.com/cloudflare/cloudflare-go@$(CLOUDFLARE_GO_VERSION)
	go mod tidy

docs: tools
	@sh -c "'$(CURDIR)/scripts/generate-docs.sh'"

check-docs: docs
	@if [ "`git status --porcelain docs/`" ]; then \
	    git diff; \
		echo "Uncommitted changes were detected in the autogenerated docs folder. Please run 'make docs' to autogenerate the docs, and commit the changes" && echo `git status --porcelain docs/` && exit 1; \
	else \
		echo "Success: No generated documentation changes detected"; \
	fi

.PHONY: build test sweep testacc lint terraform-provider-lint vet fmt fmtcheck errcheck test-compile website website-test build-dev clean-dev generate-changelog golangci-lint tools deps-check update-go-client docs check-docs
