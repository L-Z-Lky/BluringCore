<script lang="ts">
    import {createEventDispatcher} from "svelte";

    export let title: string;
    export let icon: string;  // 保留参数，但不用

    const dispatch = createEventDispatcher();
</script>

<button class="icon-text-button" on:click={() => dispatch("click")}>
    <span class="title">{title}</span>
</button>

<style lang="scss">
    .icon-text-button {
        background-color: var(--Tint, rgba(0, 0, 0, 0.5));
        backdrop-filter: blur(16px);
        -webkit-backdrop-filter: blur(16px);
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;
        padding: 8px 20px;
        border-radius: 5px;
        border: none;
        transition: all 0.25s ease;
        min-width: 60px;
        width: max-content;

        &:hover {
            box-shadow: 0 0 30px var(--Accent, #0018ff), 0 0 60px rgba(0, 24, 255, 0.3);
            transform: scale(1.02);

            .title {
                text-shadow: 0 0 20px var(--Accent, #0018ff), 0 0 40px rgba(0, 24, 255, 0.3);
            }
        }

        .title {
            font-size: 16px;
            font-weight: 500;
            color: #ffffff;
            transition: all 0.25s ease;
            white-space: nowrap;
        }
    }
</style>