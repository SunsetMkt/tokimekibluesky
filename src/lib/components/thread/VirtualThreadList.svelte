<script lang="ts">
  import {agent} from '$lib/stores';
  import { createVirtualizer } from '@tanstack/svelte-virtual';
  import VirtualThreadItem from "$lib/components/thread/VirtualThreadItem.svelte";
  import {_} from "svelte-i18n";
  import {tick} from "svelte";

  export let column;
  export let _agent = $agent;
  export let rootIndex = column.data.feed.findIndex(_feed => _feed.depth === 0);
  let virtualListEl: HTMLDivElement
  let virtualItemEls: HTMLDivElement[] = []

  $: virtualizer = createVirtualizer<HTMLDivElement, HTMLDivElement>({
      count: column.data.feed.length,
      getScrollElement: () => virtualListEl,
      estimateSize: () => 186,
      overscan: 5,
  })

  $: items = $virtualizer.getVirtualItems()

  $: {
      if (virtualItemEls.length)
          virtualItemEls.forEach((el) => $virtualizer.measureElement(el))
  }

  $: changeRootIndex(column);
  $: scrollOffset = $virtualizer.scrollOffset;

  function changeRootIndex(column) {
      const _offset = scrollOffset;

      if (scrollOffset) {
          tick().then(() => {
              $virtualizer.scrollToOffset(_offset, {
                  align: 'start',
              });
          })

          return false;
      }

      tick().then(() => {
          $virtualizer.scrollToIndex(rootIndex, {
              align: 'start',
          });
      })
  }
</script>

<div class="thread-timeline-scroller" bind:this={virtualListEl}>
  <div style="position: relative; height: {$virtualizer.getTotalSize()}px; width: 100%;">
    <div style="position: absolute; top: 0; left: 0; width: 100%; transform: translateY({items[0] ? items[0].start : 0}px);">
      {#each items as data, index (data.index)}
        {#if (!column.data.feed[data.index].blocked && !column.data.feed[data.index].notFound)}
          <div
            bind:this={virtualItemEls[index]}
            data-index={data.index}
            data-depth={column.data.feed[data.index]?.depth}
            class="thread-item"
            class:is-root={!column.data.feed[0]?.post?.record?.reply}
            class:is-final={column.data.feed[data.index].post.replyCount === 0}
            class:has-child={column.data.feed[data.index].post.replyCount > 0}
          >
            <VirtualThreadItem {column} index={data.index} {_agent}></VirtualThreadItem>

            {#if (column.data.feed[data.index]?.depth > 1)}
              <span class="thread-round-border"></span>
            {/if}

            {#if (column.data.feed[data.index]?.post?.replyCount > 0 && column.data.feed[data.index]?.depth === 6)}
              <a href={'/profile/' + column.data.feed[data.index].post.author.handle + '/post/' + column.data.feed[data.index].post.uri.split('/').slice(-1)[0]} class="thread-depth-more">{$_('read_more_thread')}</a>
            {/if}
          </div>
        {:else}
          <article class="timeline-hidden-item">
            <p class="timeline-hidde-item__text">{$_('deleted_post')}</p>
          </article>
        {/if}
      {/each}
    </div>
  </div>
</div>

<style lang="postcss">
  .thread-timeline-scroller {
      position: relative;
      height: 100%;
      width: 100%;
      overflow-y: scroll;
      contain: strict;
      padding: 0 16px calc(94vh - 120px - var(--root-client-height, 0px));
      overflow-anchor: none;
  }

  .thread-item {
      position: relative;

      &[data-depth='0'] {
          position: relative;

          &::after {
              content: '';
              left: -16px;
              top: 0;
              bottom: 0;
              position: absolute;
              width: 4px;
              background-color: var(--primary-color);
          }
      }

      /* &[data-depth='1'] {
          margin-left: 0;
      }

      &[data-depth='2'] {
          margin-left: 32px;
      }

      &[data-depth='3'] {
          margin-left: 64px;
      }

      &[data-depth='4'] {
          margin-left: 96px;
      }

      &[data-depth='5'] {
          margin-left: 128px;
      }

      &[data-depth='6'] {
          margin-left: 160px;
      } */
  }

  /* .thread-round-border {
      position: absolute;
      width: 16px;
      height: 16px;
      border-bottom: 2px solid var(--border-color-1);
      border-left: 2px solid var(--border-color-1);
      border-radius: 0 0 0 10px;
      left: -10px;
      top: 10px;
      z-index: -1;
  } */
</style>