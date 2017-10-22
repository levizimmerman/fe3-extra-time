# Extra Time - Learning New Things
A description of the time I spent learning new things about D3 or Javascript.

## Table of Contents
Subject | Resources | Time
--- | --- | ---
[TopoJSON][sectionTopoJSON] | [eu.topojson resource][euJSON], [example][exampleTopoJSON], [`.geoPath()` docs][geoPathDocs] | 6h
[CSV Hierarchy with D3][sectionCsvHierarchyWithD3] | [`.hierarchy()` docs][hierarchyDocs] | 2h
[D3 Nesting][sectionD3Nesting] | [example][nestExample], [docs][nestDocs] | 5h
[Cleaning and Transforming Data (Advanced)][sectionCleaningAndTransformingData] | [XML parsing][xmlParsing], [My knowledge about JS and logic][leviLinkedIn] | 7h
[D3 Dispatch][sectionD3Dispatch] | [`.dispatch()` docs][dispatchDocs], [Pub-sub pattern definition][pubSubPattern] | 2h

## TopoJSON (6h)
For the [first assessment][firstAssessment] I have created a map of Europe displaying indoor toilets per country per year. To display the data on a map I had to figure out how to draw countries in SVG. I have used multiple examples ([example 1][euJSON], [example 2][exampleTopoJSON], [exmaple 3][zoomExampleMap]) to learn from. One of the biggest challenges was translating D3 V3 functions to D3 V4 functions. Also figuring out how each country is drawn from a TopoJSON file was interesting to learn.

This is how constructing the map looks like in Javascript:
```javascript
d3.json(topoFile, function(err, data) {
      if (err) {
        throw err;
      }
      countries.selectAll('.country')
        .data(topojson.feature(data, data.objects.europe).features)
        .enter().append('path')
        .attr('class', 'country')
        .attr('d', path)
        .attr('fill', disabledColor)
        .attr('id', getCountryNameFromFeature)
        .attr('data-country-name', getCountryNameFromFeature)
        // listeners
        .on('click', clickedCountry)
        .on('mousemove', showCountryName)
        .on('mouseleave', function(data, index) {
          countryLabel.classed('hidden', true); // toggle hidden class
        });

      Events.emit('map/done', {
        dataFile: dataFile,
        year: year
      });
      return;
    });
```

In this example I have used [topoJSON][topoJSONResource] feature of D3 to extract path coordinates pointing the `.feature()` to `data.objects.europe` object. Then foreach country object a path will be drawn and attributes will be added to identify each country using `data-country-name` as link. This link makes it possible to call each country and change it path options based on user interaction.

## CSV Hierarchy with D3 (2h)
For the multiple exercises ([class-3-transition][class3Transition] and [class-4-interactivity][class4Interactivity]) I have used hierarchical data from this [example][flareExample]. The goal was creating and understanding clustered data. That means data that have a parent in common. In this way I could color, transform, filter or sort elements that share the same level of depth within the hierarchy.

The following is an example of the CSV data that can be made hierarchical in D3:
```csv
id,value
flare,
flare.analytics,
flare.analytics.cluster,
flare.analytics.cluster.AgglomerativeCluster,3938
flare.analytics.cluster.CommunityStructure,3812
flare.analytics.cluster.HierarchicalCluster,6714
flare.analytics.cluster.MergeEdge,743
flare.analytics.graph,
flare.analytics.graph.BetweennessCentrality,3534
flare.analytics.graph.LinkDistance,5731
```

The level of depth is signified with a dot (`.`). In this example you can see that `analytics` is a child of `flare` and `cluster` is a child of `analytics` etc. Within D3 I have used [`.hierarchy()`][hierarchyDocs] to create a data object that holds information about each element, e.g.:

```json
"class": "Axis",
"data": {
   "id": "flare.vis.axis.Axis",
   "value": 24593
},
"depth": 1,
"height": 0,
"id": "flare.vis.axis.Axis",
"package": "flare.vis.axis",
"parent": {
  "data": {},
  "height": 1,
  "depth": 0,
  "parent": null,
  "children": []
}
"r": 32.26686879996167,
"value": 24593,
"x": 885.4027311176321,
"y": 236.31475240847757
```

