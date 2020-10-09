# Operationalizing Machine Learning
This project is part of the Udacity Azure ML Nanodegree with the goal to operationalize a machine learning task in Azure. To achieve this different steps are needed.

## Architectural Diagram

This picture gives an overview of the general project steps.

![Main task](pictures/Project_main_steps.PNG)

__1. Authentication__
 
This project is developt in a provided Udacity lab. Because of this the authentification is done by the VM. The step is skipped.

__2. Auto ML model__

To set up an endpoint, a trained machine learning model has to be available. To get this, an AutoML run is started and different models are trained on the [Bank marketing dataset](https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_train.csv).

__3. Deploy the best model__

AutoML will find and provide the best fitting model. This model can than be deployed and receive data as an endpoint.

__4. Enable logging__

In case of an error or problems with the endpoint the developer has to react fast. A key tool to find a solution is to analyze logs. To provide this tool, the logging has to be enabled for the endpoint.

__5. Consume model endpoints__

To consume and test the endpoint a test POST request is sent. If the model works correctly the expected response is returned. Additionally, the endpoint can be benchmarked with an Apache tool.

__6. Create and publish a pipeline__

The final step is to create and publish a pipeline to automate the workflow. 

In the next section, key steps of the project are discussed in more detail.

## Key Steps

__1. Create dataset__

The basis of a machine learning project is the dataset. Azure Machine Learning studio provides the possibility to upload datasets and make them available for learning tasks.

For this project the [Bank marketing dataset](https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_train.csv) is uploaded. The data is organized in a tabular form and provides the target column 'y'.

As you can see on the picture, the upload was successful and the dataset 'marketing-data' is available:

![Dataset](pictures/Registered%20dataset.PNG)

__2. Execute AutoML experiment__

The next step is to execute an AutoML run to find the best model to make predictions on the dataset. The next pictures shows the completed experiment:

![Executed Experiment](pictures/Experiment%20completed.PNG)


As you can see, the expertiment ran for ~ 17min on automl-compute (this is a Standard_DS12_v2 compute cluster) and the best model, a VotingEnsemble algorithm, has the accuracy 0.91563 (right upper corner).

__3. Receive a trained model__

After completion we can check out the found models (use the models tab in the experiment view). The next picture shows the best models:

![Best model](pictures/Best_models_overview.PNG)

It's clearly to see that the first models are all very good and don't differ a lot in the accuracy score. The two first best models are Ensemble algorithms that provide a great robustness because of their combination of different models. The VotingEnsemble algorithm as best model is a good choice to deploy.

__4. Deploy best model__

Clicking on the VotingEnsemble model opens the specific run in which the model was found. In this section, details and metrics of the model could be analyized. Additonally, there is a 'Deploy' button. Before deployment, some configurations have to be set:

![Deploy](pictures/Deploy.PNG)

Beside the name and the description, the Compute Type has to be Azure Container Instance (ACI) and the authentication has to enabled. After deployment, customizations can be done with the SDK.

__5. Enable logging__

One of the most important tools for a developer is logging. To use this feature, application insights of the endpoint have to be activated. This could be done with the Azure SDK:

```
activate_application_insights.py


from azureml.core import Workspace
from azureml.core.webservice import Webservice

# Requires the config to be downloaded first to the current working directory
ws = Workspace.from_config()

# Set with the deployment name
name = "marketing-prediction"

# load existing web service
service = Webservice(name=name, workspace=ws)

# enable application insight
service.update(enable_app_insights=True)

```
__Note__ the config.json file has to be available next to the activate_application_insights&#46;py script.

After activation the parameter 'Application Insight enabled' is set to true on the endpoint side:

![Enable logging](pictures/Application_insights_true.PNG)

Additionally, a link to the Application Insights dashboard is provided. To check if the logging is also working with the SDK, the script logs&#46;py can be used:

```
from azureml.core import Workspace
from azureml.core.webservice import Webservice

# Requires the config to be downloaded first to the current working directory
ws = Workspace.from_config()

# Set with the deployment name
name = "marketing-prediction"

# load existing web service
service = Webservice(name=name, workspace=ws)
logs = service.get_logs()

for line in logs.split('\n'):
    print(line)

```

The next picture shows the terminal output of the script in case of working logging:

![Logging](pictures/Logs.PNG)





__6. Swagger Documentation__

Next step is to checkout the swagger documentation. In the swagger directory two files are provided:

