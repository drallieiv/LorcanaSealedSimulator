<script setup lang="ts">
  import { computed } from 'vue';
  const props = defineProps<{ set: number, id: number, data: object, locale?: number, fullView? :boolean}>();

  // a computed ref
  const url = computed(() => {
    /*
    let paddedSet = ('000' + props.set).slice(-3);
    let paddedCardId = ('000' + props.id).slice(-3);
    let locale = props.locale ? props.locale : 'en';
    return `https://images.dreamborn.ink/cards/${locale}/${paddedSet}-${paddedCardId}_1468x2048.webp`;
    */
    return props.data.image;
  });

  let fullView = props.fullView ? true : false;

</script>

<template>
  <div v-if="fullView">
    <img class="card" :src=url />
  </div>
  <div v-else class="condensedCard">
    <div class="cardCost" v-bind:style="{ backgroundImage: 'url(' + url + ')'}"></div>
    <div class="cardMain" v-bind:style="{ backgroundImage: 'url(' + url + ')'}"></div>
  </div>
</template>

<style>
  img.card {
    width: 180px;
  }
  div.condensedCard {
    height: 27px;
    border: 1px solid black;
    width: fit-content;
    margin: auto;
    display: flex;
  }
  div.cardCost {
    width: 26px;
    height: 27px;
    background-size: 620%;
    background-position-y: -3px;
    background-position-x: -3px;
    display: inline-block;
  }
  div.cardMain {
    width: 195px;
    height: 27px;
    background-size: 105%;
    display: inline-block;
    background-position-y: 705px;
    background-position-x: -4px;
  }
</style>
