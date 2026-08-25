<script lang="ts">
    import {onMount} from "svelte";
    import {
        getModuleSettings,
        setModuleSettings,
        setModuleEnabled,
    } from "../../integration/rest";
    import type {ConfigurableSetting} from "../../integration/types";
    import GenericSetting from "./setting/common/GenericSetting.svelte";
    import {slide} from "svelte/transition";
    import {quintOut} from "svelte/easing";
    import {description as descriptionStore, highlightModuleName} from "./clickgui_store";
    import {setItem} from "../../integration/persistent_storage";
    import {convertToSpacedString, spaceSeperatedNames} from "../../theme/theme_config";
    import {scaleFactor} from "./clickgui_store";

    export let name: string;
    export let enabled: boolean;
    export let description: string;
    export let aliases: string[];

    let moduleNameElement: HTMLElement;
    let configurable: ConfigurableSetting;
    const path = `clickgui.${name}`;
    let expanded = false;
    let hasSettings = false;

    onMount(async () => {
        await fetchModuleSettings();

        setTimeout(() => {
            expanded = localStorage.getItem(path) === "true"
        }, 500);
    });

    highlightModuleName.subscribe((m) => {
        if (name !== m) {
            return;
        }

        setTimeout(() => {
            if (!moduleNameElement) {
                return;
            }
            moduleNameElement.scrollIntoView({
                behavior: "smooth",
                block: "center",
            });
        }, 1000);
    });

    async function fetchModuleSettings() {
        configurable = await getModuleSettings(name);
        hasSettings = configurable.value.filter(v => v.name !== "Bind" && v.name !== "Hidden").length > 0;
    }

    async function updateModuleSettings() {
        await setModuleSettings(name, configurable);
        await fetchModuleSettings();
    }

    async function toggleModule() {
        await setModuleEnabled(name, !enabled);
        enabled = !enabled;  // ← 手动更新状态触发样式刷新
    }

    function setDescription() {
        if (!moduleNameElement) return;

        const boundingRect = moduleNameElement.getBoundingClientRect();
        const y = (boundingRect.top + (moduleNameElement.clientHeight / 2)) * (2 / $scaleFactor);

        let moduleDescription = description;
        if (aliases.length > 0) {
            moduleDescription += ` (aka ${aliases.map(name => $spaceSeperatedNames ? convertToSpacedString(name) : name).join(", ")})`;
        }

        if (window.innerWidth - boundingRect.right > 300) {
            const x = boundingRect.right * (2 / $scaleFactor);
            descriptionStore.set({
                x,
                y,
                anchor: "right",
                description: moduleDescription
            });
        } else {
            const x = boundingRect.left * (2 / $scaleFactor);
            descriptionStore.set({
                x,
                y,
                anchor: "left",
                description: moduleDescription
            });
        }
    }

    async function toggleExpanded(e: MouseEvent) {
        e.stopPropagation();

        expanded = !expanded;
        await setItem(path, expanded.toString());
    }
</script>

<!-- 模板部分不变 -->
<div
        class="module-wrapper"
        class:expanded
>
    <div
            class="module-row"
            on:contextmenu|preventDefault={toggleExpanded}
            on:click={toggleModule}
            on:mouseenter={setDescription}
            on:mouseleave={() => descriptionStore.set(null)}
            bind:this={moduleNameElement}
            class:enabled
            class:highlight={name === $highlightModuleName}
    >
        <span class="module-name">
            {$spaceSeperatedNames ? convertToSpacedString(name) : name}
        </span>

        {#if hasSettings}
            <button
                    class="expand-arrow"
                    aria-label="Expand settings"
                    aria-expanded={expanded}
                    on:click|stopPropagation={toggleExpanded}
            >
                <span class="expand-arrow-icon"></span>
            </button>
        {/if}
    </div>

    {#if expanded && configurable}
        <div class="settings" transition:slide|global={{duration: 200, easing: quintOut}}>
            {#each configurable.value as setting (setting.name)}
                <GenericSetting {path} bind:setting on:change={updateModuleSettings}/>
            {/each}
        </div>
    {/if}
</div>

<style lang="scss">
    .module-wrapper {
        display: flex;
        flex-direction: column;
        border-bottom: 1px solid rgba(255, 255, 255, 0.04);

        &:last-child {
            border-bottom: none;
        }
    }

    .module-row {
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 8px 14px;
        cursor: pointer;
        transition: all 0.15s ease;
        min-height: 36px;

        &:hover {
            background-color: rgba(255, 255, 255, 0.04);
        }

        &.enabled .module-name {
            color: var(--Accent, #0018ff);
            text-shadow: 0 0 12px rgba(0, 24, 255, 0.3);
        }

        &.highlight {
            background-color: rgba(0, 24, 255, 0.1);
        }
    }

    .module-name {
        font-size: 13px;
        font-weight: 500;
        color: var(--clickgui-text-dimmed-color);
        transition: all 0.15s ease;
        flex: 1;
    }

    .expand-arrow {
        background: transparent;
        border: none;
        cursor: pointer;
        padding: 4px 8px;
        display: flex;
        align-items: center;
        justify-content: center;
        color: var(--clickgui-text-dimmed-color);
        transition: all 0.2s ease;

        &:hover {
            color: var(--Accent, #0018ff);
        }
    }

    .expand-arrow-icon {
        display: inline-block;
        width: 10px;
        height: 10px;
        border-right: 2px solid currentColor;
        border-bottom: 2px solid currentColor;
        transform: rotate(-45deg);
        transition: transform 0.25s ease;
    }

    .module-wrapper.expanded .expand-arrow-icon {
        transform: rotate(45deg);
    }

    .settings {
        padding: 4px 14px 10px 14px;
        background-color: rgba(0, 0, 0, 0.15);
        border-top: 1px solid rgba(255, 255, 255, 0.04);
        display: flex;
        flex-direction: column;
        gap: 4px;
    }
</style>