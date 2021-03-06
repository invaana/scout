# Lucy Scout  



 

| Master | Develop   |  
|------- | -------   | 
| [![Master Build Status](https://travis-ci.org/invaana/lucy-scout.svg?branch=master)](https://travis-ci.org/invaana/lucy-scout) | [![Develop Build Status](https://travis-ci.org/invaana/lucy-scout.svg?branch=develop)](https://travis-ci.org/invaana/lucy-scout)  | 
| [![Master codecov](https://codecov.io/gh/invaana/lucy-scout/branch/master/graph/badge.svg)](https://codecov.io/gh/invaana/lucy-scout) |  [![Develop codecov](https://codecov.io/gh/invaana/lucy-scout/branch/develop/graph/badge.svg)](https://codecov.io/gh/invaana/lucy-scout/branch/develop) | 


This is a data aggregation framework for scouting and aggregating Scientific Data. This is part of a project called 
Lucy - an assistant for scientific researcher. Lucy aims at keeping an eye on the latest trends in the research from 
 research blogs, research journals.
 
 
The framework contains 3 modules:

- **scider** - a scientific data spider/scouting module  
- **sanitizer** - sanitising the aggregated HTML data to use it further for text mining and data processing. 
The module will remove the html attributes like `class` which are need for styling.
- **db** - database module that stored the data into database. Since we use MongoDB as the database,
 you can scrape the data and add dynamic fields too. (Currently supports MongoDB only)
- **pubmed** - this module imports the data from [pubmed](https://www.ncbi.nlm.nih.gov/pubmed) into the database.


## Technical Stack

- Python 2.7.x
- Redis Server 3.2.x
- MongoDB v3.x



## How to install
```
#Install scout development version, no stable version yet
pip install -e  git+https://github.com/invaana/lucy-scout.git#egg=scout

```


## 
```
# Install the following packages (Ubuntu )
sudo apt-get install libxml2-dev libxslt1-dev  libxml2-devel libxslt1-devel python-devel gcc \
 libxslt-devel libxml2-devel
```

 
## How to use

### 1. 2step blog data gathering

When we say 2 step blog, it means just any blog structure(post list and post detail pages). In this scenario, 

1. **STEP1:** gather the URLs of the blogs including the paginated contain.
2. **STEP2:** gather the full details of each blog.


```
Step1:  Create a scider input json file 
# example : examples/configs/github.json

from scout.scider.tasks import scrape_website_task
from scout.scider import helpers


from scout.db.mongo import connect
connect('scout')

config_file = "configs/github.json"
config = helpers.read_json_file(config_file)

scrape_website_task(config=config, max_limit=30, save=True) 

:param config: config file in dict format
:param max_limit: max number of entry scraping after which, the scraper should halt
:param save: should the data be saved to db.

```

To run the job in queue `scrape_website_task.delay(config=config, max_limit=30, save=True)`



### 3. 3step website as a blog scraping with topics

When we say 3 step blog, it means categorised sub-blogs in a blog/website.  

```
from scout.scider.tasks import scrape_website_task, scrape_website_topics_task
from scout.scider import helpers

from scout.db.mongo import connect
connect('scout')


CONFIG_FOLDER = 'configs'
config_file = "%s/newscientist.json"%CONFIG_FOLDER
config = helpers.read_json_file(config_file)


# get the topic urls
topics_configs =  scrape_website_topics_task(config=config,
                                             config_folder=CONFIG_FOLDER)['topics_configs']
topics_configs_count = len(topics_configs)
print topics_configs
print "Found %s topics "%topics_configs_count

# gather data from each topic url
for i, each_config_loc in enumerate(topics_configs):
    if i ==0:
        continue
    print "Now detailed scrapping %s/%s topics" %(i+1, topics_configs_count)
    config_file = each_config_loc
    config = helpers.read_json_file(config_file)
    scrape_website_task(config, 10000, True)

```

 


This module is designed by Data Science team for internal usage at Invaana. 
If you are a scientific data enthusiast, we'd love to know more about your interests. 
Let us know [@invaana](http://twitter.com/invaana) !