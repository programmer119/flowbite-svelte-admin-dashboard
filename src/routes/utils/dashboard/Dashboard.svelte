<script lang="ts">
  import thickbars from '../graphs/thickbars';
  import options from '../graphs/thinmultibars';
  import trafficOptions from '../graphs/traffic';
  import users from '../graphs/users';
  import { DesktopPcOutline, MobilePhoneOutline, TabletOutline, ArrowRightOutline } from 'flowbite-svelte-icons';
  import { Chart, P, Button, Timeline, TimelineItem } from 'flowbite-svelte';
  import { ChartWidget, Stats, More, ActivityList, ProductMetricCard, CategorySalesReport, DarkChart, Traffic, getChartOptions  } from '$lib';
  import { setOriginalSeries } from '$lib/chart_options';
  import type { DeviceOption, ProductType } from '$lib/types';
  import Chat from './Chat.svelte';
  import Insights from './Insights.svelte';
  import Transactions from './Transactions.svelte';
  import Customers from '../../data/users.json';
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  const IDCURL = import.meta.env.VITE_IDCURL; // 'http://xxx.xxx.xxx.xxx:4500' // ← 환경 변수에서 URL 가져오기
  const startdate = '20250625';
  const enddate = new Date().toISOString().slice(0, 10).replace(/-/g, '');
  let products: any = $state({});
  const customers = Customers.slice(0, 5);
  let chartOptions = $state(getChartOptions(false));
  let curdb_id = $state('srhsha_real_kor');
  let initialized = false;
  $effect(()=>{
    console.log('curdb_id changed:', curdb_id);
    if(!initialized)
    {
      initialized = true;
      return
    }
    const id = curdb_id;
    changeAccount(id);
    
  })
  //let chartoptionmap =$state([]);
  const chartoptionmap = new Map<string, Record<string, any>>();
  let dark = $state(false);
  let statsCont = $state({
    title: 'Statistics this month',
    popoverTitle: 'Statistics',
    tab1Title: 'KOR',
    tab2Title: 'KOR_MOK',
    tab3Title: 'NAQ',
    tab4Title: 'NAQ_MOK',
    changedbid: ChangeDB_ID,
  });


  onMount(() => {
    console.log('${}  IDCURL:', IDCURL);
  });

  function normalizeSeries(data:any) {
    const min = Math.min(...data);
    const max = Math.max(...data);
    return data.map((v: number) => (v - min) / (max - min));
  }

  async function GetTodayindexdatas(){
    const res = await fetch(`${IDCURL}/info/todayindexdatalists`); // 실제 API URL
    if (!res.ok) throw new Error('Failed to fetch todayindexdatalists');
    const data = await res.json();
    return data;
  }
  let indexData: any = null;
  async function GetIndexdatas(db_id:string,ticker:string,startdate:string,enddate:string) {
    // const res = await fetch(`/mokdata/${ticker}.json`); // mokdata
    //indexdatalists?db_id=${curdb_id}&ticker=u201&startdate=20250328&enddate=20250829
    const res = await fetch(`${IDCURL}/info/indexdatalists?db_id=${db_id}&ticker=${ticker}&startdate=${startdate}&enddate=${enddate}`); // 실제 API URL
    if (!res.ok) throw new Error('Failed to fetch indexdatalists');
    indexData = await res.json();
    
    const filteredData: Record<string, number> = {};
    for (const date in indexData) {
      if (date >= startdate && date <= enddate) {
        filteredData[date] = indexData[date];
      }
    }

    return filteredData;

    return indexData;
  }

  async function changeAllSnapshot(db_id: string) {

    try {
      const res = await fetch(`${IDCURL}/info/snapshotaccountlists?db_id=${db_id}`); // 실제 API URL
      if (!res.ok) throw new Error('Failed to fetch snapshotaccountlists');
      const data: any = await res.json();
      const fetchedData = data; // ← 받아온 JSON

      for (const outerKey in fetchedData) {
        const innerObj = fetchedData[outerKey];
        const str = outerKey;
        const parts = str.split('_'); // ['srhsha', 'real', 'kor', '2025', '03', '28']

        const mapid = parts.slice(0, 3).join('_'); // 'srhsha_real_kor'
        const pushid = parts.slice(3, 6).join('');
        console.log(mapid);

        // chartoptionmap.set(mapid, [{ date: pushid, value: innerObj }]);
        // const prev = chartoptionmap.get(mapid) ?? [];
        // chartoptionmap.set(mapid, [prev, [pushid,innerObj]]);
        
        if(pushid >= startdate)
        {
          const prev = chartoptionmap.get(mapid) ?? {};
          chartoptionmap.set(mapid, {
            ...prev,
            [pushid]: innerObj
          });  
        }
        
      }

      console.log('Fetched products:', data); 
      // products = data; // $state 감싼 변수에 값 대입
    } catch (error) {
      console.error('Error fetching products:', error);
    }

    const chartops = chartoptionmap.get(db_id)
    const chartdatas:any = [];
    const chartdates:any = [];
    let ticker = 'u001';//u001
    const u001datas = await GetIndexdatas(db_id,ticker,startdate,enddate);
    ticker = 'u201';
    const u201datas = await GetIndexdatas(db_id,ticker,startdate,enddate);
    let chartu001datas: number[] = [];
    let chartu201datas: number[] = [];
    // let chartindexdatas: number[] = [];
    
    Object.entries(chartops ?? {}).forEach(([key, value]) => {
      if(key in u001datas)
      {
        chartdates.push(
          key
        )

        chartdatas.push(
          value.a0.nowprice
        ) 
        
        let close = u001datas[key]??0;
        chartu001datas.push(close);

        close = u201datas[key]??0;
        chartu201datas.push(close);
      }
    });
    
    const rawSeries = [
      chartdatas,
      chartu001datas,
      chartu201datas
    ];
    
    function normalizeByFirstValue(series: number[]): number[] {
      const first = series[0];
      if (first === 0) {
        return series.map((v, i) => i === 0 ? 0 : v); 
      }
      return series.map(v => v / first);
    }
    const normalizedSeries = rawSeries.map(series => normalizeByFirstValue(series));

    setOriginalSeries(rawSeries);
    // setOriginalSeries([
    //   chartdatas,
    //   chartindexdatas
    // ]);
    
    chartOptions.series = [
      {
        name: db_id,
        data: normalizedSeries[0],
        color: '#EF562F'
      },
      {
        name: 'KOSPI',
        data: normalizedSeries[1],
        color: '#FDBA8C'
      },
      {
        name: 'KOSDAQ',
        data: normalizedSeries[2],
        color: '#9D9A9C'
      }      
    ];
    
    chartOptions.stroke = {
      width: [4, 2, 2],
      curve: 'smooth'
    }
    // chartOptions.fill = {
    //   opacity: [1, 0.2, 0.2] // 빨간선 진하게, 나머지 연하게
    // }


   
    chartOptions.xaxis = {
      ...chartOptions.xaxis,
      categories: chartdates
    };

    if((chartOptions.responsive && chartOptions.responsive[0].options))
    {
      chartOptions.responsive = [
      {
        breakpoint: 1024,
        options: {
          xaxis: {
            labels: {
              show: true
            }
          },
          yaxis: {
            labels: {
              show: true
            }
          }
        }
      }
    ]
    }
  }

  async function changeAllAccounts() {
    try {
      const res = await fetch(`${IDCURL}/info/accountlists`); // 실제 API URL
      if (!res.ok) throw new Error('Failed to fetch products');
      const data: any = await res.json();
      const fetchedData = data; // ← 받아온 JSON

      for (const outerKey in fetchedData) {
        const innerObj = fetchedData[outerKey];

        // 국가 코드 추출
        const nation = outerKey.includes('kor')
          ? 'kor'
          : outerKey.includes('naq')
          ? 'naq'
          : 'jp';
        const ismok = outerKey.includes('mok')
          ? true
          : false;

        products[outerKey]={
          src: 'iphone.png',
          nation: nation,
          ismok:ismok,
          image: 'iphone',
          label: `${outerKey}`,
          change: Math.random() * 5 - 2.5, // -2.5 ~ 2.5
          price: `$${Number( innerObj.a0.nowprice).toFixed(0)}`
        };
        // innerKey들 순회
        // for (const innerKey in innerObj) {
        //   products.push({
        //     src: 'iphone.png',
        //     nation: nation,
        //     image: 'iphone',
        //     label: `${outerKey}_${innerKey}`,
        //     change: Math.random() * 5 - 2.5, // -2.5 ~ 2.5
        //     price: `$${(Math.random() * 1000000).toFixed(0)}`
        //   });
        // }
      }

      console.log('Fetched products:', products); 
      // products = data; // $state 감싼 변수에 값 대입
    } catch (error) {
      console.error('Error fetching products:', error);
    }
  }

  function ChangeDB_ID(db_id:string){
    curdb_id = db_id;
    console.log('ChangeDB_ID:', curdb_id);
  }

  async function changeAccount(dbid: string) {
    try {
      const res = await fetch(`${IDCURL}/info/accountrefresh?db_id=${dbid}`); // 실제 API URL
      if (!res.ok) throw new Error('Failed to fetch products');
      const data: any = await res.json();
      const fetchedData = data; // ← 받아온 JSON

      let accumvalue = 0;
      for (const outerKey in fetchedData) {
        const innerObj = fetchedData[outerKey];
        
        const value = Number(innerObj.amount) * ((outerKey != 'a0') ? innerObj.nowprice??0 : 1);
        accumvalue += value;
        // 국가 코드 추출
        // const nation = outerKey.includes('kor')
        //   ? 'kor'
        //   : outerKey.includes('naq')
        //   ? 'naq'
        //   : 'jp';
        // const ismok = outerKey.includes('mok')
        //   ? true
        //   : false;

      }
        products[dbid].price = accumvalue;  
        const chartops = chartoptionmap.get(dbid);
        const arrchartops = Object.entries(chartops ?? {});
        const chartop = arrchartops[arrchartops.length-1];
        products[dbid].change = ((accumvalue / chartop[1].a0.nowprice -1)*100).toFixed(2);
        // Object.entries(chartops ?? {}).forEach(([key, value]) => {

        // })
        // products[outerKey].push({
        //   src: 'iphone.png',
        //   nation: nation,
        //   ismok:ismok,
        //   image: 'iphone',
        //   label: `${outerKey}`,
        //   change: Math.random() * 5 - 2.5, // -2.5 ~ 2.5
        //   price: `$${Number( innerObj.a0.nowprice).toFixed(0)}`
        // });

      console.log('Fetched products:', data); 
      // products = data; // $state 감싼 변수에 값 대입
    } catch (error) {
      console.error('Error fetching products:', error);
    }

  }

  const devices: DeviceOption[] = [
    {
      title: 'Desktop',
      subtitle: '234k',
      change: 4,
      IconOption: {
        icon: DesktopPcOutline
      }
    },
    {
      title: 'Phone',
      subtitle: '94k',
      change: -1,
      IconOption: {
        icon: MobilePhoneOutline
      }
    },
    {
      title: 'Tablet',
      subtitle: '16k',
      change: -0.6,
      IconOption: {
        icon: TabletOutline
      }
    }
  ];
