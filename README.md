<!-- Improved compatibility of back to top link: See: https://github.com/othneildrew/Best-README-Template/pull/73 -->
<div id="top"></div>

<!-- PROJECT LOGO -->
<br />
<div align="center">
  <h2 align="center">Text2SQL Demo Application</h2>
  <p align="center">
    Case Study - Text 2 SQL
  </p>
  <!--div>
    <img src="images/profile_pic.png" alt="Logo" width="80" height="80">
  </div-->
</div>

<!-- TABLE OF CONTENTS -->

## Table of Contents

<!-- <details> -->
<ol>
    <li>
        <a href="#about-the-project">About The Project</a>
        <ul>
            <li><a href="#architecture">Architecture</a></li>
        </ul>
    </li>
    <li>
        <a href="#setup">Setup</a>
        <ul>
            <li><a href="#enable-services">Enable Services</a></li>
            <li><a href="#iam-permissions">IAM Permissions</a></li>
            <li><a href="#bigquery-dataset">BigQuery Dataset</a></li>
            <li><a href="#python-dependencies">Python Dependencies</a></li>
            <li><a href="#application-default-credentials">Application Default Credentials</a></li>
        </ul>
    </li>
    <li>
        <a href="#implementation">Implementation</a>
        <ul>
            <li><a href="#text2sql-model">Text2SQL Model</a></li>
            <li><a href="#llm-prompt">LLM Prompt</a></li>
        </ul>
    </li>
    <li>
        <a href="#usage">Usage</a>
        <ul>
            <li><a href="#via-streamlit">Via Streamlit</a></li>
            <li><a href="#via-service">Via Service</a></li>
        </ul>
    </li>
    <li><a href="#challenges">Challenges</a></li>
    <li><a href="#possible-enhancement">Possible Enhancements</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
</ol>
<!-- </details> -->

<!-- ABOUT THE PROJECT -->

## About The Project

This project is created to showcase how we can run a `Text2SQL` model to translate natural language to `BigQuery` statements.

The following are some of the requirements:

- Deploy `Text2SQL` model to `Vertex AI` endpoint
- Get _user input_ and translate them into `BigQuery` statements
- Execute the translated `BigQuery` statements and return human readable answer to _user_

<p align="right">(<a href="#top">back to top</a>)</p>

<!-- Architecture -->

### Architecture

The following depicts the architecture of the following implementation.
The `Text2SQL` model will be deployed to a `Vertex AI` endpoint and user input will be pass to get the `BigQuery` equivalent statements.
The `BigQuery` statements will be executed and the result will be pass through `Langchain` agent to provide a more human readable answer.
More information about the model can be found [here][text2sql-notebook]

| ![vertex-ai-architecture][vertex-ai-architecture] |
| :-----------------------------------------------: |
|             _Vertex AI Architecture_              |

<p align="right">(<a href="#top">back to top</a>)</p>

<!-- Setup -->

## Setup

The following are the components necessary for the implementation:

- `Enable Services`
- `IAM Permissions`
- `BigQuery Dataset`
- `Python Dependencies`
- `Application Default Credentials`

### Enable Services

The following service is required for the application to run:

| #   | Service         | Description                                                                |
| --- | --------------- | -------------------------------------------------------------------------- |
| 1   | `Vertex AI` API | To trigger the API to facilitate the uploading and deployment of the model |

| ![vertex-ai-enable-services][vertex-ai-enable-services] |
| :-----------------------------------------------------: |
|               _Vertex AI Enable Services_               |

<p align="right">(<a href="#top">back to top</a>)</p>

<div id="iam"></div>

### IAM Permissions

Provide the service account that will be used for the application with the following permissions: <br>

| #   | Privilege                 | Description                                                               |
| --- | ------------------------- | ------------------------------------------------------------------------- |
| 1   | `BigQuery Admin`          | To execute jobs in `BigQuery` to get the result of the generated query    |
| 2   | `Vertex AI Administrator` | To use the `Vertex AI` apis to facilitate deployment and prediction tasks |
| 3   | `AI Platform Admin`       | To use the `Vertex AI` apis to facilitate uploading of models             |

P.S In this demo, we are using the `xxxxxxxxxx`-compute@developer.gserviceaccount.com. While the image shows that _adminstrator_ privilege is given to the service account, we can choose the appropriate privileges that is just right for the application to run.

| ![vertex-ai-service-account][vertex-ai-service-account] |
| :-----------------------------------------------------: |
|               _Vertex AI Service Account_               |

<p align="right">(<a href="#top">back to top</a>)</p>

