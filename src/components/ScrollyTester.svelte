<script>
    import Scroller from "@sveltejs/svelte-scroller";

    let count,
        index,
        offset,
        progress = 0;
    
    let lol = [0, 1, 2, 3, 4, 5]
</script>

<main>
    <Scroller
        top={0.0}
        bottom={1}
        threshold={0.55}
        bind:count
        bind:index
        bind:offset
        bind:progress
    >
        <div class="foreground" slot="foreground">
            <!-- <div> -->
                {#each lol as i}
                <section>{i}</section>
                {/each}
            <!-- </div> -->
        </div>
        <div class="background" slot="background">
            <div class="progress-bars">
                <p>current section: <strong>{index + 1}/{count}</strong></p>
                <progress value={count ? (index + 1) / count : 0} />
                <p>offset in current section</p>
                <progress value={offset || 0} />
                <p>total progress</p>
                <progress value={progress || 0} />
            </div>
        </div>
    </Scroller>
</main>
<style>
    main {
        font-family: "Lato", sans-serif;
        
    }
    .foreground {
        height: 100vh;
        width: 70%;
        margin: 0 auto;
        /* height: auto; */
        /* position: relative; */

        /* overflow: auto;
        scroll-snap-type: y mandatory; */
    }
    .background {
        width: 100%;
        height: 100vh;
        position: relative;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-flow: column wrap;
    }
    section {
        height: 99vh;
        background-color: rgba(0, 0, 0, 0);
        color: white;
        text-align: center;
        max-width: 1000%;
        color: black;
        padding: 1em;
        margin: 0 0 2em 0;
        align-items: center;

        position: sticky;
        top: 10%;
        outline: magenta solid 3px;
    }
    section {
        scroll-snap-align: start;
    }
</style>
