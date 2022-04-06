+++
title = "Face Mask Detection"
date = 2022-03-30
description = "This Python program detects whether the person has a mask on. It does this by first collecting face data and then using that data with the help of Scikit-learn library to predict whether the detected face has a mask."
in_search_index = true
[taxonomies]
tags = ["python", "CV", "ML"]
+++
## Intro
The COVID-19 virus is still spreading these days. In hopes to reduce the morbidity of individuals, governments require masks that cover the nose and mouth in closed public places (shops, buses, etc.). While it would seem easy to comply with this request, some individuals refuse to wear a mask, even risking getting fined. What is more, you can't supervise everyone. With the help of computers and security cameras, we can ensure that the person is wearing a mask. This project uses machine learning and computer vision to determine where a person is and whether they are wearing a mask.

We will do this in 3 steps:
- We will create an algorithm that can record and store face data.
- We will create a function that can learn to identify faces and masks.
- We will create a program that can use a live camera to detect if the person is wearing a mask.

## Dependencies
Since we're planning to use machine learning for this project, the best programming language to use would be Python since it has the most mature index of machine learning libraries.

### OpenCV

One of the libraries we'll use is ["OpenCV"](https://docs.opencv.org/4.x/d1) (_Open Computer Vision_). It will handle reading images (in our case with a camera) and transforming them into usable data.

This library is in a python package, so use pip to install it:
```
pip install opencv-python
```
### Scikit-learn
["Scikit-learn"]("TOADD") library is responsible for the machine learning part of our program. It will teach the computer to recognize human faces and whether they have a mask on.

This library is in a python package, so use pip to install it:
```
pip install sklearn
```

## Face Detection
The first task of our program is to collect face data, when the subject has a mask and when he is without it.
First, the computer needs to be able to see us.
```python
def collect_data():
	capture = cv2.VideoCapture(0)
	while True:
		flag, img = capture.read()
		if not flag: continue
			cv2.imshow('Result', img)
			if cv2.waitKey(2) == 27:
				break
	cv2.destroyAllWindows()
	capture.release()
```
We start by assigning the camera we will use.
Then we loop until the user presses `ESC`. After he does, we break out of the loop.
In the loop, we capture a frame using our chosen camera. Out of the capture function, we get two variables:
- `flag` - a bool that shows if we captured a frame
- `img` - the captured video frame.
If we succeed, then we show the captured image.

Now that we see ourselves let's use face cascades provided by the _"Scikit-learn"_ library to detect our face and draw a rectangle around it.
```python
SCALE_FACTOR: float = 1.1
MIN_NEIGHBORS: int = 4

def collect_data():
	capture = cv2.VideoCapture(0)

	while True:
		flag, img = capture.read()
		if not flag: continue

		face_cascades = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_default.xml')
		gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
		faces = face_cascades.detectMultiScale(gray, SCALE_FACTOR, MIN_NEIGHBORS)

		for x, y, w, h in faces:
			cv2.rectangle(img, (x, y), (x+w, y+h), (0, 255, 0), 4)

		cv2.imshow('Result', img)
		if cv2.waitKey(2) == 27:
			break
	cv2.destroyAllWindows()
	capture.release()
```
We assign the constants used in face detection as variables for later configuration.

To teach the program to detect masks, we need to store the face data. For testing, we're only saving the first 200 data entries each.

```python
SCALE_FACTOR: float = 1.1
MIN_NEIGHBORS: int = 4
SAMPLE_COUNT: int = 200

def collect_data(save_destination: str):
	capture = cv2.VideoCapture(0)
	data = []
	while True:
		flag, img = capture.read()
		if not flag: continue

		face_cascades = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_default.xml')
		gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
		faces = face_cascades.detectMultiScale(gray, SCALE_FACTOR, MIN_NEIGHBORS)

		if len(faces) > 2: continue

		for x, y, w, h in faces:
			cv2.rectangle(img, (x, y), (x+w, y+h), (0, 255, 0), 4)
			face = img[y:y+h, x:x+w, :]
			face = cv2.resize(face, (50, 50))
			print('Capturing face data: ', round((len(data) + 1) / SAMPLE_COUNT * 100, 2), "%")
			if len(data) < SAMPLE_COUNT:
				data.append(face)

		cv2.imshow('Result', img)
		if cv2.waitKey(2) == 27 or len(data) >= SAMPLE_COUNT:
			break

	np.save(save_destination, data)
	cv2.destroyAllWindows()
	capture.release()
```

## Machine Learning
Now that we have the needed data, we can start teaching our program how to differentiate whether or not a person is wearing a mask. We begin by loading the necessary data.
```python
WITH_MASK_PATH: str = "yes_mask.npy"
WITHOUT_MASK_PATH: str = "no_mask.npy"

def detect_masks():
	data_with_mask = np.load(WITH_MASK_PATH)
	data_without_mask = np.load(WITHOUT_MASK_PATH)
```
To improve performance, we reshape the data into three columns.
We also need to store the labels of the data.
```python
WITH_MASK_PATH: str = "yes_mask.npy"
WITHOUT_MASK_PATH: str = "no_mask.npy"

def detect_masks():
	data_with_mask = np.load(WITH_MASK_PATH)
	data_without_mask = np.load(WITHOUT_MASK_PATH)

	data_with_mask = data_with_mask.reshape(SAMPLE_COUNT, 50 * 50 * 3)
	data_without_mask = data_without_mask.reshape(SAMPLE_COUNT, 50 * 50 * 3)

	training_data = np.r_[data_with_mask, data_without_mask]

	data_labels = np.zeros(training_data.shape[0])
	data_labels[SAMPLE_COUNT: ] = 1.0
```
If the person doesn't have a mask - the label will be 0. Else will be 1.
Then using the prepared data, we call the machine learning function `train_test_split` passing in our data. This function is called twice because calling it once results in overfitting, accuracy becomes 1.0. Overfitting is harmful since it results in the program detecting only the data that we used at the start. Wanting to avoid overfitting, we shuffle the data by calling the function twice.
```python
TEST_SIZE: float = 0.25

def detect_masks():
	# ...
	x_train, _, _, _ = train_test_split(training_data, data_labels, test_size=TEST_SIZE)
	_, x_test, y_train, y_test = train_test_split(training_data, data_labels, test_size=TEST_SIZE)
	svc = SVC()
	svc.fit(x_train, y_train)
	y_pred = svc.predict(x_test)
	print("Tested accuracy => ", accuracy_score(y_test, y_pred))
```
## Face Mask Detection
After teaching the program to identify if a person is wearing a mask, we use a similar algorithm to the one we used in face detection, just with a few more lines of code.
```python
	# ...
	capture = cv2.VideoCapture(0)
	data = []
	while True:
		flag, img = capture.read()
		if not flag: continue

		face_cascades = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_default.xml')
		gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
		faces = face_cascades.detectMultiScale(gray, SCALE_FACTOR, MIN_NEIGHBORS)

		for x, y,w, h in faces:
			face = img[y:y+h, x:x+w, :]
			face = cv2.resize(face, (50, 50))
			face = face.reshape(1, -1)

			pred = svc.predict(face)[0]

			if int(pred) == 0:
				cv2.rectangle(img, (x, y), (x+w, y+h), (0, 255, 0), 4)
			if int(pred) == 1:

				cv2.rectangle(img, (x, y), (x+w, y+h), (0, 0, 255), 4)

		cv2.imshow('Result', img)
		if cv2.waitKey(2) == 27:
			break
	cv2.destroyAllWindows()
	capture.release()
```
If the program predicts that the face has a mask, it will draw a green square. Otherwise, a red square is drawn around the face.
