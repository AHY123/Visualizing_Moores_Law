<script>
    // imports
    import * as d3 from "d3";
    import tv_data from "./tv_data.json";

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

    let data = tv_data;

    const formatYAxis = (value) => {
        return value;
    };
    const formatXAxis = (value) => {
        return value;
    };

    $: x = d3
        .scaleLinear()
        .domain([1950, 1995])
        .range([margin.left, width - margin.right]);

    $: y = d3
        .scaleLinear()
        .domain([0, d3.max(data, (d) => d.inf_adjusted_price)])
        .nice()
        .range([height - margin.bottom, margin.top]);

    $: d3.select(gx).call(
        d3
            .axisBottom(x)
            .tickFormat(formatXAxis)
            .ticks(width / 80),
    );

    $: d3.select(gy)
        .call(d3.axisLeft(y).tickFormat(formatYAxis).ticks(null, "s"))
        .call((g) =>
            g
                .selectAll(".tick line")
                .attr("x2", width - margin.right - margin.left)
                .attr("stroke-opacity", 0.1),
        );

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
    let mousePosition = [0, 0];
    let selectedPoint = null;

    const bisect = d3.bisector((d) => d.year).center;

    function setSelected(event) {
        mousePosition = d3.pointer(event);
        if (
            margin.left < mousePosition[0] &&
            mousePosition[0] < width - margin.right &&
            margin.top < mousePosition[1] &&
            mousePosition[1] < height - margin.bottom
        ) {
            const x0 = x.invert(mousePosition[0]);
            const i = bisect(data, x0, 0);
            selectedPoint = {
                year: data[i].year,
                inf_adjusted_price: data[i].inf_adjusted_price,
            };
            // console.log(selectedPoint);
        } else {
            selectedPoint = null;
        }
    }
</script>

<div class="container">
    <div class="spacer"/>
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <svg
        class="graph"
        style="border:3px solid black"
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
            font-size="22px"
            transform="translate({-120}, {margin.top - 30})"
        >
            Tv Prices Through Time
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
                Price ($)
            </text>
        </g>
        <!-- original price-->
        {#each data.slice(0, -1) as d, i}
            <path
                d={"M" +
                    x(d.year) +
                    "," +
                    y(d.original_price) +
                    " " +
                    "L" +
                    x(data[i + 1].year) +
                    "," +
                    y(data[i + 1].original_price)}
                fill="none"
                stroke="gray"
                stroke-dasharray="5 2"
                stroke-width="2"
            />
        {/each}
        {#each data as d, i}
            <circle
                stroke="#bebada"
                fill="#bebada"
                key={i}
                cx={x(d.year)}
                cy={y(d.original_price)}
                r="2"
                on:mousemove={() => setSelected(d)}
            />
        {/each}
        <!-- Connecting Lines -->
        {#each data.slice(0, -1) as d, i}
            <path
                d={"M" +
                    x(d.year) +
                    "," +
                    y(d.inf_adjusted_price) +
                    " " +
                    "L" +
                    x(data[i + 1].year) +
                    "," +
                    y(data[i + 1].inf_adjusted_price)}
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
                cx={x(d.year)}
                cy={y(d.inf_adjusted_price)}
                r="3"
                on:mousemove={() => setSelected(d)}
            />
        {/each}

        <!-- Legend -->
        <circle cx={240} cy={78} r="5" fill="#bebada" />
        <text x="250" y="83" fill="#bebada" font-size="14px">
            Original Price
        </text>
        <circle cx={450} cy={78} r="5" fill="#80b1d3" />
        <text x="460" y="83" fill="#80b1d3" font-size="14px">
            Inflation Adjusted Price
        </text>

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
                transform="translate({x(selectedPoint.year)}, {margin.top})"
            />
            <!-- Highlight Point -->
            <circle
                cx={x(selectedPoint.year)}
                cy={y(selectedPoint.inf_adjusted_price)}
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
                        {selectedPoint.year}
                    </text>
                    <text class="tooltip-name" stroke="black" y={30}>
                        Price: {selectedPoint.inf_adjusted_price}
                    </text>
                </g>
            </g>
        {/if}
    </svg>
</div>

<style>
    .container {
        width: 100%;
        height: 100%;
        display: flex;
        flex-flow: column;
        justify-content: center;
        align-items: center;
    }
    .spacer {
        height: 10%;
    }
</style>
