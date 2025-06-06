<template>
  <div class="faqs layout mt140">
    <Typography tag="h35">FAQs</Typography>
    <ul class="faqsContent">
      <li
        v-for="faq in faqs"
        :key="faq.title"
        :class="[activedFaqs.includes(faq.title) ? 'active' : '']"
      >
        <div
          class="title"
          @click="
            () => {
              if (activedFaqs.includes(faq.title)) {
                activedFaqs = activedFaqs.filter((item) => item !== faq.title);
              } else {
                activedFaqs = [...activedFaqs, faq.title];
              }
            }
          "
        >
          <span><img :src="faq.iconSrc" /></span>
          <Typography tag="p">{{ faq.title }}</Typography>
        </div>
        <div class="animation">
          <div class="flexCol gp24" style="padding: 24px">
            <template v-for="cont in faq.content" :key="cont">
              <Typography tag="p" v-html="cont"></Typography>
            </template>
          </div>
        </div>
      </li>
    </ul>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, ref } from "vue";
import { useRouter } from "vue-router";
import Typography from "./Typography.vue";

const allNetworks = ref<{ networks: {}[] }[]>([]);
const activedFaqs = ref<string[]>([]);
const router = useRouter();

const fetchAllNetworks = () => {
  fetch("https://templates.subquery.network/all").then(async (data) => {
    const json = await data.json();
    allNetworks.value = json.templates;
  });
};

const faqs = computed(() => {
  const counts =
    allNetworks.value.reduce((cur, add) => cur + add.networks.length, 0) ||
    "165+";

  return [
    {
      title: "What networks do you support?",
      iconSrc: "/doc/assets/img/faqIcon.svg",
      content: [
        `We support over ${counts} leading layer-1 chains, including Ethereum, Cosmos, Polkadot, Avalanche, Algorand, Near and Flare. The list of supported layer-1 chains keeps growing every week, and it's our goal to support them all. Wherever you plan to build your next dApp, we want to be there to help you index it. <a href='https://subquery.network/networks'>View the full list here</a>`,
        "If you would like us to index your new layer-1 chain, we would be happy to consider it, send us a message at <a href='mailto:hello@subquery.network'>hello@subquery.network</a>.",
      ],
    },
    {
      title: "How much does it cost?",
      iconSrc: "/doc/assets/img/faqIcon.svg",
      content: [
        "SubQuery is open-source, and free for all to use forever. You can write, run, and scale your SubQuery project in your own infrastructure with complete control, many of our biggest customers do just this. Since it's open source, you can even just run the parts of it that you want.",
        `We're big believers in open source technology and really appreciate it when we 
        <a style="cursor:pointer;" onclick="router.push('/miscellaneous/contributing.html')">receive contributions</a>.`,
      ],
    },
    {
      title: "Do you provide hosting, or do I have to run it myself?",
      iconSrc: "/doc/assets/img/faqIcon.svg",
      content: [
        `We provide a 
        <a style="cursor:pointer;" onclick="router.push('/indexer/run_publish/run.html')">long guide</a> 
        on how you can run SubQuery in your infrastructure, which includes both the indexer, Postgres database, and query service.`,
        `Don't want to worry about running your own SubQuery infrastructure? There are a number of  <a style="cursor:pointer;" onclick="router.push('/indexer/run_publish/introduction.html')">external centralised hosting services</a> that offer reliable hosting of production SubQuery data indexing projects. These services are managed by external partners and are designed to make it easy to deploy, manage, and scale your SubQuery projects.`,
        `Alternatively, publish your project to the SubQuery Network, We're building the most open, performant, reliable, and scalable data service for dApp developers. The SubQuery Network indexes and services data to the global community in an incentivised and verifiable way. After publishing your project to the SubQuery Network, anyone can index and host it - providing data to users around the world faster and reliably. <a style="cursor:pointer;" onclick="router.push('/subquery_network/architects/publish.html')">Find out how</a>.`,
      ],
    },
    {
      title: "How is the data stored?",
      iconSrc: "/doc/assets/img/faqIcon.svg",
      content: [
        "SubQuery stores indexed data in a high performance PostgreSQL database.",
      ],
    },
    {
      title: "Why should I use SubQuery?",
      iconSrc: "/doc/assets/img/faqIcon.svg",
      content: [
        "SubQuery is the most efficient option for web3 builders to index data from multiple chains without the hassle of building your own indexing solution.",
        "In addition to a flexible SDK, SubQuery offers superior indexing speeds and will eventually be a decentralised solution (upon the launch of the SubQuery Network) where you can have a stake in the future of the project.",
      ],
    },
    {
      title: "How are you different from The Graph?",
      iconSrc: "/doc/assets/img/faqIcon.svg",
      content: [
        `SubQuery is a flexible, cross-chain indexing service similar to The Graph. In fact, 
        <a style="cursor:pointer;" onclick="router.push('/indexer/build/graph-migration.html')">migrating from the Graph takes only a few hours</a>. 
        Like The Graph, there are endless possibilities for the variety of data sources that can be analysed and served using SubQuery.`,
        "We build SubQuery with the following key competitive advantages in mind:",
        "<ul><li>Faster than others. We’re focusing on making SubQuery faster than other solutions with advanced indexing caches and precomputed indices saving developers time, our solution is fast to set-up, fast to manage and fast to index.</li><li>More Flexible and Feature rich. SubQuery is a scaffold for building custom APIs and we provide additional features like GraphQL subscriptions, multi-chain indexing, automated historical tracking and more.</li><li>Open. Customers have already extended our open source SDK to suit their own custom implementation.</li><li>Universal. A universal infrastructure stack bringing communities together, developers now have a tool to search, sort, filter and query any data for their app across multiple blockchains.</li></ul>",
        "Additionally, we are committed to running our Managed hosting service over the long term. We have made huge investments into it and have many customers relying on it. This provides a safe home and a reliable alternative to customers that are currently threatened by the imminent sunsetting of The Graph's hosted service.",
      ],
    },
  ];
});

