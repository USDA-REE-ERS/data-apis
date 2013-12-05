Python Example
--------------

Below is a simple Python example of implementing the API to the [ARMS](http://ers.usda.gov/data-products/arms-farm-financial-and-crop-production-practices.aspx) data.

__pyusda.py__
```python
import sys
from collections import namedtuple

import requests


class APIObject(object):
    """
API Objects pulls data and stores the response.
"""

    def __init__(self, api_url, api_key):

        self.url = '%s?api_key=%s' % (api_url, api_key)
        self.response = requests.get(self.url)
        self.data = self.response.json()


if __name__ == "__main__":
    """
Run APIObject from the commandline:
$ python pyusda.py http://api.data.gov/USDA/ERS/data/Arms/Surveys put_your_api_key_here

The first arg is the api_url. The second is your API_KEY.
$ python pyusda.py $API_URL $API_KEY

Where $API_URL is the url that you want to hit and $API_KEY is replaced with your API_KEY.

If you would like to capture the output run the following:
$ python pyusda.py 'data/Arms/Surveys' $API_KEY > output.json

"""
    import pprint
    ap = APIObject(sys.argv[1], sys.argv[2])
    pprint.pprint(ap.data)
```

Script and instructions to interact with the ARMS API


__Run APIObject from the commandline__
```
$ python pyusda.py http://api.data.gov/USDA/ERS/data/Arms/Surveys put_your_api_key_here
```

The first arg is the api_url. The second is your API_KEY (obtained from [http://api.data.gov/](http://api.data.gov)).
```
$ python pyusda.py $API_URL $API_KEY
```
Where $API_URL is the url that you want to hit and $API_KEY is replaced with your API_KEY.

If you would like to capture the output run the following:
```
$ python pyusda.py 'data/Arms/Surveys' $API_KEY > output.json
```
The output.json file in this repo has an example of how the output would be stored.


__Example of usage from ipython, a python interpretor.__
```python
In [1]: api_key = 'put_your_api_key_here'

In [2]: from pyusda import APIObject

In [3]: ap = APIObject('http://api.data.gov/USDA/ERS/data/Arms/Surveys', api_key)

In [4]: ap.url
Out[4]: 'http://api.data.gov/USDA/ERS/data/Arms/Surveys?api_key=wpe3bvzE0cO9VpVMSCfo6ULSq4ecjy2BKVZ6sOvF'

In [5]: ap.response
Out[5]: <Response [200]>

In [6]: ap.data
Out[6]:
{u'dataTable': [{u'surveyDesc': u'Crop production practices',
   u'survey_abb': u'CROP'},
  {u'surveyDesc': u'Farm finances', u'survey_abb': u'FINANCE'}],
 u'infoTable': [{u'message': u'NO ERROR', u'recordCount': 2}]}

In [7]: ap.data['dataTable']
Out[7]:
[{u'surveyDesc': u'Crop production practices', u'survey_abb': u'CROP'},
 {u'surveyDesc': u'Farm finances', u'survey_abb': u'FINANCE'}]

In [8]: for survey in ap.data['dataTable']:
   ...:         print survey
   ...:
{u'survey_abb': u'CROP', u'surveyDesc': u'Crop production practices'}
{u'survey_abb': u'FINANCE', u'surveyDesc': u'Farm finances'}
