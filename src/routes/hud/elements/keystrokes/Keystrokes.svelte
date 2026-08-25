<script lang="ts">
    import Key from "./Key.svelte";
    import {onMount} from "svelte";
    import {getMinecraftKeybinds} from "../../../../integration/rest";
    import type {MinecraftKeybind} from "../../../../integration/types";
    import {listen} from "../../../../integration/ws";

    let keyForward: MinecraftKeybind | undefined;
    let keyBack: MinecraftKeybind | undefined;
    let keyLeft: MinecraftKeybind | undefined;
    let keyRight: MinecraftKeybind | undefined;
    let keyJump: MinecraftKeybind | undefined;
    let keySneak: MinecraftKeybind | undefined;

    async function updateKeybinds() {
        const keybinds = await getMinecraftKeybinds();

        keyForward = keybinds.find(k => k.bindName === "key.forward");
        keyBack = keybinds.find(k => k.bindName === "key.back");
        keyLeft = keybinds.find(k => k.bindName === "key.left");
        keyRight = keybinds.find(k => k.bindName === "key.right");
        keyJump = keybinds.find(k => k.bindName === "key.jump");
        keySneak = keybinds.find(k => k.bindName === "key.sneak");
    }

    onMount(updateKeybinds);

    listen("keybindChange", updateKeybinds)
</script>

<div class="keystrokes-wrapper">
    <div class="keystrokes">
        <Key key={keyForward} gridArea="w" displayName="W" />
        <Key key={keyLeft} gridArea="a" displayName="A" />
        <Key key={keyBack} gridArea="s" displayName="S" />
        <Key key={keyRight} gridArea="d" displayName="D" />
        <Key key={keySneak} gridArea="sneak" displayName="下蹲" />
        <Key key={keyJump} gridArea="jump" displayName="跳跃" />
    </div>
</div>

<style lang="scss">
    .keystrokes-wrapper {
        background-color: rgba(0, 0, 0, 0.5);
        border-radius: 12px;
        padding: 10px;
        backdrop-filter: blur(16px);
        -webkit-backdrop-filter: blur(16px);
        box-shadow: 0 4px 24px rgba(0, 0, 0, 0.3);
        width: max-content;
    }

    .keystrokes {
        display: grid;
        grid-template-areas:
            ". w ."
            "a s d"
            "sneak sneak jump jump";
        grid-template-columns: repeat(4, 50px);
        grid-template-rows: repeat(3, 50px);
        gap: 4px;
    }
</style>