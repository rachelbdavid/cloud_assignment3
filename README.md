# cloud_assignment3
	1. Creating project
		a. A google cloud platform project is needed. If you don't have one follow the steps to create it.
		b. In the navigation menu, select IAM & Admin -> Manage resources.
		c. Click on Create Project.
		d. Give your desired project name. Project name should be unique.
		e. If your mail-id is a part of an organization, your organization will be displayed. You can switch to No organization if you don't want your organization name.
		f. If your mail-id is personal, it will be set to No organization by default.
		g. Click on Create after giving the needed details.
		
	2. Loading data into BigQuery
		From navigation menu, go to BigQuery.
		a. Creating dataset
			i. Select the project name from the top of the console.
			ii. In the left pane, click on the three dots next to the project name and select Create dataset.
			iii. In Dataset ID, give the value as titanic.
			iv. In Data location, select europe-west4(Netherlands).
			v. Leave the rest of the details as default and click on Create Dataset.
		b. Creating table
			i. Download the dataset called titanic.csv.
			ii. Click on the three dots next to the titanic dataset created in the previous step and select Open.
			iii. Click on Create Table option visible in the main console.
			iv. In the sidebar that opened, do the following:
				1) Create table from: Upload
				2) Select file: upload the downloaded titanic dataset.
				3) File format: CSV
				4) Table name: survivors
				5) Auto-detect: Select auto-detect checkbox - Schema and input parameters
			v. Click Create Table.
			
	3. Creating dataset
		a. Creating ML dataset
			i. In the navigation menu, click on Vertex AI under Artificial intelligence.
			ii. In the dashboard, select Region as europe-west4(Netherlands).
			iii. Click on Create dataset displayed in Preparing your training data.
			iv. Give dataset name as titanic.
			v. In Select a data type and objective, click on Tabular tab and select Regression/Classification
			vi. Select Region as europe-west4(Netherlands) and click Create.
		b. Selecting data source
			i. Now we should connect previously created dataset to this source.
			ii. Under Select a data source, select Select a table or view from BigQuery. 
			iii. Under Select a table or view from BigQuery, click on Browse and select survivors dataset.
			iv. Click on continue.
			v. Now the details of the dataset will be visible. In order to run the statistical analysis, click on Generate Statistics.
			
	4. Custom training package using Notebooks
		a. Creating notebook instance
			i. Select Workbench from left pane and click on New Notebook.
			ii. Select Python3 and on the pop up, leave it to default values and click Create.
			iii. After instance gets created, you can see OPEN JUPYTERLAB. Click on it to open Jupyter notebook.
		b. Creating package
			i. Click on '+' on the Jupyter notebook found in left upper corner, and select Terminal from launcher.
			ii. Execute these 2 commands in the terminal.
				mkdir -p /home/jupyter/titanic/trainer
				touch /home/jupyter/titanic/setup.py /home/jupyter/titanic/trainer/__init__.py /home/jupyter/titanic/trainer/task.py
			iii. After running these commands you can see now folders created.
			iv. Copy paste the task.py code in titanic/trainer/task.py
			v. Copy paste the setup.py code in titanic/setup.py
			Copy paste these four commands in terminal to create a bucket in your current project.
				export REGION="europe-west4"
				export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
				export BUCKET_NAME=$PROJECT_ID"-bucket"
				gsutil mb -l $REGION "gs://"$BUCKET_NAME
			vi. Install the required libraries using these commands in terminal
				cd /home/jupyter/titanic
				pip install setuptools
				python setup.py install
			vii. Run the following training code to verify that it executes without issues
      ```terminal
				python -m trainer.task -v \
				    --model_param_kernel=linear \
				    --model_dir="gs://"$BUCKET_NAME"/titanic/trial" \
				    --data_format=bigquery \
				    --training_data_uri="bq://"$PROJECT_ID".titanic.survivors" \
				    --test_data_uri="bq://"$PROJECT_ID".titanic.survivors" \
				    --validation_data_uri="bq://"$PROJECT_ID".titanic.survivors"
       ```
			viii. If it executes correctly, it displays the following message
				INFO:root:f1score: 0.85
				INFO:root:Training job completed. Exiting...
			ix. To create python package, run the following command in the terminal
				cd /home/jupyter/titanic
				python setup.py sdist
			x. Finally run the following command to copy the package to GCS so that the training service can use it to train a new model when you need to.
				gsutil cp dist/trainer-0.1.tar.gz "gs://"$BUCKET_NAME"/titanic/dist/trainer-0.1.tar.gz"
				
	5. Model training
		a. In the navigation menu, go to Vertex AI -> Training
		b. Select Region as europe-west4(Netherlands)  and click on Create displayed on the top of the console.
		c. Training model
			i. Dataset: titanic
			ii. Objective: Classification
			iii. Click on Custom Training and click Continue.
		d. Model details
			i. Leave the model name as it is.
			ii. Click on Advanced options. In data split, click on Random assignment.
		e. Training container
			i. Select pre-built container
			ii. Model framework: scikit-learn
			iii. Model framework version : 0.23
			iv. Pre-built container settings
				1) Package location: Click on BROWSE, your project name, titanic folder, dist folder and trainer-0.1.tar.gz file.
				2) Python module: trainer.task
			v. BigQuey project for exporting data: Project ID
			vi. Model output directory: Click on BROWSE, your project name, titanic folder, trial folder.
			vii. Click continue.
		f. Hyperparameters: Click continue
		g. Compute and pricing:
			i. Region: europe-west4
			ii. Machine type: Under Standard, n1-standard-4
			iii. Click continue.
		h. Prediction container
			i. Select Pre-built container.
			ii. Model framework: scikit-learn
			iii. Model framework version: 0.23
			iv. Model directory: gs://YOUR-BUCKET-NAME/titanic/trial/
		i. Click on Start training. It may take from 15 to 20 minutes.
		
	6. Model Evaluation
		After the training job completion artifacts will be exported under gs://YOUR-BUCKET-NAME/titanic/trial/ You can inspect the report.txt file which contains evaluation metrics and classification report of the model. Don't edit any files.
		
	7. Model Deployment
		a. In the left pane, click on Models.
		b. Click on the titanic model and DEPLOY TO ENDPOINT
		c. Define your endpoint
			i. Endpoint name: titanic-endpoint
			ii. Click continue
		d. Model settings
			i. Traffic split: 100 percent
			ii. Minimum number of compute nodes: 1
			iii. Maximum number of compute nodes: 2
			iv. Machine type: Under Standard, n1-standard-4
		e. Click CONTINUE and DEPLOY
		
	8. Model Prediction
		a. After the model is deployed you can see a text box for JSON request.
		b. Copy paste this JSON query. And click on Predict
    ```json
			{
			    "instances": [
			      ["male", 49.5, 30.6, 1, "C", "New York, NY", 0, 0], 
			      ["female", 48.0, 39.6, 1, "C", "London / Paris", 0, 1],
			      ["male", 30, 40.6, 1, "S", "London / Paris", 1, 0]]
			}
      ```
		c. You'll get prediction in JSON format in right side. Value will be 0 or 1. 0 is more likely the person will not survive and 1 means the person will survive the titanic accident.
		d. You can change the values and see the predictions. The sequence of the input features is [‘sex', ‘age', ‘fare', ‘pclass', ‘embarked', ‘home_dest', ‘parch', ‘sibsp'].
