#**Behavioral Cloning** 

**Behavrioal Cloning Project**

The goals / steps of this project are the following:
* Use the simulator to collect data of good driving behavior
* Build, a convolution neural network in Keras that predicts steering angles from images
* Train and validate the model with a training and validation set
* Test that the model successfully drives around track one without leaving the road
* Summarize the results with a written report


[//]: # (Image References)

[image1]: ./examples/placeholder.png "Model Visualization"
[image2]: ./examples/placeholder.png "Grayscaling"
[image3]: ./examples/placeholder_small.png "Recovery Image"
[image4]: ./examples/placeholder_small.png "Recovery Image"
[image5]: ./examples/placeholder_small.png "Recovery Image"
[image6]: ./examples/placeholder_small.png "Normal Image"
[image7]: ./examples/placeholder_small.png "Flipped Image"

###Files Submitted & Code Quality

####1. Submission includes all required files and can be used to run the simulator in autonomous mode

My project includes the following files:
* model.py containing the script to create and train the model
* drive.py for driving the car in autonomous mode
* model.h5 containing a trained convolution neural network 
* writeup_report.md summarizing the results

####2. Submission includes functional code
Using the Udacity provided simulator and my drive.py file, model.h5 file the car can be driven autonomously around the track by executing 
```sh
python drive.py model.h5
```

####3. Submission code is usable and readable

The model.py file contains the code for training and saving the convolution neural network. The file shows the pipeline I used for training and validating the model, and it contains comments to explain how the code works. I used python generator for memory-efficient algorithm to load the data in batches to the model.

###Model Architecture and Training Strategy

####1. An appropriate model arcthiecture has been employed

I used NVIDIA's model for this project which consists of 5 CNN with 2 3x3 and 3 5x5 filter sizes layers and depths between 24 and 64 and 3 fully connected layers. I added a few more dropout layers to the network for an efficient model to my data.
The model includes RELU layers to introduce nonlinearity, and the data is normalized in the model using a Keras lambda layer.

####2. Attempts to reduce overfitting in the model

The model contains dropout layers in order to reduce overfitting. The collected dataset is also balanced to obtain all steering angles in right proportion so that the model won't be biased to certain steering angle.

The model was trained and validated on different data sets to ensure that the model was not overfitting . The model was tested by running it through the simulator and ensuring that the vehicle could stay on the track.

####3. Model parameter tuning

The model used an adam optimizer with learning rate of 0.001

####4. Appropriate training data

Training data was chosen to keep the vehicle driving on the road. I used a combination of center lane driving, recovering from the left and right sides of the road. I augmented the training data with the images which needs steering angle > 0.5 just to balance the data with other steering angles. Steering values for left image are obtained by adding 0.2 to the steering value from the center camera view and subtracted by 0.2 for a right camera view of an image.

For details about how I created the training data, see the next section. 

###Model Architecture and Training Strategy

####1. Solution Design Approach

I preferred NVIDIA's architecture since it was used to train Self Driving Cars and the model was able to detect roads based on steering angle without ever explicitly training it to detect raods.

I have used the data set given by Udacity. My first step was to balance the data. I reduced the data for straight images and the steering angle of 0 by a fraction of 0.9, as the data has many of them. 

I used python generator to reduce the memory usage. I used ImagedataGenerator from keras. But it does not give me good results with augmenting the data. The flipping of the images is done at random. I needed at a particular steering angle. I decided to develop my own generator.

I flipped some of the images occasionally to remove bias for more number of left turns and the car went off the road and failed to recover after the bridge where there is no curb. I augmented the data set with my images at the points where it needs more steering angles as the histogram shows the data has less images which need more steering angles. This time the car tends to wobble more at the straight road. So I increased the images with straight roads by reducing the fraction to 0.6. The car started to move smoothly but at the next turn the car failed to recover. I added more dropout layers to the network to control the wobble of the car.

In order to gauge how well the model was working, I split my image and steering angle data into a training and validation set. I found that my first model had a low mean squared error on the training set but a high mean squared error on the validation set. This implied that the model was overfitting. 

To combat the overfitting, I modified the model so that ...

Then I ... 

The final step was to run the simulator to see how well the car was driving around track one. There were a few spots where the vehicle fell off the track... to improve the driving behavior in these cases, I ....

At the end of the process, the vehicle is able to drive autonomously around the track without leaving the road.

####2. Final Model Architecture

The final model architecture (model.py lines 18-24) consisted of a convolution neural network with the following layers and layer sizes ...

Here is a visualization of the architecture (note: visualizing the architecture is optional according to the project rubric)

![alt text][image1]

####3. Creation of the Training Set & Training Process

To capture good driving behavior, I first recorded two laps on track one using center lane driving. Here is an example image of center lane driving:

![alt text][image2]

I then recorded the vehicle recovering from the left side and right sides of the road back to center so that the vehicle would learn to .... These images show what a recovery looks like starting from ... :

![alt text][image3]
![alt text][image4]
![alt text][image5]

Then I repeated this process on track two in order to get more data points.

To augment the data sat, I also flipped images and angles thinking that this would ... For example, here is an image that has then been flipped:

![alt text][image6]
![alt text][image7]

Etc ....

After the collection process, I had X number of data points. I then preprocessed this data by ...


I finally randomly shuffled the data set and put Y% of the data into a validation set. 

I used this training data for training the model. The validation set helped determine if the model was over or under fitting. The ideal number of epochs was Z as evidenced by ... I used an adam optimizer so that manually training the learning rate wasn't necessary.
