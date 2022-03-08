# ENV
SHELL = /bin/zsh


define USAGE_TXT
List:
  * make ips-show-env|ips-show-list
Update/Download:
  * make ips-update
Clear:
  * make ips-clear
endef
export USAGE_TXT


usage:
	@echo "$$USAGE_TXT"

IP_LIST_BASE_DIR=./ips
IP_LIST_BASE_URL=https://www.cloudflare.com


ips-show-env:
	@find $(IP_LIST_BASE_DIR) -type f -name '.*.env' -exec printf "\n\n-> Dot env file(s): %s\n" {} \;  -exec cat {} \;


ips-show-list:
	@find $(IP_LIST_BASE_DIR) -type f -name '*.csv' -exec printf "\n\n-> IP list, csv file(s): %s\n" {} \;  -exec cat {} \;


ips-clear:
	@find $(IP_LIST_BASE_DIR) -type f -name '.*.env' -or -name '*.csv' -or -name '*.latest' | xargs rm -v;


ips-update:
	@local _ipListAllFile="$(IP_LIST_BASE_DIR)/cloudflare.ips.all.csv"; \
	local _ipEnvAllFile="$(IP_LIST_BASE_DIR)/.cloudflare.ips.all.env"; \
	:> "$${_ipListAllFile}"; \
	echo -n 'CLOUDFLARE_HOST_IPS=' > "$${_ipEnvAllFile}"; \
	for _ipt in v4 v6;do \
		local _ipListFile="$(IP_LIST_BASE_DIR)/cloudflare.ips.$${_ipt}.csv"; \
		local _ipEnvFile="$(IP_LIST_BASE_DIR)/.cloudflare.ips.$${_ipt}.env"; \
		[[ -e "$${_ipListFile}" ]] && mv "$${_ipListFile}" "$${_ipListFile}.latest"; \
		:> "$${_ipListFile}"; \
		echo -n 'CLOUDFLARE_HOST_IPS=' > "$${_ipEnvFile}"; \
		echo -n "-> Download: $${_ipt} » "; \
		curl --silent -o "$$_ipListFile" -L "$(IP_LIST_BASE_URL)/ips-$${_ipt}" \
			&& echo OK \
			|| (echo FAIL; [[ -e "$${_ipListFile}.latest" ]] && mv "$${_ipListFile}.latest" "$${_ipListFile}";); \
		cat "$${_ipListFile}" >> "$${_ipListAllFile}"; \
		echo "" >> "$${_ipListAllFile}"; \
		awk 'BEGIN{ORS=","} {print $$0;}' "$${_ipListFile}" | sed 's/,$$//' | tee -a "$$_ipEnvFile"; \
		echo; \
	done; \
	echo '-> .cloudflare.ips.all.env: ' \
		&& awk 'BEGIN{ORS=","} {print $$0;}' "$${_ipListAllFile}" | sed 's/,$$//' | tee -a "$$_ipEnvAllFile";
