[![Test pipeline](https://github.com/pdemeulenaer/cicd-databricks-github/actions/workflows/onpush.yml/badge.svg?branch=main)](https://github.com/pdemeulenaer/cicd-databricks-github/actions/workflows/onpush.yml)
[![Release pipeline](https://github.com/pdemeulenaer/cicd-databricks-github/actions/workflows/onrelease.yml/badge.svg)](https://github.com/pdemeulenaer/cicd-databricks-github/actions/workflows/onrelease.yml)

# cicd-databricks-github

This is a sample project for Databricks, generated via cookiecutter.

While using this project, you need Python 3.X and `pip` or `conda` for package management.

## Installing project requirements

```bash
pip install -r unit-requirements.txt
```

## Install project package in a developer mode

```bash
pip install -e .
```

## Testing

For local unit testing, please use `pytest`:
```
pytest tests/unit --cov
```

For an integration test on interactive cluster, use the following command:
```
dbx execute --cluster-name=<name of interactive cluster> --job=cicd-databricks-github-sample-integration-test
```

For a test on an automated job cluster, deploy the job files and then launch:
```
dbx deploy --jobs=cicd-databricks-github-sample-integration-test --files-only
dbx launch --job=cicd-databricks-github-sample-integration-test --as-run-submit --trace
```

## Interactive execution and development

1. `dbx` expects that cluster for interactive execution supports `%pip` and `%conda` magic [commands](https://docs.databricks.com/libraries/notebooks-python-libraries.html).
2. Please configure your job in `conf/deployment.json` file. 
2. To execute the code interactively, provide either `--cluster-id` or `--cluster-name`.
```bash
dbx execute \
    --cluster-name="<some-cluster-name>" \
    --job=job-name
```

Multiple users also can use the same cluster for development. Libraries will be isolated per each execution context.

## Preparing deployment file

Next step would be to configure your deployment objects. To make this process easy and flexible, we're using JSON for configuration.

By default, deployment configuration is stored in `conf/deployment.json`.

## Deployment for Run Submit API

To deploy only the files and not to override the job definitions, do the following:

```bash
dbx deploy --files-only
```

To launch the file-based deployment:
```
dbx launch --as-run-submit --trace
```

This type of deployment is handy for working in different branches, not to affect the main job definition.

## Deployment for Run Now API

To deploy files and update the job definitions:

```bash
dbx deploy
```

To launch the file-based deployment:
```
dbx launch --job=<job-name>
```

This type of deployment shall be mainly used from the CI pipeline in automated way during new release.


## CICD pipeline settings

Please set the following secrets or environment variables for your CI provider:
- `DATABRICKS_HOST`
- `DATABRICKS_TOKEN`

## Testing and releasing via CI pipeline

- To trigger the CI pipeline, simply push your code to the repository. If CI provider is correctly set, it shall trigger the general testing pipeline
- To trigger the release pipeline, get the current version from the `cicd_databricks_github/__init__.py` file and tag the current code version:
```
git tag -a v<your-project-version> -m "Release tag for version <your-project-version>"
git push origin --tags
```

## The use case

Use case: a simple random forest classifier (using scikit-learn) of the Iris dataset

The CI/CD procedure:

CI:

* unit tests (dummy, so far unrelated to the use case)

* Deploy & trigger training job. Training job made of 2 tasks:

  - task 1 (step_data_prep.py): data preparation task: Iris dataset is downloaded and split into train and test. Both are saved (test kept for validation step)
  - task 2 (step_training.py): training RF model (aka the "CI experiment"). Experiment saved to MLflow
  - task 3 (step_compare_performance.py): comparison of model performance to all experiments logged during the feature branch; Validate as a custom tag in MLflow for the CI experiment

Based on the CI experiment tag, reviewer will know if the PR is to be merged (also looking at results of unit tests, code analysis if such exists,...)  

CD:

* Deploy & trigger integration tests (dummy, so far unrelated to the use case)

* Validation job: run the scoring function on the test dataset

* Deploy the scheduled batch inference on Databricks Jobs: here the scoring function is applied on unseen data (again, test dataset?)


## TODO list

* [Done] Build the TRAIN and TEST datasets BEFORE the data preparation task. It should be there even before the CI takes place.

* [Done] Store data path and model configs in json, and read them from each task file 

* [Done] Look for a way to trigger jobs from Airflow (follow the example from the Databricks documentation: https://docs.databricks.com/dev-tools/data-pipelines.html)

* Link the Github folder to a Databricks repo

* Automate the copy of the dag file to S3 bucket

* Use a pool for clusters to keep them alive between tasks within a job

* Add a folder to contain development notebooks

* Put all functions into a utils.py module that we can refer to in any file. 




