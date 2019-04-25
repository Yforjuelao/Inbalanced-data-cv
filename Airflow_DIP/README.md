This repository shows how to apply the 
[dependency inversion principle](
https://en.wikipedia.org/wiki/Dependency_inversion_principle) 
to airflow pipelines. The high level module is the pipeline, the low level modules
are the scripts. Each script follows a common format and this abstraction
allows us to apply the DIP.  

From a single template we can generate very complex graphs of scripts.   

# Setup

This installation supposes you have Python 3 and `virtualenv` installed.
All the commands are run from the `Airflow_DIP` home folder.

Create a virtualenv

```bash
python3 -m venv venv
source venv/bin/activate
```
Install the required packages
```bash
export SLUGIFY_USES_TEXT_UNIDECODE=yes
pip install -r requirements.txt
```

If you are using Python 3.7, update the `tenacity` package to avoid errors:
```bash
pip install tenacity==5.0.2
```

Set environment variables and initialize airflow
```bash 
export PYTHONPATH=.
export AIRFLOW_HOME=`pwd`
airflow initdb
```

Modify airflow.cfg to avoid loading the airflow examples 
```bash
sed -i '' 's/load_examples = True/load_examples = False/' airflow.cfg
```
Reset the db
```bash
airflow resetdb
```

Start the webserver
```
airflow webserver
```

On a new terminal, set environment variables, activate virtualenv and 
start the scheduler
```bash
export PYTHONPATH=.
export AIRFLOW_HOME=`pwd`
source venv/bin/activate
airflow scheduler
```
In a browser, navigate to http://localhost:8080

# Examples

We created three examples:
* a basic example creating a file and copying it
* a word embedding example derived from [this tutorial exercise](https://developers.google.com/machine-learning/crash-course/embeddings/programming-exercise) 
* fine tuning the embedding

The commands to generate the three examples are respectively:
```bash
python bin/create_dag_from_template.py --template template_pipeline --dag-name test_pipeline --scripts-folder basic_example --scripts-list run_config.json
python bin/create_dag_from_template.py --template template_pipeline --dag-name embedding_dag --scripts-folder embedding_example --scripts-list run_config.json
python bin/create_dag_from_template.py --template template_pipeline --dag-name embedding_dag_extended --scripts-folder embedding_example --scripts-list run_config_extended.json
```

The code for the first example is in the `scripts/basic_example` folder. <br>
The code for the other two is in `scripts/embedding_example`. Two examples are 
generated by having two different run configurations: 
* `run_config.json`
* `run_config_extended.json` 

To run each example from the UI you need to activate it, using the OFF/ON switch 
on the left of the DAG's name. Then click on the on the leftmost button (`Play`) in the 
`Links` column.

You can see the output from each run in the `logs` directory that is generated.
This includes model performance metrics where appropriate, for example in the `eval`
output when running `embedding_dag` example.

# Build your own!

To build your own reproducible pipeline, you can follow these steps:
* create a folder under `scripts` containing an `__init__.py` file
* add the scripts you want to run
* make sure each script has a main method taking the command line arguments 
you need to pass and that it has a `--dry-run` flag to return the script inputs 
and output (check the example scripts in the `basic_example` and 
`embedding_example` for guidance)
* create a json file `run_config.json` with all the pipeline scripts 
following this format:
```python
[
  [script_identifier: [script_module, command_line arguments]...],
  ....
]
```

* As we did with our examples run:
```bash
python bin/create_dag_from_template.py --template template_pipeline --dag-name <dag_name> --scripts-folder <script_folder> --scripts-list run_config.json
```

* The DAG will be available in the airflow UI after a few minutes.
* Activate it and run it!