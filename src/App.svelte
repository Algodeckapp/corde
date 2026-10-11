<script lang="ts">
  import { onMount } from 'svelte';

  const TXT = "refactor the auth middleware to use JWT refresh tokens, add per-user rate limiting and write tests";
  
  let txContent = "Press Ctrl+Shift+V and start speaking";
  let activeKeys = [false, false, false]; // Ctrl, Shift, V
  let micActive = false;
  let micBarHeights = [6, 6, 6, 6, 6];
  let eqBars = Array(22).fill(0).map((_, i) => 14 + Math.abs(Math.sin(i * 0.8)) * 40);
  let micInterval: number;

  const IC: Record<string, string> = {
    "Claude Code": "m4.7144 15.9555 4.7174-2.6471.079-.2307-.079-.1275h-.2307l-.7893-.0486-2.6956-.0729-2.3375-.0971-2.2646-.1214-.5707-.1215-.5343-.7042.0546-.3522.4797-.3218.686.0608 1.5179.1032 2.2767.1578 1.6514.0972 2.4468.255h.3886l.0546-.1579-.1336-.0971-.1032-.0972L6.973 9.8356l-2.55-1.6879-1.3356-.9714-.7225-.4918-.3643-.4614-.1578-1.0078.6557-.7225.8803.0607.2246.0607.8925.686 1.9064 1.4754 2.4893 1.8336.3643.3035.1457-.1032.0182-.0728-.164-.2733-1.3539-2.4467-1.445-2.4893-.6435-1.032-.17-.6194c-.0607-.255-.1032-.4674-.1032-.7285L6.287.1335 6.6997 0l.9957.1336.419.3642.6192 1.4147 1.0018 2.2282 1.5543 3.0296.4553.8985.2429.8318.091.255h.1579v-.1457l.1275-1.706.2368-2.0947.2307-2.6957.0789-.7589.3764-.9107.7468-.4918.5828.2793.4797.686-.0668.4433-.2853 1.8517-.5586 2.9021-.3643 1.9429h.2125l.2429-.2429.9835-1.3053 1.6514-2.0643.7286-.8196.85-.9046.5464-.4311h1.0321l.759 1.1293-.34 1.1657-1.0625 1.3478-.8804 1.1414-1.2628 1.7-.7893 1.36.0729.1093.1882-.0183 2.8535-.607 1.5421-.2794 1.8396-.3157.8318.3886.091.3946-.3278.8075-1.967.4857-2.3072.4614-3.4364.8136-.0425.0304.0486.0607 1.5482.1457.6618.0364h1.621l3.0175.2247.7892.522.4736.6376-.079.4857-1.2142.6193-1.6393-.3886-3.825-.9107-1.3113-.3279h-.1822v.1093l1.0929 1.0686 2.0035 1.8092 2.5075 2.3314.1275.5768-.3218.4554-.34-.0486-2.2039-1.6575-.85-.7468-1.9246-1.621h-.1275v.17l.4432.6496 2.3436 3.5214.1214 1.0807-.17.3521-.6071.2125-.6679-.1214-1.3721-1.9246L14.38 17.959l-1.1414-1.9428-.1397.079-.674 7.2552-.3156.3703-.7286.2793-.6071-.4614-.3218-.7468.3218-1.4753.3886-1.9246.3157-1.53.2853-1.9004.17-.6314-.0121-.0425-.1397.0182-1.4328 1.9672-2.1796 2.9446-1.7243 1.8456-.4128.164-.7164-.3704.0667-.6618.4008-.5889 2.386-3.0357 1.4389-1.882.929-1.0868-.0062-.1579h-.0546l-6.3385 4.1164-1.1293.1457-.4857-.4554.0608-.7467.2307-.2429 1.9064-1.3114Z",
    "OpenCode": "M22 24H2V0h20zM17 4.8H7v14.4h10z"
  };

  const R1 = ["Claude Code", "OpenCode", "Gemini CLI", "Aider", "Goose", "Amp", "Cursor", "Copilot CLI", "Cline"];
  const R2 = ["GNOME Terminal", "iTerm2", "Warp", "Ghostty", "Alacritty", "kitty", "VS Code", "Windows Terminal", "Neovim", "Zed", "JetBrains", "tmux"];

  const sleep = (ms: number) => new Promise(r => setTimeout(r, ms));

  function wv(on: boolean) {
    clearInterval(micInterval);
    if (!on) {
      micBarHeights = [6, 6, 6, 6, 6];
      return;
    }
    micInterval = setInterval(() => {
      micBarHeights = micBarHeights.map(() => 5 + Math.random() * 16);
    }, 90);
  }

  async function loop() {
    if (window.matchMedia("(prefers-reduced-motion: reduce)").matches) {
      txContent = TXT;
      return;
    }
    while (true) {
      txContent = "Press Ctrl+Shift+V and start speaking";
      await sleep(1400);
      for (let i = 0; i < activeKeys.length; i++) {
        activeKeys[i] = true;
        await sleep(130);
      }
      txContent = "";
      wv(true);
      await sleep(1500);
      activeKeys = [false, false, false];
      wv(false);
      for (let i = 1; i <= TXT.length; i++) {
        txContent = TXT.slice(0, i);
        await sleep(14);
      }
      await sleep(3200);
    }
  }

  let recommendedOS = "";

  onMount(() => {
    loop();
    const u = navigator.userAgent;
    recommendedOS = /Android|iPhone|iPad/.test(u) ? "" : /Mac/.test(u) ? "mac" : /Win/.test(u) ? "win" : /Linux|X11/.test(u) ? "linux" : "";

    const cards = document.querySelectorAll("#features .card");
    if ("IntersectionObserver" in window) {
      const io = new IntersectionObserver(entries => {
        entries.forEach(entry => {
          if (entry.isIntersecting) {
            entry.target.classList.add("in");
            io.unobserve(entry.target);
          }
        });
      }, { threshold: 0.25 });
      cards.forEach(card => io.observe(card));
    } else {
      cards.forEach(card => card.classList.add("in"));
    }
  });

  function getChipBackground(i: number) {
    return `linear-gradient(135deg, hsl(${(i * 43 + 235) % 360} 70% 58%), hsl(${(i * 43 + 265) % 360} 70% 38%))`;
  }
