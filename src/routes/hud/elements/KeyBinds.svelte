<script lang="ts">
    import {onMount} from "svelte";
    import {getModules} from "../../../integration/rest";
    import {listen} from "../../../integration/ws";
    import {convertToSpacedString, spaceSeperatedNames} from "../../../theme/theme_config";
    import type {Module} from "../../../integration/types";
    import {UNKNOWN_KEY} from "../../../util/utils";
    import BindDisplay from "../../clickgui/setting/bind/BindDisplay.svelte";

    let modules: Module[] = $state([]);

    async function updateModulesWithBinds() {
        modules = (await getModules()).filter(m => m.keyBind.boundKey !== UNKNOWN_KEY);
    }

    listen("moduleToggle", updateModulesWithBinds);
    listen("valueChanged", async (e) => {
        if (e.value.name === "Bind") {
            await updateModulesWithBinds();
        }
    })

    onMount(async () => {
        await updateModulesWithBinds();
    });
</script>

<div>
    <div class="keybinds">
        <div class="header">
            <span class="title">Binds</span>
            <img class="icon" src="img/hud/keybinds/icon-keybinds.svg" alt="keybinds">
        </div>
        <div class="entries">
            {#each modules as m (m.name)}
                <div class="row" class:enabled={m.enabled}>
                    <span class="module-name">{$spaceSeperatedNames ? convertToSpacedString(m.name) : m.name}</span>
                    <span class="key-bind" class:muted={!m.enabled}>
                        [<BindDisplay boundKey={m.keyBind.boundKey} modifiers={m.keyBind.modifiers}/>]
                    </span>
                </div>
            {:else}
                <div class="no-binds">No key bindings</div>
            {/each}
        </div>
    </div>
</div>

<style lang="scss">
  .keybinds {
    width: max-content;
    border-radius: 5px;
    overflow: hidden;
    font-size: 14px;
    min-width: 150px;
    max-width: 200px;
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.3);
  }

  .header {
    background-color: var(--keybinds-background-color);
    padding: 7px 10px;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .header .title {
    color: var(--keybinds-text-color);
    font-weight: 600;
    text-shadow: 0 0 10px var(--keybinds-text-color);
  }

  .header .icon {
    width: 16px;
    height: 16px;
    filter: drop-shadow(0 0 6px var(--keybinds-text-color));
  }

  .entries {
    background-color: var(--keybinds-header-background-color);
    padding: 6px 10px;
    color: var(--keybinds-text-color);
  }

  .entries .no-binds {
    font-style: italic;
    margin-bottom: 5px;
  }

  .row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 5px;
    gap: 12px;
    min-width: 0;
  }

  .row:last-child {
    margin-bottom: 0;
  }

  .row.enabled .module-name {
    color: #ffffff;
    font-weight: 500;
    text-shadow: 0 0 10px rgba(255, 255, 255, 0.8);
  }

  .row .module-name {
    color: #888888;
    font-size: 14px;
    flex: 1;
    min-width: 0;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .row .key-bind {
    display: inline-flex;
    align-items: center;
    font-family: monospace;
    font-size: 11px;
    font-weight: 600;
    flex-shrink: 0;
    min-width: max-content;
    color: var(--keybinds-accent-color);
    text-shadow: 0 0 10px var(--keybinds-accent-color);
  }

  .row .key-bind.muted {
    color: #666666;
    font-weight: 500;
  }
</style>