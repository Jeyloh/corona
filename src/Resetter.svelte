<script>
    import { count } from "./stores.js";
    
    const reset = () => $count = 0;

</script>


<button on:click={reset}>Reset</button>


<style>
</style>