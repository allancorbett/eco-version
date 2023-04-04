<script lang="ts">
  import { onMount } from "svelte";

  async function getPackage() {
    const packageName = "@intelligentgrowthsolutions/eco";
    const url = `https://registry.npmjs.org/${packageName}`;
    const response = await fetch(url);
    const json = await response.json();
    return json;
  }

  let latest: any;
  function getLatestVersion() {
    return (latest = packageFromNpm["dist-tags"].latest);
  }

  function fetchEcoTimeEntries() {
    const timeEntries = Object.entries(packageFromNpm.time)
      .filter(([key]) => key !== "created" && key !== "modified")
      .sort(([, a], [, b]) => b.localeCompare(a))
      .slice(0, 25)
      .map(([key, value]) => {
        const date = new Date(value);
        const formattedDate = date.toLocaleString();
        return { key, formattedDate };
      });
    return timeEntries;
  }

  let timeEntries: any[] = [];
  let packageFromNpm: any;
  let isLoading = true;

  /**
   * @param {string} text
   */
  function copyToClipboard(text: string) {
    navigator.clipboard
      .writeText(text)
      .then(() => {
        console.log("Copied to clipboard:", text);
      })
      .catch((error) => {
        console.error("Failed to copy to clipboard:", error);
      });
  }

  /**
   * @param {{ target: any; }} event
   */
  function handleClick(event: { target: any }) {
    console.log("handleClick");
    const target = event.target;
    const textToCopy = target.dataset.copy;
    if (textToCopy) {
      copyToClipboard(textToCopy);
      let toast = document.createElement("eco-toast");
      toast.setAttribute("type", "success");
      toast.setAttribute("auto-dismiss", "");
      toast.setAttribute("icon", "check");
      toast.innerHTML = `Copied <strong><code>${textToCopy}</code></strong> to clipboard!
      `;
      document
        .querySelector("eco-toast-rack")
        ?.insertAdjacentElement("afterbegin", toast);
    }
  }
  onMount(async () => {
    packageFromNpm = await getPackage();
    timeEntries = fetchEcoTimeEntries();
    latest = getLatestVersion();
    isLoading = false;
  });
</script>

<svelte:head>
  <link
    rel="icon"
    href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%22.9em%22 font-size=%2290%22>🤖</text></svg>"
  />
  <link rel="stylesheet" href="https://eco.app.igs.farm/dist/eco.css" />
  <script type="module" src="https://eco.app.igs.farm/dist/eco.esm.js"></script>
  <script nomodule src="https://eco.app.igs.farm/dist/index.esm.js"></script>
  <title>Eco v{latest}</title>
</svelte:head>

{#if isLoading}
  <eco-spinner />
{:else}
  <eco-layout-body>
    <eco-toast-rack stack="false" />
    <eco-layout-nav slot="layout-nav" app-name="Eco Version Fetcher" />
    <eco-layout-aside sticky>
      <eco-messagebox slot="layout-aside" type="info" icon="info">
        <p>
          Click on <code>eco + eco-angular</code> to copy the npm installation
          command to your clipboard for both eco and eco-angular. You can also
          copy the installation command for <code>eco</code> or
          <code>eco-angular</code> individually.
        </p>
      </eco-messagebox>

      <eco-spacer gap="xl">
        <eco-card>
          <eco-layout-header slot="header" class="latest-wrapper">
            <eco-spacer gap="xs">
              <eco-text-heading level="3">
                <span class="latest"
                  >{latest} <eco-badge label="Stable release" /></span
                >
              </eco-text-heading>
            </eco-spacer>
            <span
              class="options latest"
              slot="actions-after"
              on:click={handleClick}
            >
              <span
                data-copy="npm install @intelligentgrowthsolutions/eco@{latest} @intelligentgrowthsolutions/eco-angular@{latest} --save --save-exact  --legacy-peer-deps"
              >
                eco + eco-angular
              </span>
              <span
                data-copy="npm install @intelligentgrowthsolutions/eco@{latest} --save --save-exact  --legacy-peer-deps"
              >
                eco
              </span>
              <span
                data-copy="npm install @intelligentgrowthsolutions/eco-angular@{latest} --save --save-exact  --legacy-peer-deps"
              >
                eco-angular
              </span>
            </span>
          </eco-layout-header>
          <eco-accordion>
            {#each timeEntries as { key, formattedDate }}
              <eco-accordion-item collapsable="false">
                <eco-layout-header slot="header">
                  <eco-spacer gap="xs">
                    <eco-text-heading level="5">{key} </eco-text-heading>
                    <eco-text-input-help>
                      Published {formattedDate}
                    </eco-text-input-help>
                  </eco-spacer>
                  <span
                    class="options old"
                    slot="actions-after"
                    on:click={handleClick}
                  >
                    <span
                      data-copy="npm install @intelligentgrowthsolutions/eco@{key} @intelligentgrowthsolutions/eco-angular@{key} --save --save-exact  --legacy-peer-deps"
                    >
                      eco + eco-angular
                    </span>
                    <span
                      data-copy="npm install @intelligentgrowthsolutions/eco@{key} --save --save-exact  --legacy-peer-deps"
                    >
                      eco
                    </span>
                    <span
                      data-copy="npm install @intelligentgrowthsolutions/eco-angular@{key} --save --save-exact  --legacy-peer-deps"
                    >
                      eco-angular
                    </span>
                  </span>
                </eco-layout-header>
              </eco-accordion-item>
            {/each}
          </eco-accordion>
        </eco-card>
      </eco-spacer>
    </eco-layout-aside>
  </eco-layout-body>
{/if}

<style>
  eco-text-input-help {
    font-weight: var(--fw-base);
  }
  .old {
    font-size: var(--fs-sm);
  }
  .options {
    font-weight: var(--fw-bold);
    display: flex;
    color: var(--c-cta-default);
    cursor: pointer;
    gap: var(--sp-xs);
  }
  .options span {
    transition: all var(--speed-slow) ease-in-out;
    padding: var(--sp-sm);
    border-radius: var(--br-round);
    border: var(--bw-thin) solid transparent;
  }
  .options span:hover {
    box-shadow: var(--box-shadow-inset);
    border-color: var(--c-cta-default);
  }
  .latest {
    display: flex;
    gap: var(--sp-sm);
    align-items: center;
  }
  .latest-wrapper {
    flex: 1;
    padding-inline: var(--sp-md);
    padding-block: var(--sp-xxl);
  }
</style>
