<script lang="ts">
  import NameInfo from "./NameInfo.svelte";
</script>

<div>

    <div id="InfoDisplay">I'm <NameInfo/>, I develop</div>
    <p>
        Websites, Games, Experiences, and software of all types.
    </p>

</div>

<style>

    div {
        font-family: 'Open Sans', sans-serif;
    }

    p {
        margin: 0px;
        text-align: center;
    }

    #InfoDisplay {
        display: flex;
        justify-content: center;
    }

</style>