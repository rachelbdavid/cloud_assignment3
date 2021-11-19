# cloud_assignment3
### URK18CS226
## _Create a machine learning model in the cloud to predict the future events from the stream data_
### Cloud service used: ```Google Cloud```
## 1. Creating project
* A google cloud platform project is needed. If you don't have one follow the steps to create it.
* In the navigation menu, select IAM & Admin -> Manage resources.
* Click on Create Project.
* Give your desired project name. Project name should be unique.
* If your mail-id is a part of an organization, your organization will be displayed. You can switch to No organization if you don't want your organization name.
* If your mail-id is personal, it will be set to No organization by default.
* Click on Create after giving the needed details.

## 2. Loading data into BigQuery
From navigation menu, go to ```BigQuery```.
* Creating dataset
	* Select the project name from the top of the console.
	* In the left pane, click on the three dots next to the project name and select Create dataset.
	* In Dataset ID, give the value as titanic.
	* In Data location, select ```europe-west4(Netherlands)```.
	* Leave the rest of the details as default and click on Create Dataset.
* Creating table
	* Download the dataset called ```titanic.csv```.
	* Click on the three dots next to the titanic dataset created in the previous step and select Open.
	* Click on Create Table option visible in the main console.
	* In the sidebar that opened, do the following:
		* Create table from: ```Upload```
		* Select file: upload the downloaded titanic dataset.
		* File format: ```CSV```
		* Table name: ```survivors```
		* Auto-detect: Select auto-detect checkbox - Schema and input parameters
* Click Create Table.

## 3. Creating dataset
* Creating ML dataset
	* In the navigation menu, click on ```Vertex AI``` under Artificial intelligence.
	* In the dashboard, select Region as ```europe-west4(Netherlands)```.
	* Click on Create dataset displayed in Preparing your training data.
	* Give dataset name as ```titanic```.
	* In Select a data type and objective, click on Tabular tab and select ```Regression/Classification```.
	* Select Region as ```europe-west4(Netherlands)``` and click Create.
* Selecting data source
	* Now we should connect previously created dataset to this source.
	* Under Select a data source, select ```Select a table or view from BigQuery```. 
	* Under Select a table or view from BigQuery, click on Browse and select ```survivors``` dataset.
	* Click on continue.
	* Now the details of the dataset will be visible. In order to run the statistical analysis, click on ```Generate Statistics```.

## 4. Custom training package using Notebooks
* Creating notebook instance
	* Select Workbench from left pane and click on New Notebook.
	* Select Python3 and on the pop up, leave it to default values and click Create.
	* After instance gets created, you can see OPEN JUPYTERLAB. Click on it to open Jupyter notebook.
* Creating package
	* Click on '+' on the Jupyter notebook found in left upper corner, and select Terminal from launcher.
	* Execute these 2 commands in the terminal:
	```
		mkdir -p /home/jupyter/titanic/trainer
		touch /home/jupyter/titanic/setup.py /home/jupyter/titanic/trainer/__init__.py /home/jupyter/titanic/trainer/task.py
	```
	* After running these commands you can see now folders created.
	* Copy paste the [task.py](https://github.com/rachelbdavid/cloud_assignment3/blob/main/code/task.py) code in titanic/trainer/task.py
	* Copy paste the [setup.py](https://github.com/rachelbdavid/cloud_assignment3/blob/main/code/setup.py) code in titanic/setup.py
	* Copy paste these four commands in terminal to create a bucket in your current project:
	```
		export REGION="europe-west4"
		export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
		export BUCKET_NAME=$PROJECT_ID"-bucket"
		gsutil mb -l $REGION "gs://"$BUCKET_NAME
	```
	* Install the required libraries using these commands in terminal:
	```
		cd /home/jupyter/titanic
		pip install setuptools
		python setup.py install
	```
	* Run the following training code to verify that it executes without issues:
	```
		python -m trainer.task -v \
		    --model_param_kernel=linear \
		    --model_dir="gs://"$BUCKET_NAME"/titanic/trial" \
		    --data_format=bigquery \
		    --training_data_uri="bq://"$PROJECT_ID".titanic.survivors" \
		    --test_data_uri="bq://"$PROJECT_ID".titanic.survivors" \
		    --validation_data_uri="bq://"$PROJECT_ID".titanic.survivors"
	```
	* If it executes correctly, it displays the following message
	```
		INFO:root:f1score: 0.85
		INFO:root:Training job completed. Exiting...
	```
	* To create python package, run the following command in the terminal
	```
		cd /home/jupyter/titanic
		python setup.py sdist
	```
	* Finally run the following command to copy the package to GCS so that the training service can use it to train a new model when you need to:
	```
		gsutil cp dist/trainer-0.1.tar.gz "gs://"$BUCKET_NAME"/titanic/dist/trainer-0.1.tar.gz"
	```
## 5. Model training
* In the navigation menu, go to Vertex AI -> Training
* Select Region as ```europe-west4(Netherlands)``` and click on Create displayed on the top of the console.
* Training model
	* Dataset: ```titanic```
	* Objective: ```Classification```
	* Click on Custom Training and click Continue.
* Model details
	* Leave the model name as it is.
	* Click on Advanced options. In data split, click on Random assignment.
* Training container
	* Select pre-built container
	* Model framework: ```scikit-learn```
	* Model framework version : ```0.23```
	* Pre-built container settings
		* Package location: Click on BROWSE, your project name, titanic folder, dist folder and trainer-0.1.tar.gz file.
		* Python module: ```trainer.task```
	* BigQuey project for exporting data: Project ID
	* Model output directory: Click on BROWSE, your project name, titanic folder, trial folder.
	* Click continue.
* Hyperparameters: Click continue
* Compute and pricing:
	* Region: europe-west4
	* Machine type: Under Standard, n1-standard-4
	* Click continue.
* Prediction container
	* Select Pre-built container.
	* Model framework: ```scikit-learn```
	* Model framework version: ```0.23```			
	* Model directory: ```gs://YOUR-BUCKET-NAME/titanic/trial/```
* Click on Start training. It may take from 15 to 20 minutes.

## 6. Model Evaluation
After the training job completion artifacts will be exported under ```gs://YOUR-BUCKET-NAME/titanic/trial/``` You can inspect the report.txt file which contains evaluation metrics and classification report of the model. Don't edit any files.

## 7. Model Deployment
* In the left pane, click on Models.
* Click on the titanic model and DEPLOY TO ENDPOINT
* Define your endpoint
	* Endpoint name: ```titanic-endpoint```
	* Click continue
* Model settings
	* Traffic split: 100 percent
	* Minimum number of compute nodes: 1
	* Maximum number of compute nodes: 2
	* Machine type: Under Standard, n1-standard-4
* Click CONTINUE and DEPLOY

## 8. Model Prediction
* After the model is deployed you can see a text box for JSON request.
* Copy paste this JSON query. And click on Predict:
```
	{
	    "instances": [
	      ["male", 49.5, 30.6, 1, "C", "New York, NY", 0, 0], 
	      ["female", 48.0, 39.6, 1, "C", "London / Paris", 0, 1],
	      ["male", 30, 40.6, 1, "S", "London / Paris", 1, 0]]
	}
```
* You'll get prediction in JSON format in right side. Value will be 0 or 1. 0 is more likely the person will not survive and 1 means the person will survive the titanic accident.
* You can change the values and see the predictions. The sequence of the input features is [‘sex', ‘age', ‘fare', ‘pclass', ‘embarked', ‘home_dest', ‘parch', ‘sibsp'].
