<slot />
<script>
    let pages = [
        {url: ".", title: "Home"},
        {url: "projects", title: "Projects"},
        {url: "contact", title: "Contact"},
        {url: "cv", title: "CV"},
        {url: "https://github.com/juliacamacho", title: "Github"}
    ];
    let localStorage = globalThis.localStorage ?? {};
    let colorScheme = localStorage.colorScheme ? localStorage.colorScheme : "light dark";
    let root = globalThis?.document?.documentElement;
    $: root?.style.setProperty("color-scheme", colorScheme);
    $: localStorage.colorScheme = colorScheme;

</script>

<label class="color-scheme">
    Theme:
    <select bind:value={ colorScheme }>
        <option value="light dark">Automatic</option>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
    </select>
</label>

<style>
    label.color-scheme {
        position: absolute;
        top: 1rem;
        right: 1rem;
        font-size: 80%;
        font-family: inherit;
    }
</style>