As you can see this data object has a property called `parent` this is reference to its parent. Every data element has this reference. In this way it is possible to understand the relationship between the different data elements. With the D3 [`.pack()`][packDocs] function it is possible to create a circle pack layout with the data object I just created using [`.hierarchy()`][hierarchyDocs]. The bubble chart is a stone's throw away after creating the pack layout.

## D3 Nesting (5h)
For the [last assessment][thirdAssessment] I used a dataset from the Health App of iOS. This dataset is XML formatted and has data records about step count, flights climbed, distance walked or ran, and sleep rythm. Every record is a measurement over a time period of 5 minutes, sleep rythm excluded. This means that every day multiple data entries can be available. Eventually I would display the activity data (step count, flights climbed, and distance walked or ran) in relation to the sleep rythm data. Because the sleep rythm is measured per day, I wanted to merge every activity data entry to one data entry per day. To do that I used the [`.nest()`][nestDocs] function.

To merge the data per day, you can assign an identifier for each day. Using the `key` function we can use `startDate` of each data entry and transform it to datestring (e.g. `2017-04-01`). In this way each entry, regardless time of measurement, is merged to that same day. After the [`.key()`][d3Keys] function the [`.rollup()`][d3Rollup] function is chained to it. The [`.rollup()`][d3Rollup] function 'rolls up' all data entries connected to the predefined key (e.g. `2017-04-01`). I have used the [`.sum()`][d3Sum] function to calculate the total measured value of each day. In the end the [`.entries()`][d3Entries] functions is chained. This function accepts an array of all data entries needed to be nested.

The `.save()` function is a custom function that saves the original merged data. Each saved entry can be used for filtering.

```javascript
self.mergeDataPerDay = function(data, type) {
    var dataPerDay = d3.nest()
      .key(function(data) {
        return data.startDate.toLocaleDateString();
      })
      .rollup(function(data) {
        return d3.sum(data, function(group) {
          return group.value;
        });
      })
      .entries(data);
      save(dataPerDay, type);
      return dataPerDay;
  };
```

Furthermore, the sleep rythm data is not completely measured per day. For example: I start my sleep at 11.30 p.m. and wake up at 7.30 a.m. The start of the sleep and waking are on separate dates. So the sleep rythm data also needed to be merged to one day. To merge the data I used a similar code flow as for the activity data. Only in this case I did not need a value per day, but a starting time of sleep, end time and time slept. So within the [`.rollup()`][d3Rollup] function I created a data object that has all this information. But to ensure that the starting time of sleep is correctly interpreted I added a `minThreshold` parameter to the function. This is a range of hours (24 hour clock) I could go to sleep (e.g. `[21, 3]`). The same counts for `maxThreshold` parameter. Only this regulates time waking up (e.g. `[5, 11]`). Before the actual data object is created and returned there is a check if the start and endtime or within the accepted spread of hours.

```javascript
self.createSleepCyclePerDay = function(data, minThreshold, maxThreshold) {
    var minSpread = getMinHourSpread(minThreshold);
    var maxSpread = getMaxHourSpread(maxThreshold);
    var sleepCyclePerDay = d3.nest()
      .key(function(data) {
        return data.startDate.toLocaleDateString();
      })
      .rollup(function(data) {
        return data.map(function(entry) {
          var startTime = entry.startDate.getTime();
          var endTime = entry.endDate.getTime();
          var diffTime = endTime - startTime;
          var startHours = entry.startDate.getHours();
          var endHours = entry.endDate.getHours();
          if (minSpread.indexOf(startHours) !== -1 && maxSpread.indexOf(endHours) !== -1) {
            return {
              start: entry.startDate,
              end: entry.endDate,
              slept: Math.round(diffTime / 36000) / 100 // 2 Decimal rounding: https://stackoverflow.com/questions/11832914/round-to-at-most-2-decimal-places-only-if-necessary
            };
          }
        }).filter(function(entry) {
          return entry !== undefined;
        });
      })
      .entries(data).filter(function(entry) {
        return entry.value.length > 0;
      });
      save(sleepCyclePerDay, 'sleepCycle');
      return sleepCyclePerDay;
  };
```

