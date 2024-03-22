# Getting the data

## Download the NY Taxi dataset

```bash
wget -i ./data_taxi/file_url.txt -P data_taxi/ -w 2
```


## Download the amazon-books-reviews dataset

The dataset for this project is hosted by Kaggle. To download the necessary dataset for this project, please follow the instructions below.

1. Go to https://www.kaggle.com/datasets/mohamedbakhet/amazon-books-reviews
2. Click on the 'Download' button
3. Kaggle will prompt you to sign in or to register. If you do not have a Kaggle account, you can register for one.
4. Upon signing in, the download will start automatically.
5. After the download is complete, unzip the "archive" zip file


```bash
cd data_books
unzip archive.zip
``` 

# Copy your iMessage SQLite DB
```bash
cp ~/Library/Messages/chat.db .
```



## Setup virtual python environment
Create a [virtual python](https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/) environment to keep dependencies separate. The _venv_ module is the preferred way to create and manage virtual environments.

 ```console
python3 -m venv .venv
```

Before you can start installing or using packages in your virtual environment you’ll need to activate it.

```console
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
 ```


# Optional steps


## Cleanup of virtual environment
If you want to switch projects or otherwise leave your virtual environment, simply run:

```console
deactivate
```

If you want to re-enter the virtual environment just follow the same instructions above about activating a virtual environment. There’s no need to re-create the virtual environment.