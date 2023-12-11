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

        width: 460px;

    }

    .lineBreak {
        height: 1px;
        margin-top: 5px;
        margin-bottom: 5px;
        background-color: #000;

        animation: widen 1s cubic-bezier(0.76, 0, 0.24, 1);
        animation-fill-mode: forwards;

    }

    @keyframes widen {
        0% {
            width: 0px;
        }
        100% {
            width: 460px;
        }
    }

    @keyframes widenMobile {
        0% {
            width: 0px;
        }
        100% {
            width: 90%;
        }
    }

    @media only screen and (max-width: 600px) {

        .lineBreak {
            animation: widenMobile 1s cubic-bezier(0.76, 0, 0.24, 1);
            animation-fill-mode: forwards;
        }

    }

</style>
