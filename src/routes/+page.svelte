<script lang="ts">
    import { EtilbudsavisClient, type Offer } from "limbda/avis";
    import { BindDragAnimation } from "limbda/bindings";
    import { onMount } from "svelte";

    let offers = $state<Offer[]>([]);
    let offerIndex = $state(0);
    const offer = $derived(offers[offerIndex]);
    const expiresAt = $derived(new Date(offer.run_till));

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
        getOffers();
    });
</script>

<div class="main-container">
    <div class="sub-container">
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
        {#if offer}
            <div class="post">
                <div class="thumb" {@attach dragBinding.bind}>
                    <button class="left-right-buttons left">Left</button>
                    <img id="main-image" src={offer.images.thumb} alt="" />
                    <button class="left-right-buttons right">Right</button>
                </div>
                <div class="store-logo">
                    <img src={offer.branding.logo} alt="" />
                </div>
                <p>
                    {`The price for this item is: ${offer.pricing.price} ${offer.pricing.currency}`}
                </p>
                <p>
                    {priceTable.perLiter.toFixed(2)}
                </p>
                <div class="expiry-container">
                    <p>{expiresAt.toLocaleString()}</p>
                    <br />
                    <p>
                        {(expiresAt - Date.now()) / 1000 / 60 / 60 / 60}
                    </p>
                </div>
            </div>
        {/if}
    </div>
</div>

<style>
    #search-bar-form {
        width: 5%;
    }

    #search-bar {
        width: 100%;
    }

    .main-container {
        width: 100%;
        height: 100%;
    }

    .sub-container {
        width: 100%;
        height: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    .post {
        height: 100%;
        position: relative;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    .store-logo {
        display: flex;
        align-items: center;
        justify-content: center;
        width: 50%;
        max-height: 10%;
    }

    img {
        max-width: 100%;
        max-height: 100%;
        box-sizing: content-box;
    }

    .thumb {
        height: 60%;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .thumb img {
        margin: auto;
    }

    .left-right-buttons {
        opacity: 0.2;
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
