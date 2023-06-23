---
layout: post
title: "Interactive MOSFET model"
categories: misc
excerpt_separator: <!--more-->
---

Here's another quick project: Below are interactive n-channel MOSFET current-voltage characteristic curves.
These are based on the Shichman-Hodges model, (also known as square-law model or SPICE level 1 model),
although the body effect isn't included here yet.

Move the sliders to interact!

<head>
    <style>
    #chartContainer2 {
        // width: 40%; /* Adjust the width to 66% */
        margin: 20px auto; /* Center the chart container */
        width: 49%;
        min-width: 300px;
        display: inline-block;
        // text-align:center;
    }

    .slider-container2 {
      margin-top: 20px;
      text-align: center; /* Center align the slider div */
    }

    .slider-label2 {
      display: inline-block;
      width: 220px;
      text-align: right; /* Right-align the slider labels */
    }

    .slider-value {
      display: inline-block;
      width: 150px;
      text-align: left; /* Left-align the slider values */
    }

    </style>
</head>
<body>
    <div id="chartContainer2">
        <canvas id="mosfetCanvas" width="200" height="200"></canvas>
    </div>
    <div id="chartContainer2">
        <canvas id="mosfetCanvas2" width="200" height="200"></canvas>
    </div>

    <div class="slider-container2">
        <label class="slider-label2">Gate-Source Voltage \(\nu_{DS}\) (V):</label>
        <input type="range" id="gateSourceVoltageSlider" min="0" max="2" step="0.05" value="1" class="slider">
        <span id="gateSourceVoltageValue" class="slider-value">1</span>
    </div>
    <div class="slider-container2">
        <label class="slider-label2">Threshold Voltage \(V_{t}\) (V):</label>
        <input type="range" id="thresholdVoltageSlider" min="0.3" max="1" step="0.05" value="0.5" class="slider">
        <span id="thresholdVoltageValue" class="slider-value">0.5</span>
    </div>
    <div class="slider-container2">
        <label class="slider-label2">eff. Channel Length \(L_\mathrm{eff}\) (nm):</label>
        <input type="range" id="channelLengthSlider" min="100" max="5000" step="10" value="180" class="slider">
        <span id="channelLengthValue" class="slider-value">180</span>
    </div>
    <div class="slider-container2">
        <label class="slider-label2">eff. Channel Width \(W_\mathrm{eff}\) (nm):</label>
        <input type="range" id="channelWidthSlider" min="100" max="15000" step="10" value="220" class="slider">
        <span id="channelWidthValue" class="slider-value">220</span>
    </div>
    <div class="slider-container2">
        <label class="slider-label2">Oxide Thickness \(t_\mathrm{ox}\) (nm):</label>
        <input type="range" id="oxideThicknessSlider" min="1" max="100" step="0.1" value="4.1" class="slider">
        <span id="oxideThicknessValue" class="slider-value">4.1</span>
    </div>
    <div class="slider-container2">
        <label class="slider-label2">Mobility \(\mu\) (cm²/Vs):</label>
        <input type="range" id="mobilitySlider" min="100" max="1000" step="10" value="290" class="slider">
        <span id="mobilityValue" class="slider-value">290</span>
    </div>
    <div class="slider-container2">
        <label class="slider-label2">Channel-Length modulation coeff. \(\lambda\) (1/V):</label>
        <input type="range" id="channelLengthModulationSlider" min="0.01" max="0.2" step="0.01" value="0.11" class="slider">
        <span id="channelLengthModulationValue" class="slider-value">0.11</span>
    </div>


    <script src="https://cdn.jsdelivr.net/npm/chart.js@3.5.1"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-zoom@1.1.1"></script>
    <script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
    <script id="MathJax-script" async
            src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
    </script>

    <script>
        // Simulation parameters
        var numPoints = 100;

        // MOSFET parameters
        var gateSourceVoltage = 1; // Gate-Source voltage (V)
        var thresholdVoltage = 0.5; // Threshold voltage (V)
        var channelLength = 180; // Channel length (μm)
        var channelWidth = 220; // Channel width (μm)
        var oxideThickness = 4.1; // Oxide thickness (μm)
        var mobility = 290; // Mobility (cm²/Vs)
        var channelLengthModulation = 0.11; // Channel-Length Modulation Coefficient (1/V)

        // Constants
        var permittivity = 3.45 * 1e-11 // F/m


        // Calculate the saturation region line
        function calculateSaturationRegion(drainSourceVoltage) {
            // var drainSourceVoltageTerm = drainSourceVoltage - thresholdVoltage;
            var effectiveChannelLength = channelLength * 1e-9; // Convert channel length from μm to m
            var effectiveChannelWidth = channelWidth * 1e-9; // Convert channel width from μm to m
            var effectiveOxideThickness = oxideThickness * 1e-9;

            var oxideCapacitanceTerm = permittivity / effectiveOxideThickness; // Convert oxide capacitance from F/m² to F/m²
            var overdriveVoltage = gateSourceVoltage - thresholdVoltage;
            var transconductance = mobility * oxideCapacitanceTerm *
                effectiveChannelWidth / effectiveChannelLength

            var drainCurrent = 0;
            if (gateSourceVoltage > thresholdVoltage){
              var drainCurrent = 0.5 * transconductance * Math.pow(drainSourceVoltage, 2)
              }
            return drainCurrent;
        }

        // Calculate drain current vs drain-source voltage
        function calculateDrainCurrent(drainSourceVoltage) {
            // var drainSourceVoltageTerm = drainSourceVoltage - thresholdVoltage;
            var effectiveChannelLength = channelLength * 1e-9; // Convert channel length from μm to m
            var effectiveChannelWidth = channelWidth * 1e-9; // Convert channel width from μm to m
            var effectiveOxideThickness = oxideThickness * 1e-9;

            var oxideCapacitanceTerm = permittivity / effectiveOxideThickness; // Convert oxide capacitance from F/m² to F/m²
            var overdriveVoltage = gateSourceVoltage - thresholdVoltage;
            var transconductance = mobility * oxideCapacitanceTerm *
                effectiveChannelWidth / effectiveChannelLength

            var drainCurrent = 0;
            if (gateSourceVoltage > thresholdVoltage){
              if (drainSourceVoltage < overdriveVoltage) {
                var drainCurrent = transconductance *
                    (
                      overdriveVoltage * drainSourceVoltage -
                      0.5 * Math.pow(drainSourceVoltage, 2)
                    )
                    ;
              } else {
                var drainCurrent = 0.5 * transconductance *
                    Math.pow(overdriveVoltage, 2) * (1 + channelLengthModulation * (drainSourceVoltage - overdriveVoltage))
              }
            }
            return drainCurrent;
        }

        // Calculate drain current vs gate-source voltage
        function calculateDrainCurrent2(xgateSourceVoltage) {
            var gateSourceVoltageTerm = xgateSourceVoltage - thresholdVoltage;
            var effectiveChannelLength = channelLength * 1e-9; // Convert channel length from μm to m
            var effectiveChannelWidth = channelWidth * 1e-9; // Convert channel width from μm to m
            var effectiveOxideThickness = oxideThickness * 1e-9;

            var oxideCapacitanceTerm = permittivity / effectiveOxideThickness; // Convert oxide capacitance from F/m² to F/m²


            var drainCurrent = 0;
            if (gateSourceVoltageTerm >= 0) {
              var drainCurrent = 0.5 * mobility * oxideCapacitanceTerm *
                  effectiveChannelWidth / effectiveChannelLength *
                  gateSourceVoltageTerm * gateSourceVoltageTerm;
            }
            return drainCurrent;
        }



        // Render MOSFET IV curve
        function renderMosfetIVCurve() {
            var canvas = document.getElementById("mosfetCanvas");

            // Generate voltage data
            var voltages = [];
            for (var i = 0; i <= numPoints; i++) {
                var voltage = i / numPoints * 2;
                voltages.push(voltage);
            }

            // Generate current data
            var currents = voltages.map(calculateDrainCurrent);
            var saturationCurrents = voltages.map(calculateSaturationRegion);

            // Get the chart instance
            var chart = window.mosfetChart;

            // Update the chart data
            chart.data.labels = voltages;
            chart.data.datasets[0].data = currents;
            chart.data.datasets[1].data = saturationCurrents;

            // Update the chart
            chart.update();
        }

        function renderMosfetIVCurve2() {
            var canvas = document.getElementById("mosfetCanvas2");

            // Generate voltage data
            var voltages = [];
            for (var i = 0; i <= numPoints; i++) {
                var voltage = i / numPoints * 2;
                voltages.push(voltage);
            }

            var voltagepoint = [gateSourceVoltage]

            // Generate current data
            var currents = voltages.map(calculateDrainCurrent2);
            var currentpoint = voltagepoint.map(calculateDrainCurrent2);

            // Get the chart instance
            var chart = window.mosfetChart2;

            // Update the chart data
            chart.data.labels = voltages;
            chart.data.datasets[1].data = currents;
            chart.data.datasets[0].data = [{x: voltagepoint, y: currentpoint}]

            // Update the chart
            chart.update();
        }

        // Update threshold voltage value and channel dimensions
        function updateThresholdVoltageValue() {
            var gateSourceVoltageSlider = document.getElementById("gateSourceVoltageSlider");
            var gateSourceVoltageValue = document.getElementById("gateSourceVoltageValue");
            gateSourceVoltage = parseFloat(gateSourceVoltageSlider.value);
            gateSourceVoltageValue.textContent = gateSourceVoltage;

            var thresholdVoltageSlider = document.getElementById("thresholdVoltageSlider");
            var thresholdVoltageValue = document.getElementById("thresholdVoltageValue");
            thresholdVoltage = parseFloat(thresholdVoltageSlider.value);
            thresholdVoltageValue.textContent = thresholdVoltage;

            var channelLengthSlider = document.getElementById("channelLengthSlider");
            var channelLengthValue = document.getElementById("channelLengthValue");
            channelLength = parseFloat(channelLengthSlider.value);
            channelLengthValue.textContent = channelLength;

            var channelWidthSlider = document.getElementById("channelWidthSlider");
            var channelWidthValue = document.getElementById("channelWidthValue");
            channelWidth = parseFloat(channelWidthSlider.value);
            channelWidthValue.textContent = channelWidth;

            var oxideThicknessSlider = document.getElementById("oxideThicknessSlider");
            var oxideThicknessValue = document.getElementById("oxideThicknessValue");
            oxideThickness = parseFloat(oxideThicknessSlider.value);
            oxideThicknessValue.textContent = oxideThickness;

            var mobilitySlider = document.getElementById("mobilitySlider");
            var mobilityValue = document.getElementById("mobilityValue");
            mobility = parseFloat(mobilitySlider.value);
            mobilityValue.textContent = mobility;

            var channelLengthModulationSlider = document.getElementById("channelLengthModulationSlider");
            var channelLengthModulationValue = document.getElementById("channelLengthModulationValue");
            channelLengthModulation = parseFloat(channelLengthModulationSlider.value);
            channelLengthModulationValue.textContent = channelLengthModulation;

            // Update the chart
            renderMosfetIVCurve();
            renderMosfetIVCurve2();
        }

        // Add event listeners to sliders
        var gateSourceVoltageSlider = document.getElementById("gateSourceVoltageSlider");
        gateSourceVoltageSlider.addEventListener("input", updateThresholdVoltageValue);

        var thresholdVoltageSlider = document.getElementById("thresholdVoltageSlider");
        thresholdVoltageSlider.addEventListener("input", updateThresholdVoltageValue);

        var channelLengthSlider = document.getElementById("channelLengthSlider");
        channelLengthSlider.addEventListener("input", updateThresholdVoltageValue);

        var channelWidthSlider = document.getElementById("channelWidthSlider");
        channelWidthSlider.addEventListener("input", updateThresholdVoltageValue);

        var oxideThicknessSlider = document.getElementById("oxideThicknessSlider");
        oxideThicknessSlider.addEventListener("input", updateThresholdVoltageValue);

        var mobilitySlider = document.getElementById("mobilitySlider");
        mobilitySlider.addEventListener("input", updateThresholdVoltageValue);

        var channelLengthModulationSlider = document.getElementById("channelLengthModulationSlider");
        channelLengthModulationSlider.addEventListener("input", updateThresholdVoltageValue);

        // Initialize the chart
        var canvas = document.getElementById("mosfetCanvas");
        var ctx = canvas.getContext("2d");
        window.mosfetChart = new Chart(ctx, {
          type: "line",
          data: {
              labels: [], // Empty labels initially
              datasets: [{
                  label: "Drain Current",
                  data: [], // Empty data initially
                  borderColor: "blue",
                  borderWidth: 2,
                  fill: false,
                  pointStyle: "circle", // Set point style to line
                  pointRadius: 0, // Set point radius to 0
                  pointHoverRadius: 15,
              },
              {
                  label: "Saturation Region",
                  data: [], // Empty data initially
                  borderColor: 'rgba(200, 200, 200, 0.5)',
                  backgroundColor: 'rgba(200, 200, 200, 0.2)',
                  borderDash: [5, 5],
                  borderWidth: 1,
                  fill: true,
                  pointStyle: "circle", // Set point style to line
                  pointRadius: 0, // Set point radius to 0
                  pointHoverRadius: 15,
              },
            ]
          },
          options: {
              animation: {
                duration: 250,
                easing: "easeOutQuint",
              },
              scales: {
                  x: {
                      type: "linear",
                      title: {
                          display: true,
                          text: 'Drain-Source Voltage (V)',
                          font: {
                            size: 14
                          }
                      },
                      max: 2, // Set maximum value for y-axis
                      min: 0 // Set minimum value for y-axis
                  },
                  y: {
                      type: "linear",
                      ticks: {
                          callback: function(value, index, values) {
                            return value;
                          },
                      },
                      title: {
                          display: true,
                          text: "Drain Current (A)",
                          rotation: 0,
                          position: "left",
                          font: {
                            size: 14
                          }
                      },
                      max: 1, // Set maximum value for y-axis
                      min: 0 // Set minimum value for y-axis
                  }
              },
              plugins: {
                legend: {
                  display: false // Set display to false to hide the legend
                },
                title: {
                  display: true,
                  text: 'Drain Current vs Drain-Source Voltage',
                  font: {
                    size: 16,
                    family: 'Arial',
                    weight: 'bold'
                  },
                  padding: {
                    top: 10,
                    bottom: 20
                  }
                },
                legend: {
                  display: false
                }
              }
          }
      });

      // Initialize chart 2
      var canvas = document.getElementById("mosfetCanvas2");
      var ctx = canvas.getContext("2d");
      window.mosfetChart2 = new Chart(ctx, {
        type: "line",
        data: {
            labels: [], // Empty labels initially
            datasets: [
              {
                label: "Current at selected Gate Voltage",
                data: [], // Empty data initially
                borderColor: "blue",
                borderWidth: 2,
                fill: true,
                pointStyle: "circle", // Set point style to line
                pointRadius: 3, // Set point radius to 0
                pointHoverRadius: 15,
            },
            {
                label: "Drain Current",
                data: [], // Empty data initially
                borderColor: "red",
                borderWidth: 2,
                fill: false,
                pointStyle: "circle", // Set point style to line
                pointRadius: 0, // Set point radius to 0
                pointHoverRadius: 15,
            }
          ]
        },
        options: {
            animation: {
              duration: 250,
              easing: "easeOutQuint",
            },
            scales: {
                x: {
                    type: "linear",
                    title: {
                        display: true,
                        text: 'Gate-Source Voltage (V)',
                        font: {
                          size: 14
                        }
                    },
                    max: 2, // Set maximum value for y-axis
                    min: 0 // Set minimum value for y-axis
                },
                y: {
                    type: "linear",
                    ticks: {
                        callback: function(value, index, values) {
                          return value;
                        },
                    },
                    title: {
                        display: true,
                        text: "Drain Current (A)",
                        rotation: 0,
                        position: "left",
                        font: {
                          size: 14
                        }
                    },
                    max: 1, // Set maximum value for y-axis
                    min: 0 // Set minimum value for y-axis
                }
            },
            plugins: {
              legend: {
                display: false // Set display to false to hide the legend
              },
              title: {
                display: true,
                text: 'Drain Current at Saturation',
                font: {
                  size: 16,
                  family: 'Arial',
                  weight: 'bold'
                },
                padding: {
                  top: 10,
                  bottom: 20
                }
              },
              legend: {
                display: false
              }
            }
        }
    });

        // Render the MOSFET IV curve initially
        renderMosfetIVCurve();
        renderMosfetIVCurve2();

    </script>
</body>
<br>

<!--more-->

The calculations follow those in Chapter 5 of Sedra and Smith, *Microelectronic Circuits* (2020). The grey-shaded area in the left-hand plot marks the saturation region.

Feel free to email me if you have any feedback on this!
