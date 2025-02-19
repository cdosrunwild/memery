# Memery
> Use human language to search your image folders!



## What is memery?

![meme about having too many memes](images/E2GoeMyWEAAkcLz.jpeg)

The problem: you have a huge folder of images. Memes, screenshots, datasets, product photos, inspo albums, anything. You know that somewhere in that folder is the exact image you want, but you can't remember the filename or what day you saved it. There's nothing you can do but scroll through the folder, skimming hundreds of thumbnails, hoping you don't accidentally miss it, and that you'll recognize it when you do see it. 

Humans do this amazingly well. But even with computers, local image search is still a manual effort - you're still sorting through folders of images, like an archivist of old.

**Now there's Memery**.

The `memery` package provides natural language search over local images. You can use it to search for things like "a line drawing of a woman facing to the left" and get _reasonably good results!_ 

You can do this over thousands of images (it's not optimized for performance yet, but search times scale well under O(n)). 

You can view the images in a browser GUI, or pipe them through command line tools. 

You can use `memery` or its modules in Jupyter notebooks, including GUI functions! 

Under the hood, `memery` makes use of **CLIP**, the [Contrastive Language-Image Pretraining transformer](https://github.com/openai/CLIP), released by OpenAI in 2021. CLIP trains a vision transformer and a language transformer to find the same latent space for images and their captions. This makes it perfect for the purpose of natural language image search. CLIP is a giant achievement, and `memery` stands on its shoulders.

Outline:
- Usage
  - Install locally
  - Use GUI
  - Use CLI
  - Use in Jupyter
  - Use the library
- Development
  - Notebook-driven development
  - Pull the repo
  - Notebook-driven development
  - Change the notebooks
  - Test the notebooks
  - Notebook-driven development
  - Tangle the source code
  - Weave the documentation
  - Commit your changes
- Contributing
  - Who works on this project

## Installation

With Python 3.6 or greater:

```
pip install memery
pip install git+https://github.com/openai/CLIP.git
```

Currently memery defaults to GPU installation. This will 
probably be switched in a future version. 

For now, if you want to run CPU-only, run the following command after installing memery:

`pip install torch==1.7.1+cpu torchvision==0.8.2+cpu torchaudio==0.7.2 -f https://download.pytorch.org/whl/torch_stable.html`

Someday memery will be packaged in an easy to use format, but since this is a Python project it is hard to predict when that day will be.

If you want to help develop memery, you'll need to clone the repo. See below.

## Usage

What's your use case? 

**I have images and want to search them with a GUI app**
   
   ↳  Use the Browser GUI
   
**i have a program/workflow and want to use image search as one part of it**
  
   ↳ Use in Jupyter
   
   ↳ Use as a Python module
   
   ↳ Use from command line or shell scripts
   
**i want to improve on and/or contribute to memery development**
 
   ↳ Start by cloning the repo 

### Use GUI

Currently memery has a rough browser-based GUI. To launch it, run the following in a command line: 

```memery serve```

or set up a desktop shortcut that points to the above command.

Memery will open in a browser window. The interface is pretty straightforward, but it has some quirks.

![screenshot of memery GUI displaying wholesome memes](images/streamlit-screenshot.png)

The sidebar on the left controls the location and query for the search. The "Directory" box requires a full directory path; unfortunately, Streamlit does not yet have a folder-picker component. The path is relative to your current working directory when you run `memery serve`.

The search will run once you enter a text or image query. If you enter both text and image queries, memery will search for the combination.

Beneath these widgets is the output area for temporary messages displayed with each search. Mostly this can be ignored.

The right hand panel displays the images and associated options. Major errors will appear here as giant stack traces; sometimes, changing variables in the other widgets will fix these errors live. If you get a large error here it's helpful to take a screenshot and share it with us in Github Issues.

### Use in Jupyter

If you're in a Jupyter environment, you can summon an ipywidgets GUI directly into an output cell like this:

```
from memery.gui import appPage

```

```
app = appPage()
display(app)
```


    <memery.gui.appPage at 0x7f829c87d290>


![screenshot of memery Jupyter GUI displaying wholesome memes](images/jupyter-screenshot.png)

Unfortunately the widgets won't display without an active runtime, so the above is a screenshot. In a Jupyter environment, the GUI works just like the browser-based GUI. 

Opening new appPage instances in separate cells can be helpful for exploring a dataset, but it can also cause memory overruns. For this reason the Jupyter interface is tabbed and each search will appear in a new tab.

### Use CLI

The memery command line is currently very rudimentary. So far you can use `memery` on any folder and it will search for images recursively, returning a list object to stdout. 

Pass the --n flag to control how many images are returned (default 10).

`memery recall PATH/TO/IMAGE/FOLDER 'query' --n 20`

I'm not clear yet on what behavior command-line users will expect from it. In the future we may want to make it non-recursive by default, and map the behavior more closely to POSIX standards. I would love feedback or help on this.
`

### Use as a library

The function currently called `query_flow` accepts a folder name and a query and returns a ranked list of image files. This is also recursive by default, and prints too much information into stdout. And it calls index_flow every time, which can be annoying if you have corrupted files in the directory, as it will need to rebuild the tree-index each time despite the files not changing. This will be solved in a future release.

```
from memery.core import query_flow
```

```
ranked = query_flow('./images', 'dad joke')

print(ranked[:5])
```

    89it [00:00, 44503.23it/s]
    89it [00:00, 37660.72it/s]
    0it [00:00, ?it/s]

    starting timer
    Checking files
    Indexing
    Skipping bad file: images/memes/corrupted-file.jpeg
    due to <class 'PIL.UnidentifiedImageError'>
    Skipping bad file: images/memes/.ipynb_checkpoints/corrupted-file-checkpoint.jpeg
    due to <class 'PIL.UnidentifiedImageError'>
    Loaded 82 encodings
    Encoding 0 new images


    0it [00:00, ?it/s]

    Building treemap


    


    Saving 82 encodings
    Converting query
    Searching 82 images
    Done in 0.5853581428527832 seconds
    ['images/memes/Wholesome-Meme-68.jpg', 'images/memes/Wholesome-Meme-74.jpg', 'images/memes/Wholesome-Meme-88.jpg', 'images/memes/Wholesome-Meme-78.jpg', 'images/memes/Wholesome-Meme-23.jpg']


Here's the first result from that list:

![](images/memes/Wholesome-Meme-68.jpg)


So that's how to use memery. Let's look at how you can help make it better.

## Development

Memery is a different beast than most pieces of code you've seen. It's a *literate program*: a program written for human beings to read.

Nothing in this code is particularly special. The algorithms, data structures, and pipeline are either bog-standard or even subpar. The model was developed by OpenAI, and the tree indexer by Spotify. All I did was glue a bunch of disparate things together. 

### Notebook-driven development

The thing that makes this program interesting is that it was developed **in Jupyter notebooks**. Each component has its code and documentation in the same place, a `.ipynb` notebook.

This is possible thanks to a new-ish project called `nbdev`. It provides literate programming functionality for notebooks. Specifically, it allows the programmer to automatically weave code and tangle documentation from the content of the notebooks!

This means that relevant docs, code and tests for that code all live in the same location. And ideally, that place is a notebook *written for humans*. I'm still getting the hang of this, but it means explaining *why* something works, rather than simply how to do it.

So that's the *why* of notebook-driven development. Let's look at *how* that works in practice.

### Pull the repo

Clone this repository from Github:

`git clone https://github.com/deepfates/memery.git`



### Install dependencies and memery
Enter the `memery` folder and install requirements:

```
cd memery
pip install requirements.txt
```

We also have to download the CLIP model directly from their git repo, as they decline to upload it to PyPi:

`pip install git+https://github.com/openai/CLIP.git`

And finally install your local, editable copy of memery with 

`pip install -e .`

### Notebook-driven development

Within the main memery folder there is a subfolder `memery/memery`. This contains `.py` files, which are the source code for the repo. You might be tempted to edit these Python files directly, but you must hesitate.

Remember: we are doing notebook-driven development. The Python files in that folder are auto-generated from the `.ipynb` files in this folder. If you edit them directly, the notebooks could overwrite your code in a future build. There are ways to backport code from the `.py` files to the notebooks, but it's not recommended.

The reason we write the code in notebooks is to keep documentation and tests right next to their code. This single source of truth will percolate to the docs, code packages and testing framework. 

 :warning: Always edit the notebook files, not the .py files! :warning:

### Change the notebooks

This is the part you came here for. Change the code in the notebooks as you need. 

Change the **documentation** as well, to match your new code. Code should be documented as closely to the cell where it is used as possible. The more general design should be at the top of the notebook, and implementation-specific details further down.

Each notebook should also contain **tests** for the code it defines. These can be simple assert statements which return True.


### Test the notebooks

The following command will run each notebook in the main folder and report if any cells return False or raise errors. We pause for a couple seconds between starting each notebook, to avoid collisions accessing the GPU.

`nbdev_test_nbs --pause 2`

Always run tests before committing your changes.


### Notebook-driven development

As an interpreted language, Python doesn't usually have a "compile" step. But as a literate program, Memery is written for humans first and computers second. To get a clean Python module, free of documentation and tests, we **tangle** the source code into the `/memery` directory, then **weave** the documentation.

Any code with the `#export` tag at the beginning of its cell will be tangled into the appropriate .py file. Any code cell with `#hide` will be hidden from the docs. 

### Tangle the source code

To tangle code into `/memery`, use the following in a command line:

`nbdev_build_lib`

I often use `nbdev_build_lib && pip install -e` (although I'm no longer sure if it's necessary to reinstall an editable module).

### Weave the documentation

To generate the documentation website, use `nbdev_build_docs`. Or, get a live preview using `make docs_serve`.

### Commit your changes

Before committing, clean useless metadata from your notebooks with `nbdev_clean_nbs`. Or, install the nbdev pre-commit hooks to do this automatically with `nbdev_install_git_hooks`.

Then commit your changes to the repo and push! Make sure to include the `/docs` and `/memery` folders, and any notebooks you changed when you commit.

## Contributing

Memery is open source and you can contribute. See [CONTRIBUTING.md]('/CONTRIBUTING.md') for guidelines on how you can help.

### Who works on this project

Memery was first written by Max Anton Brewer aka @deepfates in the summer of 2021. Some commits are listed from @robotface-io but that was just me using the wrong account when I first started. 

I wrote this to solve my own needs and learn notebook-based development. I hope it helps other people too. If you can help me make it better, please do. I welcome any contribution, guidance or criticism.

**The ideal way to get support is to open an issue on Github**. However, the *fastest* way to get a response from me is probably to [direct message me on twitter](twitter.com/deepfates). This is my first attempt to coordinate an open-source project, bear with me.

