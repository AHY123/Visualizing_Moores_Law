<script>
    import * as d3 from "d3";
    import ButtonComponent from "./small_components/ButtonComponent.svelte";

    export let cost_data;

    $: console.log(cost_data);

    let data = [];
    let gort = "GB";
    let storage_size = 5;
    let input_year = 1956;
    let price;

    let is_reveal = false;
    let reveal_text = "Show Answer !";
    let ready_text = "Choose a valid year (1956 - 2022)";

    $: if (input_year) {
        is_reveal = false;
    }

    function swap_reveal() {
        if (1956 <= input_year && input_year <= 2022) {
            is_reveal = !is_reveal;
        }
    }

    $: {
        if (1956 <= input_year && input_year <= 2022) {
            ready_text = "Ready?";
        } else {
            ready_text = "Choose a valid year (1956 - 2022)";
        }
    }

    $: {
        if (is_reveal) {
            reveal_text = "Hide Answer !";
        } else {
            reveal_text = "Show Answer !";
        }
    }

    const bisect = d3.bisector((d) => d.date).center;
    function formatNumber(value) {
        if (value >= 1e12) {
            return (value / 1e12).toFixed(0) + " trillion";
        } else if (value >= 1e9) {
            return (value / 1e9).toFixed(0) + " billion";
        } else if (value >= 1e6) {
            return (value / 1e6).toFixed(0) + " million";
        } else {
            return value.toString();
        }
    }

    $: {
        if (cost_data[0] != null) {
            for (let i = 0; i < cost_data.length; i++) {
                if (cost_data[i].drives != 0) {
                    data.push({
                        date: cost_data[i].date,
                        drives: cost_data[i].drives,
                    });
                }
            }
            let closest_index = bisect(data, new Date(input_year, 0, 1), 0);
            // console.log("closest index: " + closest_index);
            // console.log("closest date: " + data[closest_index].date);
            // console.log("closest price: " + data[closest_index].drives);
            if (gort == "GB") {
                price = formatNumber((data[closest_index].drives * storage_size) / 1000);
            } else {
                price = formatNumber(data[closest_index].drives * storage_size);
            }
            // console.log(price);
        }
    }
</script>

<div class="container">
    <div class="center">
        <div>
            <h1>Before we show you the graph, let's play a game first!</h1>
        </div>
        <div>
            <h2>Input below how much storage you have on your device:</h2>
        </div>
        <div>
            <input
                placeholder="enter storage size"
                bind:value={storage_size}
                size="20"
                maxlength="20"
            />
            <select bind:value={gort}>
                <option value="GB">GB</option>
                <option value="TB">TB</option>
            </select>
        </div>
        {#if !storage_size}
            <div><h2>Waiting for input ...</h2></div>
        {:else}
            <div>
                <h2>
                    How much do you think {storage_size}
                    {gort} costed in
                    <input
                        placeholder="input year"
                        size="8"
                        bind:value={input_year}
                    /> ?
                </h2>
            </div>
            <ButtonComponent on:click={swap_reveal} text={reveal_text} />
            <hr color="white" />
            <div class="answer_block">
                {#if !is_reveal}
                    <div class="container"><h1>{ready_text}</h1></div>
                {:else}
                    <div class="container">
                        <h1>
                            In {input_year} {storage_size}
                            {gort} worth of hardrives would cost approximatedly {price} dollars!
                        </h1>
                    </div>
                {/if}
            </div>
        {/if}
    </div>
</div>

<style>
    .container {
        height: 100%;
        width: 100%;
        display: flex;
        flex-flow: column;
        justify-content: center;
        align-items: center;
    }
    .center {
        height: 80%;
        width: 60%;
    }
    .answer_block {
        height: 60%;
        width: 100%;
        outline: black solid 3px;
    }
    h1 {
        font-size: 24px;
    }
    h2 {
        font-size: 16px;
    }
</style>
