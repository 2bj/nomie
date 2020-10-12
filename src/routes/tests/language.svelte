<script lang="ts">
  import type { keys } from "localforage";
  import { onMount } from "svelte";
  import Button from "../../components/button/button.svelte";
  import Card from "../../components/card/card.svelte";
  import Input from "../../components/input/input.svelte";
  import ListItem from "../../components/list-item/list-item.svelte";
  import Spacer from "../../components/spacer/spacer.svelte";

  import Text from "../../components/text/text.svelte";
  import ToolbarGrid from "../../components/toolbar/toolbar-grid.svelte";
  import Toolbar from "../../components/toolbar/toolbar.svelte";
  import Layout from "../../containers/layout/layout.svelte";

  import BaseLang from "../../lang/base";
  import download from "../../modules/download/download";
  import { SideStore } from "../../modules/storage/storage";
  import { Device } from "../../store/device-store";
  import { Interact } from "../../store/interact";

  const sideStore = new SideStore("language-editor");

  let baseLang = BaseLang;
  let activeLang = { ...BaseLang };

  async function save() {
    sideStore.put("active-lang", activeLang);
    Interact.toast("Saved Locally");
  }

  async function reset() {
    const confirmed = await Interact.confirm(`Clear ${activeLang.name} and start over?`);
    if (confirmed) {
      activeLang = { ...BaseLang };
      Device.scrollToTop();
    }
  }

  async function main() {
    let working = sideStore.get("active-lang") || {};
    activeLang = { ...baseLang, ...working };
  }

  async function send() {
    await Interact.alert(
      "Sending a Language File",
      `First, I'm going to download a file. Then I'll open up your email client to send that file to me.`
    );
    await download.json(`${activeLang.name}.lang.json`, activeLang);
    window.open(`mailto:support@happydata.org?subject=New Language File`, "_system");
  }

  onMount(main);
</script>

<Layout showTabs={false}>
  <div slot="header">
    <ToolbarGrid>
      <div slot="left" />
      <div slot="main">
        <Text bold>Nomie Language</Text>
      </div>
      <div slot="right">
        <Button type="clear" on:click={save}>Save</Button>
      </div>
    </ToolbarGrid>
  </div>

  <main>
    {#if activeLang}
      <Card pad title="Language Details">
        <Input compact placeholder="Language Name" bind:value={activeLang.name} />
        <Input compact placeholder="Author" bind:value={activeLang.author} />
      </Card>
      {#each Object.keys(baseLang.translation) as parentId}
        <Card pad title={parentId} className="my-2">
          {#each Object.keys(baseLang.translation[parentId]) as key}
            <Input
              compact
              rows={4}
              type={(baseLang.translation[parentId][key] || '').length > 20 ? 'textarea' : 'text'}
              placeholder={`${key}: ${baseLang.translation[parentId][key]}`}
              bind:value={activeLang.translation[parentId][key]} />
            <hr class="divider" />
          {/each}
        </Card>
      {/each}
    {/if}
    <ListItem className="my-4" on:click={reset}>
      <Text className="text-red">Reset Language</Text>
    </ListItem>
  </main>

  <div slot="footer">
    <Toolbar>
      <Spacer />
      All Done?
      <Button size="sm" className="ml-2" on:click={send}>Send to Brandon</Button>
      <Spacer />
    </Toolbar>
  </div>

</Layout>