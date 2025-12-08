---
title: "Blog 3"
date: "2025-10-09"
weight: 3 # Sửa từ 1 sang 3
chapter: false
pre: " <b> 3.3. </b> " # Giữ nguyên
---

# **Intelligent document processing at scale with generative AI and Amazon Bedrock Data Automation**

**by Nikita Kozodoi, Aiham Taleb, Francesco Cerizzi, Liza (Elizaveta) Zinovyeva, Nuno Castro, Ozioma Uzoegwu, Eren Tuncer, and Zainab Afolabi | on 11 JUL 2025 | in [Amazon Bedrock](#), [Amazon Bedrock Data Automation](#), [Generative AI](#), [Intermediate (200)](#), [Technical How-to](#) | [Permalink](#) | [Comments](#) | [Share](#)**

---

Extracting information from unstructured documents at scale is a recurring business task. Common use cases include creating product feature tables from descriptions, extracting metadata from documents, and analyzing legal contracts, customer reviews, news articles, and more. A classic approach to extracting information from text is named entity recognition (NER). NER identifies entities from predefined categories, such as persons and organizations. Although various AI services and solutions support NER, this approach is limited to text documents and only supports a fixed set of entities. Furthermore, classic NER models can’t handle other data types such as numeric scores (such as sentiment) or free-form text (such as summary). **Generative AI** unlocks these possibilities without costly data annotation or model training, enabling more comprehensive intelligent document processing (IDP).

**AWS** recently announced the general availability of **Amazon Bedrock Data Automation**, a feature of **Amazon Bedrock** that automates the generation of valuable insights from unstructured multimodal content such as documents, images, video, and audio. This service offers pre-built capabilities for IDP and information extraction through a unified API, alleviating the need for complex prompt engineering or fine-tuning, and making it an excellent choice for document processing workflows at scale. To learn more about Amazon Bedrock Data Automation, refer to [Simplify multimodal generative AI with Amazon Bedrock Data Automation](#).

Amazon Bedrock Data Automation is the recommended approach for IDP use case due to its simplicity, industry-leading accuracy, and managed service capabilities. It handles the complexity of document parsing, context management, and model selection automatically, so developers can focus on their business logic rather than IDP implementation details.

Although Amazon Bedrock Data Automation meets most IDP needs, some organizations require additional customization in their IDP pipelines. For example, companies might need to use self-hosted foundation models (FMs) for IDP due to regulatory requirements. Some customers have builder teams who might prefer to maintain full control over the IDP pipeline instead of using a managed service. Finally, organizations might operate in AWS Regions where Amazon Bedrock Data Automation is not available (available in us-west-2 and us-east-1 as of June 2025). In such cases, builders might use Amazon Bedrock FMs directly or perform optical character recognition (OCR) with **Amazon Textract**.

This post presents an **end-to-end IDP application** powered by Amazon Bedrock Data Automation and other AWS services. It provides a reusable AWS infrastructure as code (IaC) that deploys an IDP pipeline and provides an intuitive UI for transforming documents into structured tables at scale. The application only requires the user to provide the input documents (such as contracts or emails) and a list of attributes to be extracted. It then performs IDP with generative AI.

The application code and deployment instructions are [available on GitHub](#) under the MIT license.

## Solution overview

The IDP solution presented in this post is deployed as IaC using the **AWS Cloud Development Kit (AWS CDK)**. Amazon Bedrock Data Automation serves as the primary engine for information extraction. For cases requiring further customization, the solution also provides alternative processing paths using Amazon Bedrock FMs and Amazon Textract integration.

We use **AWS Step Functions** to orchestrate the IDP workflow and parallelize processing for multiple documents. As part of the workflow, we use **AWS Lambda** functions to call Amazon Bedrock Data Automation or Amazon Textract and Amazon Bedrock (depending on the selected parsing mode). Processed documents and extracted attributes are stored in **Amazon Simple Storage Service (Amazon S3)**.

A Step Functions workflow with the business logic is invoked through an API call performed using an AWS SDK. We also build a containerized web application running on **Amazon Elastic Container Service (Amazon ECS)** that is available to end-users through **Amazon CloudFront** to simplify their interaction with the solution. We use **Amazon Cognito** for authentication and secure access to the APIs.

The following diagram illustrates the architecture and workflow of the IDP solution.

![Architecture Diagram](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/24/architecture-3.png)

The IDP workflow includes the following steps:

1.  A user logs in to the web application using credentials managed by Amazon Cognito, selects input documents, and defines the fields to be extracted from them in the UI. Optionally, the user can specify the parsing mode, LLM to use, and other settings.
2.  The user starts the IDP pipeline.
3.  The application creates a pre-signed S3 URL for the documents and uploads them to Amazon S3.
4.  The application triggers Step Functions to start the state machine with the S3 URIs and IDP settings as inputs. The Map state starts to process the documents concurrently.
5.  Depending on the document type and the parsing mode, it branches to different Lambda functions that perform IDP, save results to Amazon S3, and send them back to the UI:
    * **Amazon Bedrock Data Automation:** Documents are directed to the “Run Data Automation” Lambda function. The Lambda function creates a blueprint with the user-defined fields schema and launches an asynchronous Amazon Bedrock Data Automation job. Amazon Bedrock Data Automation handles the complexity of document processing and attribute extraction using optimized prompts and models. When the job results are ready, they’re saved to Amazon S3 and sent back to the UI. This approach provides the best balance of accuracy, ease of use, and scalability for most IDP use cases.
    * **Amazon Textract:** If the user specifies Amazon Textract as a parsing mode, the IDP pipeline splits into two steps. First, the “Perform OCR” Lambda function is invoked to run an asynchronous document analysis job. The OCR outputs are processed using the `amazon-textract-textractor` library and formatted as Markdown. Second, the text is passed to the “Extract attributes” Lambda function (Step 6), which invokes an Amazon Bedrock FM given the text and the attributes schema. The outputs are saved to Amazon S3 and sent to the UI.
    * **Handling office documents:** Documents with suffixes like .doc, .ppt, and .xls are processed by the “Parse office” Lambda function, which uses LangChain document loaders to extract the text content. The outputs are passed to the “Extract attributes” Lambda function (Step 6) to proceed with the IDP pipeline.
6.  If the user chooses an Amazon Bedrock FM for IDP, the document is sent to the “Extract attributes” Lambda function. It converts a document into a set of images, which are sent to a multimodal FM with the attributes schema as part of a custom prompt. It parses the LLM response to extract JSON outputs, saves them to Amazon S3, and sends it back to the UI.
7.  The web application checks the state machine execution results periodically and returns the extracted attributes to the user when they are available.

## Prerequisites

You can deploy the IDP solution from your local computer or from an **Amazon SageMaker** notebook instance. The deployment steps are detailed in the solution README file.

If you choose to deploy using a SageMaker notebook, which is recommended, you will need access to an AWS account with permissions to create and launch a SageMaker notebook instance.

## Deploy the solution

To deploy the solution to your AWS account, complete the following steps:

1.  Open the **AWS Management Console** and choose the Region in which you want to deploy the IDP solution.
2.  Launch a SageMaker notebook instance. Provide the notebook instance name and notebook instance type, which you can set to `ml.m5.large`. Leave other options as default.
3.  Navigate to the Notebook instance and open the IAM role attached to the notebook. Open the role on the **AWS Identity and Access Management (IAM)** console.
4.  Attach an inline policy to the role and insert the following policy JSON:

    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "cloudformation:*",
            "s3:*",
            "iam:*",
            "sts:AssumeRole"
          ],
          "Resource": "*"
        },
        {
          "Effect": "Allow",
          "Action": [
            "ssm:GetParameter",
            "ssm:GetParameters"
          ],
          "Resource": "arn:aws:ssm:*:*:parameter/cdk-bootstrap/*"
        }
      ]
    }
    ```

5.  When the notebook instance status is marked as *InService*, choose **Open JupyterLab**.
6.  In the JupyterLab environment, choose **File**, **New**, and **Terminal**.
7.  Clone the solution repository by running the following commands:

    ```bash
    cd SageMaker
    git clone [https://github.com/aws-samples/intelligent-document-processing-with-amazon-bedrock.git](https://github.com/aws-samples/intelligent-document-processing-with-amazon-bedrock.git)
    ```

8.  Navigate to the repository folder and run the script to install requirements:

    ```bash
    cd intelligent-document-processing-with-amazon-bedrock
    sh install_deps.sh
    ```

9.  Run the script to create a virtual environment and install dependencies:

    ```bash
    sh install_env.sh
    source .venv/bin/activate
    ```

10. Within the repository folder, copy the `config-example.yml` to a `config.yml` to specify your stack name. Optionally, configure the services and indicate the modules you want to deploy (for example, to disable deploying a UI, change `deploy_streamlit` to `False`). Make sure you add your user email to the Amazon Cognito users list.
11. Configure Amazon Bedrock model access by opening the Amazon Bedrock console in the Region specified in the `config.yml` file. In the navigation pane, choose **Model Access** and make sure to enable access for the model IDs specified in `config.yml`.
12. Bootstrap and deploy the AWS CDK in your account:

    ```bash
    cdk bootstrap
    cdk deploy
    ```

Note that this step may take some time, especially on the first deployment. Once deployment is complete, you should see the message as shown in the following screenshot. You can access the Streamlit frontend using the CloudFront distribution URL provided in the **AWS CloudFormation** outputs. The temporary login credentials will be sent to the email specified in `config.yml` during the deployment.

![Deployment Output](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-2-1.jpeg)

## Using the solution

This section guides you through two examples to showcase the IDP capabilities.

### Example 1: Analyzing financial documents

In this scenario, we extract key features from a multi-page financial statement using Amazon Bedrock Data Automation. We use a sample document in PDF format with a mixture of tables, images, and text, and extract several financial metrics. Complete the following steps:

1.  Upload a document by attaching a file through the solution UI.
![Upload Document](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-3-1.png)

2.  On the **Describe Attributes** tab, either manually list the names and descriptions of the attributes or upload these fields in JSON format. We want to find the following metrics:
    * Current cash in assets in 2018
    * Current cash in assets in 2019
    * Operating profit in 2018
    * Operating profit in 2019

    ![Describe Attributes](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-4-1.png)  
3.  Choose **Extract attributes** to start the IDP pipeline.

The provided attributes are integrated into a custom blueprint with the inferred attributes list, which is then used to invoke a data automation job on the uploaded documents. After the IDP pipeline is complete, you will see a table of results in the UI. It includes an index for each document in the `_doc` column, a column for each of the attributes you defined, and a `file_name` column that contains the document name.
![Extract Attributes Button](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-5-1.png)

From the following statement excerpts, we can see that Amazon Bedrock Data Automation was able to correctly extract the values for current assets and operating profit.

![Results Table](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-6-1.png)
![Statement Excerpts](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-7-1.png)

The IDP solution is also able to do complex calculations beyond well-defined entities. Let’s say we want to calculate the following accounting metrics:
* Liquidity ratios (Current assets/Current liabilities)
* Working capitals (Current assets – Current liabilities)
* Revenue increase ((Revenue year 2/Revenue year 1) – 1)

We define the attributes and their formulas as parts of the attributes’ schema. This time, we choose an Amazon Bedrock LLM as a parsing mode to demonstrate how the application can use a multimodal FM for IDP. When using an Amazon Bedrock LLM, starting the IDP pipeline will now combine the attributes and their description into a custom prompt template, which is sent to the LLM with the documents converted to images. As a user, you can specify the LLM powering the extraction and its inference parameters, such as temperature.

![Complex Calculations](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-8-1.png)

The output, including the full results, is shown in the following screenshot.

![Upload Emails](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-9-1.png)
![Describe Attributes Emails](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-10-1.png)

### Example 2: Processing customer emails

In this scenario, we want to extract multiple features from a list of emails with customer complaints due to delays in product shipments using Amazon Bedrock Data Automation. For each email, we want to find the following:
* Customer name
* Shipment ID
* Email language
* Email sentiment
* Shipment delay (in days)
* Summary of issue
* Suggested response

Complete the following steps:
1.  Upload input emails as .txt files. You can download sample emails from GitHub.
![Extract Attributes Button](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-11.png)
2.  On the **Describe Attributes** tab, list names and descriptions of the attributes.
![Email Results](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-12.png)

You can add few-shot examples for some fields (such as delay) to explain to the LLM how these fields values should be extracted. You can do this by adding an example input and the expected output for the attribute to the description.

3.  Choose **Extract attributes** to start the IDP pipeline.
![Download Results](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-13-1.png)

The provided attributes and their descriptions will be integrated into a custom blueprint with the inferred attributes list, which is then used to invoke a data automation job on the uploaded documents. When the IDP pipeline is complete, you will see the results.

The application allows downloading the extraction results as a CSV or a JSON file. This makes it straightforward to use the results for downstream tasks, such as aggregating customer sentiment scores.

## Pricing

In this section, we calculate cost estimates for performing IDP on AWS with our solution.

Amazon Bedrock Data Automation provides a transparent pricing schema depending on the input document size (number of pages, images, or minutes). When using Amazon Bedrock FMs, pricing depends on the number of input and output tokens used as part of the information extraction call. Finally, when using Amazon Textract, OCR is performed and priced separately based on the number of pages in the documents.

Using the preceding scenarios as examples, we can approximate the costs depending on the selected parsing mode. In the following table, we show costs using two datasets: 100 20-page financial documents, and 100 1-page customer emails. We ignore costs of Amazon ECS and Lambda.

| AWS service | Use case 1 (100 20-page financial documents) | Use case 2 (100 1-page customer emails) |
| :--- | :--- | :--- |
| **IDP option 1: Amazon Bedrock Data Automation** | | |
| Amazon Bedrock Data Automation (custom output) | $20.00 | $1.00 |
| **IDP option 2: Amazon Bedrock FM** | | |
| Amazon Bedrock (FM invocation, Anthropic’s Claude 4 Sonnet) | $1.79 | $0.09 |
| **IDP option 3: Amazon Textract and Amazon Bedrock FM** | | |
| Amazon Textract (document analysis job with layout) | $30.00 | $1.50 |
| Amazon Bedrock (FM invocation, Anthropic’s Claude 3.7 Sonnet) | $1.25 | $0.06 |
| **Orchestration and storage (shared costs)** | | |
| Amazon S3 | $0.02 | $0.02 |
| AWS CloudFront | $0.09 | $0.09 |
| Amazon ECS | – | – |
| AWS Lambda | – | – |
| **Total cost: Amazon Bedrock Data Automation** | **$20.11** | **$1.11** |
| **Total cost: Amazon Bedrock FM** | **$1.90** | **$0.20** |
| **Total cost: Amazon Textract and Amazon Bedrock FM** | **$31.36** | **$1.67** |

The cost analysis suggests that using Amazon Bedrock FMs with a custom prompt template is a cost-effective method for IDP. However, this approach requires a bigger operational overhead, because the pipeline needs to be optimized depending on the LLM, and requires manual security and privacy management. Amazon Bedrock Data Automation offers a managed service that uses a choice of high-performing FMs through a single API.

## Clean up

To remove the deployed resources, complete the following steps:

1.  On the AWS CloudFormation console, delete the created stack. Alternatively, run the following command:
    ```bash
    cdk destroy --region <YOUR_DEPLOY_REGION>
    ```
2.  On the Amazon Cognito console, delete the user pool.

## Conclusion

Extracting information from unstructured documents at scale is a recurring business task. This post discussed an end-to-end IDP application that performs information extraction using multiple AWS services. The solution is powered by Amazon Bedrock Data Automation, which provides a fully managed service for generating insights from documents, images, audio, and video. Amazon Bedrock Data Automation handles the complexity of document processing and information extraction, optimizing for both performance and accuracy without requiring expertise in prompt engineering. For extended flexibility and customizability in specific scenarios, our solution also supports IDP using Amazon Bedrock custom LLM calls and Amazon Textract for OCR.

The solution supports multiple document types, including text, images, PDF, and Microsoft Office documents. At the time of writing, accurate understanding of information in documents rich with images, tables, and other visual elements is only available for PDF and images. We recommend converting complex Office documents to PDFs or images for best performance. Another solution limitation is the document size. As of June 2025, Amazon Bedrock Data Automation supports documents up to 20 pages for custom attributes extraction. When using custom Amazon Bedrock LLMs for IDP, the 300,000-token context window of Amazon Nova LLMs allows processing documents with up to roughly 225,000 words. To extract information from larger documents, you would currently need to split the file into multiple documents.

In the next versions of the IDP solution, we plan to keep adding support for state-of-the-art language models available through Amazon Bedrock and iterate on prompt engineering to further improve the extraction accuracy. We also plan to implement techniques for extending the size of supported documents and providing users with a precise indication of where exactly in the document the extracted information is coming from.

To get started with IDP with the described solution, refer to the GitHub repository. To learn more about Amazon Bedrock, refer to the documentation.

## About the authors

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/12/kozodoi.jpg" width="120" alt="Nikita Kozodoi">

**Nikita Kozodoi, PhD**, is a Senior Applied Scientist at the AWS Generative AI Innovation Center, where he works on the frontier of AI research and business. With rich experience in Generative AI and diverse areas of ML, Nikita is enthusiastic about using AI to solve challenging real-world business problems across industries.

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/12/zafolabi.jpg" width="120" alt="Zainab Afolabi">

**Zainab Afolabi** is a Senior Data Scientist at the Generative AI Innovation Centre in London, where she leverages her extensive expertise to develop transformative AI solutions across diverse industries. She has over eight years of specialised experience in artificial intelligence and machine learning, as well as a passion for translating complex technical concepts into practical business applications.

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/12/aitaleb.png" width="120" alt="Aiham Taleb">

**Aiham Taleb, PhD**, is a Senior Applied Scientist at the Generative AI Innovation Center, working directly with AWS enterprise customers to leverage Gen AI across several high-impact use cases. Aiham has a PhD in unsupervised representation learning, and has industry experience that spans across various machine learning applications, including computer vision, natural language processing, and medical imaging.

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/12/zinovyee.png" width="120" alt="Liza (Elizaveta) Zinovyeva">

**Liza (Elizaveta) Zinovyeva** is an Applied Scientist at AWS Generative AI Innovation Center and is based in Berlin. She helps customers across different industries to integrate Generative AI into their existing applications and workflows. She is passionate about AI/ML, finance and software security topics. In her spare time, she enjoys spending time with her family, sports, learning new technologies, and table quizzes.

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/24/nunoca_100x125.jpg" width="120" alt="Nuno Castro">

**Nuno Castro** is a Sr. Applied Science Manager at AWS Generative AI Innovation Center. He leads Generative AI customer engagements, helping hundreds of AWS customers find the most impactful use case from ideation, prototype through to production. He has 19 years experience in AI in industries such as finance, manufacturing, and travel, leading AI/ML teams for 12 years.

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/12/ozioma_resized.jpg" width="120" alt="Ozioma Uzoegwu">

**Ozioma Uzoegwu** is a Principal Solutions Architect at Amazon Web Services. In his role, he helps financial services customers across EMEA to transform and modernize on the AWS Cloud, providing architectural guidance and industry best practices. Ozioma has many years of experience with web development, architecture, cloud and IT management. Prior to joining AWS, Ozioma worked with an AWS Advanced Consulting Partner as the Lead Architect for the AWS Practice. He is passionate about using latest technologies to build a modern financial services IT estate across banking, payment, insurance and capital markets.

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/12/eren_resized.png" width="120" alt="Eren Tuncer">

**Eren Tuncer** is a Solutions Architect at Amazon Web Services focused on Serverless and building Generative AI applications. With more than fifteen years experience in software development and architecture, he helps customers across various industries achieve their business goals using cloud technologies with best practices. As a builder, he’s passionate about creating solutions with state-of-the-art technologies, sharing knowledge, and helping organizations navigate cloud adoption.

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/12/francesco_resized.jpg" width="120" alt="Francesco Cerizzi">

**Francesco Cerizzi** is a Solutions Architect at Amazon Web Services exploring tech frontiers while spreading generative AI knowledge and building applications. With a background as a full stack developer, he helps customers across different industries in their journey to the cloud, sharing insights on AI’s transformative potential along the way. He’s passionate about Serverless, event-driven architectures, and microservices in general. When not diving into technology, he’s a huge F1 fan and loves Tennis.
