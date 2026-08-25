<script lang="ts">
    import type {GroupedModules, Module} from "../../integration/types";
    import Panel from "./Panel.svelte";
    import Search from "./Search.svelte";
    import {fade} from "svelte/transition";
    import {onMount} from "svelte";
    import {getModules} from "../../integration/rest";
    import {groupByCategory} from "../../integration/util";

    let categories = $state<GroupedModules>({});
    let modules = $state<Module[]>([]);
    let selectedCategory = $state<string | null>(null);
    let categoryKeys = $state<string[]>([]);

    onMount(async () => {
        modules = await getModules();
        categories = groupByCategory(modules);
        categoryKeys = Object.keys(categories);
        if (categoryKeys.length > 0) {
            selectedCategory = categoryKeys[0];
        }
    });
</script>

<div class="clickgui" transition:fade|global={{duration: 200}}>
    <div class="container">
        <!-- 左侧分类列表 -->
        <div class="left-panel">
            <Search modules={modules}/>
            <div class="category-list">
                {#each categoryKeys as category}
                    <button
                        class="category-btn"
                        class:active={selectedCategory === category}
                        on:click={() => selectedCategory = category}
                    >
                        {category}
                    </button>
                {/each}
            </div>
        </div>

        <!-- 右侧模块面板 -->
        <div class="right-panel">
            {#if selectedCategory && categories[selectedCategory]}
                {#key selectedCategory}
                    <Panel category={selectedCategory} modules={categories[selectedCategory]} />
                {/key}
            {/if}
        </div>
    </div>
</div>

<style lang="scss">
    .clickgui {
        position: absolute;
        inset: 0;
        display: flex;
        align-items: center;
        justify-content: center;
        background-color: transparent;
        backdrop-filter: blur(16px);
        -webkit-backdrop-filter: blur(16px);
    }

    .container {
        display: flex;
        gap: 12px;
        height: 600px;
        max-height: 80vh;
    }

    .left-panel {
        width: 180px;
        flex-shrink: 0;
        display: flex;
        flex-direction: column;
        gap: 8px;
        height: 100%;
    }

    .category-list {
        display: flex;
        flex-direction: column;
        gap: 2px;
        background-color: var(--Tint, rgba(0, 0, 0, 0.2));
        backdrop-filter: blur(8px);
        -webkit-backdrop-filter: blur(8px);
        border-radius: 8px;
        border: 1px solid rgba(255, 255, 255, 0.06);
        padding: 4px;
        flex: 1;
        overflow-y: auto;
    }

    .category-btn {
        background: transparent;
        border: none;
        border-radius: 6px;
        padding: 8px 12px;
        font-size: 13px;
        font-weight: 500;
        color: var(--clickgui-text-dimmed-color);
        cursor: pointer;
        transition: all 0.2s ease;
        text-align: left;
        font-family: "Inter", sans-serif;
        position: relative;

        &:hover {
            color: var(--clickgui-text-color);
            background-color: rgba(255, 255, 255, 0.04);
        }

        &.active {
            color: var(--Accent, #0018ff);
            text-shadow: 0 0 20px var(--Accent, #0018ff), 0 0 40px rgba(0, 24, 255, 0.15);
            background-color: rgba(0, 24, 255, 0.06);
            border: 1px solid rgba(0, 24, 255, 0.15);
        }
    }

    .right-panel {
        flex: 1;
        min-width: 0;
        height: 100%;
        width: 360px;
        flex-shrink: 0;
        display: flex;
        flex-direction: column;
    }

    .category-list::-webkit-scrollbar {
        width: 2px;
    }

    .category-list::-webkit-scrollbar-thumb {
        background: var(--Accent, #0018ff);
        border-radius: 2px;
    }

    .category-list::-webkit-scrollbar-track {
        background: transparent;
    }
</style>