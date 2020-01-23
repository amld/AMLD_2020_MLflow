# MLflow Tracking&mdash;Keeping Track of Models
The first component discussed is MLflow Tracking. It is an API and UI for logging parameters, code versions, metrics, and artifacts when running machine learning code and for later visualizing the results. It can be used in any environment to log results to local files or to a server, then compare multiple runs, or to compare results from different users.

At first, the [setup](#setup) will be discussed. Then, a short [overview](#overview) taken from MLflow's documentation is given. And finally, we will apply this knowledge to three little examples (see [Tasks](#tasks)).

## Setup
### Locally
Windows
```bash
mlflow_server\Scripts\activate
mlflow server --host 0.0.0.0 --port 5000 --backend-store-uri file:///%cd%\mlruns --default-artifact-root file:/%cd%\mlruns
```
Linux
```bash
. mlflow_server/bin/activate
mlflow server --host 0.0.0.0 --port 5000 --backend-store-uri file:///$pwd/mlruns --default-artifact-root file:/$pwd/mlruns
```
The MLflow server is now running on <http://localhost:5000>. This value needs to be assigned to the
environment variable `MLFLOW_TRACKING_URI`, i.e., `set MLFLOW_TRACKING_URI=http://localhost:5000` on Windows
or `export MLFLOW_TRACKING_URI=http://localhost:5000` on Linux. As an alternative, it can be set in your Python code through `mlflow.set_tracking_uri("http://localhost:5000")`.

### With Docker
The folder [`100_setup_server_docker`](./100_setup_server_docker) provides a `Dockerfile` and a `startup.sh` script that enables you run the MLflow service in a Docker container. In the given script, the artifacts and metadata are stored in a folder in that container, which is certainly not the best approach. For alternatives including cloud storage by common providers, check [MLflow's documentation](https://mlflow.org/docs/latest/tracking.html#mlflow-tracking-servers).

## Overview
Here, the most important MLflow Tracking functionalities and respective Python API methods are
discussed. The [complete documentation](https://www.mlflow.org/docs/latest/tracking.html) as well as
the complete overview of the MLflow Tracking Python API
([high level](https://mlflow.org/docs/latest/python_api/mlflow.html) and
[low level](https://www.mlflow.org/docs/latest/python_api/mlflow.tracking.html)) can be found on
MLflow's webpage.

### *Runs* are core concept of MLflow
*Runs* are executions of some piece of data science code. Each run records the following
information:
* Code Version: Git commit hash used for the run, if it was run from an MLflow Project
* Start & End Time: Start and end time of the run
* Source: Name of the file to launch the run, or the project name and entry point for the run if run
  from an MLflow Project
* Parameters: Key-value input parameters of your choice. Both keys and values are strings.
* Metrics: Key-value metrics, where the value is numeric. Each metric can be updated throughout the
  course of the run (for example, to track how your model's loss function is converging), and MLflow
  records and lets you visualize the metric's full history.
* Artifacts: Output files in any format, e.g., images, models, data files

You can optionally organize runs into *experiments*, which group together runs for a
specific task.

### Multiple options to record runs
MLflow runs can be recorded to local files, to a SQLAlchemy compatible database, or remotely to a
tracking server. By default, the MLflow Python API logs runs locally to files in an `mlruns`
directory wherever you ran your program. To log runs remotely, set the `MLFLOW_TRACKING_URI`
environment variable to a tracking server's URI or call `mlflow.set_tracking_uri`.

Consult MLflow's documentation to see the different kinds of remote tracking URIs.

### Python API Logging Functions
`mlflow.set_tracking_uri("<URI>")` connects to a tracking URI. You can also set the
`MLFLOW_TRACKING_URI` environment variable to have MLflow find a URI from there. In both cases, the
`<URI>` can either be a HTTP/HTTPS URI for a remote server (e.g.,
`https://my-tracking-server:5000`), a database connection string, or a local path to log data to a
directory (e.g., `file:/path/to/dir`). The URI defaults to `mlruns`.

`mlflow.tracking.get_tracking_uri()` returns the current tracking URI.

`mlflow.create_experiment("<EXPERIMENT_NAME>")` creates a new experiment and returns its ID. Runs
can be launched under the experiment by passing the experiment ID to `mlflow.start_run()`.

`mlflow.set_experiment("<EXPERIMENT_NAME>")` sets an experiment as active. If the experiment does
not exist, creates a new experiment. If you do not specify an experiment in `mlflow.start_run()`,
new runs are launched under this experiment.

`mlflow.start_run()` returns the currently active run or starts a new run and returns a
`mlflow.ActiveRun` object usable as a context manager for the current run. You do not need to call
`start_run` explicitly: calling one of the logging functions with no active run automatically starts
a new one.

`mlflow.end_run()` ends the currently active run, if any, taking an optional run status.

`mlflow.active_run()` returns a `mlflow.entities.Run` object corresponding to the currently active
run, if any.

`mlflow.log_param("<PARNAME>", "<PARVALUE>")` logs a single key-value param in the currently active
run. The key and value are both strings. Use `mlflow.log_params(<PARDICT>)` to log multiple params
at once.

`mlflow.log_metric("<METRICNAME>", <METRICVALUE>)` logs a single key-value metric. The value must
always be a number. MLflow remembers the history of values for each metric. Use
`mlflow.log_metrics(<METRICDICT>)` to log multiple metrics at once.

`mlflow.set_tag("<TAGNAME>", "<TAGVALUE>")` sets a single key-value tag in the currently active run.
The key and value are both strings. Use `mlflow.set_tags(<TAGDICT>)` to set multiple tags at once.

`mlflow.log_artifact("<LOCALPATH>")` logs a local file or directory as an artifact.

`mlflow.log_artifacts("<LOCALDIR>")` logs all the files in a given directory as artifacts.

`mlflow.get_artifact_uri()` returns the URI that artifacts from the current run should be logged to.

The `mlflow.tracking.MLflowClient()` class creates a client of an MLflow Tracking Server that
creates and manages experiments and runs, and of an MLflow Registry Server that creates and manages
registered models and model versions. Some important methods are:
* `.create_experiment()`
* `.create_run()`
* `.download_artifacts()`
* `.get_experiment()` and `.get_experiment_by_name()`
* `.get_run()`
* `.list_artifacts()`
* `.list_experiments()`
* `.log_artifact()` and `.log_artifacts()`
* `.log_metric()`
* `.log_param()`
* `.log_batch()`

For more details on these and other methods, see MLflow's documentation.

## Tasks
In this folder, there are three Jupyter Notebooks: a 'Hello World' model
([`111_hello_world.ipynb`](./111_hello_world.ipynb)), a logistic regression done with `sklearn`
([`121_sklearn_logreg.ipynb`](./121_sklearn_logreg.ipynb)), and a more sophisticated classification
done in `sklearn` ([`131_sklearn_elasticnet_wine.ipynb`](./131_sklearn_elasticnet_wine.ipynb), taken
from MLflow's examples and modified). To open and work with these notebooks, open a CLI, go to your
project folder where you installed the `mlflow_sklearn` environment, and type the following
commands:

Windows
```bash
mlflow_sklearn\Scripts\activate
jupyter notebook
```
Linux
```bash
. mlflow_sklearn/bin/activate
jupyter notebook
```
`conda`
```bash
conda activate mlflow_sklearn
jupyter notebook
```

The Jupyter UI should open in a browser automatically, if not, follow the instructions in the CLI.

Once the UI shows up, navigate to the `10_tracking` folder and open the Jupyter Notebook you want to
work on. Follow the instructions in the respective notebook.