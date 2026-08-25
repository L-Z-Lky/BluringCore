<script lang="ts">
    import {onMount, tick} from "svelte";
    import type {Module} from "../../../integration/types";
    import {getModules} from "../../../integration/rest";
    import {listen} from "../../../integration/ws";
    import {flip} from "svelte/animate";
    import {fly} from "svelte/transition";
    import {convertToSpacedString, spaceSeperatedNames} from "../../../theme/theme_config";

    // 使用 $props() 接收属性
    let { settings }: { settings: { [name: string]: any } } = $props();

    // 声明响应式状态
    let enabledModules: Module[] = $state([]);

    // 获取最新 settings 的函数（惰性求值）
    function getSettings() {
        return settings as HudArrayListSettings;
    }

    // 计算文本实际宽度（中英文字符宽度不同）
    function getTextWidth(text: string): number {
        let width = 0;
        for (const char of text) {
            if (char.match(/[\u4e00-\u9fff\u3000-\u303f\uff00-\uffef]/)) {
                width += 2;
            } else {
                width += 1;
            }
        }
        return width;
    }

    // 计算模块完整显示文本（包含Tag）的总宽度
    function getFullDisplayWidth(module: Module): number {
        const name = $spaceSeperatedNames ? convertToSpacedString(module.name) : module.name;
        let fullText = name;

        // 如果显示Tag，把分隔符和Tag也加进去计算
        if (module.tag && getSettings().showTags) {
            // 分隔符 "-" 前后各有一个空格，占3个字符宽度
            const separator = " - ";
            const tag = module.tag;
            fullText = name + separator + tag;
        }

        return getTextWidth(fullText);
    }

    // 排序函数
    function updateEnabledModules() {
        const currentSettings = getSettings();
        getModules().then(modules => {
            const visibleModules = modules.filter(m => m.enabled && !m.hidden);

            visibleModules.sort((a, b) => {
                const widthA = getFullDisplayWidth(a);
                const widthB = getFullDisplayWidth(b);
                return widthB - widthA; // 从宽到窄（上大下小）
            });

            enabledModules = [...visibleModules];
        });
    }

    // 监听 settings 变化
    $effect(() => {
        const currentSettings = settings;
        updateEnabledModules();
    });

    // 监听命名风格变化
    $effect(() => {
        const unsubscribe = spaceSeperatedNames.subscribe(() => {
            updateEnabledModules();
        });
        return unsubscribe;
    });

    onMount(() => {
        updateEnabledModules();
    });

    listen("moduleToggle", () => {
        updateEnabledModules();
    });

    listen("refreshArrayList", () => {
        updateEnabledModules();
    });
</script>

<div class="arraylist-wrapper">
    <div class="arraylist">
        {#each enabledModules as module (module.name)}
            <div
                class="module-item"
                style="justify-content: {getSettings().itemAlignment === 'Left' ? 'flex-start' : 'flex-end'};"
                animate:flip={{ duration: 200 }}
                transition:fly={{ x: 50, duration: 200 }}
            >
                <div class="module-rect">
                    {$spaceSeperatedNames ? convertToSpacedString(module.name) : module.name}
                    {#if module.tag && getSettings().showTags}
                        <span class="tag-separator"> - </span><span class="tag">{module.tag}</span>
                    {/if}
                </div>
            </div>
        {/each}
    </div>
</div>

<style lang="scss">
    .arraylist-wrapper {
        background-color: transparent;
        border-radius: 0;
        padding: 0;
        backdrop-filter: none;
        -webkit-backdrop-filter: none;
        box-shadow: none;
        width: max-content;
        min-width: 60px;
    }

    .arraylist {
        display: flex;
        flex-direction: column;
        gap: 3px;
        width: 100%;
        padding: 0;
    }

    .module-item {
        display: flex;
        width: 100%;
        min-width: 40px;
    }

    .module-rect {
        color: var(--arraylist-text-color);
        font-size: 13px;
        font-weight: 500;
        padding: 3px 10px;
        border-radius: 4px;
        background-color: rgba(0, 0, 0, 0.45);
        backdrop-filter: blur(6px);
        -webkit-backdrop-filter: blur(6px);
        border: 1px solid rgba(255, 255, 255, 0.06);
        box-shadow: 0 1px 6px rgba(0, 0, 0, 0.15);
        width: max-content;
        white-space: nowrap;
        text-shadow: 0 0 8px var(--arraylist-text-color);
        transition: all 0.2s ease;
        line-height: 1.4;
    }

    .module-rect:hover {
        background-color: rgba(0, 0, 0, 0.65);
        transform: scale(1.02);
    }

    .tag-separator {
        color: #666666;
        font-size: 12px;
        font-weight: 400;
        margin: 0 2px;
        white-space: pre;
    }

    .tag {
        color: var(--Accent, #1a8cff);
        font-size: 12px;
        font-weight: 500;
        opacity: 0.85;
    }
</style>