### BigQuery Dataset

The dataset used for the demo is `bigquery-public-data.usa_names` and it consists the following tables

| #   | Table              | Description                               | Remarks                 |
| --- | ------------------ | ----------------------------------------- | ----------------------- |
| 1   | `usa_1910_2013`    | Common names in the various states in USA | Range: _1910_ to _2013_ |
| 2   | `usa_1910_current` | Common names in the various states in USA | Range: _1910_ to _2021_ |

<p align="right">(<a href="#top">back to top</a>)</p>

### Python Dependencies

The following are the commands to install the python libraries in the `requirements.txt`

```bash
python3 -m pip install -r requirements.txt
```

<p align="right">(<a href="#top">back to top</a>)</p>

<div id="application-default-credentials"></div>

### Application Default Credentials

The application default credentials (ADC) is used to stored the credentials locally so that no additional codes are needed to authenticate with Google Cloud Services. Before that, we will first need to install the Google Cloud Command Line. More information for the authentication method can be found [here][ref-application-default-credentials]

Run the following command to install the command line:

```bash
# Setup Google Cloud SDK Repository
sudo apt-get update -y
sudo apt-get install -y apt-transport-https ca-certificates gnupg curl sudo
curl -sS https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/cloud.google.gpg
echo "deb [signed-by=/etc/apt/trusted.gpg.d/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee /etc/apt/sources.list.d/google-cloud-sdk.list

# Install Google Cloud Command Line
sudo apt-get update -y
sudo apt-get install -y google-cloud-cli
```

Once the command line is installed, we will be able to generate the credentials in the local machine where the codes are deployed. The following command will bring you to an external browser to generate a verification code.

```bash
gcloud init
gcloud auth application-default login
```

| ![vertex-ai-application-default-credentials][vertex-ai-application-default-credentials] |
| :-------------------------------------------------------------------------------------: |
|                     _Generating Application Credentials - Text2SQL_                     |

Once the verification code is authorize the credentials will be stored in the current user home directory - `<HOME_DIRECTORY>/.config/gcloud/application_default_credentials.json`

<p align="right">(<a href="#top">back to top</a>)</p>

## Implementation

The following are some of the items used in the implementation

- `Text2SQL Model`
- `LLM Prompt`

<p align="right">(<a href="#top">back to top</a>)</p>

### Text2SQL Model

#### Deploy Model

The model are residing in a `Docker` image, hence we will have to be uploaded before deploying to the `Vertex AI`
The deployment codes can be found in _text2sql_model.py_

```Python
def upload_model(self):
    print(f"Uploading: {self.model_name} from {self.model_uri}")

    try:
        model = aiplatform.Model.upload(
            display_name=self.model_name,
            serving_container_image_uri=self.model_uri,
        )
        return model
    except Exception as e:
        print(f"Uploading: {self.model_name} fails due to {e}")
        return None

def deploy_model(self, model):
    print(f"Deploying: {self.model_name} into {self.machine_type} using {self.service_account}")
    if not model:
        return

    try:
        endpoint = model.deploy(
            machine_type=self.machine_type, service_account=self.service_account
        )

        self.endpoint = endpoint
        self.endpoint_name = endpoint.resource_name
        self.endpoint_id = self.endpoint_name.split("/")[-1]

        print(f"Deploying: {self.model_name} is deployed SUCCESSFULLY to {self.endpoint_name} with id [{self.endpoint_id}]")

        return self.endpoint_id
    except Exception as e:
        print(f"Deploying: {self.model_name} deployed FAILS to {self.endpoint_name} with id [{self.endpoint_id}] due to {e}")
        return None
```

Once the image is uploaded and deploy, we will be able to see it in the `Vertex AI - Model Registry`. <br>
P.S If you don't see your image, it could be that you might need to change the _region_ of the endpoint

| ![vertex-ai-model-registry][vertex-ai-model-registry] |
| :---------------------------------------------------: |
|              _Vertex AI Model Registry_               |

<p align="right">(<a href="#top">back to top</a>)</p>

#### Prediction

Once the model is deployed, we will be able to pass the _user query_ to this model to generate the corresponding `BigQuery` statements.
The prediction codes can be found in _query_executor.py_

