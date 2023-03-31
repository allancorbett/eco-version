<script>
  export { fetchEcoVersions };

  async function fetchEcoVersions() {
    const packageName = "@intelligentgrowthsolutions/eco";
    const baseUrl = `https://registry.npmjs.org/${packageName}`;
    const query = new URLSearchParams({
      sort: "-version",
      from: "-15",
    });
    const url = `${baseUrl}?${query.toString()}`;
    const response = await fetch(url);
    const json = await response.json();
    const versions = Object.keys(json.versions)
      .map((version) => {
        const {
          name,
          version: ver,
          publish_time: publishTime,
        } = json.versions[version];
        return { name, ver, publishTime };
      })
      .filter((version) => version.publishTime)
      .sort((a, b) => b.publishTime.localeCompare(a.publishTime));
    return versions.slice(0, 15);
  }
</script>
