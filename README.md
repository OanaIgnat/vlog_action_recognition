# IfAct: Identifying Visible Actions in Lifestyle Vlogs

This repository contains the dataset, **IfAct**, and code for our ACL 2019 paper:

[Identifying Visible Actions in Lifestyle Vlogs](https://arxiv.org/abs/1906.04236)

## Task Description

![Example instance](images/task_description.jpg)
<p align="center"> Given a video and its transcript, which human actions are visible in the video? </p>


## Miniclips

We provide a [Google Drive folder with the raw miniclips](https://drive.google.com/drive/folders/1JJApVcu5o_zvtL3M0Y9a1sAiJCVTPQ6z?usp=sharing).

A miniclip is a short video clip (maximum 1 min) extracted from a YouTube video. We segment the videos into miniclips in order to ease the annotation process.
For more details on how the segmentation is performed, see _section 3.1_ in our [paper](https://arxiv.org/abs/1906.04236).

## Videos
The __video youtube urls__ from which the miniclips were created can be found [here](https://drive.google.com/drive/folders/1t3_r0l7ca4FtP60YMTJ7CPBHWYamYtJ5?usp=sharing), together with the __video transcripts__ and __titles__.

*Important*: You can automatically download your __own videos__, __transcripts__ and perform the __text and movement filtering__ by running the code described in the __Youtube processing__ README section.
If you still need the raw videos, e-mail me.


## Data Format
The annotations of the miniclips are available at [`data/miniclip_action.json`](data/miniclip_actions.json).
The JSON file contains a dictionary: the keys represent the miniclips (e.g. "4p1_3mini_5.mp4") and the values represent the (action, label) pairs.

The miniclip name is formed by concatenating its YouTube channel, playlist, video and miniclip index. For miniclip "4p1_3mini_5.mp4":
* 4 = __channel__ index
* p1 = __playlist__ index (0 or 1) in the channel
* 3 = __video__ index in the playlist
* mini_5 = __miniclip__ index in the video

For each miniclip, we store the __extracted actions__ and their corresponding __labels__:
* 0 for __visible__
* 1 for __not visible__.

The visibile actions were manually cleaned by removing extra words like: usually, now, always, I, you, then etc.
Example format in JSON:

```json
{
  "4p1_3mini_5.mp4": [
    ["smelled it", 1],
    ["used this in my last pudding video", 1],
    ["make it smell nice", 0],
    ["loving them", 1],
    ["using them about a week", 1],
    ["using my favorite cleaner which", 0],
    ["work really really well", 1],
    ["wipe down my counters", 0],
    ["wiping down my barstools", 0],
    ["using the car shammies", 0]
  ]
}
```
## Citation

Please cite the following paper if you find this dataset useful in your research:

```bibtex
@inproceedings{ignat-etal-2019-identifying,
    title = "Identifying Visible Actions in Lifestyle Vlogs",
    author = "Ignat, Oana  and
      Burdick, Laura  and
      Deng, Jia  and
      Mihalcea, Rada",
    booktitle = "Proceedings of the 57th Annual Meeting of the Association for Computational Linguistics",
    month = jul,
    year = "2019",
    address = "Florence, Italy",
    publisher = "Association for Computational Linguistics",
    url = "https://aclanthology.org/P19-1643",
    doi = "10.18653/v1/P19-1643",
    pages = "6406--6417"
```

# Run the code

Some small parts of it are still under revision; please let me know here or by e-mail how I can help.

## Installation
To download *Stanford-postagger-full-2018-10-16* and all the required libraries.
You need *Python 3* (I have Python 3.6.7), it doesn't work with Python 2.
Comment Tensorflow (for cpu) in *requirement.txt* if you use tensoflow-gpu instead.

```bash
sh setup.sh
```

## Data Requirements
Download [glove_vectors.txt (pre-trained POS embeddings on Google N-gram corpus using POS info from 5-grams)](https://drive.google.com/open?id=1zSfeAKyPTuQMHOP53fPJDYqUqKs22tdJ).\
Download [glove.6B.50d.txt embeddings](https://drive.google.com/open?id=1TShifgw5OjUFYWZBnN5ez5uRijX5W3Ym).
Put both of them in [`data`](data).
## Usage
There are 3 main modules: **Youtube processing**, **AMT processing** and **Classification**. The first 2 modules are still under revision. The third module can be used without the first 2 ones,
as all the data is accessible from the [`data`](data) folder or *Google Drive*.

### Youtube processing
Needs an youtube downloader API key.
Given channel ids - *now 10*, and playlist ids for each channel ( 2 playlists / channel), it downloads all the videos from each playlist.
The code can be found in [`youtube_preprocessing`](youtube_preprocessing).

```bash
python main_youtube.py
```

### Amazon Mechanical Turk (AMT) processing
Does all the processing related to AMT (read data, spam removal, compute agreement). The code is in [`amt`](amt).
```bash
python main_amt.py
```

### Classification
Everything related to classification models, embeddings and features can be found in [`classify`](classify).

#### Models
The available models are: *svm*, *lstm*, *elmo*, *multimodal (video features + elmo embeddings*).

To call the methods, for example *lstm*:
```bash
python main_classify.py --do-classify lstm
```

#### Extra data
The Extra data consists of: *context* and *POS* embeddings, and also *concreteness* scores for each word in the actions.

You can find the **context** information for each action in [`data/dict_context.json`](data/dict_context.json): each action is assigned the sentence it is extracted from.
The sentences are extracted from the Youtube transcripts, using the **Stanford Parser**.

You can find both the **POS** and **context embeddings** in [`data/Embeddings`](data/Embeddings). They consist of averaging the surrounding 5 left and right *glove50d* word embeddings. For future work, we want to use *elmo* embeddings.

The **concreteness** dataset from `Brysbaert et al.` can be find in [`data`](data/) folder. Also, the data extracted from the file (just the unigrams and their concreteness scores) is in [`data/dict_all_concreteness.json`](data/dict_all_concreteness.json).

The **concreteness and POS** of all the words in the actions is stored in [`data/dict_action_pos_concreteness.json`](data/dict_action_pos_concreteness.json).

To add these **extra features** to your model: for example run *svm with context and pos embeddings*:

```bash
python main_classify.py --do-classify svm --add-extra context pos
```

To run *multimodal with concreteness*:
```bash
python main_classify.py --do-classify multimodal --add-extra concreteness
```

#### Video Features
The video features are **Inception**, **C3D** and *their concatenation*. These are found in [`data/Video/Features`](data/Video/Features). By default, the multimodal model is run with the concatenation of features:
*inception + c3d*.

To run *multimodal with inception*:
```bash
python main_classify.py --do-classify multimodal --type-feat inception
```

##### YOLO output
After running YOLOv3 object detector on all the miniclips, all the results are stored [here](https://drive.google.com/file/d/11GrSXgvKIqVpyTB0UrXhliIM--IWElll/view?usp=sharing). Copy them in [`data/Video/YOLO/miniclips_results`](data/Video/YOLO/miniclips_results).

### Useful

For all of this data, there is code available to generate your own data also.

Look in [`main_classify.py`](main_classify.py) *parse_args* method for the rest of the models and data combinations.

