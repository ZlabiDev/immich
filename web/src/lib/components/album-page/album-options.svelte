<script lang="ts">
  import Icon from '$lib/components/elements/icon.svelte';
  import { updateAlbumInfo, type AlbumResponseDto, type UserResponseDto, AssetOrder } from '@immich/sdk';
  import { mdiArrowDownThin, mdiArrowUpThin, mdiClose, mdiPlus } from '@mdi/js';
  import { createEventDispatcher } from 'svelte';

  import CircleIconButton from '../elements/buttons/circle-icon-button.svelte';
  import FullScreenModal from '$lib/components/shared-components/full-screen-modal.svelte';
  import UserAvatar from '$lib/components/shared-components/user-avatar.svelte';
  import SettingSwitch from '$lib/components/shared-components/settings/setting-switch.svelte';
  import SettingDropdown from '../shared-components/settings/setting-dropdown.svelte';
  import type { RenderedOption } from '../elements/dropdown.svelte';
  import { handleError } from '$lib/utils/handle-error';
  import { findKey } from 'lodash-es';

  export let album: AlbumResponseDto;
  export let order: AssetOrder | undefined;
  export let user: UserResponseDto;
  export let onChangeOrder: (order: AssetOrder) => void;

  const options: Record<AssetOrder, RenderedOption> = {
    [AssetOrder.Asc]: { icon: mdiArrowUpThin, title: 'Oldest first' },
    [AssetOrder.Desc]: { icon: mdiArrowDownThin, title: 'Newest first' },
  };

  $: selectedOption = order ? options[order] : options[AssetOrder.Desc];

  const dispatch = createEventDispatcher<{
    close: void;
    toggleEnableActivity: void;
    showSelectSharedUser: void;
  }>();

  const handleToggle = async (returnedOption: RenderedOption) => {
    if (selectedOption === returnedOption) {
      return;
    }
    let order = AssetOrder.Desc;
    order = findKey(options, (option) => option === returnedOption) as AssetOrder;

    try {
      await updateAlbumInfo({
        id: album.id,
        updateAlbumDto: {
          order,
        },
      });
      onChangeOrder(order);
    } catch (error) {
      handleError(error, 'Error updating album order');
    }
  };
</script>

<FullScreenModal onClose={() => dispatch('close')}>
  <div class="flex h-full w-full place-content-center place-items-center overflow-hidden p-2 md:p-0">
    <div
      class="w-[550px] rounded-3xl border bg-immich-bg shadow-sm dark:border-immich-dark-gray dark:bg-immich-dark-gray dark:text-immich-dark-fg"
    >
      <div class="px-2 pt-2">
        <div class="flex items-center">
          <h1 class="px-4 w-full self-center font-medium text-immich-primary dark:text-immich-dark-primary">Options</h1>
          <div>
            <CircleIconButton icon={mdiClose} title="Close" on:click={() => dispatch('close')} />
          </div>
        </div>

        <div class=" items-center justify-center p-4">
          <div class="py-2">
            <h2 class="text-gray text-sm mb-2">SETTINGS</h2>
            <div class="grid p-2 gap-y-2">
              {#if order}
                <SettingDropdown
                  title="Display order"
                  options={Object.values(options)}
                  selectedOption={options[order]}
                  onToggle={handleToggle}
                />
              {/if}
              <SettingSwitch
                title="Comments & likes"
                subtitle="Let others respond"
                checked={album.isActivityEnabled}
                on:toggle={() => dispatch('toggleEnableActivity')}
              />
            </div>
          </div>
          <div class="py-2">
            <div class="text-gray text-sm mb-3">PEOPLE</div>
            <div class="p-2">
              <button class="flex items-center gap-2" on:click={() => dispatch('showSelectSharedUser')}>
                <div class="rounded-full w-10 h-10 border border-gray-500 flex items-center justify-center">
                  <div><Icon path={mdiPlus} size="25" /></div>
                </div>
                <div>Invite People</div>
              </button>
              <div class="flex items-center gap-2 py-2 mt-2">
                <div>
                  <UserAvatar {user} size="md" />
                </div>
                <div class="w-full">{user.name}</div>
                <div>Owner</div>
              </div>
              {#each album.sharedUsers as user (user.id)}
                <div class="flex items-center gap-2 py-2">
                  <div>
                    <UserAvatar {user} size="md" />
                  </div>
                  <div class="w-full">{user.name}</div>
                </div>
              {/each}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</FullScreenModal>
