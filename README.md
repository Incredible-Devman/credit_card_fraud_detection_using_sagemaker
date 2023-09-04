# Credit card fraud detection 

Since July this year 2020, **Amazon Fraud Detector** is Generally Available, meaning that apart from using Amazon SageMaker doing data processing and ML training, now you can use Amazon Fraud Detector Service, which it's a fully managed service that makes it even more easier and faster to intergrate with your online application, as simple as just using API calls to the Amazon Fraud Detector service.

![How Amazon Fraud Detector Works](https://d1.awsstatic.com/diagrams/HIW_Fraud_Detector.342744016f00602e5bdc4ba551f9a2a0736e0e14.png)

To learn more about the service, please visit to the Amazon Blog:  [Amazon Fraud Detector is now Generally Available](https://aws.amazon.com/blogs/aws/amazon-fraud-detector-is-now-generally-available/)

In this Github Repository, we still stick with using Amazon SageMaker Service as the lab material, for you to understand the machine learning flow from data processing, choosing algorithm, performing ML training and model deployment.



## Resources in this Lab 

Dataset: [Credit Card Fraud Detection from Kaggle](https://www.kaggle.com/mlg-ulb/creditcardfraud)

Here are some of the screenshots for the dataset, those columns v1 to v28 are the data that already finished [feature engineering](https://en.wikipedia.org/wiki/Feature_engineering), those are in the feature format. So we can straightly use this dataset pass into the supervised learning algorithm for training.

![preview_data1.png](./images/preview_data1.png)

![preview_data2.png](./images/preview_data2.png)



## Lab Architecture
![lab_architecture.png](./images/lab_architecture.png)
![lab_architecture_old.png](./images/lab_architecture_old.png)


As illustrated in the preceding diagram, this is a big data processing in this model:
1. Developer uploads dataset to S3 and then loads dataset to SageMaker

2. Developer train machine learning job and deploy model with SageMaker

3. Create endpoint on SageMaker that can invoked by Lambda

4. Create API with API Gateway in order to send request between Application and API Gateway 

5. API Gateway send request to Lambda that invoke prediction job on SageMaker endpoint

6. Amazon SageMaker response the result of prediction from API Gateway back to Application

## AWS SageMaker introduction
Amazon SageMaker is a fully managed machine learning service. With Amazon SageMaker, data scientists and developers can quickly and easily build and train machine learning models, and then directly deploy them into a production-ready hosted environment. It provides an integrated Jupyter authoring notebook instance for easy access to your data sources for exploration and analysis, so you don't have to manage servers. It also provides common machine learning algorithms that are optimized to run efficiently against extremely large data in a distributed environment. 

## Prerequisites
1.	Sign-in a AWS account, and make sure you have select **N.Ohio region**<br>
2.	Make sure your account have permission to create IAM role for following services: **S3**, **SageMaker**, **Lambda**, **API Gateway**<br>
3.	Download **this repository** and unzip, ensure that folder including two folders and some files:<br>
Folder: **Flask-app** (contains the code to build application of demo)<br>
Folder: **data** including **crditcard.csv** (training and testing data of machine learning job of SageMaker)



## Train and build model on SageMaker


#### Create following IAM roles

The role that allow Lambda trigger prediction job with Amazon SageMaker by client request from application<br><br>
The role that allow Amazon SageMaker to execute full job and get access to S3, CloudWatchLogs (create this role in SageMaker section)<br><br>



* 	On the **service** menu, click **IAM**.<br>
* 	In the navigation pane, choose **Roles**.<br>
* 	Click **Create role**.<br><br>
* 	For role type, choose **AWS Service**, find and choose **Lambda**, and choose **Next: Permissions**.<br>
* 	On the **Attach permissions policy** page, search and choose **AmazonSageMakerFullAccess**, and choose **Next: Review**.<br>
* 	On the **Review** page, enter the following detail: <br>
**Role name**: **invoke_sage_endpoint**<br>
* 	Click **Create role**.<br>
After above step you successfully create the role for Lambda trigger prediction job with Amazon SageMaker<br>
![iam_role1.png](./images/iam_role1.png) <br>
* 	On the service menu, click **Amazon SageMaker**<br>
* 	In the navigation pane, choose **Notebook**.<br>
* 	Click **Create notebook instance** and you can find following screen<br>

* 	In **IAM role** blank select **Create a new role**<br>
* 	For **S3 buckets you specify**, choose **Any S3 bucket** and click **Create role**<br>
![iam_role2.png](./images/iam_role2.png) <br>
* 	Back to **IAM** console, click **Roles**<br>
* 	Click the role **AmazonSageMaker-ExecutionRole-xxxxxxxxxxxxxxx** you just created by before step<br>

* 	In **Permissions** tab below click **Attach policy**<br>
* 	Search and choose policy name **CloudWatchLogsFullAccess** then click **Attach policy**<br>
* 	You will see below screen on console<br>
![iam_role3.png](./images/iam_role3.png)  <br>
You successfully create the role for Amazon SageMaker to execute full job and get access to S3, CloudWatchLogs<br>



#### Create S3 bucket to store data
Create a bucket to store train_data.csv, test_data.csv which also provide the bucket location for SageMaker jobs to store result<br>
* 	On the **service** menu, click **S3**.<br>
* 	Click **Create bucket**.<br>
* 	Enter the **Bucket name “yourbucketname-dataset” (e.g., tfdnn-dataset)** and ensure that the bucket name is unique so that you can create.<br>
* 	For Region choose **US East (N. Virginia)**.<br>
* 	Click **Create**.<br>
* 	Click **“yourbucketname-dataset”** bucket.<br>
* 	Click **Upload**.<br>
* 	Click **Add files**.<br>
* 	Select file **train_data.csv** and **test_data.csv** in **data** folder then click **Upload**.<br>
* 	Make sure that your S3 contain this bucket **“yourbucketname-dataset”** and the region is **US East (N. Virginia)**<br>

Congratulations! now you can start building notebook instance on **SageMaker**<br>

#### Create notebook instance on SageMaker
* 	On the **Services** menu, click **Amazon SageMaker**.<br>
* 	In the navigation pane, choose **Notebook**.<br>
* 	Click **Create notebook instance**<br>
* 	Enter **Notebook instance name** for **“ML-cluster”**<br>
* 	For **Notebook instance type**, select **“ml.t2.medium”**<br>
* 	For IAM role, choose **AmazonSageMaker-ExecutionRole-xxxxxxxxxxxxxxx** you just created by before step<br>
* 	Leave other blank by default and click **Create notebook instance**<br><br>
![create_notebook1.png](./images/create_notebook1.png) <br>
* 	Notebook instance will pending for a while until it in service<br><br>
![create_notebook2.png](./images/create_notebook2.png) <br>
* 	Until the status of **ML-cluster** transform into **InService** click **open**<br><br>
![create_notebook3.png](./images/create_notebook3.png) <br>
* 	The jupyter notebook screen show up like below<br><br>
![create_notebook4.png](./images/create_notebook4.png) <br>


#### Deploy the trained model and get the result

* 	Move on to the cell below **“Deploy the trained Model”** heading<br>
* 	When you run this cell represent that **creates an endpoint**** which serves prediction requests in real-time with an instance<br>
* 	Set **instance_type** as **'ml.m4.xlarge'** as default and run the cell<br><br>
![deploy_model1.png](./images/capture1.png) <br> 
Wait for a while until it finish deployment<br><br>
![deploy_model2.png](./images/capture2.png) <br>  


#### Invoke the endpoint

* 	The cell below **“Invoke the Endpoint to get inferences”** heading do prediction job and output the prediction result<br>
* 	Modify the data in **iris_predictor.predict()**<br>
Change to this data below:<br>

[-15.819178720771802,8.7759971528627,-22.8046864614815,11.864868080360699,-9.09236053189517,-2.38689320657655,-16.5603681078199,0.9483485947860579,-6.31065843275059,-13.0888909176936,9.81570317447819,-14.0560611837648,0.777191846436601,-13.7610179615936,-0.353635939812489,-7.9574472262599505,-11.9629542349435,-4.7805077876172,0.652498045264831,0.992278949261366,-2.35063374523783,1.03636187430048,1.13605073696052,-1.0434137405139001,-0.10892334328197999,0.657436778462222,2.1364244708551396,-1.41194537483904,-0.3492313067728856]<br>
 <br>
**This data is selected from real fraud data that can test accuracy of our model for prediction (output contain class 0 for normal transaction and class 1 for fraud transaction )**<br><br>
![invoke_endpoint1.png](./images/invoke_endpoint1.png) <br>  
Then run the cell to get the output<br><br>
![invoke_endpoint2.png](./images/invoke_endpoint2.png) <br>   
**The result shows the fraud probability of this credit card data is 0.9518** (for example of this model)<br>

* 	Now don’t run the cells below **“(Optional) Delete the Endpoint”** heading. That means to delete this endpoint. You can delete it after this workshop.<br>
* 	Back to **SageMaker** console, click **Endpoints** you will find the endpoint you just created and click **Models** you will find the model you deployed<br><br>
![invoke_endpoint3.png](./images/invoke_endpoint3.png) <br>   
![invoke_endpoint4.png](./images/invoke_endpoint4.png) <br>   
**Congratulations! You successfully deploy a model and create an endpoint on SageMaker.**<br>
Next step you will learn how to create a Lambda function to invoke that endpoint<br>


## Integrate with serverless application

#### Create a Lambda function to invoke SageMaker endpoint

* 	On the **Services** menu, click **Lambda**.<br>
* 	Click **Create function**.<br>
* 	Choose **Author from scratch**.<br>
* 	Enter function Name **endpoint_invoker**.<br>
* 	Select **python 3.6** in Runtime blank.<br>
* 	Select **Choose an existing role** in **Role** blank and choose **invoke_sage_endpoint** as **Existing role**.<br><br>
![lambda_function1.png](./images/lambda_function1.png) <br>   
* 	Click **Create function** and you will see below screen.<br><br>
![lambda_function2.png](./images/lambda_function2.png) <br>  
* 	Click **endpoint_invoker** blank in **Designer** and replace original code that existing in **Function code** editor with below code. Remember to modify **ENDPOINT_NAME** to **your SageMaker endpoint name**<br><br>
*
      import boto3
      import json
      client = boto3.client('runtime.sagemaker')
      ENDPOINT_NAME = 'sagemaker-tensorflow-xxxx-xx-xx-xx-xx-xx-xxx'

      def lambda_handler(event, context):
          # TODO implement
          print(event['body'])
          print(type(event['body']))
          # list(event['body'])
          target = json.loads(event['body'])
          result = client.invoke_endpoint(EndpointName=ENDPOINT_NAME,Body=json.dumps(target))
          response = json.loads(result['Body'].read())

          print(response)
        
          http_response = {
              'statusCode': 200,
              'body': json.dumps(response),
              'headers':{
                  'Content-Type':'application/json',
                  'Access-Control-Allow-Origin':'*'
              }
          }
          return http_response


<br>![lambda_function3.png](./images/lambda_function3.png) <br> 
* 	Click **Save** to save the change of function.




#### Create an API with API Gateway

* 	On the **Service menu**, click **API Gateway**.<br>
* 	Click **Get Started** if you are first time to this console<br>
* 	Choose **new API** of **Create new API**<br>
* 	Enter API name **predict_request** and set **Endpoint Type** as **Reginal** then click **Create API**<br>
* 	Click **Actions** and select **Create Resource**<br>
* 	Enable **Configure as proxy resource** and enable **API Gateway CORS**<br><br>
![api_gateway1.png](./images/api_gateway1.png) <br> 
* 	Click **Create Resource**<br>
* 	In proxy setup, choose **Lambda Function Proxy** for **Integration type**, **Lambda Region** select **us-east-1**, select **“endpoint_invoker”** for **Lambda Function** then click **Save**.<br><br>
![api_gateway2.png](./images/api_gateway2.png) <br>  
Click **OK** in order to add permission to Lambda function<br><br>
![api_gateway3.png](./images/api_gateway3.png) <br>  
You will see this screen<br><br>
![api_gateway4.png](./images/api_gateway4.png) <br>  
* 	Click **Method Response**<br>
* 	Click **Add Response** and enter **200** and save<br>
* 	Spread **HTTP 200 status** blank<br><br>
![api_gateway5.png](./images/api_gateway5.png) <br>  
Click **Add Header** and enter **Access-Control-Allow-Origin** then save<br>
Click **Add Response Model** and enter **application/json** for **Content type**<br>
Select **Empty** in **Model** then save<br><br>
![api_gateway6.png](./images/api_gateway6.png) <br>  
* 	Click **Actions** then select **Deploy API**<br>
* 	Choose **New Stage** for **Deployment stage**<br>
* 	Enter **dev** for Stage name and click **Deploy**<br>
* 	You will see the **Invoke URL** for this API<br><br>
![api_gateway7.png](./images/api_gateway7.png) <br>  
Now you finish API deployment and you can try the demo on application<br>



#### Demo it

#### Demo in local environment
* 	For this workshop, we build a **Flask** web application to call API<br>
Detail: http://flask.pocoo.org/<br>
**Make sure that you have installed python2.7** in your system<br>
https://www.python.org/downloads/<br>
You also need to setup environment for **Flask**<br>
Run **pip install Flask** in terminal<br>
![demo1.png](./images/demo1.png) <br> 
* 	The source code of application is inside the **flask-app** folder<br>
* 	Modify the file **app.py** that inside **flask-app** with your editor<br>
![demo2.png](./images/demo2.png) <br> 
In function **predict_result()** find the post URL in about line 33<br>
Modify the URL to your **invoke URL{proxy+}** that have created on API Gateway<br>
![demo3.png](./images/demo3.png) <br> 

In terminal go to the same directory path of **flask-app** folder<br>
e.g., Mac example below<br><br>
![demo4.png](./images/demo4.png) <br> 
Run below two commands to run Flask web application<br>
**export FLASK_DEBUG=1**    <br>
**FLASK_APP=app.py flask run**
<br><br>
![demo5.png](./images/demo5.png) <br> 
Python will run it on **localhost:5000**<br>
We prepare 10 input data that is real fraud data for prediction at the screen below (in app.py)<br><br>
![demo6.png](./images/demo6.png) <br> 
![demo7.png](./images/demo7.png) <br> 
You can copy and paste each data in input area and click **predict**<br><br>
![demo8.png](./images/demo8.png) <br> 
Then you will get the prediction result on http://localhost:5000/result<br>
**Default input data is “input_data”**<br><br>
![demo9.png](./images/demo9.png) <br> 
Change input data to input_data5<br><br>
![demo10.png](./images/demo10.png) <br> 
You will get different result<br><br>
![demo11.png](./images/demo11.png) <br> 
For non-fraud transaction data in the real world, we also prepare 5 transactions to test<br>
You can see the results of prediction<br><br>
![demo12.png](./images/demo12.png) <br> <br>
![demo13.png](./images/demo13.png) <br> <br>
![demo14.png](./images/demo14.png) <br> <br>
* Let's try those different data to explore the fraud detection result<br><br>
![demo15.png](./images/demo15.png) <br> <br>
![demo16.png](./images/demo16.png) <br> <br>
![demo17.png](./images/demo17.png) <br> <br>

#### Demo in AWS Cloud9 environment
* On the **service** menu, click **Cloud9**.<br>
* Click **Create environment** 
* Enter the name of your environment in **Name** (e.g., **flask-env**)
* Click **Next step**
* Select **Create a new instance for environment (EC2)** and **t2.micro (1 GiB RAM + 1 vCPU)** for Environment type and Instance type
* Click **Next step**
* Click **Create environment**
You will need to wait Cloud9 for setup environment in a few minutes <br>
![cloud9_1.png](./images/cloud9_1.png) <br> <br>

* Paste command at below terminal
* 
      git clone https://github.com/ecloudvalley/Credit-card-fraud-detection-with-SageMaker-using-TensorFlow-estimators.git

* 
      sudo pip install flask

* Click **app**.**py** in
**Credit-card-fraud-detection-with-SageMaker-using-TensorFlow-estimators/flask-app** <br>
* In line 80, modify the port from **5000** to **8080** <br>
![cloud9_2.png](./images/cloud9_2.png) <br> <br>
* In line 50 ~ 51, modify the **url of post request to your API url** that you create in API Gateway step <br>
![api_url.png](./images/api_url.png) <br> <br>
* Click **Run** on the toolbar and click **Preview Running Application** in **Preview** <br>
![cloud9_3.png](./images/cloud9_3.png) <br> <br>
The screen will show as below <br>
![cloud9_4.png](./images/cloud9_4.png) <br> <br>
* Click Pop Out Window to open website in another tab in your browser <br>
![cloud9_5.png](./images/cloud9_5.png) <br> <br>

We prepare 10 input data that is real fraud data for prediction at the screen below (in app.py)<br><br>
![input.png](./images/input.png) <br> <br>
You can copy and paste each data in input area and click **predict**<br><br>
![cloud9_6.png](./images/cloud9_6.png) <br> <br>
Then you will get the prediction result on **cloud9 website**<br>
![cloud9_7.png](./images/cloud9_7.png) <br> <br>
![cloud9_8.png](./images/cloud9_8.png) <br> <br>

## Appendix

In the real-world example, when the system detects the fraud you may want to inform your client by sending the message through mobile phone or email, so actually, you can integrate with **SNS** service in this architecture

* First you need to create a topic and subscribe that topic in SNS dashboard<br>
![sns1.png](./images/sns1.png) <br> <br>
* There are various types of target to send the message<br><br>
![sns2.png](./images/sns2.png) <br> <br>
* Back to your Lambda function and add some code as below<br><br>
* 
      import boto3
      import json
      from time import gmtime, strftime
      import time
      import datetime
        
      client = boto3.client('runtime.sagemaker')
      sns = boto3.client('sns')
        
      ENDPOINT_NAME = 'YourSageMakerEndpointName'
        
      def lambda_handler(event, context):
          # TODO implement
          # print(event['body'])
          # print(type(event['body']))
          # list(event['body'])
          target = json.loads(event['body'])
          result = client.invoke_endpoint(EndpointName=ENDPOINT_NAME,Body=json.dumps(target))
          response = json.loads(result['Body'].read())
        
          print(response)
          fraud_rate = response['result']['classifications'][0]['classes'][1]['score']
          fraud = float(fraud_rate)*100
        
          if fraud>=90:
              now = datetime.datetime.now()
              tdelta = datetime.timedelta(hours=8)
              mytime = now + tdelta
        
              mail_response = sns.publish(
              TopicArn='YourSNSTopicArn',
              Message='Do you remember this transaction?' + '\n' + mytime.strftime("%Y-%m-%d %H:%M:%S") + '\n Please check your credit card account \n it might be a fraud transaction',
              Subject='Transaction Alert')
        
          http_response = {
              'statusCode': 200,
              'body': json.dumps(response),
              'headers':{
                  'Content-Type':'application/json',
                  'Access-Control-Allow-Origin':'*'
              }
          }
          return http_response 
  
 * Remember to change **ENDPOINT_NAME** of SageMaker and **TopicArn** of SNS<br>
 * In this example, SNS will push a message to my Email if the fraud rate of prediction is over 90%<br><br>

  ![alert1.png](./images/alert1.png) <br> <br>


