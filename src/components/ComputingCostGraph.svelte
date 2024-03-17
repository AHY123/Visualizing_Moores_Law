<script>
  import * as d3 from "d3";

  export let cost_data;

  let svg, gx, gy;
  let height = 500;
  let width = 800;

  let RED = "#C05065";
  let BLUE = "#192B56";
  let GREEN = "#2D8465";

  const margin = {
    top: 100,
    right: 80,
    bottom: 80,
    left: 130,
  };

  $: x = d3
    .scaleTime()
    .domain([new Date("1955"), d3.max(cost_data, (d) => d.date)])
    .range([margin.left, width - margin.right - 30]);

  $: y = d3
    .scaleLog()
    .domain([7, d3.max(cost_data, (d) => d.memory)])
    .range([height - margin.bottom, margin.top]);

  $: d3.select(gx).call(d3.axisBottom(x).ticks(10));

  const formatYAxis = (value) => {
    if (value >= 1e12) {
      return `${value / 1e12} trillion`;
    } else if (value >= 1e9) {
      return `${value / 1e9} billion`;
    } else if (value >= 1e6) {
      return `${value / 1e6} million  `;
    } else {
      return value;
    }
  };

  $: d3.select(gy)
    .call(d3.axisLeft(y).tickFormat(formatYAxis).ticks(7))
    .call((g) =>
      g
        .selectAll(".tick line")
        .clone()
        .attr("x2", width - margin.right - margin.left - 30)
        .attr("stroke-opacity", 0.1),
    );

  let startFade = false;

  // $: {
  //   if (index <= 3) {
  //     startFade = false;
  //   } else {
  //     startFade = true;
  //     console.log("trying to hide");
  //   }
  // }

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
    const i = bisect(cost_data, x0, 0);
    selectedPoint = {
      date: cost_data[i].date,
      memory: cost_data[i].memory,
      flash: cost_data[i].flash,
      drives: cost_data[i].drives,
      ssd: cost_data[i].ssd,
    };
  }

  function findNextNonZero(data, start, key) {
    for (let i = start; i < data.length; i++) {
      if (data[i][key] !== 0) {
        return { date: data[i].date, value: data[i][key] };
      }
    }
    return null;
  }

  function formatNumber(value) {
    if (value >= 1e12) {
      return (value / 1e12).toFixed(2) + " trillion";
    } else if (value >= 1e9) {
      return (value / 1e9).toFixed(2) + " billion";
    } else if (value >= 1e6) {
      return (value / 1e6).toFixed(2) + " million";
    } else {
      return value.toString();
    }
  }

  function getToolTipText(selectedPoint) {
    let year = selectedPoint.date.getYear() + 1901;
    let variables = ["memory", "flash", "drives"];
    let selectedValues = {};

    selectedValues.year = year;

    for (let variable of variables) {
      if (selectedPoint[variable] !== 0) {
        let capitalizedVariable =
          variable.charAt(0).toUpperCase() + variable.slice(1);
        selectedValues[capitalizedVariable] = formatNumber(
          selectedPoint[variable],
        );
      }
    }

    if (selectedPoint["ssd"] !== 0) {
      selectedValues.SSD = formatNumber(selectedPoint.ssd);
    }

    return selectedValues;
  }
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<div class="container" class:fade={startFade}>
  <div class="spacer" />
  <svg
    style="border:3px solid black"
    class="graph"
    this="{svg}{width}"
    {height}
    viewBox="0 0 {width} {height}"
    on:mousemove={setSelected}
  >
    <!-- legend -->
    <rect
      width="10"
      height="2"
      fill={RED}
      transform="translate({700}, {357})"
    />
    <text x="716" y="363" fill={RED}>Memory</text>
    <rect
      width="10"
      height="2"
      fill={GREEN}
      transform="translate({700}, {413})"
    />
    <text x="716" y="418" fill={GREEN}>Disk</text>
    <rect
      width="10"
      height="2"
      fill={BLUE}
      transform="translate({700}, {377})"
    />
    <text x="716" y="382" fill={BLUE}>Flash</text>
    <rect
      width="10"
      height="2"
      fill="orange"
      transform="translate({700}, {396})"
    />
    <text x="716" y="401" fill="orange">SSD</text>

    <!-- title -->
    <text
      class="title"
      x={width / 2 + 35}
      y="0"
      dy="-1em"
      fill="black"
      font-size="14px"
      transform="translate({-370}, {margin.top - 20})"
    >
      Plotting the Cost of Components Through Time (Logarithmic Scale)
    </text>
    <!-- x-axis -->
    <g
      class="axis"
      bind:this={gx}
      transform="translate(2,{height - margin.bottom})"
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
    <g class="axis" bind:this={gy} transform="translate({margin.left + 2},{0})">
      <text
        class="axis-labels"
        x="0"
        y="0"
        dy="-1em"
        fill="black"
        font-size="14px"
        transform="translate(-70, {height / 2 - 50}) rotate(-90)"
      >
        Cost ($/TB)
      </text>
    </g>

    <!-- adding dots and lines -->
    <g stroke="black" stroke-width="1.5">
      {#each cost_data.slice(0, -1) as d, i}
        {@const nextNonZeroMemory = findNextNonZero(cost_data, i + 1, "memory")}
        {#if nextNonZeroMemory !== null}
          <path
            d={"M" +
              x(d.date) +
              " " +
              y(d.memory) +
              " " +
              "L" +
              x(nextNonZeroMemory.date) +
              " " +
              y(nextNonZeroMemory.value)}
            fill="none"
            stroke={RED}
            stroke-width="2"
          />
        {/if}
        {@const nextNonZeroFlash = findNextNonZero(cost_data, i + 1, "flash")}
        {#if nextNonZeroFlash !== null}
          <path
            d={"M" +
              x(d.date) +
              " " +
              y(d.flash) +
              " " +
              "L" +
              x(nextNonZeroFlash.date) +
              " " +
              y(nextNonZeroFlash.value)}
            fill="none"
            stroke={BLUE}
            stroke-width="2"
          />
        {/if}
        {@const nextNonZeroDrives = findNextNonZero(cost_data, i + 1, "drives")}
        {#if nextNonZeroDrives !== null}
          <path
            d={"M" +
              x(d.date) +
              " " +
              y(d.drives) +
              " " +
              "L" +
              x(nextNonZeroDrives.date) +
              " " +
              y(nextNonZeroDrives.value)}
            fill="none"
            stroke={GREEN}
            stroke-width="2"
          />
        {/if}
        {@const nextNonZeroSsd = findNextNonZero(cost_data, i + 1, "ssd")}
        {#if nextNonZeroSsd !== null}
          <path
            d={"M" +
              x(d.date) +
              " " +
              y(d.ssd) +
              " " +
              "L" +
              x(nextNonZeroSsd.date) +
              " " +
              y(nextNonZeroSsd.value)}
            fill="none"
            stroke="orange"
            stroke-width="2"
          />
        {/if}
      {/each}
      {#each cost_data as d, i}
        {#if d.memory != 0}
          <circle
            stroke={RED}
            fill={RED}
            key={i}
            cx={x(d.date)}
            cy={y(d.memory)}
            r="1.5"
            on:mousemove={() => setSelected(d)}
          />
        {/if}
      {/each}
      {#each cost_data as d, i}
        {#if d.flash != 0}
          <circle
            stroke={BLUE}
            fill={BLUE}
            key={i}
            cx={x(d.date)}
            cy={y(d.flash)}
            r="1.5"
            on:mousemove={() => setSelected(d)}
          />
        {/if}
      {/each}
      {#each cost_data as d, i}
        {#if d.drives != 0}
          <circle
            stroke={GREEN}
            fill={GREEN}
            key={i}
            cx={x(d.date)}
            cy={y(d.drives)}
            r="1.5"
            on:mousemove={() => setSelected(d)}
          />
        {/if}
      {/each}
      {#each cost_data as d, i}
        {#if d.ssd != 0}
          <circle
            stroke="orange"
            fill="orange"
            key={i}
            cx={x(d.date)}
            cy={y(d.ssd)}
            r="1.5"
            on:mousemove={() => setSelected(d)}
          />
        {/if}
      {/each}

      {#if selectedPoint && mousePosition[1] < 450}
        <!-- added bigger dots when hovered over -->
        {#if selectedPoint.memory != 0}
          <circle
            cx={x(selectedPoint.date)}
            cy={y(selectedPoint.memory)}
            r="5"
            fill={RED}
          />
        {/if}
        {#if selectedPoint.flash != 0}
          <circle
            cx={x(selectedPoint.date)}
            cy={y(selectedPoint.flash)}
            r="5"
            fill={BLUE}
          />
        {/if}
        {#if selectedPoint.drives != 0}
          <circle
            cx={x(selectedPoint.date)}
            cy={y(selectedPoint.drives)}
            r="5"
            fill={GREEN}
          />
        {/if}
        {#if selectedPoint.ssd != 0}
          <circle
            cx={x(selectedPoint.date)}
            cy={y(selectedPoint.ssd)}
            r="5"
            fill="orange"
          />
        {/if}

        <!-- added vertical line -->
        <rect
          class="vertical-line"
          width="1"
          height={310}
          stroke="gray"
          stroke-opacity="0.1"
          fill-opacity="0.1"
          transform="translate({x(selectedPoint.date)}, {margin.top + 10})"
        />

        <!-- added tooltip -->
        <div class="main-tooltip"></div>
        {#if mousePosition[0] < margin.right + 300 && mousePosition[0] > margin.left && mousePosition[1] > 190 && mousePosition[1] < 420}
          {@const values = getToolTipText(selectedPoint)}
          <g
            class="tooltip"
            transform="translate({mousePosition[0] + 25},{mousePosition[1] -
              tooltipH +
              5})"
          >
            <rect
              class="top-rect"
              width={tooltipW - 35}
              height={tooltipH - 20}
              fill="#F0F0F0"
              stroke="black"
              opacity="1"
            />
            <text class="tooltip-year" x={10} dy={20} stroke="red"
              >{values.year}</text
            >
            <text
              class="tooltip-name"
              x={52}
              dy={20}
              stroke="black"
              stroke-width="0.5"
              font-size="14px"
            >
              in $ per TB
            </text>
            {#each Object.entries(values) as [name, value], i}
              {#if name !== "year"}
                <text class="tooltip-name" x={10} dy={i * 25 + 20}
                  >{name}: ${value}</text
                >
              {/if}
            {/each}
          </g>
        {:else if mousePosition[0] >= margin.right + 300 && mousePosition[0] < width - margin.right - 25 && mousePosition[1] > 190 && mousePosition[1] < 420}
          {@const values = getToolTipText(selectedPoint)}
          <g
            class="tooltip"
            transform="translate({mousePosition[0] -
              tooltipW +
              15},{mousePosition[1] - tooltipH + 5})"
          >
            <rect
              class="top-rect"
              width={tooltipW - 35}
              height={tooltipH + 5}
              fill="#F0F0F0"
              stroke="black"
              opacity="1"
            />
            <text class="tootip-year" x={10} dy={20} stroke="red"
              >{values.year}</text
            >
            <text
              class="tooltip-name"
              x={52}
              dy={20}
              stroke="black"
              stroke-width="0.5"
              font-size="14px"
            >
              in $ per TB
            </text>
            {#each Object.entries(values) as [name, value], i}
              {#if name !== "year"}
                <text class="tooltip-name" x={10} dy={i * 25 + 20}
                  >{name}: ${value}</text
                >
              {/if}
            {/each}
          </g>
        {:else if mousePosition[0] < margin.right + 300 && mousePosition[0] > margin.left && mousePosition[1] <= 190 && mousePosition[1] > 90}
          {@const values = getToolTipText(selectedPoint)}
          <g
            class="tooltip"
            transform="translate({mousePosition[0] + 25},{mousePosition[1] +
              25})"
          >
            <rect
              class="toolrect"
              width={tooltipW - 30}
              height={tooltipH - 20}
              fill="#F0F0F0"
              stroke="black"
            />
            <text class="tooltip-year" x={10} dy={20} stroke="red"
              >{values.year}</text
            >
            <text
              class="tooltip-name"
              x={52}
              dy={20}
              stroke="black"
              stroke-width="0.5"
              font-size="14px"
            >
              in $ per TB
            </text>
            {#each Object.entries(values) as [name, value], i}
              {#if name !== "year"}
                <text class="tooltip-name" x={10} dy={i * 25 + 20}
                  >{name}: ${value}</text
                >
              {/if}
            {/each}
          </g>
        {:else if mousePosition[0] >= margin.right + 300 && mousePosition[0] < width - margin.right - 25 && mousePosition[1] <= 190 && mousePosition[1] > 90}
          {@const values = getToolTipText(selectedPoint)}
          <g
            class="tooltip"
            transform="translate({mousePosition[0] -
              tooltipW -
              15},{mousePosition[1] + 25})"
          >
            <rect
              class="toolrect"
              width={tooltipW - 5}
              height={tooltipH + 5}
              fill="#F0F0F0"
              stroke="black"
            />
            <text class="tooltip-year" x={10} dy={20} stroke="red"
              >{values.year}</text
            >
            <text
              class="tooltip-name"
              x={52}
              dy={20}
              stroke="black"
              stroke-width="0.5"
              font-size="14px"
            >
              in $ per TB
            </text>
            {#each Object.entries(values) as [name, value], i}
              {#if name !== "year"}
                <text class="tooltip-name" x={10} dy={i * 25 + 20}
                  >{name}: ${value}</text
                >
              {/if}
            {/each}
          </g>
        {/if}
      {/if}
    </g>
  </svg>
</div>

<style>
  @import url("https://fonts.googleapis.com/css2?family=Lato:ital,wght@0,100;0,300;0,400;1,100;1,300;1,400&display=swap");
  .container {
    width: 100%;
    height: 100%;
    display: flex;
    flex-flow: column wrap;
    justify-content: center;
    align-items: center;
    visibility: hidden;
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
  .spacer {
    height: 10%;
  }
</style>
