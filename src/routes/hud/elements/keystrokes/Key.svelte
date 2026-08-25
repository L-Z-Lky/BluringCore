<script lang="ts">
    import {listen} from "../../../../integration/ws";
    import type {KeyEvent} from "../../../../integration/events";
    import type {MinecraftKeybind} from "../../../../integration/types";

    export let gridArea: string;
    export let key: MinecraftKeybind | undefined;
    export let displayName: string;

    let active = false;

    listen("key", (e: KeyEvent) => {
        if (e.key !== key?.key.translationKey) {
            return;
        }

        active = e.action === 1 || e.action === 2;
    });
</script>

<div class="key" style="grid-area: {gridArea};" class:active>
    {displayName}
</div>

<style lang="scss">
    .key {
        height: 50px;
        width: 50px;
        background-color: rgba(255, 255, 255, 0.06);
        color: #ffffff;
        display: flex;
        align-items: center;
        justify-content: center;
        border-radius: 8px;
        font-size: 13px;
        font-weight: 600;
        transition: all 0.12s ease;
        user-select: none;
        border: 1px solid rgba(255, 255, 255, 0.06);

        &.active {
            background-color: var(--Accent, #1a8cff);
            color: #ffffff;
            border-color: var(--Accent, #1a8cff);
            box-shadow: 0 0 20px var(--Accent, #1a8cff), 0 0 40px rgba(26, 140, 255, 0.2);
            font-size: 15px;
            transform: scale(0.92);
        }
    }

    .key[style*="sneak"],
    .key[style*="jump"] {
        width: 100%;
    }
</style>