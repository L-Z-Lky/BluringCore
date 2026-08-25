<script lang="ts">
    import {fly} from "svelte/transition";
    import {createEventDispatcher} from "svelte";
    import {backIn, backOut} from "svelte/easing";
    import TitleButtonIcon from "./TitleButtonIcon.svelte";

    export let title: string;
    export let icon: string;
    export let index: number;

    let hovered = false;

    const dispatch = createEventDispatcher();
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class="main-button" on:mouseenter={() => hovered = true} on:mouseleave={() => hovered = false} on:click={() => hovered = false}
     on:click={() => dispatch("click")} out:fly|global={{duration: 400, x: -500, delay: index * 100, easing: backIn}}
     in:fly|global={{duration: 400, x: -500, delay: index * 100, easing: backOut}}>
    <div class="icon">
        <TitleButtonIcon {icon} />
    </div>

    <div class="title">{title}</div>

    <div class="wrapped-content">
        <slot parentHovered={hovered}/>
    </div>
</div>

<style lang="scss">

  .main-button {
    background-color: var(--Tint, rgba(0, 0, 0, 0.5));
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
    width: 590px;
    padding: 8px 35px;
    display: grid;
    grid-template-columns: max-content 1fr max-content;
    align-items: center;
    cursor: pointer;
    border-radius: 5px;
    column-gap: 25px;
    transition: all 0.25s ease;

    &:hover {
      box-shadow: 0 0 30px var(--Accent, #0018ff), 0 0 60px rgba(0, 24, 255, 0.3);
      transform: scale(1.01);

      .icon {
        color: var(--Accent, #0018ff);
        text-shadow: 0 0 20px var(--Accent, #0018ff);
      }

      .title {
        text-shadow: 0 0 20px var(--Accent, #0018ff);
      }
    }
  }

  .icon {
    background-color: transparent;
    color: var(--Accent, #0018ff);
    width: 40px;
    height: 40px;
    transition: ease color 0.2s, ease text-shadow 0.2s;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .title {
    font-size: 18px;
    color: var(--menu-main-button-text-color);
    font-weight: 600;
    transition: ease text-shadow 0.2s;
  }
</style>