```Python
def generate(_self):
    print(f"Generate SQL Called - {_self.endpoint_id}")
    if len(_self.request_properties) <= 0 or not _self.endpoint_id:
        return DEFAULT_QUERY_MESSAGE

    if SESSION_DEPLOYMENT_KEY not in st.session_state:
        print("No deployment object found")
        return DEFAULT_QUERY_MESSAGE

    deployment = st.session_state[SESSION_DEPLOYMENT_KEY]
    endpoint = deployment.endpoint

    # The endpoint request
    response = endpoint.predict(instances=_self.request_properties, parameters=_self.llm_properties)

    sql_status = response.predictions[0][RESPONSE_STATUS_KEY]
    sql_query = response.predictions[0][RESPONSE_SQL_QUERY_KEY]

    print(f"Generated SQL query: {sql_query}")

    return sql_query.strip()
```

<p align="right">(<a href="#top">back to top</a>)</p>

#### Execution

With the generated `BigQuery` statement, we will then be able to get the result of the statement.
The execution codes can be found in _query_executor.py_

```python
def execute(_self, sql_query):
    print("Execute SQL Called")

    if not sql_query or sql_query == DEFAULT_QUERY_MESSAGE:
        print("No sql query found")
        return DEFAULT_RESULT_MESSAGE

    # @title [Optional] Execute generated SQL query on BQ data
    client = bigquery.Client(project=_self.project_id)
    df_res = client.query(sql_query).to_dataframe()

    return df_res
```

<p align="right">(<a href="#top">back to top</a>)</p>

#### Answering

Once the result is returned from `BigQuery`, we will convert it to a dataframe and attempt to answer the question with this dataframe.
The answering codes can be found in _palm_model.py_

```python
def answer(self, query, data):
    if not query or isinstance(data, str):
        return DEFAULT_ANSWER_MESSAGE

    self.prompt = self.prompt_template.format(question=query, context=data)

    agent = create_pandas_dataframe_agent(
        llm = self.llm,
        df = data,
        verbose=True,
        agent_type=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    )
    answer = agent.run(self.prompt)

    return answer
```

<p align="right">(<a href="#top">back to top</a>)</p>

#### Undeploy Model

Once the model is tested, we can undeploy the model and remove the endpoint to prevent any unnecessary costs.
The undeployment codes can be found in _text2sql_model.py_

```Python
def undeploy_model(self):
    print(f"Undeploying: {self.model_name} into {self.machine_type} using {self.service_account}")
    if not self.endpoint:
        return

    try:
        self.endpoint.undeploy_all()
        self.endpoint.delete()
    except Exception as e:
        print(f"Undeploying: {self.model_name} FAILS due to {e}")
```

<p align="right">(<a href="#top">back to top</a>)</p>

### LLM Prompt

#### User Query Prompt

The following is one example of the `user query` prompt. <br>
The _instructions_ are added to provide the field naming of the dataframe for use in the latter stage when trying to generate an answer.

```text
What was the most popular male name in 2013?
Instructions: Provide a descriptive field name for each generated field.
```

The following are some of the prompts that one can try as well:

- Get total number of people from state of WA grouped by name and state, sort by highest to lowest and limit the result to 10.
- What the most popular female name in the Arkansas state?
- What are the top 10 popular names?

<p align="right">(<a href="#top">back to top</a>)</p>

#### Answer Prompt

The following is one template prompt for answering the question with the dataframe. <br>
The prompt template can be found in _palm_model.py_, and is reference from [here][langchain-palm-2-api]

```text
Ignore the instructions portion of the question.
Let's think step-by-step and answer, answer the question using the context.

Do not try to make up an answer:
- If the context is empty, just say "I do not know the answer to that."

Do not give a one-word answer.
Provide the answer in a descriptive manner.

Question: {question}
Context: {context}
Answer:
```

<p align="right">(<a href="#top">back to top</a>)</p>

<!-- USAGE EXAMPLES -->

## Usage

There are 1 way to invoke the implementation

- `via-streamlit` - Run the application with `streamlit`
- `via-service` - Run the application with `systemd`

<p align="right">(<a href="#top">back to top</a>)</p>

### Via Streamlit

The following command can be use to run the application:

```bash
python3 -m streamlit run [streamlit_application_python] --server.port [streamlit_application_port]
```

| #   | Fields                         | Description                                       | Remarks         |
| --- | ------------------------------ | ------------------------------------------------- | --------------- |
| 1   | `streamlit_application_python` | The python file contain the streamlit application |
| 2   | `streamlit_application_port`   | The port which the application should run         | `Default`: 8501 |

Once the application is started, you will be able to access the application with the `streamlit_application_port` specified.

```bash
http://localhost:8501/
```

| ![text2sql-streamlit-application][text2sql-streamlit-application] |
| :---------------------------------------------------------------: |
|                _Streamlit Application - Text2SQL_                 |

