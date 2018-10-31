.PHONY: clean test lint

TEST_PATH=./


train-nlu:
	python -m rasa_nlu.train -c place_finder/config.yml --data place_finder/data/nlu_data.md -o place_finder/models --fixed_model_name nlu_model --project current --verbose
train-core:
	python -m rasa_core.train -d place_finder/domain.yml -s place_finder/data/stories.md -o place_finder/models/current/dialogue --epochs 200
train-action-server:
	python -m rasa_core_sdk.endpoint --actions ./place_finder/actions
custom-google-assistant:
	gactions update --action_package action.json --project place-finder-5bd06
enable-custom-testing:
	gactions test --action_package action.json --project place-finder-5bd06