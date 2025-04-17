
## Detecting AI-Generated Images in Social Media

<!-- 
Discuss: Value proposition: Your will propose a machine learning system that can be 
used in an existing business or service. (You should not propose a system in which 
a new business or service would be developed around the machine learning system.) 
Describe the value proposition for the machine learning system. What’s the (non-ML) 
status quo used in the business or service? What business metric are you going to be 
judged on? (Note that the “service” does not have to be for general users; you can 
propose a system for a science problem, for example.)
-->
Our project aims to build a machine learning system that can determine whether an image is AI-generated. On platforms like Twitter, there are many exaggerated or misleading images created by AI, which can easily influence children and elderly people with false information. However, these platforms currently do not provide any alerts or warnings about such images. Our system is designed to offer these social media platforms a tool that can inform their users when content is AI-generated, which may contain false information and requires cautious evaluation. This would help reduce user reports and improve the efficiency of manual reviews, providing a better environment for social media platforms.

### Contributors

<!-- Table of contributors and their roles. 
First row: define responsibilities that are shared by the team. 
Then, each row after that is: name of contributor, their role, and in the third column, 
you will link to their contributions. If your project involves multiple repos, you will 
link to their contributions in all repos here. -->

| Name                            | Responsible for                          | Link to their commits in this repo |
|---------------------------------|------------------------------------------|------------------------------------|
| All team members                | Continuous X pipeline                    |                                    |
| Chengqian Luo                   | Data collection and Data pipeline        |  https://github.com/Mypainismorethanyours/Detecting-AI-Generated-Images-in-Social-Media/commits/main?author=Ayachisan                                  |
| Shengyang Li                    | Model training and Inference                           | https://github.com/Mypainismorethanyours/Detecting-AI-Generated-Images-in-Social-Media/commits/main?author=Mypainismorethanyours                                   |
| Julio Rodriguez                 | Model serving and Monitoring	           | https://github.com/Mypainismorethanyours/Detecting-AI-Generated-Images-in-Social-Media/commits/main?author=JulioRodriguez289                                   |  



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
| TWIGMA       | Public Dataset     | [Dataset Link](https://zenodo.org/records/11374087)       |
| Twitter100k       | Public Dataset     | [Dataset Link](https://github.com/huyt16/Twitter100k)        |
| Prompts      | Generated by GPT-4o| Finetune the MLLMs        |
| Mini-InternVL-Chat-2B-V1-5       | Pretrained  Model       | [Model card](https://huggingface.co/OpenGVLab/Mini-InternVL-Chat-2B-V1-5)|
| Qwen2-VL-7B-Instruct      | Pretrained  Model       |  [Model card](https://huggingface.co/Qwen/Qwen2-VL-7B-Instruct)                         |


### Summary of infrastructure requirements

<!-- Itemize all your anticipated requirements: What (`m1.medium` VM, `gpu_mi100`), 
how much/when, justification. Include compute, floating IPs, persistent storage. 
The table below shows an example, it is not a recommendation. -->

| Requirement         | How many/when                                     | Justification                          |
|---------------------|---------------------------------------------------|----------------------------------------|
| `m1.medium` VMs     | 3 for entire project duration                     |For training, serving and tracking      |
| `A100 or gpu_mi100` | 4 hour block twice a week                         |Used for MLLM fine-tune                 |
| Floating IPs        | 1 for entire project duration, 1 for sporadic use |One for Project and one for partial test|

### Detailed design plan

<!-- In each section, you should describe (1) your strategy, (2) the relevant parts of the 
diagram, (3) justification for your strategy, (4) relate back to lecture material, 
(5) include specific numbers. -->

#### Model training and training platforms

<!-- Make sure to clarify how you will satisfy the Unit 4 and Unit 5 requirements, 
and which optional "difficulty" points you are attempting. -->
Strategy:

Use a pretrained multimodal model to detect whether an image is AI-generated or authentic. The model will be fine-tuned on a dataset consisting of both real and AI-generated images, labeled from the data sources mentioned.
Training Platform: Use a cloud-based GPU instance for training and fine-tuning, utilizing Chameleon infrastructure for scalability.
Relevant Diagram Parts:

Data collection from social media feeds (APIs).
Image preprocessing pipeline to normalize and resize images.
Training and inference using a deep neural network for image classification (e.g., convolutional neural networks or multimodal models like CLIP).
Justification:

Fine-tuning pretrained models ensures the system benefits from state-of-the-art AI capabilities while focusing on the specific task of detecting AI-generated images.
The use of cloud-based GPU instances ensures scalability and performance for model training.
Related to Course Material:

Use of pretrained models (like CLIP) and transfer learning aligns with Unit 4 (Model Training at Scale) and Unit 5 (Model Training Infrastructure and Platform).
Handling multimodal data (image and text) fits into model training using different modalities.
Difficulty points:
Use LLMs to generate prompts first, and then use MLLMs to take both the image and the prompts as inputs to obtain the result.
Use the strategies to fit training job on a low-end GPU, use distributed training and compare total model training time with one GPU vs. multiple GPUs of the same type.

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


