<script>
    import * as d3 from "d3";
    // import ButtonComponent from "./small_components/ButtonComponent.svelte";

    // svg var
    let svg, gx, gy;
    let height = 500;
    let width = 800;

    let item_price;

    let data = [];
    $: {
        if (item_price) {
            data = [];
            for (let i = 2024; i <= 2824; i += 20) {
                data.push({
                    year: i,
                    price: item_price * Math.exp(-0.061 * (i - 2024)),
                });
            }
        }
    }

    const margin = {
        top: 100,
        right: 80,
        bottom: 80,
        left: 130,
    };

    const formatPrice = (value) => {
        if (value >= 1e18) {
            return (value / 1e18).toFixed(8) + " quintillion";
        } else if (value >= 1e15) {
            return (value / 1e15).toFixed(8) + " quadrillian";
        } else if (value >= 1e12) {
            return (value / 1e12).toFixed(8) + " trillion";
        } else if (value >= 1e9) {
            return (value / 1e9).toFixed(8) + " billion";
        } else if (value >= 1e6) {
            return (value / 1e6).toFixed(8) + " million";
        } else {
            return value.toFixed(4);
        }
    };

    const formatYAxis = (value) => {
        if (value >= 1e18) {
            return value / 1e18 + " quintillion";
        } else if (value >= 1e15) {
            return value / 1e15 + " quadrillian";
        } else if (value >= 1e12) {
            return value / 1e12 + " trillion";
        } else if (value >= 1e9) {
            return value / 1e9 + " billion";
        } else if (value >= 1e6) {
            return value / 1e6 + " million";
        } else {
            return value;
        }
    };

    const formatXAxis = (value) => {
        return value;
    };

    $: x = d3
        .scaleLinear()
        .domain([2020, 2825])
        .range([margin.left, width - margin.right]);

    $: y = d3
        .scaleLinear()
        .domain([0, d3.max(data, (d) => d.price)])
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

    const tooltipW = 230;
    const tooltipH = 110;
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
        console.log("tryign to sellect");
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
                price: data[i].price,
            };
            console.log(selectedPoint);
        } else {
            selectedPoint = null;
        }
    }
</script>

<div class="container">
    <div class="spacer" />
    <div>
        <h1>TOTALLY ACCURATE TECH PRICE PREDICTOR (TATPP®)</h1>
    </div>
    <div>
        Enter the price of your tech item:
        <input
            placeholder="enter tech item price"
            bind:value={item_price}
            size="20"
            maxlength="19"
        />
    </div>
    <div class="spacer" />
    <div class="svg_container">
        {#if !item_price}
            <div class="container">
                <h1>{"Input Item Price to See Prediction"}</h1>
            </div>
        {:else}
            <!-- svelte-ignore a11y-no-static-element-interactions -->
            <svg
                class="graph"
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
                    transform="translate({-100}, {margin.top - 20})"
                >
                    Item Price Prediction
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
                <g
                    class="axis"
                    bind:this={gy}
                    transform="translate({margin.left},0)"
                >
                    <text
                        class="axis-labels"
                        x="0"
                        y="0"
                        dy="-1em"
                        fill="black"
                        font-size="14px"
                        transform="translate(-80, {height / 2 -
                            40}) rotate(-90)"
                    >
                        Price ($)
                    </text>
                </g>
                <!-- Connecting Lines -->
                {#each data.slice(0, -1) as d, i}
                    <path
                        d={"M" +
                            x(d.year) +
                            "," +
                            y(d.price) +
                            " " +
                            "L" +
                            x(data[i + 1].year) +
                            "," +
                            y(data[i + 1].price)}
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
                        cy={y(d.price)}
                        r="3"
                        on:mousemove={() => setSelected(d)}
                    />
                {/each}
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
                        cy={y(selectedPoint.price)}
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
                            <text
                                class="tooltip-year"
                                stroke="#fb8072"
                                fill="#fb8072"
                            >
                                {selectedPoint.year}
                            </text>
                            <text class="tooltip-name" stroke="black" y={30}>
                                In {selectedPoint.year - 2024} Years
                            </text>
                            <text class="tooltip-name" stroke="black" y={60}>
                                Price: {formatPrice(selectedPoint.price)}$
                            </text>
                        </g>
                    </g>
                {/if}
            </svg>
        {/if}
    </div>
</div>

<style>
    .svg_container {
        height: 500px;
        width: 800px;
        outline: black solid 3px;
        display: flex;
        flex-flow: column;
        justify-content: center;
        align-items: center;
    }
    .container {
        width: 100%;
        height: 100%;
        display: flex;
        flex-flow: column;
        justify-content: center;
        align-items: center;
    }
    h1 {
        font-size: 24px;
    }
    .spacer {
        height: 3%;
    }
</style>
