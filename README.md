# Kibana Conversion Funnel
This is hack I found to create a conversion funnel with Kibana


Trying to make the Funnel work with the existing [kbn_funnel](XXX) in Kibana 5.X and 6.X was taking too long and no updates have been made so I found a work around that actually opened the door to tons of possibilities. With this hack, you can use ANY D3 library to create interactive and fun visuals. Check out https://canvasjs.com/docs/charts/chart-types/ or https://github.com/d3/d3/wiki/Gallery for all your options. This example is for creating a funnel visual.



# How to do

Download the transform vis. In 6.2.2 I did this with:
```
kibana/bin/kibana-plugin install https://github.com/PhaedrusTheGreek/transform_vis/releases/download/6.2.2/transform_vis-6.2.2.zip
```
Go to https://github.com/PhaedrusTheGreek/transform_vis for more info.

Add `transform_vis.allow_unsafe: true` to the kibana.yml to allow Javascript in the visual. (This may come with security risks, so be careful!)

Restart Kibana and create a new transform visual 

Copy and paste these three

#### Query DSL -- create your own aggregation set, {{ meta.debug }} in teh template section will show you the response you get and you can change the JS accordingly.
```
{
  "query": {
    "bool": {
      "must": [  
        "_DASHBOARD_CONTEXT_"
      ] 
    }  
  },
  
  "size": 0, 
   "aggs": {
    "funnel": {
      "filters": {
        "filters": {
          "filter1": {
            "query_string": {
              "query": "field1:value1",
              "analyze_wildcard": true,
              "default_field": "*"
            }  
          },
          "filter2": {
            "query_string": {
              "query": "field2:value2",
              "analyze_wildcard": true,
              "default_field": "*"
            }
          },
          "filter3": {
            "query_string": {
              "query": "field3:value3",
              "analyze_wildcard": true,
              "default_field": "*"
            }
          },
          "filter4": {
            "query_string": {
              "query": "field4:value4",
              "analyze_wildcard": true,
              "default_field": "*"
            }
          },
          "filter5": {
            "query_string": {
              "query": "field5:value5",
              "analyze_wildcard": true,
              "default_field": "*"
            }
          },
          "filter6": {
            "query_string": {
              "query": "field6:value6",
              "analyze_wildcard": true,
              "default_field": "*"
            }
          }
        }
      }
    }
  }
}
```
#### Javascript

```
({
    // This is to install external script/library
    import_funnelLib: function() {
     $.getScript("https://canvasjs.com/assets/script/jquery.canvasjs.min.js", function( data, textStatus, jqxhr ) {
        console.log("Import Complete");
    }); 
 },
 // This only works in version >6.2.2
  after_render: function() {
     //ES data here
    var esData = this.response.aggregations.funnel.buckets;
    var test1 = esData.field1.doc_count;
    var test2 = esData.field2.doc_count;
    var test3 = esData.field3.doc_count;
    var test4 = esData.field4.doc_count;
    var test5 = esData.field5.doc_count;
    var test6 = esData.field6.doc_count; 

    
    var options = {
	animationEnabled: true,
	theme: "light1", //"light1", "light2", "dark1", "dark2"
	title: {
		text: "Conversion Funnel",
		horizontalAlign: "left",
// 		fontSize: 50
	},
	data: [{
    
		type: "pyramid", // "funnel"
		reversed: true, // "false" if you change to funnel
// 		neckHeight:  "60%",
        neckWidth: "35%",
        valueRepresents: "area",
		toolTipContent: "{label}: {y} ({percentage}%)",
		indexLabel: "{label} ({percentage}%)",
		dataPoints: [
			{ y: test1, label: "label1" },
			{ y: test2, label: "label2" },
			{ y: test3, label: "label3" },
			{ y: test4, label: "label4" },
			{ y: test5, label: "label5" },
			{ y: test6, label: "label6" }
		]
	}]
};
calculatePercentage();
$("#chartContainer").CanvasJSChart(options);
 
function calculatePercentage() {
	var dataPoint = options.data[0].dataPoints;
	var total = dataPoint[0].y;
	for (var i = 0; i < dataPoint.length; i++) {
		if (i === 0) {
			options.data[0].dataPoints[i].percentage = 100;
		} else {
			options.data[0].dataPoints[i].percentage = ((dataPoint[i].y / total) * 100).toFixed(2);
		}
	}
}

},
// This is for doing manipulation of the data and whatever is 
// "return"ed is in meta.data

  data: function() {
  // this is a test only
    var data = this.response.hits.total;
    return data/24 ;
 },


  debug: function() {
  return JSON.stringify(this, null, ' ');
  } 
}) 

```
#### Template
```
<style>
    #chartContainer {
        height:85%;
    }
    
    .canvasjs-chart-credit {
        /*This is to remove the canvas logo so it doesn't get in the way*/
        opacity:0;
    }
</style>
<div id="chartContainer"></div>

<!--This imports the library so that D3 can be used-->
{{meta.import_funnelLib}}


<!--{{ meta.data }}-->



<pre>
    <!-- This is to see the format of your aggregation -- >
    {{ meta.debug }}
</pre>
```