- __swagger&#46;sh__: runs a swagger ui locally.

- __serve &#46;py__: HTTP server that exposes a local swagger.json file so that a swagger-ui service can pick it up from localhost.

Additionally, the swagger.json file has to be downloaded from the endpoint.

If both files are runing the documentation could be found at http://localhost:8000/swagger.json. 

__Note__ sometimes it could happen, that the direct localhost link does not work. In this case, type https://petstore.swagger.io in the browser and change the addresse in the ui to http://localhost:8000/swagger.json. The picture shows the running documentation.

![Swagger documentation](pictures/Swagger_localhost.PNG)

__7. Consume model endpoint__

Finally we can test and consume our endpoint. For this we can use the script endpoint.py:

```
endpoint.py


import requests
import json

# URL for the web service, should be similar to:
# 'http://8530a665-66f3-49c8-a953-b82a2d312917.eastus.azurecontainer.io/score'
scoring_uri = 'http://84c0c212-05e2-4b15-9a36-9d7456258aa3.southcentralus.azurecontainer.io/score'
# If the service is authenticated, set the key or token
key = 'bTkoKnUBo1OP8L9Q5fib1FYnBZ331NcV'

# Two sets of data to score, so we get two results back
data = {"data":
        [
          {
            "age": 17,
            "campaign": 1,
            "cons.conf.idx": -46.2,
            "cons.price.idx": 92.893,
            "contact": "cellular",
            "day_of_week": "mon",
            "default": "no",
            "duration": 971,
            "education": "university.degree",
            "emp.var.rate": -1.8,
            "euribor3m": 1.299,
            "housing": "yes",
            "job": "blue-collar",
            "loan": "yes",
            "marital": "married",
            "month": "may",
            "nr.employed": 5099.1,
            "pdays": 999,
            "poutcome": "failure",
            "previous": 1
          },
          {
            "age": 87,
            "campaign": 1,
            "cons.conf.idx": -46.2,
            "cons.price.idx": 92.893,
            "contact": "cellular",
            "day_of_week": "mon",
            "default": "no",
            "duration": 471,
            "education": "university.degree",
            "emp.var.rate": -1.8,
            "euribor3m": 1.299,
            "housing": "yes",
            "job": "blue-collar",
            "loan": "yes",
            "marital": "married",
            "month": "may",
            "nr.employed": 5099.1,
            "pdays": 999,
            "poutcome": "failure",
            "previous": 1
          },
      ]
    }
# Convert to JSON string
input_data = json.dumps(data)
with open("data.json", "w") as _f:
    _f.write(input_data)

# Set the content type
headers = {'Content-Type': 'application/json'}
# If authentication is enabled, set the authorization header
headers['Authorization'] = f'Bearer {key}'

# Make the request and display the response
resp = requests.post(scoring_uri, input_data, headers=headers)
print(resp.json())


```

The score_uri and the key can be found in the 'Consume' tab of the endpoint. Executing this script in the terminal returns the following result:

![Consume Endpoint](pictures/Consume_endpoint.PNG)

So this tells us that the marketing action would not be successful for the first customer but for the second. This result and the fact that a result was return shows us, that the endpoint is working. 

