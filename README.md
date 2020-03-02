# DataMiningProject
Project for data mining subject

## Choosen tool and system

[Turi Create](https://github.com/apple/turicreate) simplifies the development of custom machine learning models. You don’t have to be a machine learning expert to add recommendations, object detection, image classification, image similarity or activity classification to your app.

* Easy-to-use: Focus on tasks instead of algorithms
* Visual: Built-in, streaming visualizations to explore your data
* Flexible: Supports text, images, audio, video and sensor data
* Fast and Scalable: Work with large datasets on a single machine
* Ready To Deploy: Export models to Core ML for use in iOS, macOS, watchOS, and tvOS apps

**Recommendation System** based on: https://apple.github.io/turicreate/docs/userguide/recommender/.

## Installation

      Below are the linux dependencies needed, turicreate uses python2, there is no announcement for python3 support till now.
      
      apt-get update -y &&  apt-get install -y python-pip python-dev libatlas-base-dev
      
      # For python we need turicreate library to be installed locally
      pip install turicreate

## Supported Platforms

### Turi Create supports:

* macOS 10.12+
* Linux (with glibc 2.12+)
* Windows 10 (via WSL)

## System Requirements

### Turi Create requires:

* Python 2.7, 3.5, 3.6
* x86_64 architecture
* At least 4 GB of RAM

We will first train the model locally to make validation based on training data.

      # SFrame is a data-structures built for out-of-core data analysis and supports turicreate by default.
      
      import turicreate as tc
      
      # Product visit's data against each user_id.
      
      parse_data = tc.SFrame.read_csv('visit_count.csv')
      sf.head()

      # We are passing some more information about our user data
      
      user_data = tc.SFrame.read_csv(‘user_data.csv’)
      print(user_data.head())

      # We are passing some more information about deal 
      
      datadeal_data = tc.SFrame.read_csv('deal_data.csv')
      print(deal_data.head())

      # Based on the parse data, we then divide the data as training and validation using turicreate function
      "recommender.util.random_split_by_user"
      
      training_data, validation_data = tc.recommender.util.random_split_by_user(parse_data, 'userId', 'dealId')
      
      # Trained the training data by turicreate inbuilt model
      "factorization_recommender"
      
      model = tc.factorization_recommender.create(training_data, user_id="userId", user_data= user_data,item_id="dealId",item_data=deal_data, target="visits")

The output will be like after the training completed.

      Optimization Complete: Maximum number of passes through the data reached.
      Computing final objective value and training RMSE.
      Final objective value: 4.94019
      Final training RMSE: 2.22264

Validation of model based on training data.

      # We then evaluate the model after training using the validation data that we separated out previously. results = model.evaluate(validation_data)Output: Precision and recall summary statistics by cutoff
      +--------+------------------+------------------+
      | cutoff |  mean_precision  |   mean_recall    |
      +--------+------------------+------------------+
      |   1    | 0.00568181818182 | 0.00568181818182 |
      |   2    | 0.00426136363636 | 0.00852272727273 |
      |   3    | 0.0104166666667  |     0.03125      |
      |   4    | 0.0127840909091  | 0.0511363636364  |
      |   5    | 0.0102272727273  | 0.0511363636364  |
      |   6    | 0.00852272727273 | 0.0511363636364  |
      |   7    | 0.00730519480519 | 0.0511363636364  |
      |   8    | 0.00639204545455 | 0.0511363636364  |
      |   9    | 0.00631313131313 | 0.0568181818182  |
      |   10   | 0.00568181818182 | 0.0568181818182  |
      +--------+------------------+------------------+
      [10 rows x 3 columns]

Save the model for production deployment.

      model.save("popular_deals")

Now we will use python famous micro web framework “Flask” for prediction in the production environment.

      #We create api endpoint as "http://<hostname>:5000/recomend/<user_id> which will return a json and  include product_id, its ranking and the user_id.
      from flask import Flask,jsonify,request
      import turicreate as tc app = Flask(__name__) 
      
      model = tc.load_model("popular_deals")  
      
      @app.route('/recomend/<int:num>', methods=['GET'])
      
      def prediction(num):
         if request.method == 'GET':                 
            return jsonify(list(model.recommend([num])))  
            
      if __name__ == '__main__':
         app.run(debug=True,host='0.0.0.0')

**Docker for deployment. (read carefully)**

      #Make sure the save model folder "popular_deals", the above script "app.py" and "requirements.txt" should exist in same directory.
      
      FROM ubuntu:16.04 RUN apt-get update -y && \    apt-get install -y python-pip python-dev
      
      COPY ./requirements.txt /app/requirements.txt 
      
      WORKDIR /app RUN pip install -r requirements.txt COPY . /app 
      
      ENTRYPOINT [ "python" ] CMD [ "app.py" ]

Execute the commands for docker build and deployment :

      " sudo docker build -t recomend:latest . " inside the folder 
      
and then deploy using the command :
      
      " sudo docker run -d -p 5000:5000 recomend ".
      
Here you are you successfully build a recommendation engine and is now ready for predictions.
