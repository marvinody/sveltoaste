<script>
  import { notification } from "./store.js";
  import { onMount, onDestroy } from "svelte";

  export let themes = {
    danger: "#bb2124",
    success: "#22bb33",
    warning: "#f0ad4e",
    info: "#5bc0de",
    default: "#aaaaaa"
  };

  export let timeout = 3000;

  let count = 0;
  let toasts = [];
  let unsubscribe;

  function animateOut(node, { delay = 0, duration = 300 }) {
    function vhTOpx(value) {
      var w = window,
        d = document,
        e = d.documentElement,
        g = d.getElementsByTagName("body")[0],
        x = w.innerWidth || e.clientWidth || g.clientWidth,
        y = w.innerHeight || e.clientHeight || g.clientHeight;

      return (y * value) / 100;
    }

    return {
      delay,
      duration,
      css: t =>
        `opacity: ${(t - 0.5) *
          1}; transform-origin: top right; transform: scaleX(${(t - 0.5) * 1});`
    };
  }

  function createToast(msg, theme, to) {
    const background = themes[theme] || themes["default"];
    toasts = [
      {
        id: count,
        msg,
        background,
        timeout: to || timeout,
        width: "100%"
      },
      ...toasts
    ];
    count = count + 1;
  }

  unsubscribe = notification.subscribe(value => {
    if (!value) {
      return;
    }
    createToast(value.message, value.type, value.timeout);
    notification.set();
  });

  onDestroy(unsubscribe);

  function removeToast(id) {
    toasts = toasts.filter(t => t.id !== id);
  }
</script>

<style>
  .toasts {
    list-style: none;
    position: fixed;
    top: 0;
    right: 0;
    padding: 0;
    margin: 0;
    z-index: 9999;
  }

  .toasts > .toast {
    position: relative;
    margin: 10px;
    min-width: 40vw;
    position: relative;
    animation: animate-in 350ms forwards;
    color: #fff;
  }

  .toasts > .toast .content {
    padding: 10px;
    display: block;
    font-weight: 500;
  }

  .toasts > .toast > .progress {
    position: absolute;
    bottom: 0;
    background-color: rgb(0, 0, 0, 0.3);
    height: 6px;
    width: 100%;
    animation-name: shrink;
    animation-timing-function: linear;
    animation-fill-mode: forwards;
  }

  .toasts > .toast:before,
  .toasts > .toast:after {
    content: "";
    position: absolute;
    z-index: -1;
    top: 50%;
    bottom: 0;
    left: 10px;
    right: 10px;
    border-radius: 100px / 10px;
  }

  .toasts > .toast:after {
    right: 10px;
    left: auto;
    transform: skew(8deg) rotate(3deg);
  }

  @keyframes animate-in {
    0% {
      width: 0;
      opacity: 0;
      transform: scale(1.15) translateY(20px);
    }
    100% {
      width: 40vw;
      opacity: 1;
      transform: scale(1) translateY(0);
    }
  }

  @keyframes shrink {
    0% {
      width: 40vw;
    }
    100% {
      width: 0;
    }
  }

  .container {
    display: flex;
    flex-direction: row;
  }
  .content {
    flex: 1;
  }
  .closer {
    justify-self: flex-end;
    margin-right: 10px;
    align-self: center;
    cursor: pointer;
  }
</style>

<ul class="toasts">
  {#each toasts as toast (toast.id)}
    <li class="toast" style="background: {toast.background};" out:animateOut>
      <div class="container">
        <div class="content">{toast.msg}</div>
        <div class="closer" on:click={() => removeToast(toast.id)}>X</div>
      </div>
      <div
        class="progress"
        style="animation-duration: {toast.timeout}ms;"
        on:animationend={() => removeToast(toast.id)} />
    </li>
  {/each}
</ul>
