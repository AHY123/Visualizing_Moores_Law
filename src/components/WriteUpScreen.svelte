<script>
    import * as d3 from "d3";
    // import { onMount } from "svelte";

    // export let index;

    let isVisible = true;

    // $: if (index == 3) {
    //     isVisible = true;
    // } else {
    //     isVisible = false;
    // }
    // onMount(() => {

    // })
</script>

<div class="container visible" class:visible={isVisible}>
    <h1>Write-Up</h1>
    <h2>
        We looked through the datasets and started cleaning them, making sure
        they reveal the relationship that we wanted to show between things like
        transistors and years. Similarly, we’ve also implemented two of our
        visualizations, both of which are interactive. Additionally, we added
        our scrolling page and are working on the transitions as well as timers
        between the pages to make sure it’s working smoothly. Finally, we’ve
        been working on our main writeup for the explanation and looking through
        additional graphs that we can add to make our page more interesting and
        interactive overall.
    </h2>
    <h2>
        The most challenging part of our project will likely be the combination
        between the page setup as well as creating different types of
        visualization. We want to make sure not to overload the viewer but also
        put enough content for a 3 minute video, so we need to work that fine
        line. Furthermore, we want to also identify key visualizations other
        than line graphs that would make it easier to understand the complex
        topic that we want to talk about. Finally, we think that the Svelte
        portion also might be a little difficult, especially if we are working
        on graphs that we haven’t looked at before in lecture or lab.
    </h2>
</div>

<style>
    h1 {
        font-size: 36px;
    }
    h2 {
        font-size: 24px;
    }
    .container {
        width: 100%;
        height: 80%;
        display: flex;
        flex-flow: column wrap;
        justify-content: center; /* Center horizontally */
        align-items: center; /* Center vertically */
        visibility: visible;
    }
    .container.visible {
        visibility: visible;
        animation: fadeIn 3s;
    }
    @keyframes fadeIn {
        0% {
            opacity: 0;
        }
        100% {
            opacity: 1;
        }
    }
</style>
