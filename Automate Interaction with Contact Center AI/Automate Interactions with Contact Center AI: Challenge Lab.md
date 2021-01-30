# Steps for Automate Interaction with Contact Center AI: Challenge Lab

* Enable Speech-to-text API

* Enable Natural Language Processing API

* Create the following resources

  * triggering bucket: e.g. t-buck88
  * Create DFaudio folder inside the triggering bucket
  * staging bucket: e.g. s-buck88
  * BigQuery dataset: e.g. mydataset
  * PubSub topic: e.g. audio-topic

* Clone the repo from github 

```bash
git clone https://github.com/GoogleCloudPlatform/dataflow-contact-center-speech-analysis.git
```

* Go to the following path to deploy a cloud function

```bash
cd dataflow-contact-center-sppech-analysis/saf-longrun-job-func
```

* Deploy cloud function that is triggered by the triggering bucket using

```bash
gcloud functions deploy safLongRunJobFunc --region=us-central1 --stage-bucket=t-buck88 --runtime=nodejs8 --trigger-event=google.storage.object.finalize --trigger-resource=t-buck88
```

* Go to the dataflow directory to deploy a streaming dataflow pipeline

```bash
cd dataflow-contact-center-sppech-analysis/saf-longrun-job-dataflow
```

* Set up a python3 environment

```bash
python -m virtualenv env -p python3
source env/bin/activate
pip install apache-beam[gcp]
pip install dateparser
```

* Running the pipeline

```bash
python3 saflongrunjobdataflow.py --region=us-central1 --project=qwiklabs-gcp-00-453b19e73372 --input_topic=projects/qwiklabs-gcp-00-453b19e73372/topics/audio-topic --runner=DataflowRunner --temp_location=gs://s-buck88/tmp --output_bigquery=mydataset.audio --requirements_file="requirements.txt"
```

* Copy some audio files to the triggering bucket

```bash
gsutil -h x-goog-meta-dlp:false -h x-goog-meta-callid:1234567 -h x-goog-meta-stereo:false -h x-goog-meta-pubsubtopicname:audio-topic -h x-goog-meta-year:2019 -h x-goog-meta-month:11 -h x-goog-meta-day:06 -h x-goog-meta-starttime:1116 cp gs://qwiklabs-bucket-gsp311/speech_commercial_mono.flac gs://t-buck88
```

```bash
gsutil -h x-goog-meta-dlp:false -h x-goog-meta-callid:1234567 -h x-goog-meta-stereo:true -h x-goog-meta-pubsubtopicname:audio-topic -h x-goog-meta-year:2019 -h x-goog-meta-month:11 -h x-goog-meta-day:06 -h x-goog-meta-starttime:1116 cp gs://qwiklabs-bucket-gsp311/speech_commercial_stereo.wav gs://t-buck88
```

* Go to BigQuery under BigData, the piepline created a table in the dataset called mydataset named audio as specified when running the pipeline.

* Run the following query to get the answer to the first question

```SQL
SELECT
  ARRAY(
  SELECT
    AS STRUCT word,
    startSecs,
    endSecs,
    speakertag,
    confidence
  FROM
    UNNEST(words)) transcript
FROM
  `qwiklabs-gcp-00-453b19e73372.mydataset.audio`
```

* Before running the DLP task query the audio table into another table in the same dataset called audio_2

```SQL
CREATE TABLE
`qwiklabs-gcp-00-453b19e73372.mydataset.audio_2` 
AS 
SELECT *
FROM
`qwiklabs-gcp-00-453b19e73372.mydataset.audio`
```

* From export table run the DLP