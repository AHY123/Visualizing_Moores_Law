<script>
    export let text;
</script>

<div class='container'>
    <p>
        {text.text}
    </p>
</div>

<style>
    p {
        font-size: 32px;
        text-wrap: balance;
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