<script lang="ts">
    export let title: string;
    export let message: string;
    export let severity: string;
</script>

<div class="notification">
    <div class="icon {severity.toString().toLowerCase()}"></div>
    <div class="title">{title}</div>
    <div class="message">{message}</div>
</div>

<style lang="scss">
  .notification {
    display: grid;
    grid-template-areas:
            "a b"
            "a c";
    grid-template-columns: max-content 1fr;
    column-gap: 10px;
    background: var(--notification-background-color);
    border-radius: 5px;
    width: 300px;
    overflow: hidden;
    padding: 6px 10px;
    margin-bottom: 10px;
    backdrop-filter: blur(4px);
    -webkit-backdrop-filter: blur(4px);
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
    height: 40px;
    min-height: 40px;
    max-height: 40px;
  }

  .icon {
    height: 24px;
    width: 24px;
    background-position: center;
    background-repeat: no-repeat;
    border-radius: 4px;
    grid-area: a;
    transition: background-color 0.2s, box-shadow 0.2s;
    position: relative;
    background-image: url("/img/hud/notification/icon-toggle.svg");
    background-size: 12px;

    &.success {
      background-color: var(--notification-success-color);
      background-image: url("/img/hud/notification/icon-success.svg");
    }

    &.error {
      background-color: var(--notification-error-color);
      background-image: url("/img/hud/notification/icon-error.svg");
    }

    // info 和 warning 使用黄色
    &.info {
      background-color: #f59e0b;  // 黄色背景
      background-image: url("/img/hud/notification/icon-info.svg");
      box-shadow: 0 0 8px #f59e0b, 0 0 16px rgba(245, 158, 11, 0.3);
    }

    &.warning {
      background-color: #f59e0b;  // 黄色背景
      background-image: url("/img/hud/notification/icon-warning.svg");
      box-shadow: 0 0 8px #f59e0b, 0 0 16px rgba(245, 158, 11, 0.3);
    }

    &.disabled,
    &.enabled {
      &::after {
        content: "";
        position: absolute;
        height: 6px;
        width: 6px;
        border-radius: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
        background: var(--notification-toggle-knob-color);
        transition: all 0.2s ease-out;
      }
    }

    &.enabled {
      background-color: var(--Accent, #1a8cff);
      box-shadow: 0 0 8px var(--Accent, #1a8cff), 0 0 16px rgba(26, 140, 255, 0.2);

      &::after {
        left: 58%;
        background: var(--notification-toggle-knob-color);
        box-shadow: 0 0 8px var(--notification-toggle-knob-color), 0 0 16px var(--notification-toggle-knob-color);
      }
    }

    &.disabled {
      background-color: #444444;
      box-shadow: 0 0 4px rgba(68, 68, 68, 0.3);

      &::after {
        left: 42%;
        background: #888888;
        box-shadow: none;
      }
    }
  }

  .title {
    grid-area: b;
    font-size: 12px;
    color: var(--notification-title-color);
    font-weight: 600;
    line-height: 1.2;
  }

  .message {
    grid-area: c;
    font-size: 10px;
    color: var(--notification-message-color);
    line-height: 1.2;
  }
</style>