To get a better feeling how performant the endpoint is, we can run the [Apache Benchmarking tool](https://httpd.apache.org/docs/2.4/programs/ab.html). The command is:

```
ab -n 10 -v 4 -p data.json -T 'application/json' -H 'Authorization: Bearer bTkoKnUBo1OP8L9Q5fib1FYnBZ331NcV' http://84c0c212-05e2-4b15-9a36-9d7456258aa3.southcentralus.azurecontainer.io/score

```

This means that the Apache Benchmarking tool __ab__ will run 10 request *interations* with data (data.json) on the endpoint. You can see that the key and the addresse are the same like in the endpoint&#46;py file. The ab result is printed with *verbosity* of 4. 

![Apache benchmark](pictures/Apache_benchmark.PNG)

The most important information is, that no runs failed. This is great and is a good sign that the endpoint is working correctly. The mean time of 474ms per request is also great but not realistic because the test is locally. The default termination time of the Azure endpoint is 60s. This shows us, that the realistic endpoint request time is higher.

__8. Create, Publish and Consume a Pipeline__

The last step was the creation, publishment and consumption of a Azure pipeline. Pipelines are a great way to automate a machine learning workflow. A good example of this process can be found in the aml-pipelines-with-automated-machine-learning-step.ipynb Jupyter notebook. For explanations some extracts of this notebook are used in this README.

 __8.1 Creation__

For the creation of a pipeline the pre-steps 'dataset generation' and 'automl step generation' have to be executed:

__Dataset generation__

We can generate the dataset with the following SDK methods:

```
key = "marketing-data "

description_text = ""

example_data = 'https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_train.csv'
dataset = Dataset.Tabular.from_delimited_files(example_data)        
#Register Dataset in Workspace
dataset = dataset.register(workspace=ws,
                            name=key,
                            description=description_text)

```

The data is now available in the dataset variable.



__AutoML step generation__

To run an AutoML process within the pipeline, an [AutoMLStep](https://docs.microsoft.com/en-us/python/api/azureml-train-automl-runtime/azureml.train.automl.runtime.automlstep?view=azure-ml-py) has be provided. This code is only an extract, other needed pre-steps are in the notebook.

```
automl_step = AutoMLStep(
    name='automl_module',
    automl_config=automl_config,
    outputs=[metrics_data, model_data],
    allow_reuse=True)
```

__Pipeline generation__

To generate the pipeline we create Pipeline object with reference to the workspace and the automl_step. Included in the automl step is the dataset as the training_data parameter.

```
from azureml.pipeline.core import Pipeline
pipeline = Pipeline(
    description="pipeline_with_automlstep",
    workspace=ws,    
    steps=[automl_step])

#Submit the Pipeline
pipeline_run = experiment.submit(pipeline)


```

With this code the Pipeline is set up and executed. It's possible to get insights to the run with the RunDetails Widget:

![Run widget](pictures/Run_widget.PNG)

Additonally it's possible to see the different steps of the pipeline with the wait_for_completion method.

![automl steps](pictures/Pipeline_step_runs.PNG)


After completion, the picture shows successful pipeline runs.

![Successful pipeline runs](pictures/Successful_pipeline-runs.PNG)

The Run 54 is the initial run, recognisable by the description. The generated pipeline is visible on the next picture (in this case it was another run):

![Generated Pipeline](pictures/Generated_pipeline.PNG)

__8.2 Pipeline publishment__

If the run was successful and the results are good, we can publish this pipeline:

```
published_pipeline = pipeline_run.publish_pipeline(
    name="Bankmarketing Train", description="Training bankmarketing pipeline", version="1.0")
```

After completion the "Bankmarketing Train" endpoint is visible in the Azure ML Studio and the status is 'Active':

![Pipeline Endpoint](pictures/Pipeline_endpoint.PNG)

It's possible to get an deeper insight in the pipeline and check out the parts:

![Pipeline modules](pictures/Pipeline_endpoint_REST.PNG)

It's clearly to see that the pipeline is splitted in dataset block and an automl_step block.

![Dataset block](pictures/Pipeline_marketing_data.PNG)

![automl block](pictures/Pipeline_automl_module.PNG)

If a pipeline runs was successful the autml block shows the tag completed. Additionally, information about the found model, the acccuracy etc. are visible when clicking on the block.

__8.3 Pipeline Consumption__

The last step is to consume the pipeline. To do this we first need an authentication header for the request. Because we are working locally we can use the [InteractiveLoginAuthentication](https://docs.microsoft.com/en-us/python/api/azureml-core/azureml.core.authentication.interactiveloginauthentication?view=azure-ml-py) class.

```

from azureml.core.authentication import InteractiveLoginAuthentication

interactive_auth = InteractiveLoginAuthentication()
auth_header = interactive_auth.get_authentication_header()


```
with this header we can than set up a POST request:

```
import requests

rest_endpoint = published_pipeline.endpoint
response = requests.post(rest_endpoint, 
                         headers=auth_header, 
                         json={"ExperimentName": "pipeline-rest-endpoint"}
                        )

```

This POST will start a new pipeline run. This run can be monitored with the following code: 

```
run_id = response.json().get('Id')

from azureml.pipeline.core.run import PipelineRun
from azureml.widgets import RunDetails

published_pipeline_run = PipelineRun(ws.experiments["pipeline-rest-endpoint"], run_id)
RunDetails(published_pipeline_run).show()

```

Additonally, the running pipeline can be seen in the Azure Machine Learning studio.

![Scheduled run](pictures/Scheduled_run.PNG)


## Screen Recording

Checkout the following video to get an overview of the whole project and all parts.
[Video](Project_screencast.mp4)


