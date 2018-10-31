.PHONY: clean test lint

TEST_PATH=./


train-nlu:
	python -m rasa_nlu.train -c config.yml --data data/nlu_data.md -o models --fixed_model_name nlu_model --project current --verbose
train-core:
	python -m rasa_core.train -d domain.yml -s data/stories.md -o models/current/dialogue --epochs 200
train-action-server:
	python -m rasa_core_sdk.endpoint --actions actions
custom-google-assistant:
	gactions update --action_package action.json --project place-finder-5bd06
enable-custom-testing:
	gactions test --action_package action.json --project place-finder-5bd06