<script lang="ts">

    import {onMount} from "svelte"

    function lerp(a: number, b: number, t: number) {
        return a + (b - a) * t;
    }

    const StartTime = Date.now() + 500

    function easeOutQuad(x: number): number {
        return 1 - (1 - x) * (1 - x);
    }

    function easeOutCirc(x: number): number {
        return Math.sqrt(1 - Math.pow(x - 1, 2));
    }

    function Update()
    {

        const timeSinceStart = Date.now() - StartTime

        const ContentHolder = document.getElementById("MainContentHolder")
        const ItemHolder = document.getElementById("ItemHolder")

        let timeIndex = Math.min(timeSinceStart / 3000, 1)
        let yOffset = ItemHolder!.getClientRects()[0].height * (1 - easeOutQuad(timeIndex))

        ItemHolder!.style.top = (ContentHolder!.getClientRects()[0].top - yOffset) + "px"

        Object.values(ItemHolder!.children).forEach((item, index) => {

            const Item = item as HTMLParagraphElement

            let Distance = Math.abs(Item.getClientRects()[0].top - ContentHolder!.getClientRects()[0].top)
            let opacity = lerp(1, 0, Distance / 20)

            Item.style.opacity = Math.max(opacity, 0).toString() 

        })

        
        if (timeSinceStart > 3000)
        { 
            let timeIndex = easeOutCirc(Math.min((timeSinceStart - 3000) / 1000, 1))
            ContentHolder!.style.width = lerp(100, 65, timeIndex) + "px  "
            // @ts-ignore
            ItemHolder!.children[ItemHolder!.children.length - 1].style.width = lerp(100, 65, timeIndex) + "px "

        } else {
            ContentHolder!.style.width = "100px"
        }

        if (timeSinceStart > 4000)
        {

            ContentHolder!.innerHTML = ItemHolder!.children[ItemHolder!.children.length - 1].innerHTML
            ContentHolder!.style.width = ""

        } else {
            window.requestAnimationFrame(Update)
        }

    }

    onMount(() => {

        window.requestAnimationFrame(Update)

    })

</script>

<div id="MainContentHolder">
    <div id="ItemHolder">
        <pre class="GlideItem">Insane</pre>
        <pre class="GlideItem">Robotic</pre>
        <pre class="GlideItem">Crazy</pre>
        <pre class="GlideItem">Mad</pre>
        <pre class="GlideItem">Kiwi</pre>
        <pre class="GlideItem">Autistic</pre>
        <pre class="GlideItem">Creative</pre>
        <pre class="GlideItem">Intrigued</pre>
        <pre class="GlideItem">Dyslexic</pre>
        <pre class="GlideItem">Human</pre>
        <pre class="GlideItem"> Meepso</pre>

    </div>
</div>

<style>

    div {
        font-family: 'Open Sans', sans-serif;
        display: inline;
    }

    pre {
        font-family: 'Open Sans', sans-serif;
        display: block;
        margin: 0px;
        text-align: center;
        width: 100px;
        cursor: default;
    }

    #MainContentHolder {
        white-space: pre;
    }

    #ItemHolder {
        position: absolute;
        display: flex;
        flex-direction: column-reverse;
    }

</style>