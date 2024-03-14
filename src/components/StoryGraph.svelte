<script>
    // imports
    import * as d3 from "d3";
    import DoubleRangeSlider from "./small_components/DoubleRangeSlider.svelte";
    import { onMount } from "svelte";

    // exports
    export let data = [];
    export let display_mode = "linear";

    // svg var
    let svg, gx, gy;
    let height = 500;
    let width = 800;

    const margin = {
        top: 100,
        right: 80,
        bottom: 80,
        left: 130,
    };

    let isVisible = true;
    let title;

    // double range slider var
    let start;
    let end;
    let min = 1970;
    let max = 2021;

    // plot_data
    $: round_convert = (d) => {
        let x = (max - min) * d + min;
        if (!x && x !== 0) return "";
        return x.toFixed(0);
    };

    // change plot_data based on slider
    let plot_data = [];
    $: {
        // console.log(data);
        plot_data = [];
        for (let i = 0; i < data.length; i++) {
            let data_point = data[i];
            // console.log([start, end]);
            if (
                (round_convert(start) <= data_point.date.getFullYear()) &
                (round_convert(end) >= data_point.date.getFullYear())
            ) {
                plot_data.push(data_point);
            }
        }
        // console.log(plot_data);
    }

    // x-axis
    $: x = d3
        .scaleTime()
        .domain([
            d3.min(plot_data, (d) => d.date),
            d3.max(plot_data, (d) => d.date),
        ])
        .range([margin.left, width - margin.right]);

    // y-axis
    let y;
    $: {
        if (display_mode == "linear") {
            y = d3
                .scaleLinear()
                .domain([0, d3.max(plot_data, (d) => d.value)])
                .nice()
                .range([height - margin.bottom, margin.top]);
        } else if (display_mode == "log") {
            y = d3
                .scaleLog()
                .domain([2308, 100000000000])
                .range([height - margin.bottom, margin.top]);
        }
    }

    // x-axis label
    $: d3.select(gx).call(d3.axisBottom(x).ticks(width / 80));

    const linearFormatYAxis = (value) => {
        if (value >= 1e9) {
            return `${value / 1e9} billion`;
        } else {
            return value;
        }
    };

    const logFormatYAxis = (value) => {
        if (value >= 1e9) {
            return `${value / 1e9} billion`;
        } else if (value >= 1e6) {
            return `${value / 1e6} million  `;
        } else {
            return value;
        }
    };

    $: {
        if (display_mode == "linear") {
            title =
                "Plotting Number of Transistors on a Microprocessor Through Time";
            d3.select(gy)
                .call(
                    d3
                        .axisLeft(y)
                        .tickFormat(linearFormatYAxis)
                        .ticks(null, "s"),
                )
                .call((g) =>
                    g
                        .selectAll(".tick line")
                        .attr("x2", width - margin.right - margin.left)
                        .attr("stroke-opacity", 0.1),
                );
        } else if (display_mode == "log") {
            title =
                "Plotting Number of Transistors on a Microprocessor Through Time (Log Scale)";
            d3.select(gy)
                .call(d3.axisLeft(y).tickFormat(logFormatYAxis).ticks(7))
                .call((g) =>
                    g
                        .selectAll(".tick line")
                        .attr("x2", width - margin.right - margin.left)
                        .attr("stroke-opacity", 0.1),
                );
        }
    }

    let mousePosition = [0, 0];
    let selectedPoint = null;

    const bisect = d3.bisector((d) => d.date).center;

    function setSelected(event) {
        mousePosition = d3.pointer(event);
        if (
            (margin.left < mousePosition[0] && mousePosition[0] < width - margin.right) &&
            (margin.top < mousePosition[1] && mousePosition[1] < height - margin.bottom)
            ) {
            const x0 = x.invert(mousePosition[0]);
            const i = bisect(plot_data, x0, 0);
            selectedPoint = {
                date: plot_data[i].date,
                value: plot_data[i].value,
            };
            // console.log(selectedPoint);
        }
        else {
            selectedPoint = null;
        }
    }

    const tooltipW = 210;
    const tooltipH = 80;
    const tooltipPaddingTop = 10;
    const tooltipPaddingLeft = 10;
    let tooltip_pos = [0, 0];
    $: {
        if (mousePosition[1] < margin.top + tooltipH + tooltipPaddingTop) {
            tooltip_pos[1] = mousePosition[1] + tooltipPaddingTop;
        } else {
            tooltip_pos[1] = mousePosition[1] - tooltipH - tooltipPaddingTop;
        }
        if (mousePosition[0] < margin.left + tooltipW + tooltipPaddingLeft) {
            tooltip_pos[0] = mousePosition[0] + tooltipPaddingLeft;
        } else {
            tooltip_pos[0] = mousePosition[0] - tooltipW - tooltipPaddingLeft;
        }
    }
