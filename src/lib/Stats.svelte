<script lang="ts">
  import { Avatar, Card, Heading, Popover, TabItem, Tabs } from 'flowbite-svelte';
  import { Change, More, DateRangeSelector } from '$lib';
  import { imagesPath } from './variables';
  import { QuestionCircleSolid } from 'flowbite-svelte-icons';
  import type { StatsProps } from './types';
  import ProductList from './Productlist.svelte';

  let { products, customers, title, popoverTitle, popoverDesc, tab1Title, tab2Title, tab3Title, tab4Title }: StatsProps = $props();
</script>

<Card size="xl" class="p-4 sm:p-6">
  <div class="mb-4 flex items-center gap-2">
    <Heading tag="h3" class="w-fit text-lg font-semibold dark:text-white">
      {title}
    </Heading>
    <button>
      <span class="sr-only">Show information</span>
      <QuestionCircleSolid size="sm" class="text-gray-400 hover:text-gray-500" />
    </button>
    <Popover placement="bottom-start">
      <div class="w-72 space-y-2 text-sm font-normal text-gray-500 dark:text-gray-300">
        <h3 class="font-semibold text-gray-900 dark:text-white">{popoverTitle}</h3>
        {#if popoverDesc}
          {@render popoverDesc()}
        {/if}
      </div>
    </Popover>
  </div>
  <Tabs style="full" class="flex divide-x divide-gray-200 shadow rtl:divide-x-reverse dark:divide-gray-700" contentClass="p-3 mt-4">
    <TabItem class="w-full" open>
      {#snippet titleSlot()}
        {tab1Title}
      {/snippet}
      <ProductList {products} nation="kor" ismok={false} />
    </TabItem>

    <TabItem class="w-full">
      {#snippet titleSlot()}
        {tab2Title}
      {/snippet}
      <ProductList {products} nation="kor" ismok={true} />
    </TabItem>

    <TabItem class="w-full" open>
      {#snippet titleSlot()}
        {tab3Title}
      {/snippet}
      <ProductList {products} nation="naq" ismok={false} />
    </TabItem>

    <TabItem class="w-full">
      {#snippet titleSlot()}
        {tab4Title}
      {/snippet}
      <ProductList {products} nation="naq" ismok={true} />
    </TabItem>

  </Tabs>

  <div class="mt-4 flex items-center justify-between border-t border-gray-200 pt-3 sm:pt-6 dark:border-gray-700">
    <DateRangeSelector />
    <More title="Full Report" href="#top" />
  </div>
</Card>

<!--
@component
[Go to docs](https://flowbite-svelte-admin-dashboard.vercel.app/)
## Type
[StatsProps](https://github.com/themesberg/flowbite-svelte-admin-dashboard/blob/main/src/lib/types.ts#L101)
## Props
@prop products
@prop customers
@prop title
@prop popoverTitle
@prop popoverDesc
@prop tab1Title
@prop tab2Title
-->
