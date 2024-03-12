<script>
  import * as d3 from "d3";

  export let data;
  export let index;

  let svg, gx, gy;
  let height = 500;
  let width = 800;

  const margin = {
    top: 100,
    right: 80,
    bottom: 80,
    left: 130,
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
        .attr("stroke-opacity", 0.1),
    );

  let startFade = true;

  $: {
    if (index <= 1 || (index >= 4) & (index <= 5)) {
      startFade = false;
    } else {
      startFade = true;
      console.log("trying to hide");
    }
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
  <div class="container" class:fade={startFade}>
    <svg
      style="border:3px solid black"
      class="graph"
      this="{svg}{width}"
      {height}
      viewBox="0 0 {width} {height}"
      on:mousemove={setSelected}
    >
      <!-- title -->
      <text
        class="title"
        x={width/2}
        y="0"
        dy="-1em"
        fill="black"
        font-size="14px"
        transform="translate({-350}, {margin.top - 20})"
      >
        Plotting Number of Transistors on a Microprocessor Through Time
      </text>
      <!-- x-axis -->
      <g
        class="axis"
        bind:this={gx}
        transform="translate(0,{height - margin.bottom})"
      >
        <text
          class="axis-labels"
          x={width / 2 + 20}
          y="40"
          dy="1em"
          fill="black"
          text-anchor="middle"
          font-size="14px"
        >
          Year
        </text>
      </g>
      <!-- y-axis -->
      <g class="axis" bind:this={gy} transform="translate({margin.left + 2},0)">
        <text
          class="axis-labels"
          x="0"
          y="0"
          dy="-1em"
          fill="black"
          font-size="14px"
          transform="translate(-70, {height / 2 - 70}) rotate(-90)"
        >
          Transistor(s)
        </text>
      </g>

      <g stroke="black" stroke-width="1.5">
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
            stroke="red"
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
            height={310}
            stroke="gray"
            stroke-opacity="0.1"
            fill-opacity="0.1"
            transform="translate({x(selectedPoint.date)}, {margin.top + 10})"
          />
          {#if mousePosition[0] < margin.right + 300 && margin.left < mousePosition[0] && mousePosition[1] > 100 && mousePosition[0] < 775}
            <div class="main-tooltip"></div>
            <g
              class="tooltip"
              transform="translate({mousePosition[0] + 5},{mousePosition[1] -
                tooltipH})"
            >
              <rect
                class="top-rect"
                width={tooltipW - 40}
                height={tooltipH - 20}
                fill="#F0F0F0"
                stroke="black"
                opacity="1"
              />
              <g
                transform="translate({tooltipPaddingLeft},{tooltipPaddingTop})"
              >
                <text class="tooltip-year" stroke="red">
                  {selectedPoint.date.getYear() + 1901}
                </text>
                <text class="tooltip-name" stroke="black" y={30}>
                  {#if selectedPoint.value > 1e9}
                    Transistors: {selectedPoint.value / 1e9} billion
                  {:else if selectedPoint.value > 1e7}
                    Transistors: {selectedPoint.value / 1e7} million
                  {:else}
                    Transistors: {selectedPoint.value}
                  {/if}
                </text>
              </g>
            </g>
          {:else if mousePosition[0] > margin.right + 300 && mousePosition[1] > 100 && mousePosition[0] < 775}
            <g
              class="tooltip"
              transform="translate({mousePosition[0] -
                tooltipW -
                5},{mousePosition[1] - tooltipH})"
            >
              <rect
                class="toolrect"
                width={tooltipW - 10}
                height={tooltipH - 20}
                fill="#F0F0F0"
                stroke="black"
              />
              <g
                transform="translate({tooltipPaddingLeft},{tooltipPaddingTop})"
              >
                <text class="tooltip-year" stroke="red">
                  {selectedPoint.date.getYear() + 1901}
                </text>
                <text class="tooltip-name" stroke="black" y={30}>
                  {#if selectedPoint.value > 1e9}
                    Transistors: {(selectedPoint.value / 1e9).toFixed(2)} billion
                  {:else if selectedPoint.value > 1e6}
                    Transistors: {(selectedPoint.value / 1e6).toFixed(2)} million
                  {:else}
                    Transistors: {selectedPoint.value}
                  {/if}
                </text>
              </g>
            </g>
          {:else if mousePosition[1] < 100 && mousePosition[0] > margin.right + 300 && mousePosition[0] < 775 && mousePosition[1] > 0}
            <g
              class="tooltip"
              transform="translate({mousePosition[0] -
                tooltipW -
                5},{mousePosition[1] + 25})"
            >
              <rect
                class="toolrect"
                width={tooltipW - 10}
                height={tooltipH - 20}
                fill="#F0F0F0"
                stroke="black"
              />
              <g
                transform="translate({tooltipPaddingLeft},{tooltipPaddingTop})"
              >
                <text class="tooltip-year" stroke="red">
                  {selectedPoint.date.getYear() + 1901}
                </text>
                <text class="tooltip-name" stroke="black" y={30}>
                  {#if selectedPoint.value > 1e9}
                    Transistors: {(selectedPoint.value / 1e9).toFixed(2)} billion
                  {:else if selectedPoint.value > 1e6}
                    Transistors: {(selectedPoint.value / 1e6).toFixed(2)} million
                  {:else}
                    Transistors: {selectedPoint.value}
                  {/if}
                </text>
              </g>
            </g>
          {:else if mousePosition[0] > 100 && mousePosition[0] < 775 && mousePosition[1] > 0}
            <g
              class="tooltip"
              transform="translate({mousePosition[0] + 5},{mousePosition[1] +
                25})"
            >
              <rect
                class="toolrect"
                width={tooltipW - 10}
                height={tooltipH - 20}
                fill="#F0F0F0"
                stroke="black"
              />
              <g
                transform="translate({tooltipPaddingLeft},{tooltipPaddingTop})"
              >
                <text class="tooltip-year" stroke="red">
                  {selectedPoint.date.getYear() + 1901}
                </text>
                <text class="tooltip-name" stroke="black" y={30}>
                  {#if selectedPoint.value > 1e9}
                    Transistors: {(selectedPoint.value / 1e9).toFixed(2)} billion
                  {:else if selectedPoint.value > 1e6}
                    Transistors: {(selectedPoint.value / 1e6).toFixed(2)} million
                  {:else}
                    Transistors: {selectedPoint.value}
                  {/if}
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
    visibility: visible;
  }
  .container.fade {
    animation: fadeOut 0.5s;
    animation-fill-mode: forwards;
  }
  @keyframes fadeOut {
    0% {
      opacity: 1;
    }
    100% {
      opacity: 0;
    }
  }
  .graph {
    opacity: 1;
    /* visibility: visible; */
  }

  .axis {
    font-family: "Lato", serif;
    font-size: 14px;
  }

  .axis-labels {
    font-family: "Lato", serif;
    font-size: 18px;
  }

  .title {
    font-family: "Lato", serif;
    font-size: 22px;
  }

  .top-rect {
    fill: "#F0F0F0";
    stroke: "black";
    opacity: "1";
  }

  .tooltip {
    font-family: "Lato", serif;
    font-size: 16px;
  }

  .tooltip-year {
    font-family: "Lato", sans-serif !important;
    font-size: 16px;
    font-weight: lighter;
  }

  .tooltip-name {
    font-family: "Lato", sans-serif !important;
    font-size: 16px;
    font-weight: lighter;
  }

  .vertical-line {
    pointer-events: none;
  }
</style>
