```yaml script=scriptloader
- tinytimer.js
```

```yaml script=dataloader
xml: Modelica.Electrical.Analog.Examples.OvervoltageProtection_init.xml 
```


# OpenModelica simulation example
## Modelica.Electrical.Analog.Examples.OvervoltageProtection

<img src=Modelica.Electrical.Analog.Examples.OvervoltageProtection.svg class="pull-right" style="width:540px; background-color:#ffffff; border:2px solid gray" />


```yaml jquery=jsonForm class="form-horizontal" name=frm 
schema: 
  stopTime:
    type: string
    title: Stop time, sec
    default: 0.4
  intervals:
    type: string
    title: Output intervals
    default: 500
  tolerance:
    type: string
    title: Tolerance
    default: 0.0000001
  solver: 
    type: string
    title: Solver
    enum: 
      - dassl
      - euler
      - rungekutta
      - dasslwort
      - dassltest
  RL: 
    type: string
    title: Load resistance, ohms
    default: 2000.0
form: 
  - "*"
params:
  fieldHtmlClass: input-medium
```

```js
if (typeof(isRunning) == "undefined") isRunning = false

if (typeof(timer) != "undefined") {clearInterval(timer.interval); timer = null};
$xml = $(xml)

// Set the default simulation parameters
defex = $xml.find("DefaultExperiment")
defex.attr("stopTime", stopTime)
defex.attr("stepSize", +stopTime / intervals)
defex.attr("tolerance", tolerance)
defex.attr("solver", solver)

// Set some model parameters
$xml.find("ScalarVariable[name = 'RL.R']").find("Real").attr("start", RL)

// Write out the initialization file
xmlstring = new XMLSerializer().serializeToString(xml)

$("#statustext").html('<img src="wait.gif" /> Simulation running')
$("#statustimer").html("");
$('#statustimer').tinyTimer({ from: Date.now() });

timer = $("#statustimer").data("tinyTimer")

// Start the simulation!
basename = "Modelica.Electrical.Analog.Examples.OvervoltageProtection"

if (typeof(wworker) != "undefined" && isRunning) wworker.terminate() 
if (typeof(wworker) == "undefined" || isRunning) wworker = new Worker(basename + ".js")
isRunning = true

wworker.postMessage({basename: basename, xmlstring: xmlstring})
wworker.addEventListener('error', function(event) {
});


```

<div id="status" style="text-align:center"><span id="statustext">
Simulation loading</span>. &nbsp Time: <span id="statustimer"> </span></div>

## Results

<div id="yaxisform"> </div>

```js
// read the csv file with the simulation results

wworker.addEventListener("message", function(e) {
    $("#statustext").html(e.data.status)
    timer.stop();
    isRunning = false
    x = $.csv.toArrays(e.data.csv, {onParseValue: $.csv.hooks.castToScalar})
    
    // `header` has the column names. The first is the time, and the rest
    // of the columns are the variables.
    header = x.slice(0,1)[0]
    
    // Select graph variables with a select box based on the header values
    if (typeof(graphvar) == "undefined") graphvar = header[1];
    if (typeof(graphvarX) == "undefined") graphvarX = header[0];
    
    var jsonform = {
      schema: {
        graphvar: {
          type: "string",
          title: "Plot variable",
          default: graphvar,
          enum: header
        }
      },
      form: [
        {
          key: "graphvar",
          onChange: function (evt) {
            calculate_forms();
            $("#plotdiv").calculate();
          }
        }
      ]
    };
    var jsonformX = {
      schema: {
        graphvarX: {
          type: "string",
          default: graphvarX,
          enum: x.slice(0,1)[0]
        }
      },
      form: [
        {
          key: "graphvarX",
          onChange: function (evt) {
            calculate_forms();
            $("#plotdiv").calculate();
          }
        }
      ]
    };
    
    $("#yaxisform").html("");
    $("#yaxisform").jsonForm(jsonform);
    $("#xaxisform").html("");
    $("#xaxisform").jsonForm(jsonformX);
    $("#plotdiv").calculate();
    
}, false);

```

```js id=plotdiv
if (typeof(header) != "undefined") {
    yidx = header.indexOf(graphvar);
    xidx = header.indexOf(graphvarX);
    // pick out the column to plot
    series = x.slice(1).map(function(x) {return [x[xidx], x[yidx]];});
    plot([series]);
}
```

<div id="xaxisform" style="left:200px; width:300px; position:relative"> </div>

## Information

This example is a simple circuit for overvoltage protection. If the
voltage `zDiode_1.n.v` is too high, the Diode `zDiode_2` breaks through
and the voltage gets down.

The simulation end time should be set to 0.4. To get the typical
behaviour please plot `sineVoltage.p.v`, `RL.p.v`, `zDiode_2.n.v` and
`zDiode_1.n.i`.

## Comments

This simulation model is from a [Modelica](http://modelica.org) model.
Modelica is a language for simulating electrical, thermal, and
mechanical, systems. [OpenModelica](http://openmodelica.org) was used
to compile this model to C. Then, [Emscripten](http://emscripten.org/)
was used to compile the C code to JavaScript.

The JavaScript code for the model is almost 2 MB, so that's why the
page loading takes so long. Once loaded, the simulation runs pretty
quickly.

For more information on compiling OpenModelica to JavaScript, see
[here](https://github.com/tshort/openmodelica-javascript).

The user interface was created in
[mdpad](http://tshort.github.io/mdpad/). See
[Modelica.Electrical.Analog.Examples.OvervoltageProtection.md](Modelica.Electrical.Analog.Examples.OvervoltageProtection.md) for the Markdown code
for this page.

This should work in both Firefox and Chrome. It doesn't work in
Internet Explorer. Sometimes, the simulation fails with "out of
memory" in Firefox 23 in Windows. I haven't seen that with Firefox 20
on Linux.