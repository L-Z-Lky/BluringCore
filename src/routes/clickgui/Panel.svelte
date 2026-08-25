<script lang="ts">
    import type {Module as TModule} from "../../integration/types";
    import Module from "./Module.svelte";

    export let category: string;
    export let modules: TModule[];
</script>

<div class="panel">
    <div class="category-title">{category}</div>
    <div class="modules">
        {#each modules as module (module.name)}
            <Module
                name={module.name}
                enabled={module.enabled}
                description={module.description}
                aliases={module.aliases}
            />
        {/each}
    </div>
</div>

<style lang="scss">
    .panel {
        background-color: var(--Tint, rgba(0, 0, 0, 0.3));
        backdrop-filter: blur(12px);
        -webkit-backdrop-filter: blur(12px);
        border-radius: 8px;
        border: 1px solid rgba(255, 255, 255, 0.06);
        overflow: hidden;
        height: 100%;
        max-height: 100%;
        display: flex;
        flex-direction: column;
        min-height: 0;  /* 允许 flex 收缩 */
    }

    .category-title {
        font-size: 13px;
        font-weight: 600;
        color: var(--Accent, #0018ff);
        padding: 10px 14px;
        border-bottom: 1px solid rgba(255, 255, 255, 0.06);
        text-shadow: 0 0 12px rgba(0, 24, 255, 0.2);
        flex-shrink: 0;
    }

    .modules {
        flex: 1 1 auto;
        overflow-y: auto;
        padding: 4px 0;
        min-height: 0;  /* 关键：允许 flex 子元素收缩到内容以下 */
        max-height: 100%;

        &::-webkit-scrollbar {
            width: 3px;
        }

        &::-webkit-scrollbar-thumb {
            background: var(--Accent, #0018ff);
            border-radius: 3px;
        }

        &::-webkit-scrollbar-track {
            background: transparent;
        }
    }
</style>