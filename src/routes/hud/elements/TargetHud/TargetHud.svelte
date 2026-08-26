<script lang="ts">
    import { listen } from "../../../../integration/ws";
    import type { PlayerData, ItemStack } from "../../../../integration/types";
    import type { TargetChangeEvent } from "../../../../integration/events";
    import TargetEntry from "./TargetEntry.svelte";

    interface TargetEntryData {
        target: PlayerData;
        id: string;
        armor: number;
    }

    let targets: TargetEntryData[] = [];
    let hideTimeouts: Map<string, number> = new Map();

    function calculateTotalArmor(armorItems: ItemStack[]): number {
        let total = 0;
        for (const item of armorItems) {
            const id = item.identifier;
            if (id.includes("netherite_helmet") || id.includes("diamond_helmet")) total += 3;
            else if (id.includes("netherite_chestplate") || id.includes("diamond_chestplate")) total += 8;
            else if (id.includes("netherite_leggings") || id.includes("diamond_leggings")) total += 6;
            else if (id.includes("netherite_boots") || id.includes("diamond_boots")) total += 3;
            else if (id.includes("iron_helmet")) total += 2;
            else if (id.includes("iron_chestplate")) total += 6;
            else if (id.includes("iron_leggings")) total += 5;
            else if (id.includes("iron_boots")) total += 2;
            else if (id.includes("golden_helmet") || id.includes("chainmail_helmet")) total += 2;
            else if (id.includes("golden_chestplate")) total += 5;
            else if (id.includes("chainmail_chestplate")) total += 5;
            else if (id.includes("golden_leggings") || id.includes("chainmail_leggings")) total += 4;
            else if (id.includes("golden_boots") || id.includes("chainmail_boots")) total += 1;
            else if (id.includes("leather_helmet")) total += 1;
            else if (id.includes("leather_chestplate")) total += 3;
            else if (id.includes("leather_leggings")) total += 2;
            else if (id.includes("leather_boots")) total += 1;
            else if (id.includes("turtle_helmet")) total += 2;
        }
        return total;
    }

    function updateTarget(data: TargetChangeEvent) {
        const targetData = data.target;
        if (!targetData) return;

        const id = targetData.uuid || targetData.username;
        const armor = calculateTotalArmor(targetData.armorItems);

        const existing = targets.find(t => t.id === id);
        if (existing) {
            existing.target = targetData;
            existing.armor = armor;
        } else {
            targets = [...targets, { target: targetData, id, armor }];
        }

        const existingTimeout = hideTimeouts.get(id);
        if (existingTimeout) clearTimeout(existingTimeout);
        hideTimeouts.set(id, setTimeout(() => {
            targets = targets.filter(t => t.id !== id);
            hideTimeouts.delete(id);
        }, 1000));
    }

    listen("targetChange", updateTarget);
</script>

<div class="target-container">
    {#each targets as entry (entry.id)}
        <TargetEntry target={entry.target} armor={entry.armor} />
    {/each}
</div>

<style lang="scss">
    .target-container {
        display: flex;
        flex-direction: column;
        align-items: flex-start;
        gap: 4px;
    }
</style>