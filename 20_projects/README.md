# MLflow Projects&mdash;Making Code Reproducible
Having found a way to track code versions and parameters, we now want to go one step further and
enable others to use our code on different machines or even run the code at scale on another
platform, thus, making our results reproducible. This usually means that the whole environment
(e.g., library dependencies) needs to be captured.

MLflow Projects are a standard format for packaging reusable data science code. Each project is
simply a directory with code or a Git repository, and uses a descriptor file or simply convention to
specify its dependencies and how to run the code.

A broad documentation on how to set up MLflow Projects can be found on
[MLflow's own documentation](https://www.mlflow.org/docs/latest/projects.html#). Here, again, a
shortened [overview](#overview) is given. Additionally, a
[template folder](200_mlflow_project_template) provides basic files to create an MLflow project.
Hands-on tasks will be given [afterwards](#tasks).

## Overview
### How to package your code
At the core, MLflow Projects are just a convention for organizing and describing your code to let
other data scientists (or automated tools) run it. Each project is simply a directory of files, or a
Git repository, containing your code. MLflow can run some projects based on a convention for placing
files in this directory (e.g., a `conda.yaml` file is treated as a Conda environment), but you can
describe your project in more detail by adding a `MLproject` file, which is a YAML formatted text
file. Each project can specify several properties:
* Name:  A human-readable name for the project.
* Entry Points: Commands that can be run within the project, and information about their parameters.
  Most projects contain at least one entry point that you want other users to call. Some projects
  can also contain more than one entry point: e.g., you might have a single Git repository
  containing multiple featurization algorithms. You can also call any `.py` or `.sh` file in the
  project as an entry point. If you list your entry points in an `MLproject` file, however, you can
  also specify parameters for them, including data types and default values.
* Environment: The software environment that should be used to execute project entry points. This
  includes all library dependencies required by the project code.

See the files in the [template folder](./200_mlflow_project_template) for more details or consult
MLflow's documentation.

### Commands
You can run any project from a Git URI or from a local directory using the `mlflow run` CLI tool, or
the `mlflow.projects.run()` Python API. For more information on how to call the CLI command, consult
```bash
mlflow run --help
```
**Attention, Windows users:** As of the moment of writing (using MLflow 1.5.0), there is a bug
running `mlflow run` in the Windows `cmd` CLI. The problem occurs when the automatically created
`conda` environment is activated. An error message occurs, telling that the CLI is not properly
configured to execeute `conda activate` although `conda init cmd.exe` has already been successfully
executed. To circumvent this, you must change one line in MLflow's code. Open
`<MLFLOWVENV>\Lib\site-packages\mlflow\projects\\__init__.py` and find the `_run_entry_point`
function (ll. 492ff. in MLflow 1.5.0). Replace the line
```
process = subprocess.Popen(command, close_fds=True, cwd=work_dir, env=env)
```
with
```
process = subprocess.Popen(["cmd", "/c", command], close_fds=True, cwd=work_dir, env=env)
```
Now, you are good to go.

## Tasks
1. Take a look at the ["Hello World" Project folder](./210_hello_world) and its specific
   implementation. Run the project with the local folder as well as with the remote GitHub folder.
   ```bash
   set MLFLOW_TRACKING_URI=http://localhost:5000
   mlflow_sklearn\Scripts\activate
   mlflow run 20_projects\210_hello_world -Palpha=.01 -Prun_origin=LocalRun -Plog_artifact=True
   mlflow run https://github.com/MynherVanKoek/AMLD_2020_MLflow.git#20_projects/210_hello_world -Palpha=.01 -Prun_origin=GitRun -Plog_artifact=True
   ```
   Again, Linux or Mac Users need to activate their virtual environment by using `. mlflow_sklearn/bin/activate`. For `conda` environments use `conda activate mlflow_sklearn`.

2. Complete the [Logistic Regression](./221_sklearn_logreg) and
   [Wine Classification](./231_sklearn_elasticnet_wine) folders and run them as well. You can also
   use their respective solution folders to them as remote GitHub projects.
