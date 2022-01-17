<script>
	let mean = 0.2
	let variance = 0.3
	let binCount = 255

	let selected = null

	let rangeStart = -4
	let rangeEnd = 4
	let rangeWidth = rangeEnd - rangeStart

	$: binWidth = rangeWidth / binCount

	function pdf(mean, variance, x) {
		if(variance == 0) {
			return Math.abs(x-mean) < 2/binCount ? Math.sqrt(binCount) : 0
		} else {
			return 1/(variance*Math.pow(2*Math.PI, 2)) * Math.exp(-0.5*Math.pow(x-mean, 2) / Math.pow(variance, 2))
		}
	}

	function cdf(mean, variance, x) {
		if(variance == 0) {
			return x-mean > -1/binCount ? 1 : 0
		} else {
			return 0.5 * (1 + erf((x-mean)/Math.sqrt(2)/variance))
		}
	}

	function erf(x) {
		const enxx = Math.exp(-x*x)
		const sqpi = Math.sqrt(Math.PI)

		return 2/sqpi * Math.sign(x) * Math.sqrt(1-enxx) * (sqpi/2 + 31/200 * enxx - 341/8000 * Math.sqrt(1-enxx*enxx))
	}
	
	$: X = Array(binCount).fill(null).map((_,i) => rangeStart + i*binWidth)

	$: samples = X.map((x, i) => ({
		id: i,
		start: x - binWidth/2,
		end: x + binWidth/2,
		height: (cdf(mean, variance, x+binWidth/2) - cdf(mean, variance, x-binWidth/2)),
		symHeight: (cdf(mean, variance, -x-binWidth/2) - cdf(mean, variance, -x+binWidth/2)),
		color: `hsl(${(x-mean)*45/1.5/(variance||1)}, 70%, 50%)`
	}))

	$: transformed = samples.flatMap(({start, end, height, color, symHeight}, i) => {

		const newStart = start*start
		const newEnd = end*end

		const origWidth = Math.abs(end-start)
		const newWidth = Math.abs(newEnd-newStart)

		const widthGrowth = newWidth / origWidth

		return newStart <= newEnd ?
			{
				id: i,
				start: start*start,
				end: end*end,
				height: height / widthGrowth,
				color,
				offset: 0,
			} : 
			{
				id: i,
				start: end*end,
				end: start*start,
				height: height / widthGrowth,
				color,
				offset: symHeight / widthGrowth,
			}
	})

	$: integrations = X.map((x) => [x, cdf(mean, variance, x)])

	function flipY(y) {
		return 180 - y;
	}

	function clickBar(evt) {
		selected = evt.currentTarget.getAttribute('data-id')
	}

	function resetSelection() {
		selected = null
	}
</script>

<style>
	:global(body) {
		margin: 0;
		padding: 1em;
	}
	:global(*) {
		box-sizing: border-box;
	}
	dl {display: grid; grid-template-columns: max-content 1fr; align-items: center;gap:1em; margin: 0;grid-auto-columns: min-content; grid-auto-flow: column; padding: 0;justify-items: stretch;}
	input[type=range] {margin: 0; width: 100%; max-width: 100%; padding: 0;}
	input {margin: 0;}
	dd {margin: 0}
	dt {grid-column-start: 1;}
	label[for]::after {content:':';}
	
	.controls {
		width: 100%;
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(18em, 1fr));
		gap: 1em;
	}
	
	svg {
		max-width: 100%;
		width: 100%;
		height: auto;
		padding: 0;
		margin: 0
	}
	
	label {
		white-space: nowrap;
	}

	.selected {
		fill: black !important;
	}
</style>

<h1>
	Function applied to random variables
</h1>

<div class="controls">
<fieldset>
	<legend>
		Parameters
	</legend>
<dl>
	<dt><label for='mean_0'>Number of bins</label></dt>
	<dd><input id="mean_0" type="range" bind:value={binCount} min="15" max="511" step="2" on:input={resetSelection} /></dd>
	<dt><label for='mean_0'>Mean</label></dt>
	<dd><input id="mean_0" type="range" bind:value={mean} min={rangeStart/2} max={rangeEnd/2} step={rangeWidth/1000} /></dd>
	<dt><label for='variance_0'>Variance</label></dt>
	<dd><input id="variance_0" type="range" bind:value={variance} min="0" max="1" step="0.001" /></dd>
</dl>	
</fieldset>

</div>




<h2>
	Original Gaußian Distribution
</h2>

