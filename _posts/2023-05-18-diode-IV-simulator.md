---
layout: post
title: "Diode IV curve simulator"
categories: misc
excerpt_separator: <!--more-->
---

Here's a quick project, creating an interactive IV curve simulator. Move the sliders to try it out!
<head>
    <style>
    #chartContainer {
        width: 80%; /* Adjust the width to 66% */
        margin: 20px auto; /* Center the chart container */
    }

    .slider-container {
      margin-top: 20px;
      text-align: center; /* Center align the slider div */
    }

    .slider-label {
      display: inline-block;
      width: 200px;
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
    <div id="chartContainer">
        <canvas id="diodeCanvas" width="400" height="400"></canvas>
    </div>

    <div class="slider-container">
        <label class="slider-label">Saturation current \(I_S\) (A):</label>
        <input type="range" id="saturationCurrentSlider" min="-15" max="-9" step="0.1" value="-12" class="slider">
        <span id="saturationCurrentValue" class="slider-value"></span>
    </div>
    <div class="slider-container">
        <label class="slider-label">Temperature \(T\) (K):</label>
        <input type="range" id="temperatureSlider" min="273" max="350" step="1" value="300" class="slider">
        <span id="temperatureValue" class="slider-value">300</span>
    </div>
    <div class="slider-container">
        <label class="slider-label">Ideality factor \(n\):</label>
        <input type="range" id="idealityFactorSlider" min="1" max="2" step="0.1" value="1" class="slider">
        <span id="idealityFactorValue" class="slider-value">1</span>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/chart.js@3.5.1"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-zoom@1.1.1"></script>
    <script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
    <script id="MathJax-script" async
            src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
    </script>

    <script>
        // Simulation parameters
        var numPoints = 200;

        // Diode characteristics
        var saturationCurrentExp = -12; // Slider value represents exponent
        var kBoltzmann = 1.38e-23; // Boltzmann constant (J/K)
        var electronCharge = 1.6e-19; // Elementary charge (C)
        var temperature = 300; // Initial temperature in Kelvin
        var thermalVoltage = (kBoltzmann * temperature) / electronCharge;
        var idealityFactor = 1; // Initial ideality factor

        // Calculate diode current
        function calculateCurrent(voltage) {
            var saturationCurrent = Math.pow(10, saturationCurrentExp); // Calculate saturation current
            var current = saturationCurrent * (Math.exp(voltage / (thermalVoltage * idealityFactor)) - 1);
            return current;
        }

        // Render diode IV curve
        function renderDiodeIVCurve() {
            var canvas = document.getElementById("diodeCanvas");

            // Generate voltage data
            var voltages = [];
            for (var i = 0; i <= numPoints; i++) {
                var voltage = i / numPoints;
                voltages.push(voltage);
            }

            // Generate current data
            var currents = voltages.map(calculateCurrent);

            // Get the chart instance
            var chart = window.diodeChart;

            // Update the chart data
            chart.data.labels = voltages;
            chart.data.datasets[0].data = currents;

            // Update the chart
            chart.update();
        }

        // Update saturation current value and temperature
        function updateSaturationCurrentValue() {
            var saturationCurrentSlider = document.getElementById("saturationCurrentSlider");
            var saturationCurrentValue = document.getElementById("saturationCurrentValue");
            saturationCurrentExp = parseFloat(saturationCurrentSlider.value);
            var sliderValue = Math.pow(10, saturationCurrentExp).toExponential(1);
            saturationCurrentValue.textContent = sliderValue;

            var temperatureSlider = document.getElementById("temperatureSlider");
            var temperatureValue = document.getElementById("temperatureValue");
            temperature = parseFloat(temperatureSlider.value);
            temperatureValue.textContent = temperature;

            thermalVoltage = (kBoltzmann * temperature) / electronCharge;

            var idealityFactorSlider = document.getElementById("idealityFactorSlider");
            var idealityFactorValue = document.getElementById("idealityFactorValue");
            idealityFactor = parseFloat(idealityFactorSlider.value);
            idealityFactorValue.textContent = idealityFactor;

            // Update the chart
            renderDiodeIVCurve();
        }

        // Add event listener to temperature slider
        var saturationCurrentSlider = document.getElementById("saturationCurrentSlider");
        saturationCurrentSlider.addEventListener("input", updateSaturationCurrentValue);

        var temperatureSlider = document.getElementById("temperatureSlider");
        temperatureSlider.addEventListener("input", updateSaturationCurrentValue);

        var idealityFactorSlider = document.getElementById("idealityFactorSlider");
        idealityFactorSlider.addEventListener("input", updateSaturationCurrentValue);

        // Render the initial diode IV curve
        var canvas = document.getElementById("diodeCanvas");
        var ctx = canvas.getContext("2d");

        window.diodeChart = new Chart(ctx, {
            type: "line",
            data: {
                labels: [], // Empty labels initially
                datasets: [{
                    label: "Diode IV Curve",
                    data: [], // Empty data initially
                    borderColor: "blue",
                    borderWidth: 2,
                    fill: false,
                    pointStyle: "line", // Set point style to line
                    pointRadius: 0 // Set point radius to 0
                }]
            },
            options: {
                animation: {
                  duration: 500,
                  easing: "easeOutQuint",
                },
                scales: {
                    x: {
                        type: "linear",
                        title: {
                            display: true,
                            text: "Bias voltage (V)",
                            font: {
                              size: 14
                            }
                        },
                        max: 1, // Set maximum value for y-axis
                        min: 0 // Set minimum value for y-axis
                    },
                    y: {
                        type: "linear",
                        ticks: {
                            callback: function(value, index, values) {
                              return value * 1000;
                            },
                        },
                        title: {
                            display: true,
                            text: "Diode current (mA)",
                            rotation: 0,
                            position: "left",
                            font: {
                              size: 14
                            }
                        },
                        max: 10e-3, // Set maximum value for y-axis
                        min: 0 // Set minimum value for y-axis
                    }
                },
                plugins: {
                  legend: {
                    display: false // Set display to false to hide the legend
                  },
                  title: {
                    display: true,
                    text: 'Ideal diode IV curve',
                    font: {
                      size: 20,
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

        // Render the initial diode IV curve
        renderDiodeIVCurve();

        // Set initial saturation current value on page load
        updateSaturationCurrentValue();

    </script>
</body>
<br>
This is based on the [Shockley diode equation](https://en.wikipedia.org/wiki/Shockley_diode_equation)
<p>
\[ I_D = I_S \left(e^{\frac{V_D}{n V_T}} - 1 \right)   \]
using the thermal voltage \(V_T = \frac{kT}{q} \), the Boltzmann constant \(k\), and the elementary charge \(q\).
</p>

<!--more-->

Interactive plots are the first thing I google for whenever I'm exploring new physics equations. However, often it's hard to find anything! Hence, my first foray into making semiconductor physics more interactive, starting with a simple diode.

This is the first time I've used Html5 and/or Javascript - however, I used the amazing powers of ChatGPT&nbsp;3.5.
I started this yesterday, and in total this took me around 4 hours. Not too bad for a complete beginner. Initially ChatGPT suggested coding the entire plot from scratch using Html5. However, this turned very buggy when either I or ChatGPT attempted to make adjustments to the plot. Hence, I switched to Chart.js for the plotting engine, and voilà.
