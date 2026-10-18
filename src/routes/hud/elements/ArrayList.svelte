<script lang="ts">
    import { onMount, onDestroy } from "svelte";
    import type { Module } from "../../../integration/types";
    import { getModules } from "../../../integration/rest";
    import { listen } from "../../../integration/ws";
    import { flip } from "svelte/animate";
    import { fly } from "svelte/transition";
    import { convertToSpacedString, spaceSeperatedNames } from "../../../theme/theme_config";

    let { settings }: { settings: { [name: string]: any } } = $props();

    let enabledModules: Module[] = $state([]);
    
    // 动态色相值，用于全局的流动渐变
    let currentHue = $state(195); // 195 是电光蓝，280 是浅紫
    let animFrame: number;

    function getSettings() {
        return settings as HudArrayListSettings;
    }

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

    function getFullDisplayWidth(module: Module): number {
        const name = $spaceSeperatedNames ? convertToSpacedString(module.name) : module.name;
        let fullText = name;
        if (module.tag && getSettings().showTags) {
            fullText = name + " - " + module.tag;
        }
        return getTextWidth(fullText);
    }

    async function updateEnabledModules() {
        const modules = await getModules();
        const visibleModules = modules.filter(m => m.enabled && !m.hidden);
        visibleModules.sort((a, b) => getFullDisplayWidth(b) - getFullDisplayWidth(a));
        enabledModules = [...visibleModules];
    }

    // 根据模块名字计算出一个固定偏移，让不同模块在紫/蓝之间有自己的固定偏好
    function getModuleColorOffset(name: string): number {
        let hash = 0;
        for (let i = 0; i < name.length; i++) {
            hash = name.charCodeAt(i) + ((hash << 5) - hash);
        }
        // 返回 -40 到 40 之间的偏移量
        return (Math.abs(hash) % 80) - 40; 
    }

    // 生成基础颜色
    function getColor(name: string): string {
        // 基础色相是全局流动的 currentHue，加上模块自己的偏移
        let hue = currentHue + getModuleColorOffset(name);
        // 限制色相在 190 到 290 之间，保证全是蓝到紫的范围
        hue = Math.max(190, Math.min(290, hue));
        return `hsl(${hue}, 100%, 75%)`; // 100% 饱和度，75% 亮度，保证发光感
    }

    function getGlowColor(name: string): string {
        let hue = currentHue + getModuleColorOffset(name);
        hue = Math.max(190, Math.min(290, hue));
        return `hsla(${hue}, 100%, 75%, 0.6)`; // 半透明用于发光
    }

    $effect(() => {
        const _ = getSettings().itemAlignment; 
        const _2 = getSettings().showTags;
        updateEnabledModules();
    });

    onMount(() => {
        updateEnabledModules();
        
        // 动画循环：让色相在 195(蓝) 和 280(紫) 之间呼吸
        let startTime = performance.now();
        function animateColor() {
            const now = performance.now();
            const time = (now - startTime) / 1000;
            // 使用正弦波在蓝和紫之间震荡
            currentHue = 237.5 + 42.5 * Math.sin(time * 0.5); 
            animFrame = requestAnimationFrame(animateColor);
        }
        animateColor();

        const unsub1 = listen("moduleToggle", updateEnabledModules);
        const unsub2 = listen("refreshArrayList", updateEnabledModules);
        const unsub3 = spaceSeperatedNames.subscribe(updateEnabledModules);
        
        return () => {
            cancelAnimationFrame(animFrame);
            unsub1?.();
            unsub2?.();
            unsub3?.();
        };
    });
</script>

<div class="arraylist-wrapper">
    <div class="arraylist">
        {#each enabledModules as module (module.name)}
            {@const displayName = $spaceSeperatedNames ? convertToSpacedString(module.name) : module.name}
            {@const textColor = getColor(displayName)}
            {@const glowColor = getGlowColor(displayName)}
            
            <div
                class="module-item"
                style="justify-content: {getSettings().itemAlignment === 'Left' ? 'flex-start' : 'flex-end'};"
                animate:flip={{ duration: 200 }}
                transition:fly={{ x: 50, duration: 200 }}
            >
                <!-- 动态颜色直接内联 -->
                <div 
                    class="module-rect" 
                    style="color: {textColor}; text-shadow: 0 0 10px {glowColor};"
                >
                    {displayName}
                    {#if module.tag && getSettings().showTags}
                        <span class="tag-separator"> - </span>
                        <!-- 标签偏亮白一点，但混合一点当前的色相 -->
                        <span class="tag" style="color: hsl(240, 100%, 85%);">{module.tag}</span>
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
        font-size: 13px;
        font-weight: 600; /* 稍微加粗，配合电光蓝和浅紫更清晰 */
        padding: 3px 10px;
        border-radius: 4px;
        background-color: rgba(0, 0, 0, 0.45);
        backdrop-filter: blur(6px);
        -webkit-backdrop-filter: blur(6px);
        border: 1px solid rgba(255, 255, 255, 0.06);
        box-shadow: 0 1px 6px rgba(0, 0, 0, 0.15);
        width: max-content;
        white-space: nowrap;
        transition: background-color 0.2s, transform 0.2s;
        line-height: 1.4;
        will-change: transform, background-color, color, text-shadow; /* 增加颜色变化的性能提示 */
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
        font-size: 12px;
        font-weight: 500;
        opacity: 0.9;
    }
</style>
