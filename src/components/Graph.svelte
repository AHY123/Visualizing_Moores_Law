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
    left: 80,
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
        .selectAll("tick line")
        .clone()
        .attr("x2", width - margin.right - margin.left)
        .attr("stroke-opacity", 0.1)
    );

  function transition(path) {
    path
      .transition()
      .duration(750)
      .attrTween("stroke-dasharray", tweenDash)
      .on("end", () => {
        d3.select(this).call(transition);
      });
  }

  function tweenDash() {
    const l = this.getTotalLength(),
      i = d3.interpolateString("0," + l, l + "," + l);
    return function (t) {
      return i(t);
    };
  }

  let isVisible = false;

  $: if (index === 0) {
    isVisible = true;
  }
</script>

<main>
  <svg
    class="graph"
    class:visible={isVisible}
    this="{svg}{width}"
    {height}
    viewBox="0 0 {width} {height}"
  >
    <!-- x-axis -->
    <g bind:this={gx} transform="translate(0,{height - margin.bottom})">
      <text
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
    <g bind:this={gy} transform="translate({margin.left},0)">
      <text
        x="0"
        y="0"
        dy="-1em"
        fill="black"
        font-size="14px"
        transform="translate(-40, {height / 2 - 70}) rotate(-90)"
      >
        Transistor(s)
      </text>
    </g>
    <g stroke="steelblue" stroke-width="1.5">
      {#each data as d, i}
        {#if i != data.length - 1}
          <line
            x1={x(data[i].date)}
            x2={x(data[i + 1].date)}
            y1={y(data[i].value)}
            y2={y(data[i + 1].value)}
            stroke="black"
            stroke-width="2"
          />
        {/if}
        <circle key={i} cx={x(d.date)} cy={y(d.value)} r="2.5" />
      {/each}
    </g>
  </svg>
</main>

<style>
  .graph.visible {
    opacity: 1;
    visibility: visible;
  }
</style>
