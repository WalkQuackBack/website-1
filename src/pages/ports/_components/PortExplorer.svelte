<script lang="ts">
  import Fuse from "fuse.js/min-basic";
  import type { PortWithIcons } from "@data/ports";
  import SearchBar from "./SearchBar.svelte";
  import PortGrid from "./PortGrid.svelte";

  interface Props {
    ports: PortWithIcons[];
  }

  let { ports }: Props = $props();

  const fuse = new Fuse(ports, {
    keys: [
      { name: "key", weight: 1 },
      { name: "name", weight: 0.5 },
      { name: "categories.name", weight: 0.25 },
      { name: "repository.current-maintainers.username", weight: 0.1 },
    ],
    includeScore: false,
    threshold: 0.2,
  });
  const url = new URL(window.location.href);

  let debounceTimeout: ReturnType<typeof setTimeout> | undefined;
  let searchTerm = $state(url.searchParams.get("q") ?? "");
  let portGrid: Array<PortWithIcons> | undefined = $state(undefined);

  handleInput();

  function handleInput() {
    // Keep the URL in sync with the search bar
    if (searchTerm === "") {
      url.searchParams.delete("q");
    } else {
      url.searchParams.set("q", searchTerm);
    }
    window.history.replaceState(null, "", url.toString());

    clearTimeout(debounceTimeout);
    debounceTimeout = setTimeout(() => {
      searchTerm ? (portGrid = fuse.search(searchTerm).map((key) => key.item)) : (portGrid = ports);
    }, 25);
  }
</script>

<SearchBar bind:searchTerm {handleInput} />
<PortGrid {portGrid} {searchTerm} />
