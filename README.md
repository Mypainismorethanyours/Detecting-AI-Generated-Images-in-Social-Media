
## A Machine Learning System for Detecting Misinformation in Media Images

<!-- 
Discuss: Value proposition: Your will propose a machine learning system that can be 
used in an existing business or service. (You should not propose a system in which 
a new business or service would be developed around the machine learning system.) 
Describe the value proposition for the machine learning system. What’s the (non-ML) 
status quo used in the business or service? What business metric are you going to be 
judged on? (Note that the “service” does not have to be for general users; you can 
propose a system for a science problem, for example.)
-->
Our project aims to develop a machine learning system that can determine whether an image contains misleading or false information. Currently, many unreliable media outlets frequently publish deceptive or false images online to mislead the public. However, some platforms still lack mechanisms to alert users about such images. Our system is designed to provide these platforms with a tool that can notify users when content may contain misinformation, urging them to exercise caution when evaluating the material. This will help reduce user reports, improve the efficiency of manual reviews, and create a better environment for these platforms.




### System diagram

<!-- Overall digram of system. Doesn't need polish, does need to show all the pieces. 
Must include: all the hardware, all the containers/software platforms, all the models, 
all the data. -->
![](https://github.com/Mypainismorethanyours/Detecting-AI-Generated-Images-in-Social-Media/blob/main/System%20Diagram.png)


### Summary of outside materials

<!-- In a table, a row for each dataset, foundation model. 
Name of data/model, conditions under which it was created (ideally with links/references), 
conditions under which it may be used. -->

|              | How it was created | Conditions of use         |
|--------------|--------------------|---------------------------|
| AMMeBa       | Public Dataset     | [Dataset Link](https://www.kaggle.com/datasets/googleai/in-the-wild-misinformation-media)       |
| Prompts: What types of incorrect information might the image contain and the reasons for them      | Generated by Mini-InternVL| Finetune the MLLMs        |
| Mini-InternVL-Chat-2B-V1-5       | Pretrained  Model       | [Model card](https://huggingface.co/OpenGVLab/Mini-InternVL-Chat-2B-V1-5)|
| Qwen2-VL-7B-Instruct      | Pretrained  Model       |  [Model card](https://huggingface.co/Qwen/Qwen2-VL-7B-Instruct)                         |


### Summary of infrastructure requirements

<!-- Itemize all your anticipated requirements: What (`m1.medium` VM, `gpu_mi100`), 
how much/when, justification. Include compute, floating IPs, persistent storage. 
The table below shows an example, it is not a recommendation. -->

| Requirement         | How many/when                                     | Justification                          |
|---------------------|---------------------------------------------------|----------------------------------------|
| `m1.xxlarge` VMs     | 3 for entire project duration                     |For training, serving and tracking      |
| `A100 or gpu_mi100` | 4 hour block twice a week                         |Used for MLLM fine-tune                 |
| Floating IPs        | 1 for entire project duration, 1 for sporadic use |One for Project and one for partial test|

### Detailed design plan

<!-- In each section, you should describe (1) your strategy, (2) the relevant parts of the 
diagram, (3) justification for your strategy, (4) relate back to lecture material, 
(5) include specific numbers. -->

#### Model training and training platforms

Use teacher MLLMs to generate prompts first, and then use student MLLMs to take both the image and the prompts as inputs to obtain the result.

[Link](https://github.com/Mypainismorethanyours/A-Machine-Learning-System-for-Detecting-Misinformation-in-Media-Images/tree/main/Finetune)

We fine-tune the Qwen2.5-VL-3B-Instruct pre-trained model. The input consists of a fixed question, "Is this image manipulated or synthesized?" along with the user-uploaded image, and the output is a text segment that includes the judgment result and possible reasons.

We support DeepSpeed for distributed training, MLFlow to track experiments, and Ray cluster to submit training jobs.

[Instruction to run](https://github.com/Mypainismorethanyours/A-Machine-Learning-System-for-Detecting-Misinformation-in-Media-Images/blob/main/Finetune/docker/README.md)

[One of Train Code](https://github.com/Mypainismorethanyours/A-Machine-Learning-System-for-Detecting-Misinformation-in-Media-Images/blob/main/Finetune/train_single_GPU_LoRA_Sample.py)

#### Model serving and monitoring platforms

<!-- Make sure to clarify how you will satisfy the Unit 6 and Unit 7 requirements, 
and which optional "difficulty" points you are attempting. -->
Strategy:

Once the model is trained, wrap the trained model into an API endpoint using Flask or FastAPI. The API will receive an image, process it through the model, and return whether the image is AI-generated or not.
Use a cloud infrastructure with auto-scaling features to manage varying traffic and ensure low latency during real-time API requests.
Relevant Diagram Parts:

The REST API endpoint that serves model predictions.
Integration with the web UI where social media moderators will interact with the system.
Justification:

Using containers (Docker) and microservices architecture allows independent scaling of the model serving API and UI components.
The cloud infrastructure can scale according to demand and handle spikes in requests (such as when moderators are processing large batches of images).
Related to Course Material:

Using containers and scaling services aligns with Unit 6 (Model Serving) and Unit 3 (DevOps).
The feedback loop for model evaluation and continuous improvement satisfies the monitoring aspect of the system (Unit 7).

To handle model serving we divided this service into two docker containers: 1 docker container for the flask app user interface and one container for the fast api endpoint that is called by the front end. These two containers are brought up with a docker-compse.yaml file that orchestrates this. The instructions for how to do this is provided below:


### Instructions on how to bring up the Flask app interface + FASTAPI docker containers 

on the node instance run : this uses a docker compose file to bring up the two containers for the

docker compose -f ~/A-Machine-Learning-System-for-Detecting-Misinformation-in-Media-Images/docker/docker-compose-fastapi.yaml up -d


sanity check to see all containers are running properly: 

docker compose ps

check logs for Jupyter container:
- This will give the token url  (http://127.0.0.1:8888/lab?token=XXXXXXXXXXXXX)
- To  access the Jupiter container notebook environment in the browser replace 127.0.0.1 with the floating IP address of the reserved instance


The system optimizations that we tested on and then applied based on the results to a  trained model targeted reduction of the inference time of our model. However, the overall includes other delays notably system related delays.

Our system level ptimizations were implemneted such that theyb are not harcoded and flexible in that whenever we have a newly trained model we can test different optimizations for that new model, then based on the results we apply the optimizations that result in the best performance( lowest latency if thats a requiremnt that we set for our model). After we have a newly trained the optimization strategies pipeline is ran and the recommended optimizations based on the results of the tests are stored in a json file. When we do model inferencing the current model is loaded as well as the recommended optimization strategies are loaded from the json file and applied to the model for inferencing. 





#### Data pipeline

<!-- Make sure to clarify how you will satisfy the Unit 8 requirements,  and which 
optional "difficulty" points you are attempting. -->
Strategy:

Collect images from social media using APIs like Twitter API, Instagram Graph API, or through web scraping.
Use an ETL (Extract, Transform, Load) pipeline to preprocess the images for model training.
Store preprocessed images and model weights in persistent cloud storage.
Continuously gather new images for model re-training to ensure the system stays up-to-date with new AI generation techniques.
Relevant Diagram Parts:

Data collection API endpoints.
Data storage and management systems.
ETL pipeline for preprocessing images.
Justification:

Continuous collection and preprocessing of data ensures that the model is regularly updated with fresh, relevant examples of social media content.
Related to Course Material:

Data pipeline management meets Unit 8 (Data Pipeline) requirements, particularly around persistent storage and managing real-time data for inference.

#### Continuous X

<!-- Make sure to clarify how you will satisfy the Unit 3 requirements,  and which 
optional "difficulty" points you are attempting. -->
Strategy:

Implement a CI/CD pipeline that automatically retrains the model with new data, evaluates it, and deploys the updated version to production.
Monitor performance in real-time and use this feedback to initiate model re-training if performance drops.
Relevant Diagram Parts:

CI/CD pipeline for training, evaluation, and deployment.
Model registry and version control for tracking model iterations.
Justification:

Automating the retraining process ensures that the model adapts to new types of AI-generated images over time, maintaining high accuracy in real-world applications.
Related to Course Material:

Automated pipelines align with Unit 3 (DevOps) for CI/CD, ensuring that the model is continuously updated with minimal manual intervention.


