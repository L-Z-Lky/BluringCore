<script lang="ts">
    import MainButton from "./buttons/MainButton.svelte";
    import FluidBackground from "./FluidBackground.svelte"; // <-- 导入你的新背景
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
    <!-- 把原来的 ConfettiBackground 换成你的流体背景 -->
    <FluidBackground />

    <div class="content">
        <div class="main-buttons">
            <!-- ... 你原来的按钮代码 ... -->
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
        /* 建议加一个深色底，防止 WebGL 加载前白屏 */
        background-color: #050208; 
    }

    .content {
        flex: 1;
        display: grid;
        grid-template-areas:
            "a"
            "b";
        grid-template-rows: 1fr max-content;
        grid-template-columns: 1fr;
        z-index: 1; /* 确保内容在流体背景之上 */
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
