## Purpose
The following [`docker_compose.yml`](https://github.com/omarsar/elastic_docker/blob/master/docker-compose.yml) allows you to easily replicate the environment used in the new ENG I and ENG II course on your local machine. I like Docker because it containerises your environment without having to worry that it will affect your host machine. This setup allowed me to work directly on the labs without needing to go through Strigo. You can achieve similar setup if you use the official instructions on how to setup your cluster locally. However, the advantage of containerising your cluster is that you achieve any configuration from one file (`docker_compose.yml`) with one command. In addition, you can run any version of the Elastic stack, even multiples ones separate from each other. Containerisation basically simplifies cluster management and it's a great way to experiment with your clusters. The other beauty is that you can share your containers with others easily and put your cluster in production very easily if so you wish. I digress!

## Setup Lab Instructions Locally

This setup is only useful if you are also willing to work with the lab instructions directly from the GitHub branches, which is sometimes the best approach as you are getting the latest lab instructions. This step is actually easy. You download or clone the instructions, type `make` in the instructions in the module's directory and then type `open labs.html`. This opens your browser and renders the labs for the particular module locally on your machine. Once you have your instructions ready, you now need access to your cluster. Below, I will discuss how to setup your cluster. 

## Setup Docker
You are now ready to leverage the power/flexibility of the docker containers. 

First, you need to have Docker installed on your machine. If you are not a docker expert like me, just download the [GUI version](https://hub.docker.com/editions/community/docker-ce-desktop-mac). This [short guide](https://docs.docker.com/docker-for-mac/) should get things running for you. You don't need to understand the whole dynamics of docker, at least not for the purpose discussed here. 

Once you have docker installed, you can check you docker version using: `docker version`.

In terms of the resources you need to assign to docker depends on what kind of cluster/s you want to setup. You are not building a monster here, so just ensure you use your resources wisely. I am assuming you want to run a few nodes with a decent heap sizes. For instance, I use the current settings for the `docker_compose.yml` file above. To be honest, this is something that you have to experiment with until you get the right setup. Look at the steps below to see how you might go about setting up docker. As I said, this is not the perfect setup, it just worked for what I wanted. 

![image](https://user-images.githubusercontent.com/7049564/60657446-ee9ef080-9e40-11e9-89b3-c285121c2f7e.png)

![image](https://user-images.githubusercontent.com/7049564/60657640-2f970500-9e41-11e9-8361-cd277bbea4f1.png)

## Setup Cluster

Once you having assigned a bit of resource to the Docker engine, you are now ready to start setting up your cluster using docker. 

The current setup uses the 7.1.1 versions of `Elasticsearch`, `Kibana`, `Logstash`, and `Filebeat`. You can find and download your desired versions here: https://www.docker.elastic.co/. Remember, you have do download them first and the command should look something like this: `docker pull docker.elastic.co/elasticsearch/elasticsearch:7.1.1`. Do this for all the solution of the Elastic stack you want in your setup. In our case, we only need Elasticsearch, Kibana, Logstash, and Filebeat. 

Great! You are half way there. Now comes the fun part. It's time to configure the `docker_compose.yml` file. Luckily, I designed a simple file for you test. This one sets up a cluster with three Elasticsearch nodes, Kibana, Logstash, and Filebeat. Take a few minutes to look at the YAML file. It actually consist of configurations, similar to what you would put in the Elasticsearch YAML file. Actually, that is the purpose of it. The cool thing is that you are doing this configuration in one file. The syntax may differ but you can figure this out pretty easily. And when you are not sure, go to the Elasticsearch guide and search for docker setup guides -- they are all there along with all the configuration you can put. I digress!

In summary, you can also easily add your configurations and whatever amount of nodes you like directly in that YAML file. Now, it's time to run the compose file. Once I am in the right directory where the docker compose file is stored, I type `docker-compose up` and the magic happens. BUT WAIT!!! Everything should work fine but we forgot to talk about one very important thing, which is where is the data coming from. In particular because we setup a Logstash and Filebeat instance. Let's do that before we run `docker-compose up`.

## Ingesting Data

For the setup to work and for your indices (e.g., blogs and logs) to be successfully ingested automatically you need to copy the datasets over to the proper containers or you can have a bridge setup (where your configs/datasets map to the configs in your containers). I prefer to work with containers directly just because it simplifies everything. 

All I was saying above is that you need to have the dataset files available in the containers so that the compose file can do its magic. In our labs for ENG 1, we are mostly using a `blogs` dataset and a `logs` dataset. Let's copy over those files to the right containers. Notice I am using containers instead of directory but containers actually contain directories themselves. Let's learn a bit about our containers. 

Type `docker ps` in your terminal and you should get something like this:

![image](https://user-images.githubusercontent.com/7049564/60658723-535b4a80-9e43-11e9-8a9c-3d254aab2ed7.png)

It lists all your containers, in our case one for each solution we are using (1 Kibana, 3 Elasticsearch nodes, 1 Filebeat, and 1 Logstash). The information that's important there is the `CONTAINER ID`. You need this information to send over data to the right container. But before that, let's look at what is inside one of the containers.

We can inspect a container using the following command: docker exec -it <CONTAINER ID> bash

Note that you can only check a container once it's running or the docker compose has been executed. This means that in order to browse the container directories, you need to have all the containers running successfully and that your container appears in the list of containers that appear after running `docker ps`.

And when you check the files inside the container using `ls`, what do you see? That's right! Basically, the folders necessary for Elasticsearch to run. This is similar to what you would get if you working directly with the zip file of Elasticsearch which you downloaded straight from the website. That's cool right! It is because there is nothing strange about it. 

![image](https://user-images.githubusercontent.com/7049564/60659051-0166f480-9e44-11e9-8e90-3023194c6c9e.png)

Great, now you can continue exploring the files in the container like you would explore the files in your host machine. In fact, if you want to add extra configurations directly in the config files of each of the solutions, you can do that with the available `vim` editor available inside the container. We don't need to this in our setup because we are configuring directly from the compose file. Let's skip that part now but know that you have that option too. The most important thing here is that you will need to figure out where you want the compose file to look at. If you look at the compose file, you will see that you are specifying a bunch of directories and these directories refer directly to the directories in the corresponding containers. If you want to tell the compose file that you want to locate a dataset or file in one of the containers, you need to figure the exact directory. 

For instance, let's look at the Filebeat configuration inside the compose file:

```yml
filebeat:
    image: docker.elastic.co/beats/filebeat:7.1.1
    volumes:
      - fbdata:/usr/share/filebeat/data/datasets
    command: filebeat -c /usr/share/filebeat/data/datasets/filebeat2.yml
    user: root
    environment:
      - "XPACK_MONITORING_ENABLED=true"
      - "ELASTIC_HOST=elasticsearch:9200"
      - "XPACK_SECURITY_ENABLED=false"
      - "XPACK_REPORTING_ENABLED=false"
      - xpack.monitoring.elasticsearch.hosts=http://elasticsearch:9200
    depends_on: ['elasticsearch']
    networks:
      - esnet
```

The `command` section executes Filebeat and tells it where to look for the YAML config file and data files. I have already copied over a few files to the Filebeat container which were stored on my local computer. I copied the files (both config and logs) from my host machine over to the Filebeat container using the following command:

`docker cp . <CONTAINER_ID>:/usr/share/filebeat/data/datasets/`

If you look at the directory in the container, you will see the files once you successfully copied them over. 

![image](https://user-images.githubusercontent.com/7049564/60659765-b77f0e00-9e45-11e9-8776-5f74556f9de9.png)

Again, note that your containers need to be live in order to inspect them. 

You can obtain the lab's datasets and config files in the corresponding Google Drive folder and GitHub repos. This step is trivial and I believe once you understand how to copy files over to container, you will get your setup up and running in no time. You can also change the config files before you copy them over to the containers, which is what I prefer to do. But you can also do this inside the containers if you wish. 

Great! Once you have the proper datasets in the proper folders in your containers and you compose file is configured to look in the proper directories, your indices should now be live in Elasticsearch and you should be able to perform queries on those indices. Keep in mind that there could be errors in the config files and this will be captures and reported in the logs provided after executing `docker-compose up`. 

These are config files I used for both Filebeat and Logstash in my case. I had to modify the config file a bit from the original config files provided in the labs: 

Filebeat logs: https://gist.github.com/omarsar/cc2ec1ef965688546c71c9b0bff299de

Logstash blogs (csv file used): https://gist.github.com/omarsar/fa1e5b3daa7322d2191c70e63eff18b2

## Testing Your Setup

If all works fine, you should be able to check if elasticsearch is running by going into the browser and typing: `http://localhost:9200`. You can also check if Kibana is running by typing: `http://localhost:5601`. All nodes should be communicating with each other without needing to configure anything extra.

If anyone is interested in this setup using docker, I will put some effort to document all the process. Let me know through slack (@elvis) if you would like me to assist you with this. Enjoy!
