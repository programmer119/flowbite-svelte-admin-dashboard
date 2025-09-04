
<script lang="ts">
    import Change from './Change.svelte';
    import type { ProductType } from './types';
    import { imagesPath } from './variables';
    export let products: Record<string, ProductType>;
    export let nation: string;
    export let ismok: boolean;
    export let changedbid: (db_id:string)=>void;
    // import { Button, P } from 'flowbite-svelte';
    
    async function onclick(db_id :string) {
    //     'http://211.255.25.125:4500/info/accountrefresh?db_id=srhsha_real_kor'
    //   console.log(`CLICK2!!! ${db_id}`);
        changedbid(db_id);
    }

</script>

<ul class="-m-3 divide-y divide-gray-200 dark:divide-gray-700 dark:bg-gray-800">
  {#each Object.values(products) as { src, image, label, price, change, nation: n, ismok: m }}
    {#if n === nation && m === ismok}
      <li class="py-3 sm:py-4">
        <div class="flex items-center justify-between">
          <div class="flex min-w-0 items-center">
            <button onclick={()=>{onclick(label??'')}}>
                <img class="h-10 w-10 flex-shrink-0" src={src ? imagesPath(src, 'products') : ''} alt={image} />
            </button>
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
