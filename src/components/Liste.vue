<template>
<div>
  <div class="displayFlexNoR">
    <div class="addItem">
      <h2 v-if="this.editable == false">Votre liste d'épicerie</h2>
      <div v-if="this.editable">
        <h2>Remplir votre liste</h2>
        <label class="h3" for="recherche">Entrez l'item recherché</label><br>
        <input type="text" class="inputRecherche" id="recherche" name="recherche" placeholder="ex: Tomate" v-model="texteRecherche" @keyup="lancerRecherche"/>
        <div id="resutats-recherche">
          <h3>Article(s):</h3>
          <p class="resultItems" v-for="(item, index) in resultatsRecherche"
             @click="ajouterItem(resultatsRecherche[index].nom, resultatsRecherche[index].prix, resultatsRecherche[index].id)" :key="index">
            {{resultatsRecherche[index].nom}}</p>
        </div>
      </div>
    </div>
    <div class="validationList lightText">
      <div class="sec1">
        <h2 class="validationListTop">Sommaire</h2>
        <p class="h2">Coût total: {{total}} $</p>
        <button class="btnSecondary" v-if="this.editable" @click="sauvegarder">Sauvegarder</button>
      </div>
      <div class="imgAddItemList">
        <img src="/img/imgApp/facture.svg" alt="image facture">
      </div>
    </div>
    <div class="contentList">
      <h3>Vos item(s)</h3>
      <ligne @maj="onMaj" v-for="(item, index) in items" :key="index" :id="item.id" :nom="item.nom" :prix="item.prix" :editable="editable"/>
    </div>
  </div>
</div>
</template>

<script>
import Ligne from '../components/Ligne.vue'

export default {
  name: 'liste',
  components: {
    Ligne
  },
  data: function () {
    let id = this.$route.query.id
    const listes = localStorage.getItem('listes')
    const donnees = JSON.parse(listes)
    id = parseInt(id)
    return {
      items: donnees[id].items,
      editable: donnees[id].editable,
      texteRecherche: '',
      resultatsRecherche: []
    }
  },
  computed: {
    total: function () {
      let total = 0
      this.items.forEach(function (item) { total += item.qte * item.prix })
      return total
    }
  },
  methods: {
    onMaj: function (id, valeur) {
      const items = this.items
      for (let i = 0; i < items.length; i++) {
        if (items[i].id === id) {
          items[i].qte = valeur
        }
      }
    },
    ajouterItem: function (nom, prix, id) {
      this.items.push({ id, nom, prix, qte: 1 })
    },
    sauvegarder: function () {
      let listes = localStorage.getItem('listes')
      listes = JSON.parse(listes)
      const id = this.$route.query.id
      listes[id].items = this.items
      listes[id].editable = false
      localStorage.setItem('listes', JSON.stringify(listes))
      this.$router.push('/')
    },
    lancerRecherche: function () {
      if (this.texteRecherche !== '') {
        const self = this
        window.$.ajax({
          url: window.urlServiceWeb + 'items/' + self.texteRecherche,
          success: function (donnees) {
            self.resultatsRecherche = JSON.parse(donnees)
          }
        })
      } else {
        this.resultatsRecherche = []
      }
    }
  }
}
</script>
