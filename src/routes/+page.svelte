<script lang="ts">
    import { EtilbudsavisClient, type Offer } from "limbda/avis";
    import { BindDragAnimation } from "limbda/bindings";
    import { onMount } from "svelte";

    let offers = $state<Offer[]>([]);
    let offerIndex = $state(0);
    const offer = $derived(offers[offerIndex]);

    const amountInSI = $derived(
        offer.quantity.pieces.to *
            offer.quantity.size.to *
            offer.quantity.unit.si.factor,
    );

    const priceTable = $derived({
        // total cans * liter in can (normalized to SI)
        perLiter: offer.pricing.price / amountInSI,
    });

    const client = new EtilbudsavisClient();

    let mounted = false;

    let query = $state("");

    const getOffers = async () => {
        offers = [];
        offerIndex = 0;
        offers = await client.fetchQuery(query);
    };

    const saveToShoppingList = (offer: Offer) => {
        const { id } = offer;
        localStorage.setItem(id, JSON.stringify(offer));
    };

    const dateToDaysAndHours = (millis: number) => {
        const totalHours = millis / 1000 / 60 / 60;
        const hours = totalHours % 24;
        const days = (totalHours - hours) / 24;
        return { totalHours, hours, days };
    };

    const expiresAt = $derived(new Date(offer.run_till));
    const expiresAtTable = $derived(dateToDaysAndHours(expiresAt - Date.now()));

    const dragBinding = new BindDragAnimation(
        {},
        {
            left: () => {
                moveIndex(-1);
            },
            right: () => {
                moveIndex(1);
            },
        },
    );

    $effect(() => {
        const q = query;
        if (mounted) {
            localStorage.setItem("query", q);
        }
    });

    function moveIndex(increment: number) {
        let ni = offerIndex + increment;
        if (ni > offers.length - 1) {
            ni = 0;
        } else if (ni < 0) {
            ni = offers.length - 1;
        }
        offerIndex = ni;
    }

    onMount(async () => {
        query = localStorage.getItem("query") || "æg";
        mounted = true;
        await getOffers();
        console.log(offers);
    });
</script>

<form
    id="search-bar-form"
    action="submit"
    onsubmit={(e) => {
        e.preventDefault();
        getOffers();
    }}
>
    <input id="search-bar" type="text" bind:value={query} />
</form>
<div class="main-container">
    <div class="sub-container">
        {#if offer}
            <div class="post">
                <div class="store-logo">
                    <img src={offer.branding.logo} alt="" />
                </div>
                <div class="main-thumb" {@attach dragBinding.bind}>
                    <button class="left-right-buttons left">Left</button>
                    <div class="sub-thumb">
                        <img id="main-image" src={offer.images.thumb} alt="" />
                        <svg
                            xmlns="http://www.w3.org/2000/svg"
                            width="24"
                            height="24"
                            viewBox="0 0 24 24"
                            fill="none"
                            stroke="currentColor"
                            stroke-width="2"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                            class="lucide lucide-star preview-icon"
                            ><path
                                d="M11.525 2.295a.53.53 0 0 1 .95 0l2.31 4.679a2.123 2.123 0 0 0 1.595 1.16l5.166.756a.53.53 0 0 1 .294.904l-3.736 3.638a2.123 2.123 0 0 0-.611 1.878l.882 5.14a.53.53 0 0 1-.771.56l-4.618-2.428a2.122 2.122 0 0 0-1.973 0L6.396 21.01a.53.53 0 0 1-.77-.56l.881-5.139a2.122 2.122 0 0 0-.611-1.879L2.16 9.795a.53.53 0 0 1 .294-.906l5.165-.755a2.122 2.122 0 0 0 1.597-1.16z"
                            /></svg
                        >
                    </div>
                    <button class="left-right-buttons right">Right</button>
                </div>
                <p class="price-display">
                    {`${offer.pricing.price.toFixed(2)} ${offer.pricing.currency}`}
                </p>
                <p class="price-per-piece-display">
                    {`${priceTable.perLiter.toFixed(2)} DKK / liter`}
                </p>
                <div class="expiry-container">
                    <p>Expires at:</p>
                    <p>{expiresAt.toLocaleString()}</p>
                    <br />
                    <p>
                        {`${expiresAtTable.days.toFixed(0)} days and ${expiresAtTable.hours.toFixed(2)} hours left`}
                    </p>
                </div>
            </div>
        {/if}
    </div>
</div>

<style>
    :root {
        --default-border-radius: 1rem;
        --color-main: oklch(0.708984 0.164063 260);
        --color-main1: oklch(0.8294 0.2072 154.59);
        --color-alt: oklch(0.524902 0.123438 260);
        --color-alt1: oklch(0.8294 0.3182 142.94);
    }

    #search-bar-form {
        width: 5%;
    }

    #search-bar {
        width: 100%;
    }

    .main-container {
        height: 100%;
        width: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .sub-container {
        font-family: "Lato";
        width: 30%;
        height: 90%;
        background-color: var(--color-main);
        border-radius: var(--default-border-radius);
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    .store-logo {
        border-radius: var(--default-border-radius);
        padding: 0.2rem;
        background-color: white;
        display: flex;
        align-items: center;
        justify-content: center;
        width: 50%;
        max-height: 10%;
    }

    .price-display {
        margin: 0;
        font-size: 200%;
        border-radius: calc(var(--default-border-radius) * 0.5);
        background-color: white;
        padding: 0.2rem;
    }

    .post {
        height: 100%;
        position: relative;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    img {
        max-width: 100%;
        max-height: 100%;
    }

    .main-thumb {
        display: contents;
        justify-content: center;
    }

    .sub-thumb:hover {
        cursor: grab;
        scale: 1.05;
    }

    .sub-thumb {
        position: relative;
        margin-block: 1rem;
        max-height: 40%;
        box-sizing: border-box;
        display: flex;
        align-items: center;
        justify-content: center;
        padding: 1rem;
        transition: scale 300ms;
        background-color: var(--color-alt);
        border-radius: var(--default-border-radius);
    }

    #main-image {
        margin: auto;
        border-radius: var(--default-border-radius);
    }

    .lucide-star {
        --offset: 1rem;
        right: var(--offset);
        bottom: var(--offset);
        padding: 0.3rem;
        border-radius: calc(var(--default-border-radius) * 0.8);
        background-color: yellow;
        position: absolute;
    }

    .left-right-buttons {
        visibility: hidden;
        opacity: 0.5;
        padding: 0;
        margin: 0;
        position: absolute;
        z-index: 2;
    }

    .left {
        width: 50%;
        left: 0;
        top: 0;
        bottom: 0;
    }

    .right {
        width: 50%;
        right: 0;
        top: 0;
        bottom: 0;
    }

    button {
        font-size: 3rem;
        margin: 3rem;
    }
</style>
