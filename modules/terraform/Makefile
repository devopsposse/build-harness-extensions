TMP ?= /tmp
TERRAFORM ?= $(BUILD_HARNESS_PATH)/vendor/terraform
TERRAFORM_VERSION ?= 1.0.11
TERRAFORM_URL ?= https://releases.hashicorp.com/terraform/$(TERRAFORM_VERSION)/terraform_$(TERRAFORM_VERSION)_$(OS)_$(BUILD_HARNESS_ARCH).zip

## Install terraform
terraformz/install:
	@[ -x $(TERRAFORM) ] || ( \
		echo "Installing Terraform $(TERRAFORM_VERSION) ($(OS)) from $(TERRAFORM_URL)" && \
		curl '-#' -fL -o $(TMP)/terraform.zip $(TERRAFORM_URL) && \
		unzip -q -d $(TMP)/ $(TMP)/terraform.zip && \
		mv $(TMP)/terraform $(TERRAFORM) && \
		rm -f $(TMP)/terraform.zip \
		)
	$(TERRAFORM) version

## (Obsolete) Ensure all plugins can be fetched
terraformz/get-plugins:
	@echo terraform -get-plugins no longer supported

## (Obsolete) Ensure all modules can be fetched
terraformz/get-modules:
	@$(TERRAFORM) init -get -backend=false -input=false >/dev/null

## Basic terraform sanity check
terraformz/validate:
	@$(TERRAFORM) validate

## Lint check Terraform
terraformz/lint:
ifeq ($(OS), darwin)
	@FAIL=`$(TERRAFORM) fmt -write=false | xargs -n 1 printf '\t- %s\n'`; \
	[ -z "$$FAIL" ] || (echo "Terraform configuration needs linting. Run '$(TERRAFORM) fmt'"; echo $$FAIL; exit 1)
else
	@FAIL=`$(TERRAFORM) fmt -write=false | xargs --no-run-if-empty -n 1 printf '\t- %s\n'`; \
	[ -z "$$FAIL" ] || (echo "Terraform configuration needs linting. Run '$(TERRAFORM) fmt'"; echo $$FAIL; exit 1)
endif

terraformz/fmt:
ifeq ($(wildcard *.tf),)
	@echo "* $@: No terraform files detected"
else
	@$(TERRAFORM) fmt -recursive .
endif

## Upgrade all terraform module sources
## This target has not been upgraded to handle registry format
terraformz/upgrade-modules:
	@echo "* $@ has been disabled because it has not been updated to handle Terraform registry syntax"
	# @$(BUILD_HARNESS_PATH)/bin/upgrade_terraform_modules.sh all

# Rewrite the *.tf files to use registry notation for modules sources
terraformz/rewrite-module-source: TERRAFORM = terraform-0.13
terraformz/rewrite-module-source:
	@sed -i -E 's,\s*source\s+=\s+"git::https://github.com/([^/]+)/terraform-([^-]+)-(.+).git\?ref=(tags/)?([0-9.]+)",  source = "\1/\3/\2"\n  version     = "\5",g' $$(find . -type f -not -name context.tf -name '*.tf')
	@$(TERRAFORM) fmt .
	@$(TERRAFORM) fmt examples/complete

terraformz/rewrite-readme-source: TERRAFORM = terraform-0.13
terraformz/rewrite-readme-source:
	@sed -i -E 's,^(\s*)source\s+=\s+"git::https://github.com/([^/]+)/terraform-([^-]+)-(.+).git\?ref=(tags/)?master",\1source = "\2/\4/\3"\n\1# Cloud Posse recommends pinning every module to a specific version\n\1# version = "x.x.x",g' README.yaml

## Rewrite versions.tf to remove upper bound for terraform core version constraint (like this ">= 0.12.0, < 0.14.0")
## and convert "~>" constraints to ">=".
terraformz/loosen-constraints: TERRAFORM = terraform-0.13
terraformz/loosen-constraints:
	@for v in $$(find . -type f -name 'versions.tf'); do \
		DIR="$$(dirname $$v)"; \
		sed -i -E 's,\s\=\s*\"(.*[0-9.]+)(\,\s*<.*)", = "\1",g' "$$v" ; \
		sed -i -E 's,=\s*"\s*~>,= ">=,g' "$$v" ; \
		$(TERRAFORM) fmt "$$DIR" ; \
	done

## Rewrite versions.tf to bump modules with minimum core version of '0.12.x' to '>= 0.12.26'
terraformz/bump-tf-12-min-version: TERRAFORM = terraform-0.13
terraformz/bump-tf-12-min-version: terraformz/loosen-constraints
	@for v in $$(find . -type f -name 'versions.tf'); do \
		DIR="$$(dirname $$v)"; \
		PIN="$$(terraform-config-inspect --json "$$DIR" | jq -r '.required_core[]')" ; \
		if vert "$$PIN" 0.12.25 >/dev/null; then \
			echo updating "$$v" ; \
			sed -i -E 's,required_version\s*\=\s*\"(\~>|>\=)\s?(0\.12(\.\d+)?)\",required_version = ">= 0.12.26",g' "$$v" ; \
			$(TERRAFORM) fmt $$DIR ; \
		else \
			echo "$$v" already pinned to "$$PIN" ; \
		fi; \
	done

## Rewrite versions.tf to update existing configuration to add an explicit source attribute for each provider
terraformz/rewrite-required-providers: TERRAFORM = terraform-0.13
terraformz/rewrite-required-providers: terraformz/loosen-constraints
	@for v in $$(find . -type f -name 'versions.tf'); do \
		DIR="$$(dirname $$v)" ; \
		KEEP_VERSION=$$(grep -Eo 'required_version\s*=\s*"(.*)"' "$$v" | cut -d '"' -f2) ; \
		echo upgrading $$v ; \
		terraform-0.13 0.13upgrade -yes "$$DIR" >/dev/null ; \
		sed -i -E 's,(required_version\s*\=\s*\")(.*)(\"),\1'"$$KEEP_VERSION"'\3,g' "$$v" ; \
	done

terraformz/v14-rewrite: TERRAFORM = terraform-0.13
terraformz/v14-rewrite: terraformz/loosen-constraints terraformz/bump-tf-12-min-version terraformz/rewrite-required-providers terraformz/rewrite-module-source terraformz/rewrite-readme-source
	@{ [[ "$(TERRAFORM_FORCE_README)" != "true" ]] && git diff --no-patch --exit-code README.yaml; } || $(MAKE) readme
