<script setup lang="ts">
  import SingleCard from './components/SingleCard.vue'
</script>
<script lang="ts">
  import cardsDatabase from './data/allCards.json'

  const nbCommons: number = 6;
  const nbUncommon: number = 3;
  const nbRares: number = 2;
  const nbFoil: number = 1;
  
  const sealedSet: number = 1;

  class Card { 
    constructor(public id: number, public set: number, public data: any) {}
  }

  class CardStack {
    constructor(public cards: Card[]) {}
  }

  class CardData {
    getRandomCardOfRarity(rarity: String) {
      let cardsOfRarity = cardsDatabase.filter(c => c.rarity == rarity);
      let randomPick = Math.floor(Math.random() * cardsOfRarity.length);
      // console.log("Card choosed for slot " + rarity, cardsOfRarity[randomPick]);
      return cardsOfRarity[randomPick]
    }
    getRandomRare() {
      let rarityRatios = {
        "rare": 4,
        "super_rare": 2,
        "legendary": 1 
      };
      let total = rarityRatios.legendary + rarityRatios.super_rare + rarityRatios.rare;
      let rand = Math.floor(Math.random() * total);

      let rarityPicked = "rare";
      if(rand > rarityRatios.rare + rarityRatios.super_rare ) {
        rarityPicked = "legendary";
      } else if(rand > rarityRatios.rare ) {
        rarityPicked = "super_rare";
      }

      // console.log("Rarity picked for rare slot : " + rarityPicked);
      return this.getRandomCardOfRarity(rarityPicked);
    }
    getRandomFoil() {
      let rarityRatios = {
        "common": 6,
        "uncommon": 5,
        "rare": 4,
        "super_rare": 2,
        "legendary": 1 
      };
      let total = rarityRatios.legendary + rarityRatios.super_rare + rarityRatios.rare + rarityRatios.uncommon  + rarityRatios.common;
      let rand = Math.floor(Math.random() * total);

      let rarityPicked = "common";
      if(rand > rarityRatios.super_rare + rarityRatios.rare + rarityRatios.uncommon + rarityRatios.common ) {
        rarityPicked = "legendary";
      } else if(rand >  rarityRatios.rare + rarityRatios.uncommon + rarityRatios.common ) {
        rarityPicked = "super_rare";
      } else if(rand >  rarityRatios.uncommon + rarityRatios.common ) {
        rarityPicked = "rare";
      } else if(rand >  rarityRatios.common ) {
        rarityPicked = "uncommon";
      }

      console.log("Rarity picked for foil slot : " + rarityPicked);
      return this.getRandomCardOfRarity(rarityPicked);
    }
  }

  let allCards = new CardData();

  class BoosterGenerator { 
    getBooster(set: number) {
      let cards = new Array();
      
      for (let c = 0; c < nbFoil; c++) {
        let cardInfo = allCards.getRandomFoil();
        cards.push(new Card(cardInfo.id, set, cardInfo));
      }

      for (let c = 0; c < nbRares; c++) {
        let cardInfo = allCards.getRandomRare();
        cards.push(new Card(cardInfo.id, set, cardInfo));
      }
      
      for (let c = 0; c < nbUncommon; c++) {
        let cardInfo = allCards.getRandomCardOfRarity("uncommon");
        cards.push(new Card(cardInfo.id, set, cardInfo));
      }

      for (let c = 0; c < nbCommons; c++) {
        let cardInfo = allCards.getRandomCardOfRarity("common");
        cards.push(new Card(cardInfo.id, set, cardInfo));
      }

      return new CardStack(cards);
    }
  }
  
  export default {
    data() {
      return {
        stacks: new Array(),
        cardToPreview: new Card(0, 0, {})
      }
    },
    computed: {
      hasCardToPreview() {
        return this.cardToPreview.set > 0;
      }
    },
    methods: {
      preview: function(set:number, id:number, data:object) {
        this.cardToPreview = new Card(id, set, data);
      },
      restart: function() {
        this.stacks = [];
        let boosterGenerator = new BoosterGenerator();
        let nbBoosters = 6;
        for (let n = 0; n < nbBoosters; n++) {
          this.stacks.push(boosterGenerator.getBooster(sealedSet));
        }
      },
      sortCards: function() {
        let mixedCards: Card[] = new Array();

        this.stacks.forEach(stack => {
          mixedCards.push(... stack.cards)
        });

        let newStacks = [];

        for (let cost = 1; cost < 6; cost++) {
          let newStack = new CardStack(mixedCards.filter(c => c.data.cost == cost));
          newStacks.push(newStack);
        }
        let newStack = new CardStack(mixedCards.filter(c => c.data.cost > 6));
        newStacks.push(newStack);

        newStacks.forEach(ns => {ns.cards.sort(function(a,b){ 
          if (a.data.color != b.data.color) {
            return a.data.color - b.data.color;
          }
          return a.id - b.id;
        })});

        this.stacks = newStacks;
      }
    },
    created() {
      let boosterGenerator = new BoosterGenerator();
      let nbBoosters = 6;
      for (let n = 0; n < nbBoosters; n++) {
        this.stacks.push(boosterGenerator.getBooster(sealedSet));
      }
    }
  }
</script>

<template>
  <div class="actions">
    <button @click="restart">Restart Pack Opening</button>
    <button @click="sortCards">Auto sort cards</button>
  </div>
  <div class="layout">
    <div class="mainView">
      <div class="cardStack" v-for="stack in stacks">
        <SingleCard v-for="card in stack.cards" @mouseover="preview(card.set, card.id, card.data)" :set=card.set :id=card.id :data=card.data />
      </div>
    </div>
    <div class="PreviewZone">
      <div v-if="hasCardToPreview">
        <SingleCard :set=cardToPreview.set :id=cardToPreview.id :data=cardToPreview.data :fullView='true'/>
      </div>
      <div v-else>
        <div class="card previewPlaceholder"></div>
      </div>
    </div>
  </div>
  
</template>

<style>
  div.mainView {
    display: flex;
    flex-wrap: wrap;
  }
  div.cardStack {
    margin: 5px;
  }
  div.layout {
    display: flex;
  }
  div.layout > * {
    flex-grow: 1;
  }
  div.previewPlaceholder {
    width: 180px;
  }
</style>
