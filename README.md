## Features
- Export Elasticsearch records to a [PrivateBin](https://privatebin.info) document

## Installation
```
# clone the repo
$ git clone https://github.com/shoujo/elk-export.git

# change the working directory to elk-export
$ cd elk-export

# install the requirements
$ python3 -m pip install -r requirements.txt
```
## Config
A configuration file must be created at `./conf/settings.ini` with the following format:

```
[default]
index = Elasticsearch index to search
elastic = 192.168.0.1:9200
site = https://PrivateBin.instance.url
```
## Usage

To upload a user's messages to PrivateBin with the default expiration time (1 week):

`python3 elk-export.py -u user`

Paste expiration time can be specified with the optional `-e, --expiration` argument followed by one of the following values:

```5min, 10min, 1hour, 1day, 1week, 1month, 1year, never```
