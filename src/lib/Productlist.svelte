
<script lang="ts">
    import Change from './Change.svelte';
    import { imagesPath } from './variables';
    export let products:any = [];
    export let nation: string;
    export let ismok: boolean;
</script>

<ul class="-m-3 divide-y divide-gray-200 dark:divide-gray-700 dark:bg-gray-800">
  {#each products as { src, image, label, price, change, nation: n, ismok: m }}
    {#if n === nation && m === ismok}
      <li class="py-3 sm:py-4">
        <div class="flex items-center justify-between">
          <div class="flex min-w-0 items-center">
            <img class="h-10 w-10 flex-shrink-0" src={src ? imagesPath(src, 'products') : ''} alt={image} />
            <div class="ml-3">
              <p class="truncate font-medium text-gray-900 dark:text-white">{label}</p>
              <Change value={change} size="sm" equalHeight class="ml-px" />
            </div>
          </div>
          <div class="inline-flex items-center text-base font-semibold text-gray-900 dark:text-white">
            {price}
          </div>
        </div>
      </li>
    {/if}
  {/each}
</ul>
