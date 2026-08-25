<script lang="ts">
    import MainButton from "./buttons/MainButton.svelte";
    import ConfettiBackground from "./ConfettiBackground.svelte";
    import IconTextButton from "../common/buttons/IconTextButton.svelte";
    import {
        exitClient,
        openScreen
    } from "../../../integration/rest";
    import {fly} from "svelte/transition";
    import {onMount} from "svelte";
    import {isAnniversary} from "../../../util/utils";

    let regularButtonsShown = true;
    let clientButtonsShown = false;

    function toggleButtons() {
        if (clientButtonsShown) {
            clientButtonsShown = false;
            setTimeout(() => {
                regularButtonsShown = true;
            }, 750);
        } else {
            regularButtonsShown = false;
            setTimeout(() => {
                clientButtonsShown = true;
            }, 750);
        }
    }
</script>

<div class="title-screen">
    {#if isAnniversary()}
        <ConfettiBackground/>
    {/if}

    <div class="content">
        <div class="main-buttons">
            {#if regularButtonsShown}
                <MainButton title="Singleplayer" icon="singleplayer" index={0}
                            on:click={() => openScreen("singleplayer")}/>

                <MainButton title="Multiplayer" icon="multiplayer"
                            on:click={() => openScreen("multiplayer")} index={1}/>

                <MainButton title="Minecraft" icon="options"
                            on:click={() => openScreen("options")} index={2}/>

                <MainButton title="Hack Options" icon="clickgui"
                            on:click={() => openScreen("clickgui")} index={3}/>

                <MainButton title="Proxy Manager" icon="proxymanager"
                            on:click={() => openScreen("proxymanager")} index={4}/>
            {:else if clientButtonsShown}
                <MainButton title="Proxy Manager" icon="proxymanager" on:click={() => openScreen("proxymanager")}
                            index={0}/>
                <MainButton title="Click GUI" icon="clickgui" on:click={() => openScreen("clickgui")} index={1}/>
                <MainButton title="Back" icon="back-large" on:click={toggleButtons} index={2}/>
            {/if}
        </div>

        <div class="additional-buttons" transition:fly|global={{duration: 700, y: 100}}>
            <IconTextButton icon="icon-exit.svg" title="Quit Game" on:click={exitClient}/>
        </div>
    </div>
</div>

<style>
    .title-screen {
        position: relative;
        isolation: isolate;
        display: flex;
        flex: 1;
        flex-direction: column;
    }

    .content {
        flex: 1;
        display: grid;
        grid-template-areas:
            "a"
            "b";
        grid-template-rows: 1fr max-content;
        grid-template-columns: 1fr;
    }

    .main-buttons {
        display: flex;
        flex-direction: column;
        row-gap: 25px;
        grid-area: a;
    }

    .additional-buttons {
        grid-area: b;
    }
</style>