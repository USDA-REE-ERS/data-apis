```html
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="DataRequest.aspx.cs" Inherits="ers.RESTexample.DataRequest" %>

<!DOCTYPE html>

<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title>ARMS Query Result</title>
</head>
<body>
    <form id="form1" runat="server">
    <div>
        <asp:PlaceHolder ID="plc_results" runat="server"></asp:PlaceHolder>
    </div>
    </form>
</body>
</html>
```


This is the code-behind, using C#.  Replace PUT_YOUR_API_KEY_HERE with your own key (from [api.data.gov](http://api.data.gov))

```c#
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Net;
using System.Reflection;
using System.Runtime.Serialization;
using System.Runtime.Serialization.Json;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;

namespace ers.RESTexample
{
    public partial class DataRequest : System.Web.UI.Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {
            string _rootArmsUrl = "http://api.data.gov/USDA/ERS/data/Arms/";
            string _myKey = "PUT_YOUR_API_KEY_HERE";

            SurveyCollection _surveys = this.RequestSurveys(_rootArmsUrl + "Surveys?api_key=" + _myKey);
            string _surveyCode = _surveys.data.FirstOrDefault(s => s.surveyDesc == "Farm finances").survey_abb;

            ReportCollection _reports = this.RequestReports(_rootArmsUrl + "Reports?api_key=" + _myKey + "&survey=" + _surveyCode);
            int _reportNum = _reports.data.FirstOrDefault(r => r.report_header == "Farm Business Income Statement").report_num;

            SubjectCollection _subjects = this.RequestSubjects(_rootArmsUrl + "Subjects?api_key=" + _myKey + "&survey=" + _surveyCode + "&report=" + _reportNum.ToString());
            int _subjectNum = _subjects.data.FirstOrDefault(s => s.subject == "Farm Operator Households").subject_num;

            SeriesCollection _series1 = this.RequestSeries(_rootArmsUrl + "Series?api_key=" + _myKey + "&survey=" + _surveyCode + "&report=" + _reportNum.ToString());
            string _series1code = _series1.data.FirstOrDefault(s => s.series_header == "Farm Typology").series_abb;

            SeriesCollection _series2 = this.RequestSeries(_rootArmsUrl + "Series?api_key=" + _myKey + "&survey=" + _surveyCode + "&report=" + _reportNum.ToString() + "&series=" + _series1code);
            string _series2code = _series1.data.FirstOrDefault(s => s.series_header == "All Farms").series_abb;

            StateCollection _states = this.RequestStates(_rootArmsUrl + "States?api_key=" + _myKey + "&survey=" + _surveyCode + "&report=" + _reportNum.ToString() + "&subject=" + _subjectNum.ToString());
            string _fipsStCode = _states.data.FirstOrDefault(s => s.state == "California").fips_st;

            string[] _years = this.RequestYears(_rootArmsUrl + "Years?api_key=" + _myKey + "&survey=" + _surveyCode + "&report=" + _reportNum.ToString() + "&subject=" + _subjectNum.ToString() + "&fipsStateCode=" + _fipsStCode);
            string _year = _years.FirstOrDefault(y => y == "2007").ToString();

            ResultCollection _results = this.RequestResults(_rootArmsUrl + "Finance?api_key=" + _myKey + "&survey=" + _surveyCode + "&report=" + _reportNum.ToString() + "&subject=" + _subjectNum.ToString() + "&series1=" + _series1code + "&series2=" + _series2code + "&fipsStateCode=" + _fipsStCode + "&year=" + _year);

            if (_results.info[0].message == "NO ERROR" && _results.info[0].recordCount > 0)
            {
                this.plc_results.Controls.Add(new LiteralControl("<table border=\"1\">"));
                this.plc_results.Controls.Add(new LiteralControl("<tr>"));
                Type _resultType = typeof(Result);
                foreach (PropertyInfo _p in _resultType.GetProperties())
                {
                    this.plc_results.Controls.Add(new LiteralControl("<th>" + _p.Name + "</th>"));
                }
                this.plc_results.Controls.Add(new LiteralControl("</tr>"));
                foreach (Result _r in _results.data)
                {
                    this.plc_results.Controls.Add(new LiteralControl("<tr>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.decimal_disp) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.element_name) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.element2_name) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.estimate) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.fips_st) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.footnote) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.report_num) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.rse) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.series) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.series_element) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.series_header) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.series2) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.series2_element) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.series2_header) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.stat_year) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.state) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.subject_num) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.survey_abb) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.topic_abb) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.topic_group) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.topic_header) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.topic_level) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.topic_sequence) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.unit_desc) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("<td>" + Convert.ToString(_r.unreliable_est) + "</td>"));
                    this.plc_results.Controls.Add(new LiteralControl("</tr>"));
                }
                this.plc_results.Controls.Add(new LiteralControl("</table>"));
            }
        }

        protected HttpWebResponse MakeRequest(string requestUrl)
        {
            HttpWebRequest _request = HttpWebRequest.Create(requestUrl) as HttpWebRequest;
            _request.ContentType = "application/json";
            HttpWebResponse _response = _request.GetResponse() as HttpWebResponse;

            if (_response.StatusCode != HttpStatusCode.OK)
            {
                throw new Exception(String.Format(
                    "Server error (HTTP {0}: {1}).",
                    _response.StatusCode,
                    _response.StatusDescription));
            }
            else
            {
                return _response;
            }
        }

        protected SurveyCollection RequestSurveys(string requestUrl)
        {
            HttpWebResponse _response = this.MakeRequest(requestUrl);
            DataContractJsonSerializer _jsonSerializer = new DataContractJsonSerializer(typeof(SurveyCollection));
            SurveyCollection _serializedObject = _jsonSerializer.ReadObject(_response.GetResponseStream()) as SurveyCollection;
            return _serializedObject;
        }

        protected ReportCollection RequestReports(string requestUrl)
        {
            HttpWebResponse _response = this.MakeRequest(requestUrl);
            DataContractJsonSerializer _jsonSerializer = new DataContractJsonSerializer(typeof(ReportCollection));
            ReportCollection _serializedObject = _jsonSerializer.ReadObject(_response.GetResponseStream()) as ReportCollection;
            return _serializedObject;
        }

        protected SubjectCollection RequestSubjects(string requestUrl)
        {
            HttpWebResponse _response = this.MakeRequest(requestUrl);
            DataContractJsonSerializer _jsonSerializer = new DataContractJsonSerializer(typeof(SubjectCollection));
            SubjectCollection _serializedObject = _jsonSerializer.ReadObject(_response.GetResponseStream()) as SubjectCollection;
            return _serializedObject;
        }

        protected SeriesCollection RequestSeries(string requestUrl)
        {
            HttpWebResponse _response = this.MakeRequest(requestUrl);
            DataContractJsonSerializer _jsonSerializer = new DataContractJsonSerializer(typeof(SeriesCollection));
            SeriesCollection _serializedObject = _jsonSerializer.ReadObject(_response.GetResponseStream()) as SeriesCollection;
            return _serializedObject;
        }

        protected StateCollection RequestStates(string requestUrl)
        {
            HttpWebResponse _response = this.MakeRequest(requestUrl);
            DataContractJsonSerializer _jsonSerializer = new DataContractJsonSerializer(typeof(StateCollection));
            StateCollection _serializedObject = _jsonSerializer.ReadObject(_response.GetResponseStream()) as StateCollection;
            return _serializedObject;
        }

        // Requesting years just returns an array, so we don't need a custom object.
        protected string[] RequestYears(string requestUrl)
        {
            HttpWebResponse _response = this.MakeRequest(requestUrl);
            DataContractJsonSerializer _jsonSerializer = new DataContractJsonSerializer(typeof(string[]));
            string[] _serializedObject = _jsonSerializer.ReadObject(_response.GetResponseStream()) as string[];
            return _serializedObject;
        }

        protected ResultCollection RequestResults(string requestUrl)
        {
            HttpWebResponse _response = this.MakeRequest(requestUrl);
            DataContractJsonSerializer _jsonSerializer = new DataContractJsonSerializer(typeof(ResultCollection));
            ResultCollection _serializedObject = _jsonSerializer.ReadObject(_response.GetResponseStream()) as ResultCollection;
            return _serializedObject;
        }
    }

    [DataContract]
    public class Info
    {
        [DataMember(Name = "message")]
        public string message { get; set; }
        [DataMember(Name = "pageCount")]
        public int pageCount { get; set; }
        [DataMember(Name = "pageIndex")]
        public int pageIndex { get; set; }
        [DataMember(Name = "pageSize")]
        public int pageSize { get; set; }
        [DataMember(Name = "recordCount")]
        public int recordCount { get; set; }
    }

    [DataContract]
    public class SurveyCollection
    {
        [DataMember(Name = "infoTable")]
        public List<Info> info { get; set; }
        [DataMember(Name = "dataTable")]
        public List<Survey> data { get; set; }
    }

    [DataContract]
    public class Survey
    {
        [DataMember(Name = "survey_abb")]
        public string survey_abb { get; set; }
        [DataMember(Name = "surveyDesc")]
        public string surveyDesc { get; set; }
   }

    [DataContract]
    public class ReportCollection
    {
        [DataMember(Name = "infoTable")]
        public List<Info> info { get; set; }
        [DataMember(Name = "dataTable")]
        public List<Report> data { get; set; }
    }

    [DataContract]
    public class Report
    {
        [DataMember(Name = "report_header")]
        public string report_header { get; set; }
        [DataMember(Name = "report_num")]
        public int report_num { get; set; }
    }

    [DataContract]
    public class SubjectCollection
    {
        [DataMember(Name = "infoTable")]
        public List<Info> info { get; set; }
        [DataMember(Name = "dataTable")]
        public List<Subject> data { get; set; }
    }

    [DataContract]
    public class Subject
    {
        [DataMember(Name = "subject")]
        public string subject { get; set; }
        [DataMember(Name = "subject_num")]
        public int subject_num { get; set; }
    }

    [DataContract]
    public class SeriesCollection
    {
        [DataMember(Name = "infoTable")]
        public List<Info> info { get; set; }
        [DataMember(Name = "dataTable")]
        public List<Series> data { get; set; }
    }

    [DataContract]
    public class Series
    {
        [DataMember(Name = "series_abb")]
        public string series_abb { get; set; }
        [DataMember(Name = "series_header")]
        public string series_header { get; set; }
    }

    [DataContract]
    public class StateCollection
    {
        [DataMember(Name = "infoTable")]
        public List<Info> info { get; set; }
        [DataMember(Name = "dataTable")]
        public List<State> data { get; set; }
    }

    [DataContract]
    public class State
    {
        [DataMember(Name = "fips_st")]
        public string fips_st { get; set; }
        [DataMember(Name = "state")]
        public string state { get; set; }
    }

    [DataContract]
    public class ResultCollection
    {
        [DataMember(Name = "infoTable")]
        public List<Info> info { get; set; }
        [DataMember(Name = "dataTable")]
        public List<Result> data { get; set; }
    }

    [DataContract]
    public class Result
    {
        [DataMember(Name = "decimal_disp")]
        public int decimal_disp { get; set; }

        [DataMember(Name = "element_name")]
        public string element_name { get; set; }

        [DataMember(Name = "element2_name")]
        public string element2_name { get; set; }

        [DataMember(Name = "estimate")]
        public int estimate { get; set; }

        [DataMember(Name = "fips_st")]
        public string fips_st { get; set; }

        [DataMember(Name = "footnote")]
        public string footnote { get; set; }

        [DataMember(Name = "report_num")]
        public int report_num { get; set; }

        [DataMember(Name = "rse")]
        public decimal rse { get; set; }

        [DataMember(Name = "series")]
        public string series { get; set; }

        [DataMember(Name = "series_element")]
        public int series_element { get; set; }

        [DataMember(Name = "series_header")]
        public string series_header { get; set; }

        [DataMember(Name = "series2")]
        public string series2 { get; set; }

        [DataMember(Name = "series2_element")]
        public int series2_element { get; set; }

        [DataMember(Name = "series2_header")]
        public string series2_header { get; set; }

        [DataMember(Name = "stat_year")]
        public int stat_year { get; set; }

        [DataMember(Name = "state")]
        public string state { get; set; }

        [DataMember(Name = "subject_num")]
        public int subject_num { get; set; }

        [DataMember(Name = "survey_abb")]
        public string survey_abb { get; set; }

        [DataMember(Name = "topic_abb")]
        public string topic_abb { get; set; }

        [DataMember(Name = "topic_group")]
        public string topic_group { get; set; }

        [DataMember(Name = "topic_header")]
        public string topic_header { get; set; }

        [DataMember(Name = "topic_level")]
        public string topic_level { get; set; }

        [DataMember(Name = "topic_sequence")]
        public int topic_sequence { get; set; }

        [DataMember(Name = "unit_desc")]
        public string unit_desc { get; set; }

        [DataMember(Name = "unreliable_est")]
        public bool unreliable_est { get; set; }
    }
}
```
