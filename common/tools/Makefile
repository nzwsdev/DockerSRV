define USAGE_TXT
Create Project (DIR) from template:
  * make snet=feapp sname=project_name create-service-dir-dev|create-service-dir-sys
Create .env file:
  * make create-dotenv-prod|create-dotenv-dev
endef
export USAGE_TXT


usage:
	@echo "$$USAGE_TXT"


## include /nzws/lib/makefile/user.mk


SERVIVE_NET := $(snet)
SERVIVE_NAME := $(sname)
SERVICE_FULL_NAME := $(SERVIVE_NET)-$(SERVIVE_NAME)


define checkDockerServiceNeNetAndName
	@if [[ -z "$(SERVIVE_NET)" ]];then echo '-> Docker service network must be defined...'; exit 1;fi
	@if [[ -z "$(SERVIVE_NAME)" ]];then echo '-> Docker service name must be defined...'; exit 2;fi
endef


create-service-dir-dev:
	@echo "Init service dir: '$(SERVICE_FULL_NAME)'"
	$(call checkDockerServiceNeNetAndName)
	@if [[ -e "$(SERVICE_FULL_NAME)" ]];then echo '-> Docker service is exists ($(SERVICE_FULL_NAME))...'; exit 3;fi
	@cp -av ./tmpl_networkapp-service_name $(SERVICE_FULL_NAME)
	SERVIVE_NAME_UC=$(SERVIVE_NAME); SERVIVE_NAME_UC=$${SERVIVE_NAME_UC:u}; \
		find $(SERVICE_FULL_NAME) -type f \
			-exec sed -i.tmpl 's/__TMPL_SERVICE_NET__/$(SERVIVE_NET)/g' {} \; \
			-exec sed -i.tmpl "s/__TMPL_SERVICE_NAME_UC__/$${SERVIVE_NAME_UC}/g" {} \; \
			-exec sed -i.tmpl 's/__TMPL_SERVICE_NAME__/$(SERVIVE_NAME)/g' {} \; \
			-exec rm -v {}.tmpl \;
	$(call createDockerServiceDirectory,$(SERVICE_FULL_NAME),dev)


create-service-dir-sys:
	@echo "Init service dir: '$(SERVICE_FULL_NAME)'"
	$(call checkDockerServiceNeNetAndName)
	$(call createDockerServiceDirectory,$(SERVICE_FULL_NAME),sys)


create-dotenv-prod: .base.env
	@echo "Create .env file from:"
	@echo " - .base.env"
	@cat .base.env > .env;
	@for _de in $$(find ./ -maxdepth 1 -type f -and -name '*.env' -not -name '.env' -not -name '.base.env' -not -path '*.dev*');do \
		echo " - $$_de"; \
		echo "\n\n# $${_de}" >> .env; \
		cat $$_de >> .env; \
	done
	@for _de in $$(find ./ -maxdepth 1 -type l -and -name '*.env' -not -name '.env' -not -name '.base.env' -not -path '*.dev*');do \
		_de=$$(readlink -n "$$_de"); \
		echo " - $$_de"; \
		echo "\n\n# $${_de}" >> .env; \
		cat $$_de >> .env; \
	done
	@echo
	@echo "Check"
	docker-compose config


create-dotenv-dev: .base.env
	@echo "Create .env file from:"
	@echo " - .base.env"
	@cat .base.env > .env;
	@for _de in $$(find ./ -maxdepth 1 -type f -and -name '*.env' -not -name '.env' -not -name '.base.env');do \
		echo " - $$_de"; \
		echo "\n\n# $${_de}" >> .env; \
		cat $$_de >> .env; \
	done
	@for _de in $$(find ./ -maxdepth 1 -type l -and -name '*.env' -not -name '.env' -not -name '.base.env');do \
		_de=$$(readlink -n "$$_de"); \
		echo " - $$_de"; \
		echo "\n\n# $${_de}" >> .env; \
		cat $$_de >> .env; \
	done
	@echo
	@echo "Check"
	docker-compose config