## Cleaning and Transforming Data (Advanced) (7h)
So in the previous section I talked about D3 Nest function. This was a big part of transforming the data to a data object I could use to create a data visualization. Throughout the [FE 3 data course][fe3DataCourse] it was needed to clean and transform different kinds of data sets. I will layout some code snippets I have written to achieve a dynamic way of parsing the handed dataset.

For [class 3 clean][class3Clean] I used a dataset from the KNMI. The data set holds information about the weather for a selected date at the selected places (De Bilt, Huibertgat, De Kooy, etc.). To connect the data rows to the different kinds of places a hardcoded array of places was found in the example code. But instead of using the array I wanted to extract the places from the given dataset. In case any place name or STN (station number) might be changed or added in the future.

This is a slice of the text data found in the downloaded data set that holds all station information:
```
# STN      LON(east)   LAT(north)     ALT(m)  NAME
# 235:         4.781       52.928       1.20  DE KOOY
# 260:         5.180       52.100       1.90  DE BILT
# 279:         6.574       52.750      15.80  HOOGEVEEN
# 285:         6.399       53.575       0.00  HUIBERTGAT
```

First I sliced the header from the whole data set. Then I sliced the header from the index of `NAME` and added `4` to the index for excluding `NAME` itself from the slice. After that I replaced all `:` and `#` characters from the string and eventually created an array based on `\n` (newlines). Using a foreach loop, I could clean each row of data. I replaced all spaces if two or more in a row are present with a comma. After cleaning each row I created a new string where each row ends in a `\n`. This is needed to successfully parse the rows using the [`d3.csvParseRows()`][d3parseCsvRows].
```javascript
function getKeysFromDoc(doc) {
  var header = doc.slice(0, doc.indexOf('STN,YYYYMMDD')); // Remove text before legend
  var headerRows = header.slice(header.indexOf('NAME') + 4, header.indexOf('# YYYYMMDD')) // remove head of identifiers
    .replace(/#|:/g, '') // replace all # and : characters with nothing
    .trim() // trim start and end of string
    .split('\n'); // define rows by newline recognition
  var newHeaderRows = []; // create return array
  headerRows.forEach(function(row) {
    newHeaderRows.push(row.replace(/\s+ /g, ',').trim()); // fill array with space stripped string and replaced by commas
  });
  return newHeaderRows.join('\n'); // create new string with join and add new line for each row
}
```

Another example of cleaning and transforming the data, which was not among the course examples, I used within the [third assessment][thirdAssessment]. I had a huge XML file (18mb+) with all health data of the past two years.

A slice of data of that XML file:
```xml
<Record type="HKQuantityTypeIdentifierStepCount" sourceName="iPhone van Levi" sourceVersion="9.2" device="&lt;&lt;HKDevice: 0x174484920&gt;, name:iPhone, manufacturer:Apple, model:iPhone, hardware:iPhone8,1, software:9.2&gt;" unit="count" creationDate="2016-01-13 19:57:52 +0200" startDate="2016-01-13 19:08:24 +0200" endDate="2016-01-13 19:09:26 +0200" value="31"/>
<Record type="HKQuantityTypeIdentifierStepCount" sourceName="iPhone van Levi" sourceVersion="9.2" device="&lt;&lt;HKDevice: 0x174486fe0&gt;, name:iPhone, manufacturer:Apple, model:iPhone, hardware:iPhone8,1, software:9.2&gt;" unit="count" creationDate="2016-01-13 19:57:52 +0200" startDate="2016-01-13 19:09:26 +0200" endDate="2016-01-13 19:10:26 +0200" value="84"/>
<Record type="HKQuantityTypeIdentifierStepCount" sourceName="iPhone van Levi" sourceVersion="9.2" device="&lt;&lt;HKDevice: 0x174486e00&gt;, name:iPhone, manufacturer:Apple, model:iPhone, hardware:iPhone8,1, software:9.2&gt;" unit="count" creationDate="2016-01-13 19:57:52 +0200" startDate="2016-01-13 19:10:26 +0200" endDate="2016-01-13 19:11:41 +0200" value="21"/>
```

