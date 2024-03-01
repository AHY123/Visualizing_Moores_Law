<script>
  import * as d3 from "d3";

  export let data;
  export let index;

  let svg, gx, gy;
  let height = 500;
  let width = 800;

  const margin = {
    top: 20,
    right: 30,
    bottom: 50,
    left: 100,
  };

  $: x = d3
    .scaleTime()
    .domain([0, d3.max(data, (d) => d.date)])
    .range([margin.left, width - margin.right]);

  $: y = d3
    .scaleLinear()
    .domain([0, d3.max(data, (d) => d.value)])
    .nice()
    .range([height - margin.bottom, margin.top]);

  $: d3.select(gx).call(d3.axisBottom(x).ticks(width / 80));

  const formatYAxis = (value) => {
    if (value >= 1e9) {
      return `${value / 1e9} billion`;
    } else {
      return value;
    }
  };

  $: d3.select(gy)
    .call(d3.axisLeft(y).tickFormat(formatYAxis).ticks(null, "s"))
    .call((g) =>
      g
        .selectAll(".tick line")
        .clone()
        .attr("x2", width - margin.right - margin.left)
        .attr("stroke-opacity", 0.1)
    );

  let isVisible = false;

  $: if (index === 0) {
    isVisible = true;
  }

  const tooltipW = 220;
  const tooltipH = 100;
  const tooltipPaddingTop = 30;
  const tooltipPaddingLeft = 15;

  let mousePosition = [0, 0];
  let selectedPoint = null;

  const bisect = d3.bisector((d) => d.date).center;

  function setSelected(event) {
    mousePosition = d3.pointer(event);
    const x0 = x.invert(mousePosition[0]);
    const i = bisect(data, x0, 0);
    selectedPoint = { date: data[i].date, value: data[i].value };
    // console.log(selectedPoint);
  }
</script>

<main>
  <!-- svelte-ignore a11y-no-static-element-interactions -->
  <div class = "container"> 

    <svg
      class="graph"
      class:visible={isVisible}
      this="{svg}{width}"
      {height}
      viewBox="0 0 {width} {height}"
      on:mousemove={setSelected}
    >
      <!-- x-axis -->
      <g
        class="axis"
        bind:this={gx}
        transform="translate(0,{height - margin.bottom})"
      >
        <text
          class="axis-labels"
          x={width / 2 + 20}
          y="20"
          dy="1em"
          fill="black"
          text-anchor="middle"
          font-size="14px"
        >
          Year
        </text>
      </g>
      <!-- y-axis -->
      <g class="axis" bind:this={gy} transform="translate({margin.left},0)">
        <text
          class="axis-labels"
          x="0"
          y="0"
          dy="-1em"
          fill="black"
          font-size="14px"
          transform="translate(-60, {height / 2 - 70}) rotate(-90)"
        >
          Transistor(s)
        </text>
      </g>
      <!-- {#each yAxisTicks as tick}
        
      {/each} -->
      <g stroke="steelblue" stroke-width="1.5">
        {#each data.slice(0, -1) as d, i}
          <path
            d={"M" +
              x(d.date) +
              "," +
              y(d.value) +
              " " +
              "L" +
              x(data[i + 1].date) +
              "," +
              y(data[i + 1].value)}
            fill="none"
            stroke="black"
            stroke-width="2"
          />
        {/each}
        {#each data as d, i}
          <circle
            key={i}
            cx={x(d.date)}
            cy={y(d.value)}
            r="2.5"
            on:mousemove={() => setSelected(d)}
          />
        {/each}
  
        {#if selectedPoint && mousePosition[1] < 450}
          <circle
            cx={x(selectedPoint.date)}
            cy={y(selectedPoint.value)}
            r="5"
            fill="red"
          />
          <rect
            class="vertical-line"
            width="1"
            height={420}
            stroke="gray"
            stroke-opacity="0.1"
            fill-opacity="0.1"
            transform="translate({x(selectedPoint.date)}, {margin.top + 10})"
          />
          {#if mousePosition[0] < margin.right + 300 && margin.left < mousePosition[0]}
            <g
              class="tooltip"
              transform="translate({mousePosition[0] + 5},{mousePosition[1] -
                tooltipH})"
            >
              <rect
                width={tooltipW}
                height={tooltipH - 20}
                fill="white"
                stroke="black"
              />
              <g transform="translate({tooltipPaddingLeft},{tooltipPaddingTop})">
                <text
                  class="tooltip-name"
                  stroke="black"
                  stroke-width="1"
                  shape-rendering="crispEdges"
                >
                  Year: {selectedPoint.date.getYear() + 1901}
                </text>
                <text
                  class="tooltip-name"
                  stroke="black"
                  y={30}
                  stroke-width="0.5"
                >
                  Transistors: {selectedPoint.value}
                </text>
              </g>
            </g>
          {:else if mousePosition[0] > margin.right + 300}
            <g
              class="tooltip"
              transform="translate({mousePosition[0] -
                tooltipW -
                5},{mousePosition[1] - tooltipH})"
            >
              <rect
                width={tooltipW}
                height={tooltipH}
                fill="white"
                stroke="black"
              />
              <g transform="translate({tooltipPaddingLeft},{tooltipPaddingTop})">
                <text
                  class="tooltip-name"
                  stroke="black"
                  stroke-width="1"
                  shape-rendering="crispEdges"
                >
                  Year: {selectedPoint.date.getYear() + 1901}
                </text>
                <text
                  class="tooltip-name"
                  stroke="black"
                  y={50}
                  stroke-width="0.5"
                >
                  Transistors: {selectedPoint.value}
                </text>
              </g>
            </g>
          {/if}
        {/if}
      </g>
    </svg>
  </div>
</main>

<style>
  @import url("https://fonts.googleapis.com/css2?family=Lato:ital,wght@0,100;0,300;0,400;1,100;1,300;1,400&display=swap");
  .container {
    display: flex;
    justify-content: center;
  }
  .graph.visible {
    opacity: 1;
    visibility: visible;
  }

  .axis {
    font-family: "Lato", serif;
    font-size: 14px;
  }

  .axis-labels {
    font-family: "Lato", serif;
    font-size: 18px;
  }

  .tooltip {
    font-family: "Lato", serif;
    font-size: 16px;
  }

  .tooltip-name {
    font-family: "Lato", serif;
    font-size: 16px;
    fill: "black";
    font-weight: 300;
  }

  .vertical-line {
    pointer-events: none;
  }
</style>
