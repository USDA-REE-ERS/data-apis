# Ruby Example #

The purpose of this document is to provide example code for interacting with the [ARMS](http://ers.usda.gov/data-products/arms-farm-financial-and-crop-production-practices.aspx) via its REST API. 

## A Basic Interface to the API ##

Below is a simple class implementing an API to the ARMS data.  Note:  ERS APIs require a key which can be obtained at [http://api.data.gov](http://api.data.gov). Please replace the ENTER_API_KEY_HERE text below with your registered key.

```ruby
module USDA
  require 'uri'
  require 'net/http'
  require 'json'

  class API
    attr_accessor :base_uri

    API_KEY = { api_key: "ENTER_API_KEY_HERE" }

    def initialize(base_uri = "http://api.data.gov/USDA/ERS/data/")
      @base_uri = base_uri
    end

    def response(endpoint, params = {})
      uri = build_uri(endpoint, params)
      response = Net::HTTP.get(uri)
      JSON.parse(response)
    end

    def build_uri(endpoint, params)
      params.merge!(API_KEY)
      uri = URI(@base_uri + endpoint)
      uri.query = URI.encode_www_form(params)
      uri
    end
  end
  ...
```

You can use this class to build an API object to interact with the ARMS API. For example, you could get a list of reports for a given survey like this:

```ruby
api = USDA::API.new
response = api.response('Arms/Reports', survey: "CROP")
puts response
```
would return

```ruby
{
    "infoTable" => [
        [0] {
            "recordCount" => 19,
                "message" => "NO ERROR"
        }
    ],
    "dataTable" => [
        [ 0] {
               "report_num" => 1,
            "report_header" => "Pesticide Use"
        },
        [ 1] {
               "report_num" => 2,
            "report_header" => "Crop Residue Management Practices"
        },
        [ 2] {
               "report_num" => 3,
            "report_header" => "Irrigation Technology and Water Use"
        },
        
        #...
        
        [18] {
               "report_num" => 19,
            "report_header" => "Precision Agriculture"
        }
    ]
}
```

## Getting Specific Data ##

To get specific data from the API, you need to build requests using the [REST paths](http://api.ers.usda.gov/) described in the documentation. The following `ARMS` class provides some pre-built methods to access these paths.

```ruby
...
  class ARMS
    def self.surveys
      # returns a list of surveys available in ARMS
      api = API.new
      api.response("Arms/Surveys")
    end

    def self.reports(survey)
      # returns a list of reports available in a given survey
      api = API.new
      api.response("Arms/Reports", survey: survey)
    end

    def self.subjects(survey, report)
      # returns a list of subjects available in a given report within a given survey
      api = API.new
      api.response("Arms/Subjects", survey: survey, report: report)
    end

    def self.crops(report, series, options = {})
      # queries the Crop survey for a given data series within a report, including any optional parameters
      options_hash = { report: report, series1: series }.merge options
      api = API.new
      api.response("Arms/Crop", options_hash)
    end
  end
  ...
```

You can use these class methods to make calls to specific routes using the optional and required paramters for each request.

For example to get a list of surveys available from ARMS, use the `.surveys` method:

` $ puts USDA::ARMS.surveys` to return

```ruby
{
    "infoTable" => [
        [0] {
            "recordCount" => 2,
                "message" => "NO ERROR"
        }
    ],
    "dataTable" => [
        [0] {
            "survey_abb" => "CROP",
            "surveyDesc" => "Crop production practices"
        },
        [1] {
            "survey_abb" => "FINANCE",
            "surveyDesc" => "Farm finances"
        }
    ]
}
```

For a list of available reports for a given survey, use the `.reports` method: 

`$ puts USDA::ARMS.reports("CROP")` returns:

```ruby
{
    "infoTable" => [
        [0] {
            "recordCount" => 19,
                "message" => "NO ERROR"
        }
    ],
    "dataTable" => [
        [ 0] {
               "report_num" => 1,
            "report_header" => "Pesticide Use"
        },
        [ 1] {
               "report_num" => 2,
            "report_header" => "Crop Residue Management Practices"
        },
        [ 2] {
               "report_num" => 3,
            "report_header" => "Irrigation Technology and Water Use"
        },
        # ...
        [18] {
               "report_num" => 19,
            "report_header" => "Precision Agriculture"
        }
    ]
}
```

The `.subjects` and `.series` methods work similarly, returning a list of subject values and series values we can use in the next step.

### The Main Event ###

The main use case of the ARMS API is used to query the Crop or Finance survey database using the endpoints `Arms/Crops` and `Arms/Finance` respectively. These endpoints accept `GET` requests with a few required and optional parameters.

The required parameters are:
-  report: An integer value specifying a report number
-  series1: A string value specifying a valid series code

The `USDA::ARMS.crops` method shown above crafts requests to the Crops survey endpoint using the required parameters, along with any optional paramters you pass it. For example, to receive the data points from the Crops survey in Report #1 (Pesticide Use) for all Farms (`series1 = "FARM"`) in the great state of Minnesota, you would use the method like this:

`USDA::ARMS.crops(1, "FARM", { fipsStateCode: "27" })`

The method returns a JSON object representing the API's response. The interesting part of the response is the series of datapoints corresponding to your query, which look like this:

```ruby
      # ...
[766] {
         "survey_abb" => "CROP",
         "report_num" => 1,
        "topic_group" => nil,
          "topic_abb" => "FUNACT",
          "topic_seq" => 10,
       "topic_header" => "Acres treated with fungicide",
        "topic_level" => nil,
          "unit_desc" => "percent of planted acres",
           "footnote" => nil,
        "subject_num" => 10,
            "fips_st" => "27",
              "state" => "Minnesota",
          "stat_year" => "1996",
             "series" => "FARM",
     "series_element" => 0,
      "series_header" => "All Farms",
       "element_name" => "TOTAL",
            "series2" => "FARM",
    "series2_element" => 0,
     "series2_header" => "All Farms",
      "element2_name" => "TOTAL",
           "estimate" => 9.372,
                "rse" => 29.625,
     "unreliable_est" => true,
       "decimal_disp" => 3
},
[767] {
         "survey_abb" => "CROP",
         "report_num" => 1,
        "topic_group" => nil,
          "topic_abb" => "FUNACT",
          "topic_seq" => 10,
          
          # ...
```
As you can see, the response is a series of datapoints (up to 1,000 per request), with different attributes that describe what the datapoint is. In order to make sense of the data, you may want to filter these results for datapoints of a specific type.

### Filtering the Results ###

Once you have a blob of JSON representing the response, you may want to filter the datapoints to only show the ones you care about. An example of how you could do this is shown below:

```ruby
  # ...
  
  class Selector
    def self.select_from_results(selectors = {}, results)
      selected = results["dataTable"]
      selectors.each do |selector, value|
        selected = selected.select { |e| e[selector] == value }
      end
      selected
    end
  end
  
  # ...
```

The `USDA::Selector.select_from_results` method takes a JSON response object and returns only the datapoints that match a set of filters (one or more) that you provide as :key => value pairs.

Using our previous example, if we wanted to find only the datapoints in the response that related to `topic_seq = 1` ("Acres treated with any pesticide") for `subject_num = 1` (corn), we could use the method like this:

```ruby
#... assume we already have a results object from the previous query (i.e. using the USDA::ARMS.crops method)
results["dataTable"].count  # => 800 (lots of data points) 
filtered_results = USDA::Selector.select_from_results( { "topic_seq" => 1,
                                                          "subject_num" => 1 }, results)
filtered_results.count   # => 16 (only the datapoints showing the total acreage of corn treated with pesticides)
```

### Getting Data in a Time Series

Let's say you wanted to get time series data for a specific data point. You could use a method like the one below:

```ruby
  # ...
  
  class DataSeries
    def self.data_by_year(topic_seq, subject_num, element2_name, results)
      selected = Selector.select_from_results({ "topic_seq" => topic_seq,
                                                "subject_num" => subject_num,
                                                "element2_name" => element2_name },
                                                results)
      selected.map { |n| { n["stat_year"] => n["estimate"] } }
    end
  end
  
  #...
```

This method will take a set of results, filter them for a specific `topic_seq`, `subject_num`, and `element2_nam`, and return an array of datapoints corresponding to a specific year. 

Again using the above results for the Pesticide use in Minnesota, to get the time series data for total acreage (`element2_name = 'TOTAL`) with pesticides (`topic_seq = 1`) planted with corn (`subject_num = 1`), we could use the method like this:

```ruby
#... assume we already have a 'results' object that was the result of a previous query
time_series = USDA::DataSeries.data_by_year(1, 1, "TOTAL", results)
#  => [{"2010"=>95.461}, {"2005"=>98.916}, {"2001"=>97.477}, {"2000"=>97.374}, {"1999"=>97.967}, {"1998"=>96.116}, {"1997"=>92.93}, {"1996"=>96.651}]
```
