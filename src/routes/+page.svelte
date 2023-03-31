<script lang="ts">
  import { onMount } from "svelte";

  async function fetchEcoTimeEntries() {
    const packageName = "@intelligentgrowthsolutions/eco";
    const url = `https://registry.npmjs.org/${packageName}`;
    const response = await fetch(url);
    const json = await response.json();
    const timeEntries = Object.entries(json.time)
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

  /**
   * @type {any[]}
   */
  let timeEntries: any[] = [];
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
    const target = event.target;
    const textToCopy = target.dataset.copy;
    if (textToCopy) {
      copyToClipboard(textToCopy);
      let toast = document.createElement("eco-toast");
      toast.setAttribute("type", "success");
      toast.setAttribute("auto-dismiss", "");
      toast.setAttribute("icon", "check");
      toast.innerHTML = `Copied command to clipboard!</br>
      <small><code>${textToCopy}</code></small>`;
      document
        .querySelector("eco-toast-rack")
        ?.insertAdjacentElement("afterbegin", toast);
    }
  }

  onMount(async () => {
    timeEntries = await fetchEcoTimeEntries();
    isLoading = false;
  });
</script>

<svelte:head>
  <link rel="stylesheet" href="https://eco.app.igs.farm/dist/eco.css" />
  <script type="module" src="https://eco.app.igs.farm/dist/eco.esm.js"></script>
  <script nomodule src="https://eco.app.igs.farm/dist/index.esm.js"></script>
</svelte:head>

{#if isLoading}
  <eco-spinner />
{:else}
  <eco-layout-body>
    <eco-toast-rack stack="false">
      <!-- {#if showToast}
        <eco-toast type="success" dismissAfter={duration} auto-dismiss>
          Copied!
        </eco-toast>
      {/if} -->
    </eco-toast-rack>
    <eco-layout-nav slot="layout-nav" app-name="Eco Version Fetcher" />
    <eco-spacer gap="xxxl">
      {#each timeEntries as { key, formattedDate }}
        <eco-card sticky>
          <eco-spacer gap="xs" slot="header">
            <eco-text-heading level="3">{key}</eco-text-heading>
            <eco-text-heading level="6">
              Published {formattedDate}
            </eco-text-heading>
          </eco-spacer>
          <!-- svelte-ignore a11y-click-events-have-key-events -->
          <eco-description-group
            vertical
            flow="h-ltr"
            term-width="20ch"
            on:click={handleClick}
          >
            <eco-description
              term="eco and eco-angular"
              description="npm install @intelligentgrowthsolutions/eco@{key} @intelligentgrowthsolutions/eco-angular@{key} --save --save-exact  --legacy-peer-deps"
              data-copy="npm install @intelligentgrowthsolutions/eco@{key} @intelligentgrowthsolutions/eco-angular@{key} --save --save-exact  --legacy-peer-deps"
            />
            <eco-description
              term="eco"
              description="npm install @intelligentgrowthsolutions/eco@{key} --save --save-exact  --legacy-peer-deps"
              data-copy="npm install @intelligentgrowthsolutions/eco@{key} --save --save-exact  --legacy-peer-deps"
            />
            <eco-description
              term="eco-angular"
              description="npm install @intelligentgrowthsolutions/eco-angular@{key} --save --save-exact  --legacy-peer-deps"
              data-copy="npm install @intelligentgrowthsolutions/eco-angular@{key} --save --save-exact  --legacy-peer-deps"
            />
          </eco-description-group>
        </eco-card>
      {/each}
    </eco-spacer>
  </eco-layout-body>
{/if}
