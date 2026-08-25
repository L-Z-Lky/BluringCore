<script lang="ts">
    import { onDestroy } from "svelte";
    import { listen } from "../../../integration/ws";
    import type { TargetHudData } from "../../../integration/types";

    export let settings: { [name: string]: any };

    let cSettings: HudTargetHudSettings;
    $: cSettings = settings as HudTargetHudSettings;

    let target: TargetHudData | null = null;

    function updateTarget(data: TargetHudData) {
        target = data;
    }

    const unlisten = listen("targetHud", updateTarget);

    onDestroy(() => {
        unlisten?.();
    });

    function getHealthColor(health: number, maxHealth: number): string {
        const ratio = health / maxHealth;
        if (ratio > 0.6) return "#33ff33";
        if (ratio > 0.3) return "#ff8800";
        return "#ff2222";
    }
</script>

{#if target}
    <div class="target-hud">
        <div class="id-line">
            <span class="name">{target.name}</span>
        </div>
        <div class="health-line">
            <span class="health-label">Health:</span>
            <span 
                class="health-value" 
                style="color: {getHealthColor(target.health, target.maxHealth)}; text-shadow: 0 0 16px {getHealthColor(target.health, target.maxHealth)};"
            >
                {target.health}
            </span>
        </div>
    </div>
{/if}

<style lang="scss">
    .target-hud {
        background: rgba(0, 0, 0, 0.6);
        backdrop-filter: blur(12px);
        -webkit-backdrop-filter: blur(12px);
        border-radius: 12px;
        padding: 14px 20px;
        min-width: 150px;
        border: 1px solid rgba(255, 255, 255, 0.08);
        box-shadow: 0 8px 32px rgba(0, 0, 0, 0.4);
    }

    .id-line {
        margin-bottom: 4px;
    }

    .name {
        font-size: 16px;
        font-weight: 600;
        color: #ffffff;
        text-shadow: 0 0 12px rgba(255, 255, 255, 0.25);
    }

    .health-line {
        display: flex;
        align-items: baseline;
        gap: 6px;
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
</style>