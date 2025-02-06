# Sign2Voice 🗣️
We want to develop an application that enables deaf people to communicate with hearing individuals who do not know sign language. The app should recognize signs, translate them into text, and then convert this text into spoken language.

## Features
- Recognition of sign language gestures from individual images (frames)
- Translation of gestures into glosses using a neural network
- Conversion of glosses into natural text
- Speech output of the generated text (Text-to-Speech)

## Architecture of Sign2Voice
Our system is structured in a modular pipeline, where each component plays a specific role in processing sign language gestures and converting them into speech. The architecture consists of four main stages:

### 1️⃣ Frame Processing (Input)
- The application takes image frames as input.
- These frames are either uploaded manually or extracted from a video.
- OpenCV is used to preprocess the images before sending them to the recognition model.

### 2️⃣ Sign Language Recognition (Gloss Prediction)
- We use **CorrNet**, a deep learning model, to recognize glosses (simplified representations of sign language).
- CorrNet processes the input frames and predicts a sequence of glosses, which represent the recognized signs.
- The output at this stage is a structured sequence of glosses that will later be converted into natural text.

### 3️⃣ Gloss-to-Text Translation
- The predicted glosses are translated into full sentences using our **Gloss2Text** model.
- Gloss2Text ensures that the generated sentences are grammatically correct and contextually meaningful.
- The final output of this step is a fully structured natural language sentence, making it easier for non-signing people to understand

### 4️⃣ Text-to-Speech Conversion (Output)
- The generated sentence is sent to the OpenAI Text-to-Speech Model via the Audio API.
- The model transforms the text into spoken words using OpenAI's advanced speech synthesis technology.
- This allows for seamless playback in the Streamlit application, enhancing the user experience.

### How the Components Work Together
- 1️⃣ User uploads frames (or extracts them from a video).
- 2️⃣ CorrNet processes the frames and predicts glosses.
- 3️⃣ Gloss2Text translates the glosses into full sentences.
- 4️⃣ The final text is converted into real-time speech using OpenAI's Text-to-Speech API and streamed via PyAudio.


To Run the Application use ```BASH streamlit run st_to_txt/streamlit_app.py ```

## Data
We used the RWTH-PHOENIX-Weather 2014 T Dataset, which you can [download](https://www-i6.informatik.rwth-aachen.de/~koller/RWTH-PHOENIX-2014-T/) here.

## Alignments
1. Focus on Minimal Viable Product (MVP): Start with a small vocabulary (e.g., numbers, basic words, or a simple sentence).
2. Iteration: Add additional gestures and improvements based on testing if time permits.


# ds-modeling-pipeline

Here you find a Skeleton project for building a simple model in a python script or notebook and log the results on MLFlow.

There are two ways to do it: 
* In Jupyter Notebooks:
    We train a simple model in the [jupyter notebook](notebooks/EDA-and-modeling.ipynb), where we select only some features and do minimal cleaning. The hyperparameters of feature engineering and modeling will be logged with MLflow

* With Python scripts:
    The [main script](modeling/train.py) will go through exactly the same process as the jupyter notebook and also log the hyperparameters with MLflow

Data used is the [coffee quality dataset](https://github.com/jldbc/coffee-quality-database).

## Requirements:

- pyenv with Python: 3.11.3

### Setup

Use the requirements file in this repo to create a new environment.

```BASH
make setup

#or

pyenv local 3.11.3
python -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements_dev.txt
```

The `requirements.txt` file contains the libraries needed for deployment.. of model or dashboard .. thus no jupyter or other libs used during development.

The MLFLOW URI should **not be stored on git**, you have two options, to save it locally in the `.mlflow_uri` file:

```BASH
echo http://127.0.0.1:5000/ > .mlflow_uri
```

This will create a local file where the uri is stored which will not be added on github (`.mlflow_uri` is in the `.gitignore` file). Alternatively you can export it as an environment variable with

```bash
export MLFLOW_URI=http://127.0.0.1:5000/
```

This links to your local mlflow, if you want to use a different one, then change the set uri.

The code in the [config.py](modeling/config.py) will try to read it locally and if the file doesn't exist will look in the env var.. IF that is not set the URI will be empty in your code.

## Usage

### Creating an MLFlow experiment

You can do it via the GUI or via [command line](https://www.mlflow.org/docs/latest/tracking.html#managing-experiments-and-runs-with-the-tracking-service-api) if you use the local mlflow:

```bash
mlflow experiments create --experiment-name 0-template-ds-modeling
```

Check your local mlflow

```bash
mlflow ui
```

and open the link [http://127.0.0.1:5000](http://127.0.0.1:5000)

This will throw an error if the experiment already exists. **Save the experiment name in the [config file](modeling/config.py).**

In order to train the model and store test data in the data folder and the model in models run:

```bash
#activate env
source .venv/bin/activate

python -m modeling.train
```

In order to test that predict works on a test set you created run:

```bash
python modeling/predict.py models/linear data/X_test.csv data/y_test.csv
```

## About MLFLOW -- delete this when using the template

MLFlow is a tool for tracking ML experiments. You can run it locally or remotely. It stores all the information about experiments in a database.
And you can see the overview via the GUI or access it via APIs. Sending data to mlflow is done via APIs. And with mlflow you can also store models on S3 where you version them and tag them as production for serving them in production.
![mlflow workflow](images/0_general_tracking_mlflow.png)

### MLFlow GUI

You can group model trainings in experiments. The granularity of what an experiment is up to your usecase. Recommended is to have an experiment per data product, as for all the runs in an experiment you can compare the results.
![gui](images/1_gui.png)

### Code to send data to MLFlow

In order to send data about your model you need to set the connection information, via the tracking uri and also the experiment name (otherwise the default one is used). One run represents a model, and all the rest is metadata. For example if you want to save train MSE, test MSE and validation MSE you need to name them as 3 different metrics.
If you are doing CV you can set the tracking as nested.
![mlflow code](images/2_code.png)

### MLFlow metadata

There is no constraint between runs to have the same metadata tracked. I.e. for one run you can track different tags, different metrics, and different parameters (in cv some parameters might not exist for some runs so this .. makes sense to be flexible).

- tags can be anything you want.. like if you do CV you might want to tag the best model as "best"
- params are perfect for hypermeters and also for information about the data pipeline you use, if you scaling vs normalization and so on
- metrics.. should be numeric values as these can get plotted

![mlflow metadata](images/3_metadata.png)