</script>

<div class="container" class:visible={isVisible}>
    <div class='spacer'></div>
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <svg
        class="graph"
        style="border:3px solid black"
        class:visible={isVisible}
        this="{svg}{width}"
        {height}
        viewBox="0 0 {width} {height}"
        on:mousemove={setSelected}
    >
        <!-- title -->
        <text
            class="title"
            x={width / 2}
            y="0"
            dy="-1em"
            fill="black"
            font-size="14px"
            transform="translate({-370}, {margin.top - 20})"
        >
            {title}
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
        <g class="axis" bind:this={gy} transform="translate({margin.left},0)">
            <text
                class="axis-labels"
                x="0"
                y="0"
                dy="-1em"
                fill="black"
                font-size="14px"
                transform="translate(-70, {height / 2 - 40}) rotate(-90)"
            >
                Transistor(s)
            </text>
        </g>
        <g stroke="black" stroke-width="1.5">
            <!-- Connecting Lines -->
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
            <!-- Data Points -->
            {#each data as d, i}
                <circle
                    stroke="#80b1d3"
                    fill="#80b1d3"
                    key={i}
                    cx={x(d.date)}
                    cy={y(d.value)}
                    r="2"
                    on:mousemove={() => setSelected(d)}
                />
            {/each}
        </g>
        <!-- tooltips -->
        {#if selectedPoint}
            <!-- Vertical Line -->
            <rect
                class="vertical-line"
                width="1"
                height={height - margin.top - margin.bottom}
                stroke="gray"
                stroke-opacity="0.1"
                fill-opacity="0.1"
                transform="translate({x(selectedPoint.date)}, {margin.top})"
            />
            <!-- Highlight Point -->
            <circle
                cx={x(selectedPoint.date)}
                cy={y(selectedPoint.value)}
                r="5"
                fill="#fb8072"
            />
            <g
                class="tooltip"
                transform="translate({tooltip_pos[0]},{tooltip_pos[1]})"
            >
                <rect
                    class="toolrect"
                    width={tooltipW}
                    height={tooltipH}
                    fill="#F0F0F0"
                    stroke="black"
                />
                <g
                    transform="translate({tooltipPaddingLeft},{tooltipPaddingTop +
                        20})"
                >
                    <text class="tooltip-year" stroke="#fb8072" fill="#fb8072">
                        {selectedPoint.date.getYear() + 1901}
                    </text>
                    <text class="tooltip-name" stroke="black" y={30}>
                        {#if selectedPoint.value > 1e9}
                            Transistors: {(selectedPoint.value / 1e9).toFixed(
                                2,
                            )} billion
                        {:else if selectedPoint.value > 1e6}
                            Transistors: {(selectedPoint.value / 1e6).toFixed(
                                2,
                            )} million
                        {:else}
                            Transistors: {selectedPoint.value}
                        {/if}
                    </text>
                </g>
            </g>
        {/if}
    </svg>
    <div class="svg_width">
        <hr />

        <DoubleRangeSlider bind:start bind:end />
        <div class="labels">
            <div class="label">{round_convert(start)}</div>
            <div class="label">{round_convert(end)}</div>
        </div>
        <hr />
        <div class="options">
            <div class="label">Choose Graph Display Mode</div>
            <select bind:value={display_mode} class="select label">
                <option value="linear">Linear Scale</option>
                <option value="log">Logarithmic Scale</option>
                <option value="other">Others</option>
            </select>
        </div>
    </div>
</div>

<style>
    .container {
        width: 100%;
        height: 100%;
        display: flex;
        flex-flow: column wrap;
        justify-content: center;
        align-items: center;
        visibility: hidden;
    }
    .container.visible {
        visibility: visible;
    }
    .svg_width {
        width: 800px;
        display: flex;
        flex-flow: column wrap;
    }
    .label:first-child {
        float: left;
    }
    .label:last-child {
        float: right;
    }
    .options {
        width: 50%;
    }
    .title {
        font-family: "Lato", serif;
        font-size: 22px;
    }
    .spacer {
        height: 10%;
    }
</style>
