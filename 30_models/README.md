# MLflow Models&mdash;Packaging and Deploying ML Models
So far, we have learnt how to integrate MLflow's tracking functionalities into our model code. Doing
so, we can now trace different runs done with different parameters and their respective outcomes to
better evaluate, compare, discard, or accept a model. Additionally, we have found a possibility to
package and re-run data science code, thus, allowing for reproducing results. The third component,
MLflow Models, offers a convention for packaging machine learning models in multiple flavors, and a
variety of tools to deploy these models.

At first, an [overview on MLflow Models](#overview) taken from
[MLflow's documentation](https://www.mlflow.org/docs/latest/models.html) is given. Then, we will
integrate these new functions in in our code (see [Tasks](#tasks)).

## Overview
### How MLflow is storing models
Each MLflow Model is defined by a directory of files that contains arbitrary files and an `MLmodel`
configuration file. For example, the output of `sklearn` models looks like follows:
```bash
# Directory written by mlflow.sklearn.save_model(model, "my_model")
my_model/
├── MLmodel
└── model.pkl
```
The `MLmodel` file describes various model attributes. It is written in YAML format and looks like
this:
```yaml
utc_time_created: '2019-12-28 07:13:54.587332'
run_id: 5c37ed12cb5a4cd68ddeec596482db6f
artifact_path: model
flavors:
  python_function:
    data: model.pkl
    env: conda.yaml
    loader_module: mlflow.sklearn
    python_version: 3.6.5
  sklearn:
    pickled_model: model.pkl
    serialization_format: cloudpickle
    sklearn_version: '0.22'
```
It contains the following fields:
* `utc_time_created` (optional): Date and time when the model was created, in UTC ISO 8601 format.
* `run_id` (optional): ID of the run that created the model, if the model was saved using
  `tracking`.
* `artifact_path`: path to the root directory of the MLflow Model
* `flavors`: definition of *flavors*, i.e., frameworks or libraries that the model can be viewed and
  used in.

### Different *flavors* for different modeling frameworks
*Flavors* are a convention that deployment tools can use to understand the model, which makes it
possible to write tools that work with models from any ML library without having to integrate each
tool with each library. MLflow defines several "standard" flavors that all of its built-in
deployment tools support, such as a "Python function" flavor that describes how to run the model as
a Python function. However, libraries can also define and use other flavors. For example, MLflow’s
`mlflow.sklearn` library allows loading models back as a scikit-learn `Pipeline` object for use in
code that is aware of scikit-learn, or as a generic Python function for use in tools that just need
to apply the model. This model can then be used with any tool that supports either the `sklearn` or
`python_function` model flavor.

**Built-in model flavors:** MLflow provides several standard flavors that might be useful in your
applications. Specifically, many of its deployment tools support these flavors, so you can export
your own model in one of these flavors to benefit from all these tools. Amongst other ones, these
are
* Python Function (`python_function`)
* Keras (`keras`)
* PyTorch (`pytorch`)
* Scikit-learn (`sklearn`)
* Spark MLlib (`spark`)
* TensorFlow (`tensorflow`)

A complete list can be found in
[MLflow's documentation](https://www.mlflow.org/docs/latest/models.html#built-in-model-flavors).

### MLflow Model Python API

MLflow Models can be saved and loaded in multiple ways. First, MLflow includes integrations with
several common libraries (*flavors*). They contain the methods
* `mlflow.<FLAVOR>.save_model(<MODEL>, "<LOCALPATH>")` to save the model locally,
* `mlflow.<FLAVOR>.log_model(<MODEL>, "<ARTIFACTPATH>")` to log the model as an artifact for the
  current run,
* `mlflow.<FLAVOR>.load_model("<MODELURI>")` to load the model from a local file or a run. More
  details on how to reference models from different locations can be found
  [here](https://www.mlflow.org/docs/latest/concepts.html#artifact-locations).

Additionally, some flavors (e.g., `keras`) offer the (experimental!) method `mlflow.<FLAVOR>.autolog()` that enable automatic logging. It logs metrics specified in the fit function, and optimizer data as parameters. Model checkpoints are logged as artifacts to a `models` directory.

Second, you can `mlflow.models.Model` class to create and write models. This class has four key
methods:
* `.add_flavor("<FLAVOR>", <PARAMDICT>)` to add a flavor to the model. Each flavor has a string name
  and a dictionary of key-value attributes, where the values can be any object that can be
  serialized to YAML.
* `.save("<LOCALPATH>")` to save the model as a YAML file to a local directory.
* `.log("<ARTIFACTPATH>", <FLAVOR>)` to log the model as an artifact in the current run using MLflow
  Tracking.
* `.load("<PATH>")` to load a model from a local directory or from an artifact in a previous run.

### Deployment

MLflow provides tools for deploying MLflow models on a local machine and to several production
environments. Currently, these are
* deployment of MLflow Models
* deployment of a `python_function` model on Microsoft Azure ML
* deployment of a `python_function` model on Amazon SageMaker
* export of a `python_function` model as an Apache Spark UDF

Here, we will concentrate on the deployment of MLflow Models. To get more insights on the other
deployment options, consult
[MLflow's documentation](https://www.mlflow.org/docs/latest/models.html#built-in-deployment-tools).

**Deploy MLflow models:** MLflow can deploy models locally as local REST API endpoints or to
directly score files. In addition, MLflow can package models as self-contained Docker images with
the REST API endpoint. The image can be used to safely deploy the model to various environments such
as Kubernetes. To deploy MLflow models locally or to generate a Docker image, the CLI interface to
the `mlflow.models` module is used. For more info, see
```bash
mlflow models --help
mlflow models serve --help
mlflow models predict --help
mlflow models build-docker --help
```
The REST API server accepts the following data formats as POST input to the `/invocations` path:
* JSON-serialized pandas DataFrames in the `split` orientation, e.g., `data =
  pandas_df.to_json(orient='split')`. This format is specified using a `Content-Type` request header
  value of `application/json` or `application/json; format=pandas-split`.
* JSON-serialized pandas DataFrames in the `records` orientation. *Using this format is not
  recommended because it is not guaranteed to preserve column ordering.* This format is specified
  using a `Content-Type` request header value of `application/json; format=pandas-records`.
* CSV-serialized pandas DataFrames. For example, `data = pandas_df.to_csv()`. This format is
  specified using a `Content-Type` request header value of `text/csv`.

Example requests:
```bash
# split-oriented
curl http://127.0.0.1:5000/invocations -H 'Content-Type: application/json' -d '{
    "columns": ["a", "b", "c"],
    "data": [[1, 2, 3], [4, 5, 6]]
}'
```

## Tasks
Open the `311_sklearn_logreg.ipynb` and `321_sklearn_elasticnet_wine.ipynb` notebooks. Follow the
respective instructions.