onMounted(() => {
  fetchAllNetworks();
  // it's a hack to make the router available when easy to render faqs
  // @ts-ignore
  window.router = router;
});
</script>

<style lang="scss">
.faqs {
  margin-top: 140px;
}

.faqs h3 {
  margin-bottom: 24px !important;
}
.faqsContent {
  border: 1px solid var(--dark-mode-border);
  background: var(--dark-mode-background);
  padding: 0;
  border-radius: 8px;
  overflow: hidden;
}
.dark .faqsContent {
  color: #2c3e50;
}
.faqsContent li {
  font-size: 16px;
  color: #fff;
  font-weight: 500;
  line-height: 1.5;
}
.faqsContent .title {
  display: flex;
  align-items: center;
  padding: 24px;
  box-sizing: border-box;
  height: 72px;
  background: var(--dark-mode-card);
  border-bottom: 1px solid var(--dark-mode-border);
  gap: 18px;
}
.faqsContent li:last-child .title {
  /* margin-bottom: -1px; */
  border-bottom: none;
}
.faqsContent li:last-child .animation {
  /* margin-bottom: -1px; */
  border-bottom: none;
}
.faqsContent .title span {
  width: 36px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s linear;
  transform: rotate(-90deg);
  color: #fff;
}
.faqsContent .ct {
  padding: 24px;
  background-color: transparent;
}
.faqsContent .ct p {
  font-weight: 500;
  font-size: 16px;
  line-height: 24px;
  margin-bottom: 10px;
  color: #fff;
}
.faqsContent .animation {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.2s linear;
}
.faqsContent li.active:last-child .title {
  border-bottom: 1px solid var(--dark-mode-border);
}
.faqsContent .active .title span {
  transform: rotate(0);
}
.faqsContent .active .animation {
  max-height: 10000px;
  border-bottom: 1px solid var(--dark-mode-border);
}

.faqs .title span {
  cursor: pointer;
}

.faqs .title img {
  pointer-events: none;
}
</style>
