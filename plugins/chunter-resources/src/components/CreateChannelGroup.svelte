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
  import { createEventDispatcher, onMount } from 'svelte'
  import { ModernEditbox, Label, Modal } from '@hcengineering/ui'
  import presentation from '@hcengineering/presentation'
  import chunter from '../plugin'

  const dispatch = createEventDispatcher()

  let groupName = ''
  let canSave = false
  export let show = false

  onMount(() => {
    console.log('Component mounted')
    console.log('Initial groupName:', groupName)
  })

  $: {
    try {
      console.log('Reactive block start')
      console.log('groupName:', groupName)
      canSave = groupName !== ''
      console.log('canSave set to:', canSave)
    } catch (error) {
      console.error('Error in reactive block:', error)
    }
  }

  async function handleCreate() {
    try {
      console.log('handleCreate called')
      console.log('groupName in handleCreate:', groupName)
      if (!canSave) return
      dispatch('create', { name: groupName })
      dispatch('close')
    } catch (error) {
      console.error('Error in handleCreate:', error)
    }
  }
</script>

<Modal
  label={chunter.string.CreateGroup}
  okLabel={chunter.string.Create}
  bind:show
  {canSave}
  on:close
  on:ok={handleCreate}
>
  <div class="flex-col gap-4">
    <div class="flex-col gap-2">
      <Label label={chunter.string.GroupName} />
      <ModernEditbox
        bind:value={groupName}
        label={chunter.string.GroupNamePlaceholder}
        kind="default"
        size="medium"
      />
    </div>
  </div>
</Modal>

<style lang="scss">
  .flex-col {
    display: flex;
    flex-direction: column;
  }

  .gap-2 {
    gap: 0.5rem;
  }

  .gap-4 {
    gap: 1rem;
  }
</style>
