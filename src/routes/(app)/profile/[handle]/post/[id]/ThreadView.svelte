<script lang="ts">
    import {agent, didHint} from "$lib/stores";
    import {onMount} from "svelte";
    import {defaultDeckSettings} from "$lib/components/deck/defaultDeckSettings";
    import DeckRow from "../../../../DeckRow.svelte";
    import {getDidByHandle, isDid} from "$lib/util";
    import {getColumnState} from "$lib/classes/columnState.svelte";

    interface Props {
      id: any;
      handle: any;
      title?: string;
    }

    let { id, handle = $bindable(), title = '', _agent = $agent }: Props = $props();
    let columnId = $derived(`thread_${id}`);
    const columnState = getColumnState(true);

    onMount(async () => {
        if ($didHint) {
            const _did = $didHint;
            didHint.set('');
            handle = _did;
        }

        if (!isDid(handle)) {
            handle = await getDidByHandle(handle, _agent);
        }

        if (!columnState.hasColumn(columnId)) {
            columnState.add({
                id: columnId,
                algorithm: {
                    algorithm: 'at://' + handle + '/app.bsky.feed.post/' + id,
                    type: 'thread',
                    name: 'Thread',
                },
                style: 'default',
                settings: defaultDeckSettings,
                did: _agent.did(),
                handle: _agent.handle(),
                data: {
                    feed: [],
                    cursor: '',
                }
            });
        }
    })
</script>

{#if (columnState.hasColumn(columnId))}
  {#key _agent}
    <DeckRow index={columnState.getColumnIndex(columnId)} isJunk={true} name={title} {_agent}></DeckRow>
  {/key}
{/if}
