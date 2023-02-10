BASE_DIR := .

.PHONY: help clean build
.DEFAULT_GOAL := help

help:
	@grep -E '^[a-zA-Z0-9_-]+:.*?## .*$$' $(MAKEFILE_LIST) \
	| sed -n 's/^\(.*\): \(.*\)##\(.*\)/\1\3/p'

clean: ## "Remove all generated deployment files"
	if [ -f "package.zip" ]; then rm package.zip; else echo "Compress file doesnt exists. Nothing to remove."; fi
	if [ -f "venv/lambda-dependencies.zip" ]; then rm venv/lambda-dependencies.zip; else echo "Compress file doesnt exists. Nothing to remove."; fi
	if [ -d "venv/python" ]; then rm -rf venv/python/; else echo "Dependencies folder doesnt exists. Nothing to remove."; fi
	if [ -d "dependency" ]; then rm -rf dependency/; else echo "Dependencies folder doesnt exists. Nothing to remove."; fi

dependency: clean ## "Create dependency zip package needed by the Python script"
	bash -C "dependency.sh"

build: dependency ## "Create zip package by building the Go Slack bot"
	zip -r package.zip main.py

aws-upload: ## Upload dependencies package into AWS S3 bucket
	aws s3 cp dependency/lambda-dependencies.zip s3://alarm-lambda-dependencies/

aws-dry: build ## "AWS Cloudformation stack dry run"
	sam deploy \
			--stack-name slack-sns-notify \
			--region us-east-1 \
			--resolve-s3 \
			--disable-rollback \
			--template-file test-notify.yaml \
			--capabilities CAPABILITY_NAMED_IAM \
			--parameter-overrides Environment="dev" Project="internaltool" EcsServiceName="internaltool" EcsClusterName="dev-default-ecs" \
			--no-execute-changeset

aws-deploy: build aws-upload ## "AWS Cloudformation stack deployment"
	sam deploy \
		--stack-name slack-sns-notify \
		--region us-east-1 \
		--resolve-s3 \
		--disable-rollback \
		--template-file test-notify.yaml \
		--parameter-overrides Environment="dev" Project="internaltool" EcsServiceName="internaltool" EcsClusterName="dev-default-ecs" \
		--capabilities CAPABILITY_NAMED_IAM

aws-destroy: ## "Clean all the resources created through the AWS Cloudformation stack"
	sam delete \
		--stack-name slack-sns-notify