<script>
    export let info = {};

    export let hLevel = 2;
</script>
<article>
    <svelte:element this={"h" + hLevel}>{ info.title }</svelte:element>
    <img src="{info.image}" alt="">
    <div>
        <p>{info.description}</p>
        <p>c. {info.year}</p>
    </div>
</article>