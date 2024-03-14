<script>
    export let text;
</script>

<div class='container'>
    <h1>
        {text.subtitle}
    </h1>
    <h2>
        {text.text}
    </h2>
</div>

<style>
    h1 {
        font-size: 90px;
    }
    h2 {
        font-size: 50px;
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
</style>