After loading the XML file I start a map function on an empty array and using a DOM selector function to create an array of data. Of every `<Record>` passed to the map function I get the following attributes: `type`, `value`, `startDate`, `endDate` and `creationDate`.
```javascript
function mapData(error, data) {
  if (error) {
    throw error;
  }

  data = [].map.call(data.querySelectorAll('Record'), function(record) {
    return {
      type: parseType(record.getAttribute('type')),
      value: parseValue(record.getAttribute('value')),
      startDate: parseTime(record.getAttribute('startDate')),
      endDate: parseTime(record.getAttribute('endDate')),
      creationDate: parseTime(record.getAttribute('creationDate'))
    }
  });

  Events.emit('data/map/done', {
    data: data
  });
}
```

Every attribute that holds a date is consistent with following format `%Y-%m-%d %H:%M:%S %Z`. For example a start date could be `2017-04-01 12:01:52 +0200`. These date strings are parsed with [`d3.timeParse()`][d3TimeParse], which returns a date object. This comes in handy when drawing data onto a timescale.
```javascript
var parseTime = d3.timeParse('%Y-%m-%d %H:%M:%S %Z');
```

This function parses all values to a number format. If a value cannot be parsed to a number, null will be returned. This is prevents weird bugs later in the development because now we know that value can be a number or `null`.
```javascript
function parseValue(value) {
  return isNaN(Number(value)) ? null : Number(value);
}
```

Every type attribute has a prefix: 'HKQuantityTypeIdentifier' or 'HKCategoryTypeIdentifier'. Hence the `OR` operator within the [`.replace()`][jsReplace] function. Also this functions pushes the tyoe to a global `types` array. In this way you could log which types of records are transformed.
```javascript
function parseType(type) {
  type = type.replace(/HKQuantityTypeIdentifier|HKCategoryTypeIdentifier/g, '');
  if (types.indexOf(type) === -1) {
    types.push(type)
  }
  return type;
}
```

## D3 Dispatch (2h)
Last but not least the [`d3.dispatch()`][d3Dispatch] function. This may be misleading but I did not use the D3 dispatch function within any of my code. Instead I used a custom library that enables a [pub-sub code pattern][pubsubPattern] while developing the assigments and assessments.

It is a simple library that looks like this:
```javascript
var Events = {
    events: {},
    on: function (eventName, fn) {
        // Early exit
        if (!eventName || !fn) {
            return;
        }
        this.events[eventName] = this.events[eventName] || [];
        this.events[eventName].push(fn);
    },
    off: function (eventName, fn) {
        // Early exit
        if (!eventName || !fn) {
            return;
        }
        if (this.events[eventName]) {
            for (var i = 0; i < this.events[eventName].length; i++) {
                if (this.events[eventName][i] === fn) {
                    this.events[eventName].splice(i, 1);
                    break;
                }
            }
        }
    },
    emit: function (eventName, data) {
        // Early exit
        if (!eventName) {
            return;
        }
        if (this.events[eventName]) {
            this.events[eventName].forEach(function (fn) {
                fn(data);
            });
        }
    }
};
```

I have used this pattern for multiple reasons:
1. It ensures that my application is scaleable. I could write new functions that simply 'hooks' on emitted events.
2. It prevents [spaghetti code][spaghetti]. You will not be caught in a crossfire of function calls.
3. It creates a loggable overview of all possible events within your application and which function calls are connected to which event.

