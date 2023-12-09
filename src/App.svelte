<script lang="ts">
    import Contact from "./Contact.svelte";
    import Info from "./Info.svelte";
    import TitleBox from "./TitleBox.svelte";
</script>

<main>

    <TitleBox/>
    <Info/>

    <div class="lineBreak"/>

    <Contact/>

</main>

<style>

    main {

        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;

    }

    .lineBreak {
        width: 460px;
        height: 1px;
        margin-top: 5px;
        margin-bottom: 5px;
        background-color: #000;
    }

    @media only screen and (max-width: 600px) {

        .lineBreak {
            width: 90%;
        }

    }

</style>
