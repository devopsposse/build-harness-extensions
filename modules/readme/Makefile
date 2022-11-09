export README_LINT ?= $(TMP)/README.md
export README_FILE ?= README.md
export README_YAML ?= README.yaml
export README_TEMPLATE_FILE = $(BUILD_HARNESS_EXTENSIONS_PATH)/templates/README.md.gotmpl
export README_TEMPLATE_YAML := $(if $(findstring http,$(README_YAML)),$(README_YAML),$(BUILD_HARNESS_PATH)/templates/$(README_YAML))
export README_INCLUDES ?= $(file://$(shell pwd)/?type=text/plain)

## [Readme Custom] Alias for readmez/build
readmez: readmez/build
	@exit 0

readmez/deps: packages/install/gomplate
	@exit 0

## [Readme Custom] Create basic minimalistic .README.md template file
readmez/init:
	@if [ -f $(README_YAML) ]; then \
	  echo "$(README_YAML) already exists!"; \
	else \
	  cp $(README_TEMPLATE_YAML) $(README_YAML) ; \
	  echo "$(README_YAML) created!"; \
	fi;

## [Readme Custom] Verify the `README.md` is up to date
readmez/lint:
	@$(SELF) readme/build README_FILE=$(README_LINT) >/dev/null
	@diff -ruN $(README_LINT) $(README_FILE)
	@rm -f $(README_LINT)

## [Readme Custom] Create README.md by building it from README.yaml
readmez/build: readmez/deps $(README_DEPS)
	@gomplate --file $(README_TEMPLATE_FILE) \
		--out $(README_FILE) 
	@echo "Generated $(README_FILE) from $(README_TEMPLATE_FILE) using data from $(README_TEMPLATE_YAML)"

readmez/generate-related-references:
	@$(BUILD_HARNESS_PATH)/bin/generate_related_references.py
