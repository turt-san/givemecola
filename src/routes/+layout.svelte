<script lang="ts">
    import favicon from "#lib/assets/favicon.svg";

    let { children } = $props();

    let navHidden = $state(true);

    const toggleNav = () => {
        navHidden = !navHidden;
    };

    const routes = [
        {
            display: "Saved",
            route: "/saved",
        },
    ];
</script>

<svelte:head>
    <link rel="icon" href={favicon} />
</svelte:head>

<div class={["main-nav", { hidden: navHidden }]}>
    {#each routes as route}
        <a class="route-item" href={route.route}>{route.display}</a>
    {/each}
</div>
<div class="main-nav-button" onpointerdown={toggleNav}></div>
{@render children()}

<style>
    @font-face {
        font-family: "Lato";
        src: url("/fonts/Lato-Regular.ttf") format("truetype");
        font-weight: 400;
        font-style: normal;
    }

    .hidden {
        visibility: hidden;
    }

    .main-nav-button {
        position: absolute;
        background-color: red;
        padding: 1rem;
        left: 0;
        bottom: 0;
    }

    .main-nav {
        position: absolute;
        background-color: magenta;
        width: 10rem;
        left: 0;
        top: 0;
        bottom: 0;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    .route-item {
        background-color: white;
        padding: 0.5rem;
    }
</style>
