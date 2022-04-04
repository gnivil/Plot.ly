# Plot.ly Challenge - Belly Button Biodiversity
In this assignment, you will build an interactive dashboard to explore the [Belly Button Biodiversity dataset](http://robdunnlab.com/projects/belly-button-biodiversity/), which catalogs the microbes that colonize human navels.

The dataset reveals that a small handful of microbial species (also called operational taxonomic units, or OTUs, in the study) were present in more than 70% of people, while the rest were relatively rare.

-----

## Step 1: Plot.ly
* Use the D3 library to read in samples.json.
* Create a horizontal bar chart with a dropdown menu to display the top 10 OTUs found in that individual.
* Create a bubble chart that displays each sample.
* Display the sample metadata, i.e., an individual's demographic information.
* Display each key-value pair from the metadata JSON object somewhere on the page.
* Update all of the plots any time that a new sample is selected.

```js
function buildMetadata(sample) {
  d3.json("samples.json").then((data) => {
    var metadata = data.metadata;
    // Filter the data for the object with the desired sample number
    var resultArray = metadata.filter(sampleObj => sampleObj.id == sample);
    var result = resultArray[0];
    // Use d3 to select the panel with id of `#sample-metadata`
    var PANEL = d3.select("#sample-metadata");

    // Use `.html("") to clear any existing metadata
    PANEL.html("");

    // Use `Object.entries` to add each key and value pair to the panel
    // Hint: Inside the loop, you will need to use d3 to append new
    // tags for each key-value in the metadata.
    Object.entries(result).forEach(([key, value]) => {
      PANEL.append("h6").text(`${key.toUpperCase()}: ${value}`);
    });

    // BONUS: Build the Gauge Chart
    buildGauge(result.wfreq);
  });
}

function buildCharts(sample) {
  d3.json("samples.json").then((data) => {
    var samples = data.samples;
    var resultArray = samples.filter(sampleObj => sampleObj.id == sample);
    var result = resultArray[0];

    var otu_ids = result.otu_ids;
    var otu_labels = result.otu_labels;
    var sample_values = result.sample_values;

    // Build a Bubble Chart
    var bubbleLayout = {
      title: "Bacteria Cultures Per Sample",
      margin: { t: 0 },
      hovermode: "closest",
      xaxis: { title: "OTU ID" },
      margin: { t: 30}
    };
    var bubbleData = [
      {
        x: otu_ids,
        y: sample_values,
        text: otu_labels,
        mode: "markers",
        marker: {
          size: sample_values,
          color: otu_ids,
          colorscale: "Earth"
        }
      }
    ];

    Plotly.newPlot("bubble", bubbleData, bubbleLayout);

    var yticks = otu_ids.slice(0, 10).map(otuID => `OTU ${otuID}`).reverse();
    var barData = [
      {
        y: yticks,
        x: sample_values.slice(0, 10).reverse(),
        text: otu_labels.slice(0, 10).reverse(),
        type: "bar",
        orientation: "h",
      }
    ];

    var barLayout = {
      title: "Top 10 Bacteria Cultures Found",
      margin: { t: 30, l: 150 }
    };

    Plotly.newPlot("bar", barData, barLayout);
  });
}

function init() {
  // Grab a reference to the dropdown select element
  var selector = d3.select("#selDataset");

  // Use the list of sample names to populate the select options
  d3.json("samples.json").then((data) => {
    var sampleNames = data.names;

    sampleNames.forEach((sample) => {
      selector
        .append("option")
        .text(sample)
        .property("value", sample);
    });

    // Use the first sample from the list to build the initial plots
    var firstSample = sampleNames[0];
    buildCharts(firstSample);
    buildMetadata(firstSample);
  });
}

function optionChanged(newSample) {
  // Fetch new data each time a new sample is selected
  buildCharts(newSample);
  buildMetadata(newSample);
}

// Initialize the dashboard
init();
```

-----

## Bonus: Gauge Chart
* Adapt the Gauge Chart from [alt text](https://plot.ly/javascript/gauge-charts/) to plot the weekly washing frequency of the individual.
* Modify the example gauge code to account for values ranging from 0 through 9.
* Update the chart whenever a new sample is selected.

```js
function buildGauge(wfreq) {
  // Enter the washing frequency between 0 and 180
  var level = parseFloat(wfreq) * 20;

  var data = [
    {
      domain: { x: [0, 1], y: [0, 1]},
      value: wfreq,
      title: { text: "<b>Belly Button Washing Frequency</b> <br><span style='font-size:0.8em;color:gray'>Scrubs per Week</span>" },
      type: "indicator",
      mode: "gauge",
      gauge: {
        axis: { range: [null, 9], tickwidth: 2, tickmode: "linear" },
        bar: { color: "rbga(0, 128, 0, .8)" },
        steps:
          [
            { range: [0, 1], color: "rgba(0, 128, 128, .05 )" },
            { range: [1, 2], color: "rgba(0, 128, 128, .1)" },
            { range: [2, 3], color: "rgba(0, 128, 128, .2)" },
            { range: [3, 4], color: "rgba(0, 128, 128, .3)" },
            { range: [4, 5], color: "rgba(0, 128, 128, .4)" },
            { range: [5, 6], color: "rgba(0, 128, 128, .5)" },
            { range: [6, 7], color: "rgba(0, 128, 128, .6)" },
            { range: [7, 8], color: "rgba(0, 128, 128, .7)" },
            { range: [8, 9], color: "rgba(0, 128, 128, .8)" },
          ],
        threshold: {
          line: { color: "purple", width: 7 },
          thickness: .75,
          value: wfreq
        }
      }
    }
  ];

  var layout = { width: 500, height: 500, margin: { t: 0, b: 0 } };
  var GAUGE = document.getElementById("gauge");
  Plotly.newPlot(GAUGE, data, layout);
}
```
-----