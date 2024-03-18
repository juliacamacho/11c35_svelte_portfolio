<script>
    export let info = {};
    let p = info;

    export let hLevel = 2;
</script>
<article>
    <svelte:element this={"h" + hLevel}>{ p.title }</svelte:element>
    <img src="{p.image}" alt="">
    <div>
        <p>{p.description}</p>
        <p>c. {p.year}</p>
    </div>
</article>