I used this library because I was not familiar with the [`d3.dispatch()`][d3Dispatch] function already built in D3. I have stumbed upon this when [@wooorm][wooorm] mentioned it on the 'frontend-3' Slack channel. That is when I have decided to read up on this. I will probably try to implement it within the resit of assessment three.

[nestExample]: http://www.d3noob.org/2014/02/grouping-and-summing-data-using-d3nest.html
[nestDocs]: https://github.com/d3/d3-collection/blob/master/README.md#nest
[filterDocs]: https://developer.mozilla.org/nl/docs/Web/JavaScript/Reference/Global_Objects/Array/filter
[mapDocs]: https://developer.mozilla.org/nl/docs/Web/JavaScript/Reference/Global_Objects/Array/map
[euJSON]: http://bl.ocks.org/milafrerichs/69035da4707ea51886eb
[exampleTopoJSON]: https://gist.github.com/milafrerichs/69035da4707ea51886eb
[geoPathDocs]: https://github.com/d3/d3-geo/blob/master/README.md#geoPath
[dispatchDocs]: https://github.com/d3/d3-dispatch/blob/master/README.md#dispatch
[pubsubPattern]: https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern
[hierarchyDocs]: https://github.com/d3/d3-hierarchy/blob/master/README.md#hierarchy
[xmlParsing]: https://bl.ocks.org/mbostock/ec585e034819c06f5c99
[leviLinkedIn]: https://www.linkedin.com/in/levi-zimmerman-8772242b/
[firstAssessment]: https://github.com/levizimmerman/fe3-assessment-1
[zoomExampleMap]: https://bl.ocks.org/mbostock/2206590
[topoJSONResource]: https://d3js.org/topojson.v1.min.js
[class3Transition]: https://github.com/cmda-fe3/course-17-18/tree/master/site/class-3-transition/levizimmerman
[class4Interactivity]: https://github.com/cmda-fe3/course-17-18/tree/master/site/class-4-interaction/levizimmerman
[flareExample]: https://bl.ocks.org/mbostock/4063269
[packDocs]: https://github.com/d3/d3-hierarchy/blob/master/README.md#pack
[thirdAssessment]: https://github.com/levizimmerman/fe3-assessment-3
[d3Sum]: https://github.com/d3/d3-array/blob/master/README.md#sum
[d3Keys]: https://github.com/d3/d3-collection/blob/master/README.md#nest_key
[d3Entries]: https://github.com/d3/d3-collection/blob/master/README.md#nest_entries
[d3Rollup]: https://github.com/d3/d3-collection/blob/master/README.md#nest_rollup
[fe3DataCourse]: https://github.com/cmda-fe3/fe3-assessment-3
[class3Clean]: https://github.com/cmda-fe3/course-17-18/tree/master/site/class-3-clean/levizimmerman
[d3parseCsvRows]: https://github.com/d3/d3-dsv/blob/master/README.md#csvParseRows
[d3TimeParse]: https://github.com/d3/d3-time-format/blob/master/README.md#timeParse
[jsReplace]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace
[d3Dispatch]: https://github.com/d3/d3-dispatch/blob/master/README.md#dispatch
[spaghetti]: https://media.giphy.com/media/nOC6xDYlLXd3q/giphy.gif
[wooorm]: https://github.com/wooorm
[sectionTopoJSON]: https://github.com/levizimmerman/fe3-extra-time/blob/master/readme.md#topojson-6h
[sectionCsvHierarchyWithD3]: https://github.com/levizimmerman/fe3-extra-time/blob/master/readme.md#csv-hierarchy-with-d3-2h
[sectionD3Nesting]: https://github.com/levizimmerman/fe3-extra-time/blob/master/readme.md#d3-nesting-5h
[sectionCleaningAndTransformingData]: https://github.com/levizimmerman/fe3-extra-time/blob/master/readme.md#cleaning-and-transforming-data-advanced-7h
[sectionD3Dispatch]: https://github.com/levizimmerman/fe3-extra-time/blob/master/readme.md#d3-dispatch-2h