#### Deploy Model

To start running the application, click the `deploy` button. <br>
Once the model is deployed, you should be able to see a status message indicating that the model has been deployed and the `endpoint id` should be generated

| ![text2sql-streamlit-application-model-deployed][text2sql-streamlit-application-model-deployed] |
| :---------------------------------------------------------------------------------------------: |
|                       _Streamlit Application - Text2SQL (Model Deployed)_                       |

> If you encountered an message that indicates that the model is not deployed due to errors, and the logs show that _PermissionDenied: 403 Request had insufficient authentication scopes. [reason: "ACCESS_TOKEN_SCOPE_INSUFFICIENT"...]_, this might due to <a href="#iam">IAM</a> configuration

<p align="right">(<a href="#top">back to top</a>)</p>

#### Getting Text to SQL

Once the model is deployed, you can enter the query inside the `Ask your question about the dataset` segment. <br>
It will then generate the corresponding `BigQuery` statement and execute to get the result and try to answer the question with the result

| ![text2sql-streamlit-application-user-query][text2sql-streamlit-application-user-query] |
| :-------------------------------------------------------------------------------------: |
|                     _Streamlit Application - Text2SQL (User Query)_                     |

<p align="right">(<a href="#top">back to top</a>)</p>

#### Checking History

Once you have a query made, you can look back under the `history` tab for the result. <br>
P.S The `history` is not persistent and will get reset each time the application load up

| ![text2sql-streamlit-application-history][text2sql-streamlit-application-history] |
| :-------------------------------------------------------------------------------: |
|                   _Streamlit Application - Text2SQL (History)_                    |

<p align="right">(<a href="#top">back to top</a>)</p>

### Via Service

This method combines the `streamlit` method and running of the application as a `service` via `systemd`. The following are some of the setup scripts that are required:

- `Application script` - The set of configurations (e.g port and address) to run the `streamlit` application
- `Startup script` - The set of instructions to start the `streamlit` application in a virtual environment
- `Service file` - The set of instructions to run the `streamlit` application as backend service

<p align="right">(<a href="#top">back to top</a>)</p>

#### Application Script

The `application` script is used to set the running configuration of the application file. The following shows the command inside the `application` script:

```bash
python3 -m streamlit run <APPLICATION_FILE> --server.port <SERVER_PORT> --server.address <SERVER_ADDRESS>
```

[NOTE]

- Replace `<APPLICATION_FILE>` with the `streamlit` application file
- Replace `<SERVER_PORT>` with the port to run the `streamlit` application
- Replace `<SERVER_ADDRESS>` with the server address to run `streamlit` application

<p align="right">(<a href="#top">back to top</a>)</p>

#### Startup Script

The `startup` script is used to run the `streamlit` application in a virtual environment. The following shows the command inside the `startup` script: <br>

```bash
base_directory="<HOME_DIRECTORY>/text2sql"
app_directory="${base_directory}/app"
virtual_env_setup_script="${base_directory}/bin/activate"
streamlit_setup_script="${app_directory}/run_streamlit.sh"
application_script="${app_directory}/text2sql.py"

source ${virtual_env_setup_script} && ${streamlit_setup_script} ${application_script}
```

[NOTE]

- For this example the application file is assume to be in the `text2sql/app` folder
- Replace `<HOME_DIRECTORY>` with the appropriate directory

<p align="right">(<a href="#top">back to top</a>)</p>

#### Service Script

The following are the configuration for the `service` script, which is to be created in the `/etc/systemd/system` directory: <br>

```bash
[Unit]
Description=Text2SQL Streamlit Application

[Service]
User=<SERVICE_USER>
WorkingDirectory=<HOME_DIRECTORY>/text2sql/app
ExecStart=<HOME_DIRECTORY>/text2sql/app/start.sh

[Install]
WantedBy=multi-user.target
```

[NOTE]

- For this example the application file is assume to be in the `text2sql/app` folder
- Replace `<SERVICE_USER>` with the appropriate user to run the application as. This `<SERVICE_USER>` home directory should contains the application credentials created <a href="#application-default-credentials">here</a>.
- Replace `<HOME_DIRECTORY>` with the appropriate directory where the application file reside

<p align="right">(<a href="#top">back to top</a>)</p>

#### Install Service

The following are the commands to install the `service` script: <br>

```bash
# Refresh the service configuration
sudo systemctl daemon-reload

# Start the service
sudo systemctl start text2sql.service

# Check the status of the service
sudo systemctl status text2sql.service

# Enable the service to start on boot
sudo systemctl enable text2sql.service
```