</script>

<div class="mt-px space-y-4">
  <div class="grid gap-4 xl:grid-cols-2 2xl:grid-cols-3">
    <ChartWidget value={12.5} {chartOptions} title="$45,385" subtitle="Sales this week" />
    <button on:click={()=>changeAllSnapshot(curdb_id)}>
    일별계좌
    </button>
    <button on:click={changeAllAccounts}>
    오늘계좌
    </button>
      
    <button on:click={()=>{}}>
      TEST
    </button>
    <!-- <button on:click={GetIndexdatas}>
    코스피
    </button> -->
    
   
    <Stats {products} {customers} {...statsCont}>
      {#snippet popoverDesc()}
        <P>Statistics is a branch of applied mathematics that involves the collection, description, analysis, and inference of conclusions from quantitative data.</P>
        <More title="Read more" href="#top" flat />
      {/snippet}
    </Stats>
  </div>
  <!-- <div class="grid grid-cols-1 gap-4 xl:grid-cols-2 2xl:grid-cols-3">
    <ProductMetricCard title="New products" subTitle="2,340" changeProps={{ size: 'sm', value: 12.5, since: 'Since last month' }}>
      {#snippet chart()}
        <Chart options={thickbars} class="w-full" />
      {/snippet}
    </ProductMetricCard>

    <ProductMetricCard title="Users" subTitle="4,420" changeProps={{ size: 'sm', value: -3.4, since: 'Since last month' }}>
      {#snippet chart()}
        <DarkChart configFunc={users} class="w-full" />
      {/snippet}
    </ProductMetricCard>

    <ProductMetricCard title="Users" subTitle="4,420" changeProps={{ size: 'sm', value: -3.4, since: 'Since last month' }}>
      {#snippet chart()}
        <DarkChart
          configFunc={(d) => {
            const x = users(d);
            if (x.plotOptions?.bar) {
              x.plotOptions.bar.horizontal = true;
            } else {
              x.plotOptions = {
                bar: {
                  horizontal: true
                }
              };
            }
            return x;
          }}
          class="w-full"
        />
      {/snippet}
    </ProductMetricCard>
  </div>

  <div class="grid grid-cols-1 gap-4 xl:grid-cols-2">
    <Chat />
    <div class="flex flex-col gap-4">
      <CategorySalesReport title="Sales by category" subtitle="Desktop PC" changeProps={{ value: 2.5, since: 'Since last month', size: 'sm' }}>
        {#snippet chart()}
          <Chart {options} />
        {/snippet}
      </CategorySalesReport>
      <Traffic {devices}>
        {#snippet chart()}
          <Chart options={trafficOptions(dark)} />
        {/snippet}
      </Traffic>
    </div>
  </div>
  <div class="grid grid-cols-1 gap-4 xl:grid-cols-2">
    <ActivityList title="Latest Activity">
      {#snippet actions()}
        <a href="#top" class="text-primary-700 dark:text-primary-500 inline-flex items-center rounded-lg p-2 text-sm font-medium hover:bg-gray-100 dark:hover:bg-gray-700"> View all </a>
      {/snippet}
      <Timeline>
        <TimelineItem title="Application UI design in Figma" date="April 2025">
          <p class="mb-4 text-base font-normal text-gray-500 dark:text-gray-300">
            Get access to over 20+ pages including a dashboard layout, charts, kanban board, calendar, and pre-order E-commerce & Marketing pages.
          </p>
          <Button color="alternative">Learn more<ArrowRightOutline class="ms-2" size="sm" /></Button>
        </TimelineItem>
        <TimelineItem title="Marketing UI code in Flowbite" date="March 2025">
          <p class="text-base font-normal text-gray-500 dark:text-gray-300">Get started with dozens of web components and interactive elements built on top of Tailwind CSS.</p>
          <a href="#top" class="text-primary-700 dark:text-primary-500 inline-flex items-center text-xs font-medium hover:underline sm:text-sm">
            Go to Flowbite Blocks<ArrowRightOutline class="ms-2" size="sm" />
          </a>
        </TimelineItem>
        <TimelineItem title="Marketing UI design in Figma" date="February 2025">
          <p class="text-base font-normal text-gray-500 dark:text-gray-300">Get started with dozens of web components and interactive elements built on top of Tailwind CSS.</p>
        </TimelineItem>
      </Timeline>
    </ActivityList>
    <Insights />
  </div>

  <Transactions {dark} /> -->
</div>
