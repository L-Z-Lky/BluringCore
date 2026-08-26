<script lang="ts">
    import type { PlayerData } from "../../../../integration/types";
    import TextComponent from "../../../menu/common/TextComponent.svelte";

    export let target: PlayerData;
    export let armor: number;

    function getHealthColor(health: number): string {
        if (health <= 5) return "#ff2222";
        if (health <= 10) return "#ff8800";
        return "#33ff33";
    }
</script>

<div class="target-entry">
    <div class="id-line">
        <TextComponent textComponent={target.username} fontSize={16} allowPreformatting={true} />
    </div>
    <div class="health-line">
        <span class="health-label">Health:</span>
        <span
            class="health-value"
            style="color: {getHealthColor(Math.floor(target.actualHealth))}; text-shadow: 0 0 16px {getHealthColor(Math.floor(target.actualHealth))};"
        >
            {Math.floor(target.actualHealth)}
        </span>
    </div>
    <div class="armor-line">
        <span class="armor-label">Armor:</span>
        <span class="armor-value">{armor}</span>
    </div>
</div>

<style lang="scss">
    .target-entry {
        background: rgba(0, 0, 0, 0.5);
        backdrop-filter: blur(16px);
        -webkit-backdrop-filter: blur(16px);
        border-radius: 8px;
        padding: 10px 16px;
        min-width: 120px;
        border: 1px solid rgba(255, 255, 255, 0.06);
        box-shadow: 0 4px 24px rgba(0, 0, 0, 0.3);
        margin-bottom: 4px;
    }

    .id-line {
        margin-bottom: 2px;
    }

    .health-line {
        display: flex;
        align-items: baseline;
        gap: 6px;
    }

    .armor-line {
        display: flex;
        align-items: baseline;
        gap: 6px;
        margin-top: 2px;
    }

    .health-label {
        font-size: 14px;
        font-weight: 500;
        color: rgba(255, 255, 255, 0.6);
    }

    .health-value {
        font-size: 18px;
        font-weight: 700;
        font-variant-numeric: tabular-nums;
        transition: color 0.3s ease, text-shadow 0.3s ease;
    }

    .armor-label {
        font-size: 14px;
        font-weight: 500;
        color: rgba(255, 255, 255, 0.6);
    }

    .armor-value {
        font-size: 16px;
        font-weight: 600;
        color: #ffffff;
    }
</style>