<!--
// Copyright 2024 Hardcore Engineering Inc.
//
// Licensed under the Eclipse Public License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License. You may
// obtain a copy of the License at https://www.eclipse.org/legal/epl-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
//
// See the License for the specific language governing permissions and
// limitations under the License.
-->
<script lang="ts">
  import { createQuery } from '@hcengineering/presentation'
  import { Class, Doc, Ref } from '@hcengineering/core'
  import { DocNotifyContext } from '@hcengineering/notification'
  import { InboxNotificationsClientImpl } from '@hcengineering/notification-resources'
  import { Widget } from '@hcengineering/workbench'
  import { ActivityMessage } from '@hcengineering/activity'
  import { ChatWidgetTab } from '@hcengineering/chunter'
  import { updateTabData } from '@hcengineering/workbench-resources'
  import { Button } from '@hcengineering/ui'

  import Channel from './Channel.svelte'
  import { closeThreadInSidebarChannel } from '../navigation'
  import { ThreadView } from '../index'
  import ChannelHeader from './ChannelHeader.svelte'
  import CreateChannelGroup from './CreateChannelGroup.svelte'

  export let widget: Widget
  export let tab: ChatWidgetTab
  export let height: string
  export let width: string

  const notificationsClient = InboxNotificationsClientImpl.getClient()
  const contextByDocStore = notificationsClient.contextByDoc
  const objectQuery = createQuery()

  let object: Doc | undefined = undefined
  let context: DocNotifyContext | undefined = undefined
  let selectedMessageId: Ref<ActivityMessage> | undefined = tab.data.selectedMessageId
  let showCreateGroup = false

  $: context = object ? $contextByDocStore.get(object._id) : undefined
  $: void loadObject(tab.data._id, tab.data._class)

  $: threadId = tab.data.thread

  $: if (tab.data.selectedMessageId !== undefined && tab.data.selectedMessageId !== '') {
    selectedMessageId = tab.data.selectedMessageId
    updateTabData(widget._id, tab.id, { selectedMessageId: '' })
  }

  async function loadObject (_id?: Ref<Doc>, _class?: Ref<Class<Doc>>): Promise<void> {
    if (_id === undefined || _class === undefined) {
      object = undefined
      objectQuery.unsubscribe()
      return
    }

    objectQuery.query(
      _class,
      { _id },
      (res) => {
        object = res[0]
      },
      { limit: 1 }
    )
  }
  let renderChannel = tab.data.thread === undefined

  $: if (tab.data.thread === undefined) {
    renderChannel = true
  }

  $: visible = height !== '0px' && width !== '0px'
</script>

<div class="flex flex-col h-full">
  {#if showCreateGroup}
    <CreateChannelGroup 
      bind:show={showCreateGroup}
      on:create={(event) => {
        // TODO: Handle group creation
        console.log('Create group:', event.detail.name)
      }}
    />
  {/if}

  <div class="flex items-center justify-between px-4 py-2">
    <span class="text-sm font-medium">Channels</span>
    <Button
      icon={chunter.icon.Plus}
      kind="ghost"
      size="small"
      on:click={() => { showCreateGroup = true }}
    />
  </div>

  {#if object && renderChannel && visible}
    <div class="channel" class:invisible={threadId !== undefined} style:height style:width>
      <ChannelHeader
        _id={object._id}
        _class={object._class}
        {object}
        withAside={false}
        withSearch={false}
        canOpen={true}
        allowClose={true}
        canOpenInSidebar={false}
        closeOnEscape={false}
        on:close
      />
      {#key object._id}
        <Channel {object} {context} syncLocation={false} freeze={threadId !== undefined} {selectedMessageId} />
      {/key}
    </div>
  {/if}
  {#if threadId && visible}
    <div class="thread" style:height style:width>
      <ThreadView
        {...tab.data.props}
        _id={threadId}
        {selectedMessageId}
        syncLocation={false}
        on:channel={() => closeThreadInSidebarChannel(widget, tab)}
        on:close
      />
    </div>
  {/if}
</div>

<style lang="scss">
  .flex {
    display: flex;
  }

  .flex-col {
    flex-direction: column;
  }

  .h-full {
    height: 100%;
  }

  .items-center {
    align-items: center;
  }

  .justify-between {
    justify-content: space-between;
  }

  .px-4 {
    padding-left: 1rem;
    padding-right: 1rem;
  }

  .py-2 {
    padding-top: 0.5rem;
    padding-bottom: 0.5rem;
  }

  .text-sm {
    font-size: 0.875rem;
  }

  .font-medium {
    font-weight: 500;
  }

  .channel {
    display: inline-flex;
    flex-direction: column;
    flex: 1;
    min-width: 0;
    min-height: 0;
    position: absolute;
    top: 0;
    left: 0;

    &.invisible {
      visibility: hidden;
    }
  }

  .thread {
    position: absolute;
    display: inline-flex;
    flex-direction: column;
    flex: 1;
    min-width: 0;
    min-height: 0;
    top: 0;
    left: 0;
    background-color: var(--theme-panel-color);
  }
</style>
