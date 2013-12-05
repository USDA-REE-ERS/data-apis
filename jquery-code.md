# jQuery Example #

The purpose of this document is to provide example code for interacting with [ARMS](http://ers.usda.gov/data-products/arms-farm-financial-and-crop-production-practices.aspx) data via its REST API. 

```javascript
  <html>
    <title>USDA ERS API web service call using jSON</title>
        <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js" type="text/javascript"></script>
        <script type="text/javascript">

          /* CODE SAMPLE: retrieve list of ARMS reports using ERS API */**_
          function GetARMS() {       
            jQuery.support.cors = true;

            var jsonp_url = "http://api.data.gov/USDA/ERS/data/Arms/Reports?api_key=API_KEY&survey=Crop&callback=?";

            $.ajax({
              url: jsonp_url,                    
              type: 'GET',           
              dataType: 'jsonp', 
              jsonpCallback: 'jsonCallback',                
              crossDomain: true,  
              success: function() { console.log("success"); }, 
              error: function() { console.log("error"); }                  
            });
          }

          function jsonCallback(data) {
               //alert('in jsonCallback');
            var reports = data.dataTable; // this array has the data
            alert(reports);      
            var strResult = "<table><th>Report Number</th><th>Report Description</th>";

            $.each(reports, function (index, Report) {
              strResult += "<tr><td>" + Report.report_num + "</td><td> " + Report.report_header + "</td></tr>";
            });
            strResult += "</table>";  

            $("#divResultARMS").html(strResult);         
          }

        </script>

      <body>
        <h3>USDA ERS API demo </h3>
        <p><button onClick="GetARMS();return false;">ARMS Surveys</button></p>
        <div id="divResultARMS"></div>
      </body>
  </html>
```