</script>

<div class="glow" aria-hidden="true"></div><div class="haze" aria-hidden="true"></div>
<div class="wrap">
  <nav>
    <a class="brand" href="#top" aria-label="Cordė home">
      <svg viewBox="0 0 420 240" aria-hidden="true">
        <path class="l" d="M20,150 C40,162 55,152 66,118 C76,84 88,66 98,84 C110,108 112,176 132,176 C152,176 186,100 206.4,56.4"/>
        <path class="l" d="M333.6,56.4 A90,90 0 1 0 333.6,183.6"/>
      </svg>Cordė
    </a>
    <div class="links">
      <a href="#features">Product</a><a href="#teams">Use cases</a><a href="#pricing">Pricing</a><a href="#faq">FAQ</a>
    </div>
    <a class="btn" href="#pricing">Get Started</a>
  </nav>

  <header class="hero" id="top">
    <h1>Smart Voice Input for Your Terminal</h1>
    <p class="sub">Speak your prompts and Cordė types them into any terminal or window. Private, instant, and processed entirely on your machine.</p>
    <div class="box" aria-label="Animated demo: speech typed into a terminal prompt">
      <div class="keys">
        <span class="k" class:dn={activeKeys[0]}>Ctrl</span>
        <span class="k" class:dn={activeKeys[1]}>Shift</span>
        <span class="k" class:dn={activeKeys[2]}>V</span>
      </div>
      <div class="ph"><b>&gt;</b> <span>{txContent}</span><span class="cur"></span></div>
      <div class="mic">
        {#each micBarHeights as height}
          <i style="height: {height}px"></i>
        {/each}
      </div>
      <span class="btn go">Send</span>
    </div>
    <div class="cta">
      <a class="btn lg" href="#pricing">Get Cordė</a>
      <a class="btn lg ghost" href="#how">See how it works</a>
    </div>
    <div class="note">Pronounced “cord-ay” · macOS, Windows, Linux · $1.99/month</div>
  </header>

  <div class="dl" id="download">
    <a class="dlb" href="#pricing" class:rec={recommendedOS === 'mac'} data-os="mac">
      <svg class="fi" viewBox="0 0 24 24" aria-hidden="true"><path d="M12.152 6.896c-.948 0-2.415-1.078-3.96-1.04-2.04.027-3.91 1.183-4.961 3.014-2.117 3.675-.546 9.103 1.519 12.09 1.013 1.454 2.208 3.09 3.792 3.039 1.52-.065 2.09-.987 3.935-.987 1.831 0 2.35.987 3.96.948 1.637-.026 2.676-1.48 3.676-2.948 1.156-1.688 1.636-3.325 1.662-3.415-.039-.013-3.182-1.221-3.22-4.857-.026-3.04 2.48-4.494 2.597-4.559-1.429-2.09-3.623-2.324-4.39-2.376-2-.156-3.675 1.09-4.61 1.09zM15.53 3.83c.843-1.012 1.4-2.427 1.245-3.83-1.207.052-2.662.805-3.532 1.818-.78.896-1.454 2.338-1.273 3.714 1.338.104 2.715-.688 3.559-1.701"/></svg>
      <span><b>Download for macOS</b><small>Apple Silicon &amp; Intel</small></span>
    </a>
    <a class="dlb" href="#pricing" class:rec={recommendedOS === 'win'} data-os="win">
      <svg viewBox="0 0 24 24" fill="none" stroke="#fff" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="3" y="4" width="18" height="12" rx="2"/><path d="M8 20h8M12 16v4"/></svg>
      <span><b>Download for Windows</b><small>Windows 10 &amp; 11</small></span>
    </a>
    <a class="dlb" href="#pricing" class:rec={recommendedOS === 'linux'} data-os="linux">
      <svg class="fi" viewBox="0 0 24 24" aria-hidden="true"><path d="M12.504 0c-.155 0-.315.008-.48.021-4.226.333-3.105 4.807-3.17 6.298-.076 1.092-.3 1.953-1.05 3.02-.885 1.051-2.127 2.75-2.716 4.521-.278.832-.41 1.684-.287 2.489a.424.424 0 00-.11.135c-.26.268-.45.6-.663.839-.199.199-.485.267-.797.4-.313.136-.658.269-.864.68-.09.189-.136.394-.132.602 0 .199.027.4.055.536.058.399.116.728.04.97-.249.68-.28 1.145-.106 1.484.174.334.535.47.94.601.81.2 1.91.135 2.774.6.926.466 1.866.67 2.616.47.526-.116.97-.464 1.208-.946.587-.003 1.23-.269 2.26-.334.699-.058 1.574.267 2.577.2.025.134.063.198.114.333l.003.003c.391.778 1.113 1.132 1.884 1.071.771-.06 1.592-.536 2.257-1.306.631-.765 1.683-1.084 2.378-1.503.348-.199.629-.469.649-.853.023-.4-.2-.811-.714-1.376v-.097l-.003-.003c-.17-.2-.25-.535-.338-.926-.085-.401-.182-.786-.492-1.046h-.003c-.059-.054-.123-.067-.188-.135a.357.357 0 00-.19-.064c.431-1.278.264-2.55-.173-3.694-.533-1.41-1.465-2.638-2.175-3.483-.796-1.005-1.576-1.957-1.56-3.368.026-2.152.236-6.133-3.544-6.139zm.529 3.405h.013c.213 0 .396.062.584.198.19.135.33.332.438.533.105.259.158.459.166.724 0-.02.006-.04.006-.06v.105a.086.086 0 01-.004-.021l-.004-.024a1.807 1.807 0 01-.15.706.953.953 0 01-.213.335.71.71 0 00-.088-.042c-.104-.045-.198-.064-.284-.133a1.312 1.312 0 00-.22-.066c.05-.06.146-.133.183-.198.053-.128.082-.264.088-.402v-.02a1.21 1.21 0 00-.061-.4c-.045-.134-.101-.2-.183-.333-.084-.066-.167-.132-.267-.132h-.016c-.093 0-.176.03-.262.132a.8.8 0 00-.205.334 1.18 1.18 0 00-.09.4v.019c.002.089.008.179.02.267-.193-.067-.438-.135-.607-.202a1.635 1.635 0 01-.018-.2v-.02a1.772 1.772 0 01.15-.768c.082-.22.232-.406.43-.533a.985.985 0 01.594-.2zm-2.962.059h.036c.142 0 .27.048.399.135.146.129.264.288.344.465.09.199.14.4.153.667v.004c.007.134.006.2-.002.266v.08c-.03.007-.056.018-.083.024-.152.055-.274.135-.393.2.012-.09.013-.18.003-.267v-.015c-.012-.133-.04-.2-.082-.333a.613.613 0 00-.166-.267.248.248 0 00-.183-.064h-.021"/></svg>
      <span><b>Download for Linux</b><small>.AppImage, .deb, .rpm</small></span>
    </a>
  </div>

  <div class="works">
    <p>Works with every coding agent and terminal</p>
    <div class="mq" id="mq1">
      <div class="tr">
        <div class="set">
          {#each R1 as n, i}
            <div class="chip">
              {#if IC[n]}
                <svg viewBox="0 0 24 24" width="22" height="22" fill="currentColor" aria-hidden="true"><path d={IC[n]}/></svg>
              {:else}
                <b style="background: {getChipBackground(i)}">{n[0].toUpperCase()}</b>
              {/if}
              {n}
            </div>
          {/each}
        </div>
        <div class="set" aria-hidden="true">
          {#each R1 as n, i}
            <div class="chip">
              {#if IC[n]}
                <svg viewBox="0 0 24 24" width="22" height="22" fill="currentColor" aria-hidden="true"><path d={IC[n]}/></svg>
              {:else}
                <b style="background: {getChipBackground(i)}">{n[0].toUpperCase()}</b>
              {/if}
              {n}
            </div>
          {/each}
        </div>
      </div>
    </div>
    <div class="mq rev" id="mq2">
      <div class="tr">
        <div class="set">
          {#each R2 as n, i}
            <div class="chip">
              <b style="background: {getChipBackground(i)}">{n[0].toUpperCase()}</b>
              {n}
            </div>
          {/each}
        </div>
        <div class="set" aria-hidden="true">
          {#each R2 as n, i}
            <div class="chip">
              <b style="background: {getChipBackground(i)}">{n[0].toUpperCase()}</b>
              {n}
            </div>
          {/each}
        </div>
      </div>
    </div>
  </div>

  <section id="features">
    <div class="hd">
      <h2>Everything You Need to Talk to Your Terminal</h2>
      <p>A small, fast desktop app built for real daily use, not just a demo.</p>
    </div>
    <div class="grid">
      <div class="card hot" style="--i: 0">
        <div class="fv">
          <div class="kc"><span>Ctrl</span><span>Shift</span><span>V</span></div>
        </div>
        <h3>Global hotkey</h3>
        <p>Hold to talk or toggle. Rebindable.</p>
      </div>
      <div class="card" style="--i: 1">
        <div class="fv">
          <svg viewBox="0 0 120 64" aria-hidden="true">
            <circle class="ring" cx="60" cy="32" r="28" fill="none" stroke="#8f83ff" stroke-width="1.5" stroke-dasharray="4 6" opacity=".7"/>
            <rect x="40" y="12" width="40" height="40" rx="8" fill="#14143a" stroke="#8f83ff" stroke-width="2"/>
            <path d="M48 8v4M60 8v4M72 8v4M48 52v4M60 52v4M72 52v4M36 22h4M36 32h4M36 42h4M80 22h4M80 32h4M80 42h4" stroke="#8f83ff" stroke-width="2" stroke-linecap="round"/>
            <g class="lock">
              <rect x="52" y="30" width="16" height="12" rx="2.5" fill="#fff"/>
              <path d="M55 30v-4a5 5 0 0 1 10 0v4" fill="none" stroke="#fff" stroke-width="2.2"/>
            </g>
          </svg>
        </div>
        <h3>100% on-device</h3>
        <p>Runs locally. Audio never leaves your machine.</p>
      </div>
      <div class="card" style="--i: 0">
        <div class="fv">
          <svg viewBox="0 0 120 64" aria-hidden="true">
            <rect x="8" y="6" width="104" height="52" rx="8" fill="#0b0b1c" stroke="rgba(255,255,255,.2)"/>
            <circle cx="18" cy="15" r="2.2" fill="#ff5f57"/>
            <circle cx="25" cy="15" r="2.2" fill="#febc2e"/>
            <circle cx="32" cy="15" r="2.2" fill="#28c840"/>
            <text x="16" y="35" fill="#8f83ff" font-size="9" font-family="monospace">&gt;</text>
            <rect class="ty" x="26" y="29" width="64" height="5" rx="2.5" fill="#d9d9f2"/>
            <rect class="cu" x="93" y="27" width="3" height="9" rx="1" fill="#8f83ff"/>
            <rect class="ty t2" x="16" y="42" width="48" height="5" rx="2.5" fill="#8d8dab"/>
          </svg>
        </div>
        <h3>Types anywhere</h3>
        <p>Types at your cursor in any window.</p>
      </div>
      <div class="card" style="--i: 1">
        <div class="fv">
          <svg viewBox="0 0 120 64" aria-hidden="true">
            <circle cx="60" cy="32" r="24" fill="none" stroke="rgba(255,255,255,.12)" stroke-width="5"/>
            <circle class="pg" cx="60" cy="32" r="24" fill="none" stroke="#8f83ff" stroke-width="5" stroke-linecap="round"/>
            <text x="60" y="37" text-anchor="middle" fill="#fff" font-size="14" font-weight="700" font-family="Inter,Arial,sans-serif">&lt;1s</text>
          </svg>
        </div>
        <h3>Sub-second speed</h3>
        <p>Text appears in under a second.</p>
      </div>
      <div class="card" style="--i: 0">
        <div class="fv">
          <svg viewBox="0 0 120 64" aria-hidden="true">
            <rect x="10" y="12" width="100" height="10" rx="5" fill="rgba(255,255,255,.08)"/>
            <rect class="hv" x="10" y="12" width="100" height="10" rx="5" fill="rgba(255,255,255,.3)"/>
            <rect x="10" y="40" width="100" height="10" rx="5" fill="rgba(255,255,255,.08)"/>
            <rect class="lw" x="10" y="40" width="22" height="10" rx="5" fill="#8f83ff"/>
            <text x="38" y="48" fill="#fff" font-size="8" font-weight="600" font-family="Inter,Arial,sans-serif">&lt; 50 MB</text>
          </svg>
        </div>
        <h3>Featherweight</h3>
        <p>Idle memory under 50 MB.</p>
      </div>
      <div class="card" style="--i: 1">
        <div class="fv">
          <div class="vd" aria-hidden="true">
            <i class="s"></i><i class="s"></i>
            {#each Array(6) as _, i}
              <i class="v" style="animation-delay: {0.14 + i * 0.07}s"></i>
            {/each}
            <i class="s"></i><i class="s"></i><i class="s"></i>
            {#each Array(3) as _, i}
              <i class="v" style="animation-delay: {0.77 + i * 0.07}s"></i>
            {/each}
            <i class="s"></i><i class="s"></i><i class="s"></i>
          </div>
        </div>
        <h3>Voice activity detection</h3>
        <p>Silence ignored. Only your words typed.</p>
      </div>
    </div>
  </section>

  <section id="how" style="padding-top:20px">
    <div class="hd">
      <h2>Three Steps. No New Workflow.</h2>
      <p>Cordė sits in your system tray until you call it.</p>
    </div>
    <div class="bento">
      <div class="tile t2">
        <h3>1 · Press the shortcut</h3>
        <p>Hold to talk or tap to toggle. A subtle tray indicator shows when it's listening.</p>
        <div class="vis"><div class="orb"></div></div>
      </div>
      <div class="tile t2">
        <h3>2 · Speak naturally</h3>
        <p>A local Whisper model transcribes as you talk.</p>
        <div class="vis">
          <div class="eq">
            {#each eqBars as height, i}
              <i style="height: {height}px; animation-delay: {i * 0.07}s"></i>
            {/each}
          </div>
        </div>
      </div>
      <div class="tile t2">
        <h3>3 · It types for you</h3>
        <p>Your words land at the cursor, ready to send to your agent.</p>
        <div class="vis"><div class="mini"><b>&gt;</b> refactor the auth middleware…<br><em>✓</em> typed by Cordė</div></div>
      </div>
    </div>
  </section>

  <section id="teams" style="padding-top:20px">
    <div class="hd">
      <h2>Designed for Developers Who Live in the CLI</h2>
      <p>From long agent prompts to quick commands, speaking is faster than typing.</p>
    </div>
    <div class="bento">
      <div class="tile t3">
        <h3>Agent prompting</h3>
        <p>Dictate long, detailed instructions to Claude Code and other CLI agents without breaking flow.</p>
        <div class="vis"><div class="mini"><b>&gt;</b> add per-user rate limiting and write tests for every edge case<span class="cur"></span></div></div>
      </div>
      <div class="tile t3">
        <h3>Private by design</h3>
        <p>Proprietary code stays proprietary. Nothing is uploaded and nothing needs an API key.</p>
        <div class="vis">
          <div class="chips">
            <span>Offline</span><span>No cloud</span><span>No API keys</span><span>No uploads</span>
          </div>
        </div>
      </div>
    </div>
  </section>

  <section id="pricing">
    <div class="hd">
      <h2>Simple Pricing</h2>
      <p>One plan. Everything included.</p>
    </div>
    <div class="price">
      <div class="plan pop">
        <small>Cordė</small>
        <div class="n">$1.99 <s>/ month</s></div>
        <ul>
          <li>Unlimited on-device dictation</li>
          <li>macOS, Windows and Linux</li>
          <li>All models and hotkey modes</li>
          <li>Automatic updates</li>
        </ul>
        <a class="btn" href="#top">Get Cordė</a>
      </div>
    </div>
  </section>

  <section id="faq" style="padding-top:20px">
    <div class="hd"><h2>Questions</h2></div>
    <div class="faq">
      <details><summary>Does my voice leave my computer?</summary><p>No. Transcription runs locally with Faster-Whisper, so audio is never sent anywhere.</p></details>
      <details><summary>Which apps does it work with?</summary><p>Any app that accepts keyboard input. It was designed for terminals and CLI agents like Claude Code.</p></details>
      <details><summary>How accurate is it?</summary><p>It uses OpenAI's Whisper models in an optimized runtime. Smaller models are faster, and you can pick a larger one for more accuracy.</p></details>
      <details><summary>How do I say the name?</summary><p>“Cord-ay”.</p></details>
    </div>
  </section>

  <section style="padding-top:20px">
    <div class="final">
      <h2>Stop typing. Start speaking.</h2>
      <a class="btn lg" href="#pricing">Get Cordė</a>
    </div>
  </section>

  <footer>
    <div>© Cordė · Voice to terminal</div>
    <div>Code at the speed of thought.</div>
  </footer>
</div>

<style>
  /* Local overrides if needed, but most is in app.css */
  .keys {
    display: flex;
    gap: 6px;
  }
</style>
