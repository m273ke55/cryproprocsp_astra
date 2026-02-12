.PHONY: lint syntax

lint:
	ansible-lint

syntax:
	for playbook in examples/*.yml; do \
		ANSIBLE_COLLECTIONS_PATH=$(PWD) \
		ansible-playbook --syntax-check -i localhost, $$playbook; \
	done
