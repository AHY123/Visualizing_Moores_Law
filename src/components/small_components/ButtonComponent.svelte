<script>
    // Declaring props
    export let text;
</script>

<button on:click>
    {text}
</button>

<style>
    button {
        /* border: black; */
        stroke: black;
        padding: 0.5rem 2rem;
        color: black;
        font-size: 1.5rem;
        /* border-radius: 0.5rem; */
        transition: all 250ms;
        transform-origin: center;
        /* inset: 0px -2px 3px rgba(0, 0, 0, 0.25); */
    }
    button:hover {
        cursor: pointer;
        transform: scale(0.975);
    }
</style>