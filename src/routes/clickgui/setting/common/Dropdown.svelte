<script lang="ts">
    import {createEventDispatcher, tick} from "svelte";
    import {convertToSpacedString, spaceSeperatedNames} from "../../../../theme/theme_config";

    export let name: string | null;
    export let options: string[];
    export let value: string;

    const dispatch = createEventDispatcher();

    let expanded = false;
    let dropdownHead: HTMLElement;

    function updateValue(v: string) {
        value = v;
        expanded = false;
        dispatch("change");
    }

    function toggleExpanded() {
        expanded = !expanded;
    }
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
<div class="dropdown" class:expanded>
    <div class="head" bind:this={dropdownHead} on:click={toggleExpanded}>
        {#if name !== null}
            <span class="text">{$spaceSeperatedNames ? convertToSpacedString(name) : name}
                &bull; {$spaceSeperatedNames ? convertToSpacedString(value) : value}</span>
        {:else}
            <span class="text">{$spaceSeperatedNames ? convertToSpacedString(value) : value}</span>
        {/if}
    </div>

    {#if expanded}
        <div class="options">
            {#each options as o (o)}
                <div
                        class="option"
                        class:active={o === value}
                        on:click={() => updateValue(o)}
                >
                    {$spaceSeperatedNames ? convertToSpacedString(o) : o}
                </div>
            {/each}
        </div>
    {/if}
</div>

<style lang="scss">
    @use "../../icon-settings-expand" as *;

    .dropdown {
        display: flex;
        flex-direction: column;
        width: 100%;
        border-bottom: 1px solid rgba(255, 255, 255, 0.04);

        &:last-child {
            border-bottom: none;
        }

        &.expanded {
            .text::after {
                transform: translateY(-50%) rotate(0);
                opacity: 1;
            }
        }
    }

    .head {
        background-color: var(--clickgui-dropdown-trigger-background-color);
        padding: 6px 10px;
        cursor: pointer;
        display: flex;
        align-items: center;
        position: relative;
        border-radius: 3px 3px 0 0;
        transition: ease border-radius .2s;

        .text {
            font-weight: 500;
            color: var(--clickgui-text-color);
            font-size: 12px;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
            margin-right: 20px;
            flex: 1;
        }

        .text::after {
            @include icon-settings-expand();
        }
    }

    .options {
        padding: 4px 0;
        background-color: rgba(0, 0, 0, 0.2);
        border-top: 1px solid rgba(255, 255, 255, 0.04);
        border-radius: 0 0 3px 3px;
        display: flex;
        flex-direction: column;

        .option {
            color: var(--clickgui-dropdown-option-color);
            font-weight: 500;
            font-size: 12px;
            padding: 6px 10px;
            cursor: pointer;
            transition: all 0.15s ease;

            &:hover {
                background-color: rgba(255, 255, 255, 0.05);
                color: var(--clickgui-dropdown-option-hover-color);
            }

            &.active {
                color: var(--Accent, #0018ff);
                background-color: rgba(0, 24, 255, 0.08);
            }
        }
    }
</style>