<svg viewBox="-20 -180 1080 400" stroke-width="2" font-size="24">
	<defs>
		<marker id="arrowhead" markerWidth="10" markerHeight="7" 
    refX="0" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" />
    </marker>
	</defs>

	<g>
	<line x1="0" y1={flipY(0)} x2="1000" y2={flipY(0)} stroke="black" marker-end="url(#arrowhead)" />
	<line x1="500" y1={flipY(-15)} x2="500" y2={flipY(300)} stroke="black" marker-end="url(#arrowhead)" />
	<text x={525} y={flipY(300)} text-anchor="start">p(X)</text>
	<text x={1000} y={flipY(20)} text-anchor="start">X</text>

	<line x1={500 + 1000/rangeWidth} y1={flipY(0)} x2={500 + 1000/rangeWidth} y2={flipY(-15)} stroke="black" />
	<line x1={500 + 2*1000/rangeWidth} y1={flipY(0)} x2={500 + 2*1000/rangeWidth} y2={flipY(-15)} stroke="black" />
	<line x1={500 + 3*1000/rangeWidth} y1={flipY(0)} x2={500 + 3*1000/rangeWidth} y2={flipY(-15)} stroke="black" />
	<line x1={500 - 1000/rangeWidth} y1={flipY(0)} x2={500 - 1000/rangeWidth} y2={flipY(-15)} stroke="black" />
	<line x1={500 - 2*1000/rangeWidth} y1={flipY(0)} x2={500 - 2*1000/rangeWidth} y2={flipY(-15)} stroke="black" />
	<line x1={500 - 3*1000/rangeWidth} y1={flipY(0)} x2={500 - 3*1000/rangeWidth} y2={flipY(-15)} stroke="black" />

	<text x={500 + 1000/rangeWidth} y={flipY(-40)} text-anchor="middle">1</text>
	<text x={500 + 2*1000/rangeWidth} y={flipY(-40)} text-anchor="middle">2</text>
	<text x={500 + 3*1000/rangeWidth} y={flipY(-40)} text-anchor="middle">3</text>
	<text x={500 - 1000/rangeWidth} y={flipY(-40)} text-anchor="middle">-1</text>
	<text x={500 - 2*1000/rangeWidth} y={flipY(-40)} text-anchor="middle">-2</text>
	<text x={500 - 3*1000/rangeWidth} y={flipY(-40)} text-anchor="middle">-3</text>


		{#each samples as {start, end, height, color, id}}
		<rect class:selected={selected==id} on:click={clickBar}  data-id={id} x={500+1000/rangeWidth*start} width={1000/rangeWidth*(end-start)} y={flipY(100/rangeWidth*height*(binCount)+1)} height={100/rangeWidth*height*(binCount)} fill={color} />
		{/each}
		
		</g>
</svg>

<h2>
	Distribution after applying the square-function (X<sup>2</sup>)
</h2>


<svg viewBox="-20 -180 1080 500" stroke-width="2" font-size="24">

	<g>
	<line x1="0" y1={flipY(0)} x2="1000" y2={flipY(0)} stroke="black" marker-end="url(#arrowhead)" />
	<line x1="500" y1={flipY(-15)} x2="500" y2={flipY(300)} stroke="black" marker-end="url(#arrowhead)" />
	<text x={525} y={flipY(300)} text-anchor="start">p(X<tspan dy="-10" font-size=".7em">2</tspan><tspan dy="10">)</tspan></text>
	<text x={1000} y={flipY(20)} text-anchor="start">X<tspan dy="-10" font-size=".7em">2</tspan></text>

	<line x1={500 + 1000/rangeWidth} y1={flipY(0)} x2={500 + 1000/rangeWidth} y2={flipY(-15)} stroke="black" />
	<line x1={500 + 2*1000/rangeWidth} y1={flipY(0)} x2={500 + 2*1000/rangeWidth} y2={flipY(-15)} stroke="black" />
	<line x1={500 + 3*1000/rangeWidth} y1={flipY(0)} x2={500 + 3*1000/rangeWidth} y2={flipY(-15)} stroke="black" />
	<line x1={500 - 1000/rangeWidth} y1={flipY(0)} x2={500 - 1000/rangeWidth} y2={flipY(-15)} stroke="black" />
	<line x1={500 - 2*1000/rangeWidth} y1={flipY(0)} x2={500 - 2*1000/rangeWidth} y2={flipY(-15)} stroke="black" />
	<line x1={500 - 3*1000/rangeWidth} y1={flipY(0)} x2={500 - 3*1000/rangeWidth} y2={flipY(-15)} stroke="black" />

	<text x={500 + 1000/rangeWidth} y={flipY(-40)} text-anchor="middle">1</text>
	<text x={500 + 2*1000/rangeWidth} y={flipY(-40)} text-anchor="middle">2</text>
	<text x={500 + 3*1000/rangeWidth} y={flipY(-40)} text-anchor="middle">3</text>
	<text x={500 - 1000/rangeWidth} y={flipY(-40)} text-anchor="middle">-1</text>
	<text x={500 - 2*1000/rangeWidth} y={flipY(-40)} text-anchor="middle">-2</text>
	<text x={500 - 3*1000/rangeWidth} y={flipY(-40)} text-anchor="middle">-3</text>

		{#each transformed as {start, end, height, color, id, growth, offset}}
		<rect class:selected={selected==id} on:click={clickBar} data-id={id} x={500+1000/rangeWidth*start} width={1000/rangeWidth*(end-start)} y={flipY(100/rangeWidth*(height-offset)*(binCount)+Math.sign(height))} height={Math.abs(100/rangeWidth*height*(binCount))} fill={color} />
		{/each}
		
		</g>
</svg>


<p>
	Applying a function to a random variable affects the resulting distribution. The colored rectangles represent the density of the random variable along the X axis. You can see how squaring the variable shifts all rectangles into the positive range. Each colored rectangles keeps its area but is stretched along the X axis by 2*x (the derivative of x<sup>2</sup>). Accordingly the height of each rectangle is compressed by 2*x to keep the area constant.
</p>
<p>
	As x<sup>2</sup> is not injective multiple rectangles end up at the same target along the x axis. They simply stack on top of each other to represent the resulting probability density at each location.
</p>
<p>
	Clicking on a single rectangle highlights its corresponding partner rectangle in the other distribution.
</p>
<p>
	Lowering the number of bins makes the individual rectangles more recognizable but reduces the precision of the probability density.
</p>