<script lang="ts">
    import Icon from '@iconify/svelte';

    function OnClick(RedicrectURL: String)
    {
        return function(){
            window.open(RedicrectURL.toString(), "_blank")
        }
    }
</script>

<div>

    <button style="animation-delay: 500ms;" on:click={OnClick("https://discord.com/users/262685099575869451")}>
        <Icon icon="ic:baseline-discord" width="30" color="fff" />
    </button>

    <button style="animation-delay: 700ms;" on:click={OnClick("https://github.com/DrMeepso")}>
        <Icon icon="mdi:github" width="30" color="fff" />
    </button>

    <button style="animation-delay: 900ms;" on:click={OnClick("https://steamcommunity.com/id/DrMeepso/")}>
        <Icon icon="mdi:steam" width="30" color="fff" />
    </button>

</div>

<style>

    div {
        display: flex;
    }

    button {
        background-color: #000;
        border: none;
        border-radius: 50%;
        width: 50px;
        height: 50px;
        margin: 5px;
        display: flex;
        justify-content: center;
        align-items: center;
        cursor: pointer;
        transition: all 0.2s ease-in-out;

        opacity: 0;

        animation: fadeIn 1s ease-in-out;
        animation-fill-mode: forwards;

        z-index: 10;
    }

    button:hover {
        transition: all 0.2s ease-in-out;
        background-color: #707070;
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