[NOTE]

- For this example the `service` file is assume to be named `text2sql.service`

<p align="right">(<a href="#top">back to top</a>)</p>

<!-- Challenges -->

## Challenges

The following are some challenges encountered:

- Multiple Unused Models Deployed in Model Registry
<p align="right">(<a href="#top">back to top</a>)</p>

### Challenge #1: Multiple Unused Models Deployed in Model Registry

**Observations**

Sometimes when we forget to undeploy previous model and launch the application again, the previous model will stay in the model registry. This resulted in many past model instances that are deployed and will likely not be used again.

<br/>

**Resolution**

As such, we created a script `housekeep_endpoints.py`, by passing in the `project_id` and `location`, we are able to list all the available endpoints and undeployed and delete any models that are still existing in these endpoints.

```python
def housekeep_endpoints(project, location):
    # Make sure you have your GCP credentials authenticated on your local
    aiplatform.init(project=project, location=location)

    endpoints = aiplatform.Endpoint.list()
    for endpoint in endpoints:
        endpoint.undeploy_all()
        endpoint.delete()
```

Run the following command to trigger cleanup the endpoints and its unused models:

```bash
python3 housekeep_endpoints.py -p <PROJECT_ID> -l <LOCATION>
```

[NOTE]

- Replace `<PROJECT_ID>` with the appropriate project ID
- Replace `<LOCATION>` with the appropriate location

| ![vertex-ai-housekeep-endpoints][vertex-ai-housekeep-endpoints] |
| :-------------------------------------------------------------: |
|                _Vertex AI - Housekeep Endpoints_                |

<br/>

<p align="right">(<a href="#top">back to top</a>)</p>

<!-- ENHANCEMENTS -->

## Possible Enhancements

- [ ] Load Dropdown Options from Config Files Instead of Hardcoding
- [ ] Explore authentication for AI platform init using credentials instead of Application Default Credentials (ADC) so as to facilitate docker deployment
- [ ] Package application into docker image
- [ ] Reuse any existing model in the endpoint
- [ ] Display full answer from Langchain Pandas Agent (Currently, answer is cut off if it is tool long)

<p align="right">(<a href="#top">back to top</a>)</p>

<!-- ACKNOWLEDGMENTS -->

## Acknowledgments

- [Readme Template][template-resource]
- [Langchain with Palm API][langchain-palm-2-api]
- [Langchain with Pandas Dataframe][langchain-pandas]

<p align="right">(<a href="#top">back to top</a>)</p>

<!-- MARKDOWN LINKS & IMAGES -->

[template-resource]: https://github.com/othneildrew/Best-README-Template/blob/master/README.md
[text2sql-notebook]: https://499d902414c42365-dot-us-central1.notebooks.googleusercontent.com/lab/tree/sia-demo/%5BExternal%5D_Experimental_Text2SQL_V2_launch.ipynb
[langchain-pandas]: https://python.langchain.com/docs/integrations/toolkits/pandas
[langchain-output-parsers]: https://python.langchain.com/docs/modules/model_io/output_parsers/structured
[langchain-partial-prompt]: https://python.langchain.com/docs/modules/model_io/prompts/prompt_templates/partial
[langchain-palm-api]: https://github.com/GoogleCloudPlatform/generative-ai/blob/main/language/orchestration/langchain/intro_langchain_palm_api.ipynb
[langchain-palm-2-api]: https://cloud.google.com/blog/products/ai-machine-learning/generative-ai-applications-with-vertex-ai-palm-2-models-and-langchain
[vertex-ai-architecture]: ./images/vertex_ai_architecture.png
[vertex-ai-enable-services]: ./images/vertex_ai_enable_services.png
[vertex-ai-service-account]: ./images/vertex_ai_service_account.png
[vertex-ai-model-registry]: ./images/vertex_ai_model_registry.png
[vertex-ai-application-default-credentials]: ./images/vertex_ai_application_default_credentials.png
[vertex-ai-housekeep-endpoints]: ./images/vertex_ai_housekeep_endpoints.png
[text2sql-streamlit-application]: ./images/text2sql_streamlit_application.png
[text2sql-streamlit-application-model-deployed]: ./images/text2sql_streamlit_application_model_deployed.png
[text2sql-streamlit-application-user-query]: ./images/text2sql_streamlit_application_user_query.png
[text2sql-streamlit-application-history]: ./images/text2sql_streamlit_application_history.png
[ref-application-default-credentials]: https://cloud.google.com/docs/authentication/gcloud#gcloud-credentials
